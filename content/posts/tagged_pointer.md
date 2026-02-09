---
title: 'Tagged Pointer'
date: 2026-02-09
tags: [c++, eda]
categories: [c++]
---


Tagged Pointer是一个经典的时间优化技术,通过巧妙利用内存对齐特性，在零额外内存开销下避免堆分配。


## 基础原理

为了防止未对齐访问,编译器会插入填充0以根据类型的对齐要求对齐值，分配内存对齐到8字节(在64位系统上是16字节)，指针地址以000结尾 。可以利用这些未使用的低位来指示这不是真实指针,然后将对象数据直接存储在这个"非指针"的剩余部分中,而不是单独的内存分配。


```
Top                       		Bottom
▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮▮000 
                             					^^^
                             				可用于存储标记
```


## 简单实现:存储小整数

以一个能将小整数直接编码进指针的系统为例:

```c
#include <stdint.h>
#include <stdbool.h>

// 标记位定义
#define TAG_MASK    0x7      // 低3位掩码: 0b111
#define TAG_INT     0x1      // 整数标记: 0b001
#define TAG_PTR     0x0      // 指针标记: 0b000

// 类型判断
static inline bool is_tagged_int(uintptr_t value) {
    return (value & TAG_MASK) == TAG_INT;
}

static inline bool is_pointer(uintptr_t value) {
    return (value & TAG_MASK) == TAG_PTR;
}

// 整数编码:左移3位后设置标记
static inline uintptr_t encode_int(int64_t num) {
    return (num << 3) | TAG_INT;
}

// 整数解码:去除标记后右移
static inline int64_t decode_int(uintptr_t tagged) {
    return (int64_t)tagged >> 3;  // 算术右移保留符号
}

// 指针编码:确保低3位为0
static inline uintptr_t encode_ptr(void* ptr) {
    uintptr_t addr = (uintptr_t)ptr;
    // 断言指针已对齐
    assert((addr & TAG_MASK) == 0);
    return addr | TAG_PTR;
}

// 指针解码:清除标记位
static inline void* decode_ptr(uintptr_t tagged) {
    return (void*)(tagged & ~TAG_MASK);
}
```

使用示例

```c
// 示例:实现简单的动态类型值
typedef uintptr_t Value;

// 创建整数值
Value create_number(int64_t num) {
    // 小整数:直接编码
    if (num >= -(1LL << 60) && num < (1LL << 60)) {
        return encode_int(num);
    }
    // 大整数:堆分配
    int64_t* heap_num = malloc(sizeof(int64_t));
    *heap_num = num;
    return encode_ptr(heap_num);
}

// 整数加法
Value add_numbers(Value a, Value b) {
    int64_t a_val, b_val;
    
    // 提取值a
    if (is_tagged_int(a)) {
        a_val = decode_int(a);
    } else {
        int64_t* ptr = decode_ptr(a);
        a_val = *ptr;
    }
    
    // 提取值b
    if (is_tagged_int(b)) {
        b_val = decode_int(b);
    } else {
        int64_t* ptr = decode_ptr(b);
        b_val = *ptr;
    }
    
    return create_number(a_val + b_val);
}

// 使用示例
int main() {
    Value num1 = create_number(42);       // Tagged pointer
    Value num2 = create_number(58);       // Tagged pointer
    Value sum = add_numbers(num1, num2);  // 结果:100
    
    printf("Result: %lld\n", decode_int(sum));  // 输出: 100
    return 0;
}
```

## 经典应用场景:And-Inverter Graph (AIG)

AIG是由二输入AND门和反相器组成的多级逻辑网络,用于表示二进制顺序逻辑电路。在数字电路设计中,任何布尔逻辑都可以用AND和NOT门表示。如果不使用Tagged Pointer,AIG节点结构可能是这样:

```c
// 传统实现:每条边需要单独存储反相信息
typedef struct AigEdge_ {
    AigNode *node;           // 8字节
    unsigned int complement : 1;  // 反相标记
} AigEdge;

typedef struct AigNode_ {
    unsigned int id;
    AigEdge *fanin0;  // 指向输入边0 (16字节对象)
    AigEdge *fanin1;  // 指向输入边1 (16字节对象)
} AigNode;
```

- 每个AigEdge占用16字节(8字节指针 + 对齐填充)

- 每个AND门需要2个fanin,额外32字节仅用于存储边信息

  


