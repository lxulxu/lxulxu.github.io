---
title: 'C++安全指南'
date: 2024-04-28 08:20:35
tags: [c++]
categories: [笔记]
---
## 编程习惯
- switch中应有default
- 不应在debug或错误信息中提供过多内容
- 不应该在客户端代码中硬编码对称加密秘钥
```c++
    // Bad
    char g_aes_key[] = {...};
    void Foo() {
      ....
      AES_func(g_aes_key, input_data, output_data);
    }
```
```c++
    // Good
    char* g_aes_key;
    void Foo() {
      ....
      AES_encrypt(g_aes_key, input_data, output_data);
    }
    void Init() {
      g_aes_key = get_key_from_https(user_id, ...);
    }
```
- 函数不可以返回栈上的变量的地址，而应当使用堆来传递非简单类型变量，强烈建议返回 string、vector 等类型。
```c++
    // Bad
    char* Foo(char* sz, int len){
      char a[300] = {0};
      if (len > 100) {
        memcpy(a, sz, 100);
      }
      a[len] = '\0';
      return a;  // WRONG
    }
```
```c++
    // Good
    char* Foo(char* sz, int len) {
        char* a = new char[300];
        if (len > 100) {
            memcpy(a, sz, 100);
        }
        a[len] = '\0';
        return a;  // OK
    }
```
- 有逻辑联系的数组必须仔细检查
```c++
    // Good
    const int nWeekdays[] = {1, 2, 3, 4, 5, 6, 7};
    const char* sWeekdays[] = {"Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"};
    assert(ARRAY_SIZE(nWeekdays) == ARRAY_SIZE(sWeekdays));//确保有关联的nWeekdays和sWeekdays数据统一
    for (int x = 0; x < ARRAY_SIZE(sWeekdays); x++) {
      if (strcmp(sWeekdays[x], input) == 0) {
        return nWeekdays[x];
      }
    }
```
- 在头文件、源代码、文档中列举的函数声明应当一致，不应当出现定义内容错位的情况
错误示例：
foo.h
```c++
    int CalcArea(int width, int height);
```
foo.cc
```c++
    int CalcArea(int height, int width) {  // Different from foo.h
      if (height > real_height) {
        return 0;
      }
      return height * width;
    }
```
- 检查复制粘贴的重复代码（相同代码通常代表错误）
- 左右一致的重复判断/永远为真或假的判断（通常代表错误）
- **函数每个分支都应有返回值**：开启适当级别的警告（GCC 中为 -Wreturn-type 并已包含在 -Wall 中）并设置为错误，可以在编译阶段发现这类错误。
```c++
    // Bad
    int Foo(int bar) {
      if (bar > 100) {
        return 10;
      } else if (bar > 10) {
        return 1;
      }
    }
```
上述例子当bar<10时，其结果是未知的值。
- 不得使用栈上未初始化的变量
- 不得直接使用刚分配的未初始化的内存（如realloc），在 C++ 中，再次强烈推荐用 string、vector 代替手动内存分配。
```c++
    // Bad
    char* Foo() {
      char* a = new char[100];
      a[99] = '\0';
      memcpy(a, "char", 4);
      return a;
    }
```
```c++
    // Good
    char* Foo() {
      char* a = new char[100];
      memcpy(a, "char", 4);
      a[4] = '\0';
      return a;
    }
```
- 与内存分配相关的函数需要检查其返回值是否正确，以防导致程序崩溃或逻辑错误
```c++
    // Bad
    void Foo() {
      char* bar = mmap(0, 0x800000, .....);
      *(bar + 0x400000) = '\x88'; // Wrong
    }
```
```c++
    // Good
    void Foo() {
      char* bar = mmap(0, 0x800000, .....);
      if(bar == MAP_FAILED) {
        return;
      }
      *(bar + 0x400000) = '\x88';
    }
```
- 不要在if里面赋值
- if里，非bool类型和非bool类型的按位操作可能代表代码存在错误
## 文件操作
- **避免路径穿越问题**：在进行文件操作时，需要判断外部传入的文件名是否合法，如果文件名中包含 ../ 等特殊字符，则会造成路径穿越，导致任意文件的读写。
```c++
    void Foo() {
      char file_path[PATH_MAX] = "/home/user/code/";
      // 如果传入的文件名包含../可导致路径穿越
      // 例如"../file.txt"，则可以读取到上层目录的file.txt文件
      char name[20] = "../file.txt";
      memcpy(file_path + strlen(file_path), name, sizeof(name));
      int fd = open(file_path, O_RDONLY);
      if (fd != -1) {
        char data[100] = {0};
        int num = 0;
        memset(data, 0, sizeof(data));
        num = read(fd, data, sizeof(data));
        if (num > 0) {
          write(STDOUT_FILENO, data, num);
        }
        close(fd);
      }
    }
```
```c++
    void Foo() {
      char file_path[PATH_MAX] = "/home/user/code/";
      char name[20] = "../file.txt";
      // 判断传入的文件名是否非法，例如"../file.txt"中包含非法字符../，直接返回
      if (strstr(name, "..") != NULL){
        // 包含非法字符
        return;
      }
      memcpy(file_path + strlen(file_path), name, sizeof(name));
      int fd = open(file_path, O_RDONLY);
      if (fd != -1) {
        char data[100] = {0};
        int num = 0;
        memset(data, 0, sizeof(data));
        num = read(fd, data, sizeof(data));
        if (num > 0) {
          write(STDOUT_FILENO, data, num);
        }
        close(fd);
       }
    }
```
- 避免相对路径导致的安全问题（DLL、EXE劫持等问题）
- **文件权限控制**：在创建文件时，需要根据文件的敏感级别设置不同的访问权限，以防止敏感数据被其他恶意程序读取或写入。
```c++
    int Foo() {
      // 不要设置为777权限，以防止被其他恶意程序操作
      if (creat("file.txt", 0777) < 0) {
        printf("文件创建失败！\n");
      } else {
        printf("文件创建成功！\n");
      }
      return 0;
    }
```
## 内存操作
- 防止各种越界写（向前/向后）
```c++
    int a[5];
    a[5] = 0;
```
- **防止任意地址写**:任意地址写会导致严重的安全隐患，可能导致代码执行。因此，在编码时必须校验写入的地址。
错误示例：
```c++
    void Write(MyStruct dst_struct) {
      char payload[10] = { 0 };
      memcpy(dst_struct.buf, payload, sizeof(payload));
    }
    int main() {
      MyStruct dst_stuct;
      dst_stuct.buf = (char*)user_controlled_value;
      Write(dst_stuct);
      return 0;
    }
```
## 数字操作
- 防止整数溢出
```c++
    const kMicLen = 4;
    // 整数溢出
    void Foo() {
      int len = 1;
      char payload[10] = { 0 };
      char dst[10] = { 0 };
      // Bad, 由于len小于4字节，导致计算拷贝长度时，整数溢出
      // len - MIC_LEN == 0xfffffffd
      memcpy(dst, payload, len - kMicLen);
    }
```
```c++
    void Foo() {
      int len = 1;
      char payload[10] = { 0 };
      char dst[10] = { 0 };
      int size = len - kMicLen;
      // 拷贝前对长度进行判断
      if (size > 0 && size < 10) {
        memcpy(dst, payload, size);
        printf("memcpy good\n");
      }
    }
```
- 防止Off-By-One：在进行计算或者操作时，如果使用的最大值或最小值不正确，使得该值比正确值多1或少1，可能导致安全风险。
```c++
    char firstname[20];
    char lastname[20];
    char fullname[40];
    fullname[0] = '\0';
    strncat(fullname, firstname, 20);
    // 第二次调用strncat()可能会追加另外20个字符。如果这20个字符没有终止空字符，则存在安全问题
    strncat(fullname, lastname, 20);
```
```c++
    char firstname[20];
    char lastname[20];
    char fullname[40];
    fullname[0] = '\0';
    // 当使用像strncat()函数时，必须在缓冲区的末尾为终止空字符留下一个空字节，避免off-by-one
    strncat(fullname, firstname, sizeof(fullname) - strlen(fullname) - 1);
    strncat(fullname, lastname, sizeof(fullname) - strlen(fullname) - 1);
```
- 避免大小端错误
- 检查除以零异常
- 防止数字类型的错误强转
```c++
    int Foo() {
      int len = 1;
      unsigned int size = 9;
      // 1 < 9 - 10 ? 由于运算中无符号和有符号混用，导致计算结果以无符号计算
      if (len < size - 10) {
        printf("Bad\n");
      } else {
        printf("Good\n");
      }
    }
```
```c++
    void Foo() {
      // 统一两者计算类型为有符号
      int len = 1;
      int size = 9;
      if (len < size - 10) {
        printf("Bad\n");
      } else {
        printf("Good\n");
      }
    }
```
- 比较数据大小时加上最小/最大值的校验
```c++
    void Foo(int index) {
      int a[30] = {0};
      // 此处index是int型，只考虑了index小于数组大小，但是并未判断是否大于0
      if (index < 30) {
        // 如果index为负数，则越界
        a[index] = 1;
      }
    }
```
```c++
    void Foo(int index) {
      int a[30] = {0};
      // 判断index的最大最小值
      if (index >=0 && index < 30) {
        a[index] = 1;
      }
    }
```
## 指针操作
- 检查在pointer上使用sizeof：除了测试当前指针长度，否则一般不会在pointer上使用sizeof。
可能错误：
```c++
    size_t structure_length = sizeof(Foo*);
```
```c++
    size_t structure_length = sizeof(Foo);
```
- 检查直接将数组和0比较的代码：开启足够的编译器警告（GCC 中为 -Waddress，并已包含在 -Wall 中），并设置为错误，可以在编译期间发现该问题。
- 不应当向指针赋予写死的地址：特殊情况需要特殊对待（比如开发硬件固件时可能需要写死），但是如果是系统驱动开发之类的，写死可能会导致后续的问题。
- 检查空指针
```c++
    *foo = 100;
    if (!foo) {
      ERROR("foobar");
    }
```
```c++
    if (!foo) {
      ERROR("foobar");
    }
    *foo = 100;
```
- 释放完后置空指针
```c++
    void foo() {
      char* p = (char*)malloc(100);
      memcpy(p, "hello", 6);
      // 此时p所指向的内存已被释放，但是p所指的地址仍然不变
      printf("%s\n", p);
      free(p);
      // 未设置为NULL，可能导致UAF等内存错误
      if (p != NULL) {  // 没有起到防错作用
        printf("%s\n", p); // 错误使用已经释放的内存
      }
    }
```
```c++
    void foo() {
      char* p = (char*)malloc(100);
      memcpy(p, "hello", 6);
      // 此时p所指向的内存已被释放，但是p所指的地址仍然不变
      printf("%s\n", p);
      free(p);
      //释放后将指针赋值为空
      p = NULL;
      if (p != NULL)  { // 没有起到防错作用
        printf("%s\n", p); // 错误使用已经释放的内存
      }
    }
```
- 防止错误的类型转换
```c++
    const int NAME_TYPE = 1;
    const int ID_TYPE = 2;
    // 该类型根据 msg_type 进行区分，如果在对MessageBuffer进行操作时没有判断目标对象，则存在类型混淆
    struct MessageBuffer {
      int msg_type;
      union {
        const char *name;
        int name_id;
      };
    };
    void Foo() {
      struct MessageBuffer buf;
      const char* default_message = "Hello World";
      // 设置该消息类型为 NAME_TYPE，因此buf预期的类型为 msg_type + name
      buf.msg_type = NAME_TYPE;
      buf.name = default_message;
      printf("Pointer of buf.name is %p\n", buf.name);
      // 没有判断目标消息类型是否为ID_TYPE，直接修改nameID，导致类型混淆
      buf.name_id = user_controlled_value;
      if (buf.msg_type == NAME_TYPE) {
        printf("Pointer of buf.name is now %p\n", buf.name);
        // 以NAME_TYPE作为类型操作，可能导致非法内存读写
        printf("Message: %s\n", buf.name);
      } else {
        printf("Message: Use ID %d\n", buf.name_id);
      }
    }
```
```c++
    void Foo() {
      struct MessageBuffer buf;
      const char* default_message = "Hello World";
      // 设置该消息类型为 NAME_TYPE，因此buf预期的类型为 msg_type + name
      buf.msg_type = NAME_TYPE;
      buf.name = default_msessage;
      printf("Pointer of buf.name is %p\n", buf.name);
      // 判断目标消息类型是否为 ID_TYPE，不是预期类型则做对应操作
      if (buf.msg_type == ID_TYPE)
        buf.name_id = user_controlled_value;
      if (buf.msg_type == NAME_TYPE) {
        printf("Pointer of buf.name is now %p\n", buf.name);
        printf("Message: %s\n", buf.name);
      } else {
        printf("Message: Use ID %d\n", buf.name_id);
      }
    }
```
- 智能指针使用安全
```c++
    class Foo {
     public:
      explicit Foo(int num) { data_ = num; };
      void Function() { printf("Obj is %p, data = %d\n", this, data_); };
     private:
      int data_;
    };
    std::unique_ptr<Foo> fool_u_ptr = nullptr;
    Foo* pfool_raw_ptr = nullptr;
    void Risk() {
      fool_u_ptr = make_unique<Foo>(1);
      // 从独占智能指针中获取原始指针,<Foo>(1)
      pfool_raw_ptr = fool_u_ptr.get();
      // 调用<Foo>(1)的函数
      pfool_raw_ptr->Function();
      // 独占智能指针重新赋值后会释放内存
      fool_u_ptr = make_unique<Foo>(2);
      // 通过原始指针操作会导致UAF，pfool_raw_ptr指向的对象已经释放
      pfool_raw_ptr->Function();
    }
    // 输出：
    // Obj is 0000027943087B80, data = 1
    // Obj is 0000027943087B80, data = -572662307
```
```c++
    void Safe() {
      fool_u_ptr = make_unique<Foo>(1);
      // 调用<Foo>(1)的函数
      fool_u_ptr->function();
      fool_u_ptr = make_unique<Foo>(2);
      // 调用<Foo>(2)的函数
      fool_u_ptr->function();
    }
    // 输出：
    // Obj is 000002C7BB550830, data = 1
    // Obj is 000002C7BB557AF0, data = 2
```