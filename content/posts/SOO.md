---
title: '小对象优化'
date: 2026-02-11
tags: [c++]
categories: [c++]
---


## 核心原理

小对象优化采用空间换时间的策略，在容器对象内部预留一个固定大小的内部缓冲区，专门用于存储小对象数据。

以libstdc++ std::string为例, std::string通常需要动态分配内存来存储实际数据。对于大型对象，堆分配的开销相对于数据处理成本而言是可以接受的。但当频繁处理小对象时，情况就不同了：

- 分配器开销：每次调用new/delete或malloc/free都涉及复杂的内存管理算法，包括寻找合适大小的内存块、维护空闲列表等
- 内存碎片：大量小内存块的分配和释放会导致堆内存碎片化，降低内存利用率
- 缓存局部性差：堆上分配的小对象在内存中分布散乱，访问时缓存命中率低


```cpp
class string {
    struct _Alloc_hider {
        char* _M_p;  // 指向数据的指针
    } _M_dataplus;
    
    size_t _M_string_length;
    
    enum { _S_local_capacity = 15 };
    
    union {
        char _M_local_buf[_S_local_capacity + 1];  // 16字节栈缓冲
        size_t _M_allocated_capacity;               // 堆容量
    };
    
    // 指针比较法:本地缓冲地址固定
    bool _M_is_local() const {
        return _M_dataplus._M_p == _M_local_buf;
    }
    
    char* _M_data() const {
        return _M_dataplus._M_p;
    }
    
    size_t capacity() const {
        return _M_is_local() ? _S_local_capacity 
                             : _M_allocated_capacity;
    }
};
```

区分当前使用内部缓冲区还是堆内存常见策略包括：

- 利用容量字段的特殊值：当某成员变量为某个特殊值时表示使用内部缓冲区
- 专用标志位：使用某个成员变量字段既存储剩余空间信息，又作为状态标识
- 指针值判断：通过检查指针是否指向内部缓冲区来判断状态(libstdc++ std::string采用)

```cpp
// 构造时设置
string(const char* s) {
    size_t len = strlen(s);
    if (len <= _S_local_capacity) {
        _M_dataplus._M_p = _M_local_buf;  // 指向本地
        memcpy(_M_local_buf, s, len + 1);
    } else {
        _M_dataplus._M_p = allocate(len + 1);  // 堆分配
        memcpy(_M_dataplus._M_p, s, len + 1);
        _M_allocated_capacity = len;
    }
    _M_string_length = len;
}

// 拷贝构造
string(const string& other) {
    size_t len = other._M_string_length;
    if (other._M_is_local()) {
        // 短字符串: 栈拷贝
        memcpy(_M_local_buf, other._M_local_buf, len + 1);
        _M_dataplus._M_p = _M_local_buf;
    } else {
        // 长字符串: 堆分配+拷贝
        _M_dataplus._M_p = allocate(len + 1);
        memcpy(_M_dataplus._M_p, other._M_dataplus._M_p, len + 1); 
        _M_allocated_capacity = len;
    }
    _M_string_length = len;
}

// 移动构造
string(string&& other) noexcept {
    if (other._M_is_local()) {
        // 短字符串: 必须拷贝(数据在other的栈上)
        memcpy(_M_local_buf, other._M_local_buf, 
               other._M_string_length + 1);
        _M_dataplus._M_p = _M_local_buf;
    } else {
        // 长字符串: 指针转移
        _M_dataplus._M_p = other._M_dataplus._M_p;
        _M_allocated_capacity = other._M_allocated_capacity;
        other._M_dataplus._M_p = other._M_local_buf;  // 重置为local
        other._M_local_buf[0] = '\0';
    }
    _M_string_length = other._M_string_length;
    other._M_string_length = 0;
}

~string() {
    if (!_M_is_local()) {
        deallocate(_M_dataplus._M_p);
    }
}
```

## 应用场景

- **图形/游戏引擎** - 小型数学对象（2D/3D向量、四元数、颜色值）内联存储避免cache miss
- **JSON/XML解析器** - 短字符串、小数组节点内联存储，减少解析时的内存碎片
