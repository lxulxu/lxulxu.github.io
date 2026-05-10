---
title: '字符串池化解析：以Yosys IdString为例'
date: 2026-02-13
tags: [c++, eda]
categories: [c++]
---



## 问题背景

在C++中，两个内容相同的字符串`std::string`是不同的对象：

```cpp
std::string s1 = "clk";
std::string s2 = "clk";
s1 == s2;  // true，但O(n)逐字符比较
&s1[0] != &s2[0];  // 不同内存地址

```

 - 比较开销： O(n)逐字符比较 
 - 内存冗余： 相同字符串多份拷贝 
 - 哈希表性能： 重复计算哈希值 
 - 缓存不友好： 分散存储，cache miss 

## [Yosys IdString](https://github.com/YosysHQ/yosys/blob/main/kernel/rtlil.h)的解决方案

IdString本质上是一个int

```cpp
struct IdString {
    int index_;  // 4字节ID代表字符串的身份
    
    // O(1)比较
    bool operator==(const IdString &rhs) const { 
        return index_ == rhs.index_; 
    }
};
```

### 全局存储架构

```cpp

// 实际字符串存储
static std::vector<char*> global_id_storage_;
// 索引 -> 字符串指针
// [0] -> "" 
// [1] -> "$auto$1"
// [2] -> "\\clk" 
// [3] -> "$and"

// 字符串到索引映射
static dict<char*, int> global_id_index_;
// "$auto$1" -> 1
// "\\clk"   -> 2

//引用计数
static std::vector<int> global_refcount_storage_;
// [1] -> 5   (5个地方引用"$auto$1")
// [2] -> 100 (100个地方引用"clk")

//空闲索引复用
static std::vector<int> global_free_idx_list_;
// [42, 57, 83]  // 这些索引可以复用
```

### String Interning

```cpp
static int get_reference(const char *p)
{
    log_assert(destruct_guard_ok);

    if (!p[0])
        return 0;

    // 查找是否已存在
    auto it = global_id_index_.find((char*)p);
    if (it != global_id_index_.end()) {
        // 已存在：增加引用计数，返回索引
        global_refcount_storage_.at(it->second)++;
        return it->second;
    }

    log_assert(p[0] == '$' || p[0] == '\\');
    log_assert(p[1] != 0);
    for (const char *c = p; *c; c++)
        if ((unsigned)*c <= (unsigned)' ')
            log_error("Found control character or space (0x%02x) in string '%s'"
                      " which is not allowed in RTLIL identifiers\n", *c, p);

    //分配新索引
    int idx;
    if (global_free_idx_list_.empty()) {
        // 无空闲索引：扩容
        idx = global_id_storage_.size();
        global_id_storage_.push_back(nullptr);
        global_refcount_storage_.push_back(0);
    } else {
        // 复用空闲索引
        idx = global_free_idx_list_.back();
        global_free_idx_list_.pop_back();
    }

    // 存储字符串
    global_id_storage_.at(idx) = strdup(p);
    global_id_index_[global_id_storage_.at(idx)] = idx;
    global_refcount_storage_.at(idx) = 1;

    return idx;
}
```

### 引用计数生命周期管理

```cpp
// 获取引用
static inline int get_reference(int idx)
{
    if (idx) {
        global_refcount_storage_[idx]++;
    }
    return idx;
}

// 释放引用
static inline void put_reference(int idx)
{
    if (!destruct_guard_ok || !idx)
        return;

    int &refcount = global_refcount_storage_[idx];

    if (--refcount > 0)
        return;  // 还有引用，不释放

    // 引用归零：释放字符串，回收索引
    log_assert(refcount == 0);
    free_reference(idx);
}

static inline void free_reference(int idx)
{
    // 从哈希表移除
    global_id_index_.erase(global_id_storage_.at(idx));
    // 释放C字符串
    free(global_id_storage_.at(idx));
    global_id_storage_.at(idx) = nullptr;
    // 索引加入空闲列表
    global_free_idx_list_.push_back(idx);
}
```

### 析构保护机制