ABC项目是加州大学伯克利分校开发的开源逻辑综合与验证工具，在其核心数据结构AIG的实现中，节点对象[Aig_Obj_t](https://github.com/berkeley-abc/abc/blob/master/src/aig/aig/aig.h)使用tagged pointer优化，利用最低1位存储complement(反相)标记。在大规模电路网表中，这种优化的收益是显著的。

```c
struct Aig_Obj_t_ {
    // ===== Tagged Fanin指针 =====
    Aig_Obj_t *pFanin0;    // 输入端0
    Aig_Obj_t *pFanin1;    // 输入端1
    
    unsigned int Type    :  3;
    unsigned int fPhase  :  1;
    unsigned int fMarkA  :  1;
    unsigned int fMarkB  :  1;
    unsigned int nRefs   : 26;
    
    unsigned     Level   : 24;
    unsigned     nCuts   :  8;
    
    int          TravId;       
    int          Id;            
    
    union {
        void *   pData;
        int      iData;
        float    dData;
    };
};
```

核心Tagged Pointer操作:

```c
// 检查边是否为反相 (检查最低位)
#define Aig_IsComplement(p)   \
    (((int)((ABC_PTRUINT_T)(p) & 01)))

// 获取真实节点指针 (清除最低位)
#define Aig_Regular(p)        \
    ((Aig_Obj_t *)((ABC_PTRUINT_T)(p) & ~01))

// 获取非反相版本
#define Aig_Not(p)            \
    ((Aig_Obj_t *)((ABC_PTRUINT_T)(p) ^ 01))

// 带条件的反相
#define Aig_NotCond(p,c)      \
    ((Aig_Obj_t *)((ABC_PTRUINT_T)(p) ^ (c)))

// ===== 读取fanin操作 =====

// 获取fanin0的真实节点指针
static inline Aig_Obj_t* Aig_ObjFanin0(Aig_Obj_t* pObj) {
    return Aig_Regular(pObj->pFanin0);
}

// 获取fanin1的真实节点指针  
static inline Aig_Obj_t* Aig_ObjFanin1(Aig_Obj_t* pObj) {
    return Aig_Regular(pObj->pFanin1);
}

// 检查fanin0是否反相
static inline int Aig_ObjFaninC0(Aig_Obj_t* pObj) {
    return Aig_IsComplement(pObj->pFanin0);
}

// 检查fanin1是否反相
static inline int Aig_ObjFaninC1(Aig_Obj_t* pObj) {
    return Aig_IsComplement(pObj->pFanin1);
}

// ===== 构造操作 =====

// 创建AND门 (根据条件设置反相标记)
Aig_Obj_t* Aig_And(Aig_Man_t* p, 
                    Aig_Obj_t* p0,   // fanin0 (可能已tagged)
                    Aig_Obj_t* p1) { // fanin1 (可能已tagged)
    Aig_Obj_t* pNode = Aig_TableLookup(p, p0, p1);
    if (pNode) return pNode;
    
    // 创建新节点
    pNode = Aig_ManCreateNode(p);
    pNode->pFanin0 = p0;  // 直接存储tagged pointer
    pNode->pFanin1 = p1;  
    pNode->Type = AIG_OBJ_AND;
    
    return pNode;
}

// 创建反相边: NOT(p) = AND(p, 1) 的特殊表示
Aig_Obj_t* Aig_Not_Func(Aig_Obj_t* p) {
    return Aig_Not(p);  // 翻转最低位
}
```

实际使用示例

- 构造简单逻辑电路

```c
// 构造电路: out = NOT(AND(a, NOT(b)))  即 a NAND b
void build_nand_circuit(Aig_Man_t* pMan) {
    // 创建输入端口
    Aig_Obj_t* pA = Aig_ObjCreateCi(pMan);  // 输入a
    Aig_Obj_t* pB = Aig_ObjCreateCi(pMan);  // 输入b
    
    // 创建 NOT(b) - 只是翻转指针的最低位!
    Aig_Obj_t* pNotB = Aig_Not(pB);  
    
    // 创建 AND(a, NOT(b))
    Aig_Obj_t* pAnd = Aig_And(pMan, pA, pNotB);
    
    // 创建 NOT(AND(...))
    Aig_Obj_t* pNand = Aig_Not(pAnd);
    
    // 创建输出端口
    Aig_ObjCreateCo(pMan, pNand);
}
```

- 遍历电路拓扑

```c
// 递归计算节点逻辑值
int Aig_ObjEvaluate(Aig_Obj_t* pObj, int* pInputVals) {
    // 常量节点
    if (Aig_ObjIsConst1(pObj)) {
        return 1;
    }
    
    // 输入节点
    if (Aig_ObjIsCi(pObj)) {
        return pInputVals[Aig_ObjCioId(pObj)];
    }
    
    // AND节点:需要处理fanin的complement标记
    assert(Aig_ObjIsAnd(pObj));
    
    // 1. 获取真实的fanin节点指针
    Aig_Obj_t* pFanin0 = Aig_ObjFanin0(pObj);  
    Aig_Obj_t* pFanin1 = Aig_ObjFanin1(pObj);
    
    // 2. 递归计算fanin的值
    int val0 = Aig_ObjEvaluate(pFanin0, pInputVals);
    int val1 = Aig_ObjEvaluate(pFanin1, pInputVals);
    
    // 3. 根据complement标记进行反相
    if (Aig_ObjFaninC0(pObj)) val0 = !val0;
    if (Aig_ObjFaninC1(pObj)) val1 = !val1;
    
    // 4. 返回AND结果
    return val0 & val1;
}
```
