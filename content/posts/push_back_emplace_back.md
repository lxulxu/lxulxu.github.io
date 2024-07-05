---
title: 'C++中的push_back与emplace_back'
date: 2024-07-05 18:48:00
tags: [c++]
categories: [c++]
---

**1. 引言**

C++标准库提供了`push_back`和`emplace_back`两种向容器末尾添加元素的方法。本文将深入分析这两个函数的区别、使用场景，以及在实际应用中的性能考虑。

**2. 基本概念**

2.1 push_back
`push_back`有两个重载版本：

```cpp
void push_back(const T& value);
void push_back(T&& value);
```

第一个版本复制元素，第二个版本移动元素。

2.2 emplace_back
`emplace_back`是C++11引入的变参模板函数：

```cpp
template <class... Args>
void emplace_back(Args&&... args);
```

它直接在容器中构造对象，参数被完美转发给元素的构造函数。

**3. 主要区别**

1. **构造方式**：`push_back`需要预先构造的对象，`emplace_back`在容器内构造对象。
2. **参数传递**：`push_back`接受对象，`emplace_back`接受构造函数参数。
3. **效率**：`emplace_back`可能避免不必要的临时对象创建和复制/移动操作。
4. **灵活性**：`emplace_back`可直接传递构造函数参数。
5. **编译复杂度**：`emplace_back`作为变参模板可能增加编译时间和内存使用。

**4. 使用示例**

4.1 简单类型

```cpp
std::vector<int> vec;
vec.push_back(10);  // push_back 足够简单高效
```

4.2 复杂对象构造

```cpp
std::vector<std::pair<int, std::string>> vec;
// 使用 push_back
vec.push_back(std::make_pair(1, "one"));
// 使用 emplace_back
vec.emplace_back(1, "one");  // 更简洁，直接传递构造函数参数
```

4.3 不可移动类型

```cpp
std::vector<std::mutex> mutexes;
mutexes.emplace_back();  // 可以工作，直接在容器中构造 mutex
// mutexes.push_back(std::mutex()); // 编译错误，mutex 不可复制或移动
```

**5. 性能考虑：emplace_back并非总是更优**

虽然通常认为 `emplace_back` 在性能上优于 `push_back`，但实际情况可能并非如此简单。

5.1 理论上的优势

```cpp
class MyClass {
public:
    MyClass(int a, double b) : x(a), y(b) {
        std::cout << "MyClass constructed\n";
    }
    MyClass(const MyClass&) {
        std::cout << "MyClass copied\n";
    }
    MyClass(MyClass&&) noexcept {
        std::cout << "MyClass moved\n";
    }
private:
    int x;
    double y;
};

std::vector<MyClass> vec;
vec.push_back(MyClass(10, 3.14));  // 构造 + 移动
vec.emplace_back(10, 3.14);  // 直接构造，理论上更高效
```

理论上，`emplace_back` 通过直接在容器内构造对象，避免了额外的移动操作，因此应该更高效。

5.2 编译器优化的影响

```cpp
std::vector<std::string> vec;
vec.push_back(std::string("Hello"));  // 可能被优化，避免额外复制
vec.emplace_back("World");  // 直接构造
```

现代编译器的优化能力可能会显著减小 `push_back` 和 `emplace_back` 之间的性能差距。

5.3 C++17及以后版本的改进

```cpp
struct Expensive {
    Expensive() { std::cout << "Constructed\n"; }
    Expensive(const Expensive&) { std::cout << "Copied\n"; }
    Expensive(Expensive&&) noexcept { std::cout << "Moved\n"; }
};

std::vector<Expensive> vec;
vec.push_back(Expensive());  // C++17: 直接构造，不会调用移动构造函数
vec.emplace_back();          // 直接构造
```

C++17引入的保证复制省略（guaranteed copy elision）进一步模糊了 `push_back` 和 `emplace_back` 之间的界限。

5.4 性能结论

尽管 `emplace_back` 在某些情况下确实可能提供性能优势，但这种优势并不像人们通常认为的那样普遍或显著。实际上，由于编译器优化和语言标准的演进，在许多常见情况下，`push_back` 和 `emplace_back` 的性能可能非常接近。

**6. 代码维护性**

在追求性能和保持代码可读性之间找平衡很重要：

```cpp
// 可能性能稍好，但可读性较差
vec.emplace_back(std::piecewise_construct, 
                 std::forward_as_tuple(1), 
                 std::forward_as_tuple("complex"));

// 性能可能稍差，但更易读和维护
vec.push_back(std::make_pair(1, std::string("complex")));
```

**7. 结论**

选择`push_back`还是`emplace_back`取决于多个因素，包括对象类型、构造复杂度、代码可读性和具体的性能需求。`emplace_back` 经常被误认为比 `push_back` 更好，或者与移动语义相关，但这是错误的。在大多数情况下，选择最清晰、最直观的方法通常是最好的做法。

**建议在日常使用中优先选择`push_back`。**只有在需要`emplace_back`的特定功能时（例如，当处理`deque<mutex>`或其他不可移动的类型时），或者在性能确实成为问题时，才考虑使用`emplace_back`。

**8. 参考链接**

[1] https://stackoverflow.com/questions/4303513/push-back-vs-emplace-back 
[2] https://quuxplusone.github.io/blog/2021/03/03/push-back-emplace-back/