```cpp
// 防止静态析构顺序问题
static bool destruct_guard_ok;  // POD，初始化为false

static struct destruct_guard_t {
    destruct_guard_t() { destruct_guard_ok = true; }
    ~destruct_guard_t() { destruct_guard_ok = false; }
} destruct_guard;

```

```cpp
// 问题场景：全局IdString在静态析构时
static IdString g_clk("\\clk");  // 全局对象

// main结束后析构顺序不确定：
// 如果global_refcount_storage_先析构，g_clk析构时访问已释放内存
// destruct_guard在global_refcount_storage_之后析构，保护机制生效
```

### Sticky IDs优化

```cpp
#ifdef YOSYS_USE_STICKY_IDS
static int last_created_idx_ptr_;
static int last_created_idx_[8];  // 环形缓冲区
#endif

// 在get_reference中：
#ifdef YOSYS_USE_STICKY_IDS
// 避免 Create->Delete->Create 模式
// 场景：临时IdString创建后立即销毁，然后再次创建相同字符串
// 问题：索引被回收又分配，导致缓存失效
// 解决：保留最近创建的8个索引不回收

if (last_created_idx_[last_created_idx_ptr_])
    put_reference(last_created_idx_[last_created_idx_ptr_]);
last_created_idx_[last_created_idx_ptr_] = idx;
get_reference(last_created_idx_[last_created_idx_ptr_]);
last_created_idx_ptr_ = (last_created_idx_ptr_ + 1) & 7;
#endif
```


## IdString 相比 std::string 的优势

### 性能比较

| 场景 | IdString | std::string |
|:---|:---|:---|
| 对象大小 | 4 字节 | 32 字节 |
| 字符串存储 | 全局唯一存储，无重复 | 每个 string 独立管理或共享 |
| 构造（已存在） | **O(1)** 哈希查找 | O(n) 拷贝 |
| 比较 | **O(1)** 整数比较 | O(n) 逐字符 |
| 哈希 | **O(1)** 预计算 | O(n) 实时计算 |
| 拷贝 | **O(1)** 整数拷贝 | O(n) 深拷贝或引用计数 |
| 析构 | **O(1)** | 取决于实现 |

### 缓存局部性

```cpp
// IdString：索引访问连续内存，缓存友好
for (auto id : id_list) {
    const char* str = global_id_storage_[id.index_];  // 顺序访问
}

// std::string：指针跳跃，缓存不友好
for (const auto& s : string_list) {
    // s.data() 指向堆上随机位置
}
```


## 底层用 `char*` 而非 `std::string`的优势

`char*` + `strdup` 实现隐式去重

```cpp
static dict<char*, int> global_id_index_;  // char* → 索引
static std::vector<char*> global_id_storage_;  // 存储

// 插入时
char* dup = strdup(str);  // 新分配
if (global_id_index_.count(dup)) {
    free(dup);  // 已存在，释放重复
    return existing_index;
}
// 新字符串保留，地址唯一
```

利用 `strdup` 的堆分配特性——相同内容的字符串若已存在，其指针地址必然相同，哈希表可直接用指针比较替代字符串比较，达到 **O(1)** 的查找和比较性能。

| 考量 | `char*` | `std::string` |
|:---|:---|:---|
| **内存开销** | 8 字节指针 | ≥32 字节（SSO 除外） |
| **哈希表键比较** | 指针直接比较（地址相同即相等） | 逐字符比较或自定义哈希 |
| **去重机制** | `strdup` 保证相同内容相同地址 | 需额外处理 |

## 应用场景

### 推荐场景

- **字符串重复度高**（如编译器符号表、电路信号名），池化可显著节省内存
- **频繁进行比较或哈希操作**，整数索引替代逐字符比较
- **内存受限环境**，用索引替换指针+长度+容量的冗余存储

### 不适用场景

- **字符串基本唯一**，池化引入额外开销却无去重收益
- **比较操作极少**，池化的查找成本无法摊平
- **工具运行时间短**，构造和销毁池的开销占比过高
- **多线程高并发**，全局池成为锁竞争热点，反而降低性能
