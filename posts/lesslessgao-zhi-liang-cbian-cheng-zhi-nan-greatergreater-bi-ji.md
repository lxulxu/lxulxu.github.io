---
title: '《高质量C++编程指南》笔记'
date: 2022-09-04 11:51:33
tags: [c++,笔记]
published: true
hideInList: false
feature: 
isTop: false
---
## 文件结构

### 头文件结构

  ```c++
//版权和版本声明
/*
* Copyright (c) 2001,上海贝尔有限公司网络应用事业部
* All rights reserved.
*
* 文件名称：graphics.h
* 文件标识：见配置管理计划书
* 摘   要：简要描述本文件的内容
*
* 当前版本：1.1
* 作   者：输入作者（或修改者）名字
* 完成日期：2001年7月20日
*
* 取代版本：1.0
* 原作者 ：输入原作者（或修改者）名字
* 完成日期：2001年5月10日
*/

#ifndef GRAPHICS_H // 防止 graphics.h 被重复引用
#define GRAPHICS_H
#include <math.h> // 引用标准库的头文件
...
#include “myheader.h” // 引用非标准库的头文件
...
void Function1(...); // 全局函数声明
...
class Box // 类结构声明
{
...
};
#endif
  ```

#### 版权和版本的声明

1. 版权信息。
2. 文件名称，标识符，摘要。
3. 当前版本号，作者/修改者，完成日期。
4. 版本历史信息。

#### 预处理块

- 为了防止头文件被重复引用，应当用 ifndef/define/endif 结构产生预处理块

- 用 #include <filename.h> 格式来引用标准库的头文件（编译器将从标准库目录开始搜索）

- 用 #include “filename.h” 格式来引用非标准库的头文件（编译器将从用户的工作目录开始搜索）

- 【建议】*头文件中只存放“声明”而不存放“定义”*

- 【建议】*不提倡使用全局变量，尽量不要在头文件中出现象 extern int value 这类声明*

#### 函数和类结构声明等

### 定义文件结构

graphics.cpp

```c++
//版权和版本声明
/*
...
*/

#include “graphics.h” // 引用头文件
...
// 全局函数的实现体
void Function1(...)
{
...
}
// 类成员函数的实现体
void Box::Draw(...)
{
...
}
```

## 程序版式

### 空行

- 在每个类声明之后、每个函数定义结束之后都要加空行

```c++
// 空行
void Function1(...)
{
...
}
// 空行
void Function2(...)
{
...
}
```

- 在一个函数体内，逻揖上密切相关的语句之间不加空行，其它地方应加空行分隔


```c++
// 空行
while (condition)
{
	statement1;
	// 空行
	if (condition)
	{
		statement2;
	}
	else
	{
		statement3;
	}
	// 空行
	statement4;
}
```

### 代码行

- 一行代码只做一件事情，如只定义一个变量，或只写一条语句

- if、for、while、do 等语句自占一行，执行语句不得紧跟其后。不论执行语句有多少都要加{}，这样可以防止书写失误
- 【建议】*尽可能在定义变量的同时初始化该变量（就近原则）*

| 风格良好                                                     | 风格不良                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| int width; // 宽度<br/>int height; // 高度<br/>int depth; // 深度 | int width, height, depth; // 宽度高度深度                    |
| for (initialization; condition; update)<br/>{<br/>dosomething();<br/>}<br/>// 空行<br/>other(); | for (initialization; condition; update)<br/>dosomething();<br/>other(); |

### 代码行内空格

- 关键字之后要留空格。象 const、virtual、inline、case 等关键字之后至少要留一个空格，否则无法辨析关键字。象 if、for、while 等关键字之后应留一个空格再跟左括号‘（’，以突出关键字
- 函数名之后不要留空格，紧跟左括号‘（’，以与关键字区别
- ‘（’向后紧跟，‘）’、‘，’、‘;’向前紧跟，紧跟处不留空格
- ‘，’之后要留空格，如 Function(x, y, z)。如果‘;’不是一行的结束符号，其后要留空格，如 for (initialization; condition; update)
- 赋值操作符、比较操作符、算术操作符、逻辑操作符、位域操作符，如“=”、“+=” “>=”、“<=”、“+”、“*”、“%”、“&&”、“||”、“<<”,“^”等二元操作符的前后应当加空格
- 一元操作符如“!”、“~”、“++”、“--”、“&”（地址运算符）等前后不加空格
- 象“［］”、“.”、“->”这类操作符前后不加空格
- 对于表达式比较长的 for 语句和 if 语句，为了紧凑起见可以适当地去掉一些空格，如 for (i=0; i<10; i++)和 if ((a<=b) && (c<=d))

| 风格良好                                             | 风格不良                                                  |
| ---------------------------------------------------- | --------------------------------------------------------- |
| void Func1(int x, int y, int z);                     | void Func1 (int x,int y,int z);                           |
| if ((a>=b) && (c<=d))                                | if(a>=b&&c<=d)                                            |
| for (i=0; i<10; i++)                                 | for(i=0;i<10;i++)<br/>for (i = 0; I < 10; i ++)           |
| x = a < b ? a : b;                                   | x=a<b?a:b;                                                |
| int *x = &y;                                         | int * x = & y;                                            |
| array[5] = 0; <br/>a.Function(); <br/>b->Function(); | array [ 5 ] = 0;<br/>a . Function();<br/>b -> Function(); |

### 对齐

- 程序的分界符‘{’和‘}’应独占一行并且位于同一列，同时与引用它们的语句左对齐
- { }之内的代码块在‘{’右边数格处左对齐

```c++
void Function(int x)
{
	for (initialization; condition; update)
	{
		... // program code
	}
}
```

### 长行拆分

- 代码行最大长度宜控制在 70 至 80 个字符以内。代码行不要过长，否则眼睛看不过来，也不便于打印
- 长表达式要在低优先级操作符处拆分成新行，操作符放在新行之首（以便突出操作符）。拆分出的新行要进行适当的缩进，使排版整齐，语句可读

```c++
if ((very_longer_variable1 >= very_longer_variable12)
		&& (very_longer_variable3 <= very_longer_variable14)
		&& (very_longer_variable5 <= very_longer_variable16))
{
	dosomething();
}
	
virtual CMatrix CMultiplyMatrix (CMatrix leftMatrix,
																 CMatrix rightMatrix);
	
for (very_longer_initialization;
		 very_longer_condition;
		 very_longer_update)
{
	dosomething();
}
```

### 修饰符的位置

修饰符 * 和 ＆ 应该靠近数据类型还是该靠近变量名，是个有争议的活题。若将修饰符 * 靠近数据类型，例如：int* x; 从语义上讲此写法比较直观，即 x 是 int 类型的指针。上述写法的弊端是容易引起误解，例如：int* x, y; 此处 y 容易被误解为指针变量。虽然将 x 和 y 分行定义可以避免误解，但并不是人人都愿意这样做。

### 注释

C++语言中，程序块的注释常采用“/*...*/”，行注释一般采用“//...”。注释通常用于版本、版权声明/函数接口说明/重要的代码行或段落提示。


- 注释是对代码的“提示”，而不是文档，程序中的注释**不可喧宾夺主**。如果代码本来就是清楚的，则不必加注释。否则多此一举，令人厌烦。例如`i++; // i 加 1`
- 边写代码边注释，修改代码同时修改相应的注释
- 尽量**避免在注释中使用缩写**，特别是不常用缩写
- 注释的位置应与被描述的代码相邻，**可以放在代码的上方或右方**，不可放在下方
- 当代码比较长，特别是有**多重嵌套**时，应当在一些段落的结束处加注释，便于阅读

```c++
/*
* 函数介绍：
* 输入参数：
* 输出参数：
* 返回值 ：
*/

void Function(float x, float y, float z)
{
	if (...)
	{
		...
		while (...)
		{
			...
		} // end of while
		...
	} // end of if
}
```

### 类的版式

```c++
//以数据为中心版式
class A
{
	private:
		int i, j;
		float x, y;
		...
	public:
		void Func1(void);
		void Func2(void);
		...
}

//以行为为中心的版式
class A
{
	public:
		void Func1(void);
		void Func2(void);
		...
	private:
		int i, j;
		float x, y;
		...
}
```

## 命名规则

### 共性规则

- 标识符应当直观且可以拼读，可望文知意，不必进行“解码”。标识符最好采用**英文单词**或其组合，便于记忆和阅读。切忌使用汉语拼音来命名。

- 标识符的长度应当符合“min-length && max-information”原则。例如变量名 maxval 就比 maxValueUntilOverflow 好用。单字符的名字也是有用的，常见的如 i,j,k,m,n,x,y,z 等，它们通常可用作函数内的局部变量。

- 命名规则尽量与所采用的操作系统或开发工具的风格保持一致。例如 Windows 应用程序的标识符通常采用“大小写”混排的方式，如 AddChild。而 Unix 应用程序的标识符通常采用“小写加下划线”的方式，如 add_child。别把这两类风格混在一起用。

- 程序中不要出现仅靠大小写区分的相似的标识符

- 程序中不要出现标识符完全相同的局部变量和全局变量

- 变量的名字应当使用“名词”或者“形容词＋名词”如value、oldValue

- **全局函数**的名字应当使用“动词”或者“**动词＋名词**”（动宾词组）如DrawBox()。**类的成员函数**应当只使用“**动词**”，被省略掉的名词就是对象本身如box->Draw()。

- 用正确的反义词组命名具有互斥意义的变量或相反动作的函数等如minVaule/maxValue、SetValue/GetValue。

- 【建议】*尽量避免名字中出现数字编号，如 Value1,Value2 等，除非逻辑上的确需要编号。*

### Windows 命名规则

- 类名和函数名用大写字母开头的单词组合而成
- 变量和参数用小写字母开头的单词组合而成
- 常量全用大写的字母，用下划线分割单词
- 静态变量加前缀 s_（表示 static）
- 如果不得已需要全局变量，则使全局变量加前缀 g_（表示 global）
- 类的数据成员加前缀 m_（表示 member），这样可以避免数据成员与成员函数的参数同名。
- 为了防止某一软件库中的一些标识符和其它软件库中的冲突，可以为各种标识符加上能反映软件性质的前缀。例如三维图形标准 OpenGL 的所有库函数均以 gl 开头，所有常量（或宏定义）均以 GL 开头。

## 表达式和基本语句

### 运算符优先级

如果代码行中的运算符比较多，用括号确定表达式的操作顺序，避免使用默认的优先级，例如：word = (high << 8) | low。

### 复合表达式

- 不要编写太复杂的复合表达式，如`i = a >= b && c < d && c + f <= g + h ;`

- 不要有多用途的复合表达式。如`d = (a = b + c) + r ;`

- 不要把程序中的复合表达式与“真正的数学表达式”混淆，如`if (a < b < c)`

### if 语句

各类型变量与零值比较

| 风格良好                                         | 风格不良                                                     |
| ------------------------------------------------ | ------------------------------------------------------------ |
| 布尔变量<br/>if (flag)<br>if (!flag)             | if (flag == TRUE)<br/>if (flag == 1 )<br/>if (flag == FALSE)<br/>if (flag == 0) |
| 整形变量<br/>if (value == 0)<br/>if (value != 0) | if (value)<br/>if (!value)                                   |
| 浮点变量<br/>if ((x>=-EPSINON) && (x<=EPSINON))  | if (x == 0.0)                                                |
| 指针变量<br/>if (p == NULL)<br/>if (p != NULL)   | if (p == 0)<br/>if (p != 0)<br/>if (p)<br/>if (!p)           |

### 循环语句

- 【建议】*在多重循环中，如果有可能，应当将最长的循环放在最内层，最短的循环放在最外层，以减少 CPU 跨切循环层的次数。*
- 【建议】*如果循环体内存在逻辑判断，并且循环次数很大，宜将逻辑判断移到循环体的外面。循环次数N较大时逻辑判断会打断循环“流水线”作业使编译器不能对循环进行优化处理，N较小时则不必移动以保持程序简洁。*
- 不可在 for 循环体内修改循环变量，防止 for 循环失去控制。
- 【建议】*for 语句的循环控制变量的取值采用“半开半闭区间”写法，如`for (int x=0; x<N; x++)`而不是`for (int x=0; x<=N-1; x++)`。*
- switch 循环每个 case 语句的结尾不要忘了加 break，否则将导致多个分支重叠（除非有意使多个分支重叠）。
- switch 循环不要忘记 default 分支。即使程序真的不需要 default 处理。

## 常量

- 尽量使用含义直观的常量来表示那些将在程序中多次出现的数字或字符串，如`const float PI = 3.14159;`。

- 在 C++ 程序中只使用 const 常量而不使用宏常量，即 const 常量完全取代宏常量。

- 需要对外公开的常量放在头文件中，不需要对外公开的常量放在定义文件的头部。为便于管理，可以把不同模块的常量集中存放在一个公共的头文件中。

- 如果某一常量与其它常量密切相关，应在定义中包含这种关系，而不应给出一些孤立的值。如：

```c++
 const float RADIUS = 100;
 const float DIAMETER = RADIUS * 2;
```

- 类中常量通过枚举常量而非const实现

## 函数设计

### 参数规则

参数的书写要完整，不要贪图省事只写参数的类型而省略参数名字。如果函数没有参数，则用 void 填充。

参数命名要恰当，顺序要合理。例如编写字符串拷贝函数 StringCopy，它有两个参数。如果把参数名字起为 str1 和str2，例如 void StringCopy(char *str1, char *str2);那么我们很难搞清楚究竟是把 str1 拷贝到 str2 中，还是刚好倒过来。可以把参数名字起得更有意义，如叫 strSource 和 strDestination。这样从名字上就可以看出应该把 strSource 拷贝到 strDestination。还有一个问题，这两个参数那一个该在前那一个该在后？参数的顺序要遵循程序员的习惯。一般地，应将目的参数放在前面，源参数放在后面。

如果输入参数以值传递的方式传递对象，则宜改用“const &”方式来传递，这样可以省去临时对象的构造和析构过程，从而提高效率。

【建议】*避免函数有太多的参数，参数个数尽量控制在 5 个以内。*

【建议】*尽量不要使用类型和数目不确定的参数*

### 返回值规则

- 不要省略返回值的类型
- 函数名字与返回值类型在语义上不可冲突。违反这条规则的典型代表是 C 标准库函数 getchar，按照 getchar 名字的意思，将变量 c 声明为 char 类型是很自然的事情。但不幸的是 getchar 返回 int 类型。
- 不要将正常值和错误标志混在一起返回。正常值用输出参数获得，而错误标志用 return 语句返回。
- 【建议】*有时候函数原本不需要返回值，但为了增加灵活性如支持链式表达，可以附加返回值。例如字符串拷贝函数 strcpy 的原型：`char *strcpy(char *strDest，const char *strSrc);`，可以获得如下灵活性*：

```c++
char str[20];
int length = strlen( strcpy(str, “Hello World”) );
```

- 【建议】*如果函数的返回值是一个对象，有些场合用“引用传递”替换“值传递”可以提高效率。而有些场合只能用“值传递”而不能用“引用传递”，否则会出错。比如*：

```c++
class String
{...
	// 赋值函数
	String & operate=(const String &other);
  // 相加函数，如果没有 friend 修饰则只许有一个右侧参数
  friend String operate+( const String &s1, const String &s2);
	private:
  	char *m_data;
}
	

String & String::operate=(const String &other)
{
	if (this == &other)
	return *this;
	delete m_data;
	
	m_data = new char[strlen(other.data)+1];
	strcpy(m_data, other.data);
	return *this; // 返回的是 *this 的引用，无需拷贝过程
}
	
String operate+(const String &s1, const String &s2)
{
	String temp;
	delete temp.data; // temp.data 是仅含‘\0’的字符串
	temp.data = new char[strlen(s1.data) + strlen(s2.data) +1];
	strcpy(temp.data, s1.data);
	strcat(temp.data, s2.data);
	return temp;
}
```

对于赋值函数，应当用“引用传递”的方式返回 String 对象。如果用“值传递”的方式，虽然功能仍然正确，但由于 return 语句要把 *this 拷贝到保存返回值的外部存储单元之中，增加了不必要的开销，降低了赋值函数的效率。

对于相加函数，应当用“值传递”的方式返回 String 对象。如果改用“引用传递”，那么函数返回值是一个指向局部对象 temp 的“引用”。由于 temp 在函数结束时被自动销毁，将导致返回的“引用”无效。

### 函数内部实现规则

- 在函数体的“入口处”，对参数的有效性进行检查。

- 在函数体的“出口处”，对 return 语句的正确性和效率进行检查。


1. return 语句不可返回指向“栈内存”的“指针”或者“引用”，因为该内存在函数体结束时被自动销毁。例如


```c++
char * Func(void)
{
	char str[] = “hello world”; // str 的内存位于栈上
	...
	return str; // 将导致错误
}
```
2. 要搞清楚返回的究竟是“值”、“指针”还是“引用”
3. 如果函数返回值是一个对象，要考虑 return 语句的效率。例如 

```c++
return String(s1 + s2);
```

这是临时对象的语法，表示“创建一个临时对象并返回它”。不要以为它与“先创建一个局部对象 temp 并返回它的结果”是等价的，如

```c++
String temp(s1 + s2);
return temp;
```

实质不然，上述代码将发生三件事。首先，temp 对象被创建，同时完成初始化；然后拷贝构造函数把 temp 拷贝到保存返回值的外部存储单元中；最后，temp 在函数结束时被销毁（调用析构函数）。然而“创建一个临时对象并返回它”的过程是不同的，编译器直接把临时对象创建并初始化在外部存储单元中，省去了拷贝和析构的化费，提高了效率。

- 【建议】*函数的功能要单一，不要设计多用途的函数。*

- 【建议】*函数体的规模要小，尽量控制在 50 行代码之内。*

- 【建议】*尽量避免函数带有“记忆”功能。相同的输入应当产生相同的输出。带有“记忆”功能的函数，其行为可能是不可预测的，因为它的行为可能取决于某种“记忆状态”。这样的函数既不易理解又不利于测试和维护。在 C/C++语言中，函数的static 局部变量是函数的“记忆”存储器。建议尽量少用 static 局部变量，除非必需。*

- 【建议】*不仅要检查输入参数的有效性，还要检查通过其它途径进入函数体内的变量的有效性，例如全局变量、文件句柄等。*

- 【建议】*用于出错处理的返回值一定要清楚，让使用者不容易忽视或误解错误情况。*

### 使用断言

- 使用断言捕捉不应该发生的非法情况。不要混淆非法情况与错误情况之间的区别，后者是必然存在的并且是一定要作出处理的。
- 在函数的入口处，使用断言检查参数的有效性（合法性）。
- 【建议】*一般教科书都鼓励程序员们进行防错设计，但要记住这种编程风格可能会隐瞒错误。当进行防错设计时，如果“不可能发生”的事情的确发生了，则要使用断言进行报警。*

### 引用与指针

| 引用                                           | 指针                   |
| ---------------------------------------------- | ---------------------- |
| 被创建的同时必须被初始化                       | 可以在任何时候被初始化 |
| 不能有 NULL 引用，引用必须与合法的存储单元关联 | 可以是 NULL            |
| 一旦被初始化，就不能改变引用的关系             | 可以随时改变所指的对象 |

```c++
//指针传递
void Func1(int *x)
{
	(* x) = (* x) + 10;
}
...
int n = 0;
Func1(&n);
cout << “n = ” << n << endl; // n = 10

//引用传递
void Func2(int &x)
{
	x = x + 10;
}
...
int n = 0;
Func2(n);
cout << “n = ” << n << endl; // n = 10
```

对比上述两个个示例程序，会发现“引用传递”的性质象“指针传递”，而书写方式象“值传递”。实际上“引用”可以做的任何事情“指针”也都能够做，为什么还要“引用”这东西？答案是“**用适当的工具做恰如其分的工作**”。指针能够毫无约束地操作内存中的如何东西，尽管指针功能强大，但是非常危险。就象一把刀，它可以用来砍树、裁纸、修指甲、理发等等，谁敢这样用？如果的确只需要借用一下某个对象的“别名”，那么就用“引用”，而不要用“指针”，以免发生意外。比如说，某人需要一份证明，本来在文件上盖上公章的印子就行了，如果把取公章的钥匙交给他，那么他就获得了不该有的权利。

## 内存管理

### 内存分配方式

内存分配方式有三种：

1. 从**静态存储区域**分配。内存在程序编译的时候就已经分配好，这块内存在**程序的整个运行期间**都存在。例如**全局变量**，**static 变量**。
2. 在**栈**上创建。在执行函数时，函数内**局部变量**的存储单元都可以在栈上创建，**函数执行**结束时这些存储单元自动被释放。栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
3. 从**堆**上分配，亦称动态内存分配。程序在运行的时候用 **malloc** 或 **new** 申请任意多少的内存，程序员自己负责在何时用 free 或 delete 释放内存。动态内存的生存期**由我们决定**，使用非常灵活，但问题也最多。

### 常见内存错误

1. 内存**分配未成功**，却使用了它。在使用内存之前检查指针是否为 NULL。
2. 内存分配虽然成功，但是**尚未初始化**就引用它。犯这种错误主要有两个起因：一是没有初始化的观念；二是误以为内存的缺省初值全为零，导致引用初值错误（例如数组）。
3. 内存分配成功并且已经初始化，但**操作越过了内存的边界**。例如在使用数组时经常发生下标“多 1”或者“少 1”的操作。特别是在 for 循环语句中，循环次数很容易搞错，导致数组操作越界。
4. **忘记释放内存**，造成内存泄露。动态内存的**申请与释放必须配对**，程序中 malloc 与 free 的使用次数一定要相同，否则肯定有错误（ new/delete 同理）。
5. **释放了内存却继续使用它**。有三种情况：

​	1）程序中的对象调用关系过于复杂，实在难以搞清楚某个对象究竟是否已经释放了内存，此时应该重新设计数据结构，从根本上解决对象管理的混乱局面。

​	2）函数的 return 语句写错了，注意**不要返回指向“栈内存”的“指针”或者“引用”**，因为该内存在函数体结束时被自动销毁。

​	3）使用 free 或 delete 释放了内存后，没有将指针设置为 NULL。导致产生“**野指针**”。

  建议如下：

- 用 malloc 或 new 申请内存之后，应该立即**检查指针值是否为 NULL**。防止使用指针值为 NULL 的内存。

- 不要忘记**为数组和动态内存赋初值**。防止将未被初始化的内存作为右值使用。

- 避免数组或指针的**下标越界**，特别要当心发生“多 1”或者“少 1”操作。

- 动态内存的**申请与释放必须配对**，防止内存泄漏。

- 用 free 或 delete 释放了内存之后，立即将指针设置为 NULL，防止产生“野指针”。

### 数组与指针

- 数组要么在静态存储区被创建（如全局数组），要么在栈上被创建。数组名对应着（而不是指向）一块内存，其地址与容量在生命期内保持不变，只有数组的内容可以改变。

- 指针可以随时指向任意类型的内存块，它的特征是“可变”，所以我们常用指针来操作动态内存。指针远比数组灵活，但也更危险。

  举几例进行比较：

```c++
//修改内容 
//数组
char a[] = “hello”;
a[0] = ‘X’;
cout << a << endl;

//指针
char *p = “world”; // 注意 p 指向常量字符串（位于静态存储区，内容为 world\0）
p[0] = ‘X’; // 编译器不能发现该错误，但是该语句企图修改常量字符串的内容而导致运行错误
cout << p << endl;

//复制与比较
//数组
char a[] = "hello";
char b[10];
strcpy(b, a); // 不能用 b = a;
if(strcmp(b, a) == 0) // 不能用 if (b == a)

//指针
int len = strlen(a);
char *p = (char *)malloc(sizeof(char)*(len+1));
strcpy(p,a); // 不要用 p = a;
if(strcmp(p, a) == 0) // 不要用 if (p == a)
  
//内存容量
char a[] = "hello world";
char *p = a;
cout<< sizeof(a) << endl; // 12 字节（注意别忘了’\0’）
cout<< sizeof(p) << endl; // 4 字节，一个指针变量的字节数

//注意当数组作为函数的参数进行传递时，该数组自动退化为同类型的指针
void Func(char a[100])
{
	cout<< sizeof(a) << endl; // 4 字节而不是 100 字节
}
```

### 指针参数

如果函数的参数是一个指针，不要指望用该指针去申请动态内存。

```c++
void GetMemory(char *p, int num)
{
	p = (char *)malloc(sizeof(char) * num);
}
void Test(void)
{
	char *str = NULL;
	GetMemory(str, 100); // str 仍然为 NULL
	strcpy(str, "hello"); // 运行错误
}
```

毛病出在函数 GetMemory 中。编译器总是要**为函数的每个参数制作临时副本**，指针参数 `p` 的副本是 `_p`，编译器使 `_p` = `p`。如果函数体内的程序修改了 `_p` 的内容，就导致参数 `p` 的内容作相应的修改。这就是指针可以用作输出参数的原因。在本例中，`_p` 申请了新的内存，只是把 `_p` 所指的内存地址改变了，但是 `p` 丝毫未变。所以函数 GetMemory 并不能输出任何东西。事实上，每执行一次 GetMemory 就会泄露一块内存，因为没有用 free 释放内存。解决方法有以下两种：

```c++
//指向指针的指针
void GetMemory2(char **p, int num)
{
	*p = (char *)malloc(sizeof(char) * num);
}
void Test2(void)
{
	char *str = NULL;
	GetMemory2(&str, 100); // 注意参数是 &str，而不是 str
	strcpy(str, "hello");
	cout<< str << endl;
	free(str);
}

//函数返回值来传递动态内存
char *GetMemory3(int num)
{
	char *p = (char *)malloc(sizeof(char) * num);
	return p;
}
void Test3(void)
{
	char *str = NULL;
	str = GetMemory3(100);
	strcpy(str, "hello");
	cout<< str << endl;
	free(str);
}
```

用函数返回值来传递动态内存这种方法虽然好用，但是常常有人把 return 语句用错了。这里强调不要用 return 语句返回指向“栈内存”的指针，因为该内存在函数结束时自动消亡。

```c++
char *GetString(void)
{
	char p[] = "hello world";
	return p; // 编译器将提出警告
}
void Test4(void)
{
	char *str = NULL;
	str = GetString(); // str 的内容是垃圾
	cout<< str << endl;
}
```

而当 return 语句返回指向静态存储区的指针时，返回的始终是同一个“只读”的内存块。

```c++
char *GetString2(void)
{
	char *p = "hello world";
	return p;
}
void Test5(void)
{
	char *str = NULL;
	str = GetString2();
	cout<< str << endl;
}
```

### 其他

- free 和 delete 只是把指针所指的内存给释放掉，但并没有把指针本身干掉。如果后续不把指针设置为 NULL，会让人误以为是合法的指针。
- 指针消亡了，并不表示它所指的内存会被自动释放。内存被释放了，并不表示指针会消亡或者成了 NULL 指针。

```c++
void Func(void)
{
	char *p = (char *) malloc(100); // 动态内存会自动释放吗？
}
```

如果程序终止了运行，一切指针都会消亡，动态内存会被操作系统回收。既然如此，在程序临终前，就可以不必释放内存、不必将指针设置为 NULL 了。终于可以偷懒而不
会发生错误了吧？想得美。如果别人把那段程序取出来用到其它地方怎么办？

- **野指针**不是 NULL 指针，是指向“垃圾”内存的指针。成因主要有三种：
  1. 指针变量没有被初始化。任何指针变量刚被创建时不会自动成为 NULL 指针，它的缺省值是随机的，它会乱指一气。所以，指针变量在创建的同时应当被初始化，要么将指针设置为 NULL，要么让它指向合法的内存。
  2. 指针被 free 或者 delete 之后，没有置为 NULL，让人误以为它是合法指针。
  3. 指针操作超越了变量的作用范围。示例程序如下，函数 Test 在执行语句 p->Func()时，对象 a 已经消失，而 p 是指向 a 的，所以 p 就成了“野指针”。

```c++
class A
{
	public:
		void Func(void){ cout << “Func of class A” << endl; }
};
void Test(void)
{
	A *p;
	{
		A a;
		p = &a; // 注意 a 的生命期
	}
	p->Func(); // p 是“野指针”
}
```

- malloc/free 和 new/delete。malloc 与 free 是 C++/C 语言的**标准库函数**，new/delete 是C++的**运算符**。对于非内部数据类型的对象而言，光用 maloc/free 无法满足动态对象的要求。对象在创建的同时要自动执行构造函数，对象在消亡之前要自动执行析构函数。由于 malloc/free 是库函数而不是运算符，不在编译器控制权限之内，不能够把执行构造函数和析构函数的任务强加于 malloc/free。因此C++语言需要一个能完成动态内存分配和初始化工作的运算符 new，以及一个能完成清理与释放内存工作的运算符 delete。
- 如果在申请动态内存时找不到足够大的内存块，malloc 和 new 将返回 NULL 指针，宣告内存申请失败。通常有三种方式处理“内存耗尽”问题。
  1. 判断指针是否为 NULL，如果是则马上用 return 语句终止本函数。
  2. 判断指针是否为 NULL，如果是则马上用 exit(1) 终止整个程序的运行。
  3. 为 new 和 malloc 设置异常处理函数。例如Visual C++可以用 _set_new_hander 函数为 new 设置用户自己定义的异常处理函数，也可以让 malloc 享用与 new 相同的异常处理函数。

## C++函数的高级特性

### 函数重载

- 重载函数通过参数不同来进行区分，编译器根据参数为每个重载函数产生不同的内部标识符。如`void foo(int x, int y);`C++编译器会产生像`_foo_int_int`之类的名字用来支持函数重载和类型安全连接，C编译器会产生`_foo`这样的名字。由于**编译后的名字不同**，C++程序不能直接调用C函数。C++提供了一个C连接交换指定符号 extern“C” 来解决这个问题。
- 当心隐式类型转换导致重载函数产生二义性。

```c++
# include <iostream.h>
void output( int x); // 函数声明
void output( float x); // 函数声明

void output( int x)
{
	cout << " output int " << x << endl ;
}

void output( float x)
{
	cout << " output float " << x << endl ;
}

void main(void)
{
	int x = 1;
	float y = 1.0;
	output(x); // output int 1
	output(y); // output float 1
	output(1); // output int 1
	// output(0.5); // error! ambiguous call, 因为自动类型转换
	output(int(0.5)); // output int 0
	output(float(0.5)); // output float 0.5
}
```

### 成员函数的重载、覆盖与隐藏

- 重载与覆盖


| 重载                       | 覆盖                               |
| -------------------------- | ---------------------------------- |
| 相同的范围（在同一个类中） | 不同的范围（分别位于派生类与基类） |
| 函数名字相同               | 函数名字相同                       |
| 参数不同                   | 参数相同                           |
| virtual 关键字可有可无     | 基类函数必须有 virtual 关键字      |

如下，函数 Base::f(int) 与 Base::f(float) 相互重载，而 Base::g(void) 被 Derived::g(void) 覆盖。

```c++
#include <iostream.h>
class Base
{
	public:
		void f(int x){ cout << "Base::f(int) " << x << endl; }
		void f(float x){ cout << "Base::f(float) " << x << endl; }
		virtual void g(void){ cout << "Base::g(void)" << endl; }
};

class Derived : public Base
{
	public:
		virtual void g(void){ cout << "Derived::g(void)" << endl; }
};

void main(void)
{
	Derived d;
	Base *pb = &d;
	pb->f(42); // Base::f(int) 42
	pb->f(3.14f); // Base::f(float) 3.14
	pb->g(); // Derived::g(void)
}
```

- 隐藏规则

“隐藏”是指派生类的函数屏蔽了与其同名的基类函数，规则如下：

1. 如果派生类的函数与基类的函数同名，但是**参数不同**。此时，不论有无 virtual 关键字，基类的函数将被隐藏（注意别与重载混淆）。
2. 如果派生类的函数与基类的函数同名，并且**参数相同**，但是基类函数没有 virtual 关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）。

如下,函数 Derived::f(float)覆盖了 Base::f(float)，函数 Derived::g(int) 隐藏了 Base::g(float)，而不是重载，函数 Derived::h(float) 隐藏了 Base::h(float)，而不是覆盖。

```c++
#include <iostream.h>
class Base
{
	public:
		virtual void f(float x){ cout << "Base::f(float) " << x << endl; }
		void g(float x){ cout << "Base::g(float) " << x << endl; }
		void h(float x){ cout << "Base::h(float) " << x << endl; }
};
class Derived : public Base
{
	public:
		virtual void f(float x){ cout << "Derived::f(float) " << x << endl; }
		void g(int x){ cout << "Derived::g(int) " << x << endl; }
		void h(float x){ cout << "Derived::h(float) " << x << endl; }
};

void main(void)
{
	Derived d;
	Base *pb = &d;
	Derived *pd = &d;
	// Good : behavior depends solely on type of the object
	pb->f(3.14f); // Derived::f(float) 3.14
	pd->f(3.14f); // Derived::f(float) 3.14
	// Bad : behavior depends on type of the pointer
	pb->g(3.14f); // Base::g(float) 3.14
	pd->g(3.14f); // Derived::g(int) 3 (surprise!)
	// Bad : behavior depends on type of the pointer
	pb->h(3.14f); // Base::h(float) 3.14 (surprise!)
	pd->h(3.14f); // Derived::h(float) 3.14
}
```

- 摆脱隐藏

  如下，语句 pd->f(10)的本意是想调用函数 Base::f(int)，但是 Base::f(int)不幸被 Derived::f(char *) 隐藏了。由于数字 10 不能被隐式地转化为字符串，所以在编译时出错。

```c++
class Base
{
	public:
		void f(int x);
};
class Derived : public Base
{
	public:
		void f(char *str);
};
void Test(void)
{
	Derived *pd = new Derived;
	pd->f(10); // error
}
```

如果语句 pd->f(10)一定要调用函数 Base::f(int)，那么将类 Derived 修改为如下即可。

```c++
class Derived : public Base
{
	public:
		void f(char *str);
		void f(int x) { Base::f(x); }
};
```

### 参数的缺省值

- 参数缺省值只能出现在函数的声明中，而不能出现在定义体中。

```c++
void Foo(int x=0, int y=0); // 正确，缺省值出现在函数的声明中

void Foo(int x=0, int y=0) // 错误，缺省值出现在函数的定义体中
{
	...
}
```

- 如果函数有多个参数，参数只能从后向前挨个儿缺省，否则将导致函数调用语句怪模怪样。

```c++
void Foo(int x, int y=0, int z=0); // 正确

void Foo(int x=0, int y, int z=0); // 错误
```

防止不合理地使用参数的缺省值将导致重载函数产生二义性。

```c++
#include <iostream.h>
void output( int x);
void output( int x, float y=0.0);

void output( int x)
{
	cout << " output int " << x << endl ;
}

void output( int x, float y)
{
	cout << " output int " << x << " and float " << y << endl ;
}

void main(void)
{
	int x=1;
	float y=0.5;
	// output(x); // error! ambiguous call
	output(x,y); // output int 1 and float 0.5
}
```

### 运算符重载

在 C++语言中，可以用关键字 operator 加上运算符来表示函数，叫做运算符重载。例如两个复数相加函数：`Complex Add(const Complex &a, const Complex &b);`可以用运算符重载来表示：`Complex operator +(const Complex &a, const Complex &b);`。运算符与普通函数在调用时的不同之处是：对于普通函数，参数出现在圆括号内；而对于运算符，参数出现在其左、右侧。

```c++
Complex a, b, c;
...
c = Add(a, b); // 用普通函数
c = a + b; // 用运算符 +
```

运算符定义为函数的规则如下。

| 运算符                           | 规则               |
| -------------------------------- | ------------------ |
| 所有的一元运算符                 | 建议重载为成员函数 |
| = () [] ->                       | 只能重载为成员函数 |
| += -= /= *= &= \|= ~= %= >>= <<= | 建议重载为成员函数 |
| 所有其它运算符                   | 建议重载为全局函数 |

在 C++运算符集合中，有一些运算符是不允许被重载的。这种限制是出于安全方面的考虑，可防止错误和混乱。

- 不能改变 C++**内部数据类型**（如 int,float 等）的运算符。
- 不能重载‘.’，因为‘.’在类中对任何成员都有意义，已经成为标准用法。
- 不能重载目前 C++运算符集合中没有的符号，如#,@,$等。原因有两点，一是难以理解，二是难以确定优先级。
- 对已经存在的运算符进行重载时，不能改变优先级规则，否则将引起混乱。 

### 函数内联

关键字 inline 必须与**函数定义体**放在一起才能使函数成为内联，仅将 inline 放在函数声明前面不起任何作用。

```c++
void Foo(int x, int y);
inline void Foo(int x, int y) // inline 与函数定义体放在一起
{
	...
}
```

内联是以代码膨胀（复制）为代价，仅仅省去了函数调用的开销，从而提高函数的执行效率。如果执行函数体内代码的时间，相比于函数调用的开销较大，那么效率的收
获会很少。另一方面，每一处内联函数的调用都要复制代码，将使程序的总代码量增大，消耗更多的内存空间。以下情况不宜使用内联：

- 如果函数体内的**代码比较长**，使用内联将导致内存消耗代价较高。
- 如果函数体内**出现循环**，那么执行函数体内代码的时间要比函数调用的开销大。

## 类

对于任意一个类 A，C++编译器将自动为 A 产生四个缺省的函数，如

```c++
A(void); // 缺省的无参数构造函数
A(const A &a); // 缺省的拷贝构造函数
~A(void); // 缺省的析构函数
A & operate =(const A &a); // 缺省的赋值函数
```

注意“缺省的拷贝构造函数”和“缺省的赋值函数”均采用“位拷贝”而非“值拷贝”的方式来实现，倘若类中含有指针变量，这两个函数注定将出错。

### 构造函数的初始化表

构造函数有个特殊的初始化方式叫“初始化表达式表”（简称初始化表）。初始化表位于函数参数表之后，却在函数体 {} 之前。这说明该表里的初始化工作发生在函数体内的任何代码被执行之前。构造函数初始化表的使用规则：

- 如果类存在继承关系，派生类必须在其初始化表里**调用基类的构造函数**。

```c++
class A
{...
	A(int x); // A 的构造函数
};

class B : public A
{...
	B(int x, int y);// B 的构造函数
};

B::B(int x, int y)
	: A(x) // 在初始化表里调用 A 的构造函数

{
	...
}
```

- 类的 **const 常量**只能在初始化表里被初始化，因为它不能在函数体内用赋值的方式来初始化。
- 类的数据成员的初始化可以采用初始化表或函数体内赋值两种方式，这两种方式的效率不完全相同。非内部数据类型的成员对象应当采用第一种方式初始化，以获取更高的效率。对于内部数据类型的数据成员而言，两种初始化方式的效率几乎没有区别，但后者的程序版式似乎更清晰些。

```c++
class A
{...
	A(void); // 无参数构造函数
	A(const A &other); // 拷贝构造函数
	A & operate =( const A &other); // 赋值函数
}；
  
class B
{
	public:
		B(const A &a); // B 的构造函数
	private:
		A m_a; // 成员对象
};

//第一种
B::B(const A &a)
	: m_a(a)
{
	...
}

//第二种
B::B(const A &a)
{
	m_a = a;
	...
}
```

### 拷贝构造函数与赋值函数

- 如果不主动编写拷贝构造函数和赋值函数，编译器将以“位拷贝”的方式自动生成缺省的函数。倘若类中含有指针变量，那么这两个缺省的函数就隐含了错误。以类 String 的两个对象 a,b 为例，假设 a.m_data 的内容为“hello”，b.m_data的内容为“world”。现将 a 赋给 b，缺省赋值函数的“位拷贝”意味着执行 b.m_data = a.m_data。这将造成三个错误：一是 b.m_data 原有的内存没被释放，造成内存泄露；二是 b.m_data 和 a.m_data 指向同一块内存，a 或 b 任何一方变动都会影响另一方；三是在对象被析构时，m_data 被释放了两次。
- 拷贝构造函数是在对象被创建时调用的，而赋值函数只能被已经存在了的对象调用。

```c++
String a(“hello”);
String b(“world”);
String c = a; // 调用了拷贝构造函数，风格较差最好写成 c(a);
c = b; // 调用了赋值函数
```

- 如果我们实在不想编写拷贝构造函数和赋值函数，又不允许别人使用编译器生成的缺省函数，偷懒的办法是只需将拷贝构造函数和赋值函数声明为私有函数，不用编写代码。

### 类String

```c++
class String
{
	public:
		String(const char *str = NULL); // 普通构造函数
		String(const String &other); // 拷贝构造函数
		~ String(void); // 析构函数
		String & operate =(const String &other); // 赋值函数
	private:
		char *m_data; // 用于保存字符串
};

// String 的普通构造函数
String::String(const char *str)
{
  if (str == NULL)
  {
    m_data = new char[1];
    *m_data = '\0';
  }
  else
  {
    int length = strlen(str);
    m_data = new char[length + 1];
    strcpy(m_data, str);
  }
}

// 拷贝构造函数
String::String(const String &other)
{
  int length = strlen(other.m_data);
  m_data = new char[length + 1];
  strcpy(m_data, other.m_data);
}

// String 的析构函数
String::~String(void)
{
  delete []m_data;// 由于 m_data 是内部数据类型，也可以写成 delete m_data;
}

// 赋值函数
String::String& operate =(const String &other)
{
  // (1) 检查自赋值
	if (this == &other)
    return *this;
  
  // (2) 释放原有的内存资源
  delete []m_data;
  
  // （3）分配新的内存资源，并复制内容
	int length = strlen(other.m_data);
	m_data = new char[length+1];
	strcpy(m_data, other.m_data);
  
	// （4）返回本对象的引用
	return *this;
}
```



### 派生类中实现类的基本函数

- 派生类的构造函数应在其初始化表里调用基类的构造函数。
- 基类与派生类的**析构函数应该为虚**（即加 virtual 关键字）。
- 在编写派生类的赋值函数时，注意不要忘记对基类的数据成员重新赋值，如下所示：

```c++
class Base
{
	public:
		Base & operate =(const Base &other); // 类 Base 的赋值函数
	private:
		int m_i, m_j, m_k;
};

class Derived : public Base
{
	public:
		Derived & operate =(const Derived &other); // 类 Derived 的赋值函数
	private:
		int m_x, m_y, m_z;
};

Derived & Derived::operate =(const Derived &other)
{
	//（1）检查自赋值	
	if(this == &other)
	return *this;
  
	//（2）对基类的数据成员重新赋值
	Base::operate =(other); // 因为不能直接操作私有数据成员
  
	//（3）对派生类的数据成员赋值
	m_x = other.m_x;
	m_y = other.m_y;
	m_z = other.m_z;
  
	//（4）返回本对象的引用
	return *this;
}
```

### 类的继承

如果 A 是基类，B 是 A 的派生类，类 A 和类 B 毫不相关，不可以为了使 B 的功能更多些而让 B 继承 A 的功能和属性。若在逻辑上 B 是 A 的“一种”（a kind of ），则允许 B 继承 A 的功能和属性。例如男人（Man）是人（Human）的一种，男孩（Boy）是男人的一种。那么类 Man 可以从类 Human 派生，类 Boy 可以从类 Man 派生。

### 类的组合

若在逻辑上 A 是 B 的“一部分”（a part of），则不允许 B 从 A 派生，而是要用 A 和其它东西组合出 B。例如眼（Eye）、鼻（Nose）、口（Mouth）、耳（Ear）是头（Head）的一部分，所以类 Head 应该由类 Eye、Nose、Mouth、Ear 组合而成，不是派生而成。如下所示。

```c++
class Eye
{
	public:
		void Look(void);
};

class Nose
{
	public:
		void Smell(void);
};

class Mouth
{
	public:
		void Eat(void);
};

class Ear
{
	public:
		void Listen(void);
};

// 正确的设计，虽然代码冗长。
class Head
{
	public:
		void Look(void) { m_eye.Look(); }
		void Smell(void) { m_nose.Smell(); }
		void Eat(void) { m_mouth.Eat(); }
		void Listen(void) { m_ear.Listen(); }
  
	private:
		Eye m_eye;
		Nose m_nose;
		Mouth m_mouth;
		Ear m_ear;
};
```

如果允许 Head 从 Eye、Nose、Mouth、Ear 派生而成，那么 Head 将自动具有 Look、Smell、Eat、Listen 这些功能，这种设计方法却是不对的。

## 其他

### 使用 const 提高函数的健壮性

#### 修饰输入参数

- 如果输入参数采用“指针传递”，那么加 const 修饰可以**防止意外地改动该指针**，起到保护作用。例如 StringCopy 函数`void StringCopy(char *strDestination, const char *strSource)`，其中 strSource 是输入参数，strDestination 是输出参数。给 strSource 加上 const 修饰后，如果函数体内的语句试图改动 strSource 的内容，编译器将指出错误。
- 如果输入参数采用“值传递”，由于函数将自动产生临时变量用于复制该参数，该输入参数本来就无需保护，所以不要加 const 修饰。例如不要将函数`void Func1(int x)` 写成`void Func1(const int x)`。同理不要将函数`void Func2(A a)`写成`void Func2(const A a)`。其中 A 为用户自定义的数据类型。
- 对于非内部数据类型的参数而言，象`void Func(A a)`这样声明的函数效率较低。因为函数体内将产生 A 类型的临时对象用于复制参数 a，而临时对象的构造、复制、析构过程都将消耗时间。为了提高效率，可以将函数声明改为 `void Func(A &a)`，因为“引用传递”仅借用一下参数的别名而已，不需要产生临时对象。但是函数`void Func(A &a)` 存在一个缺点：“引用传递”有可能改变参数 a，这是我们不期望的。解决这个问题很容易，加 const 修饰即可，因此函数最终成为`void Func(const A &a)`。

#### 修饰返回值

- 如果给以“指针传递”方式的函数返回值加 const 修饰，那么函数返回值（即指针）的内容不能被修改，该返回值只能被赋给加 const 修饰的同类型指针。

```c++
const char * GetString(void);

//错误
char *str = GetString();

//正确
const char *str = GetString();
```

- 如果函数返回值采用“值传递方式”，由于函数会把返回值复制到外部临时的存储单元中，加 const 修饰没有任何价值。例如不要把函数 int GetInt(void) 写成 const int GetInt(void)。
- 函数返回值采用“引用传递”的场合并不多，这种方式一般只出现在类的赋值函数中，目的是为了实现链式表达。例如

```c++
class A
{...
	A & operate = (const A &other); // 赋值函数
};
A a, b, c; // a, b, c 为 A 的对象
...
a = b = c; // 正常的链式赋值

(a = b) = c; // 不正常的链式赋值，但合法
```

如果将赋值函数的返回值加 const 修饰，那么该返回值的内容不允许被改动。上例中，语句 a = b = c 仍然正确，但是语句 (a = b) = c 则是非法的。

#### 修饰成员函数

任何不会修改数据成员的函数都应该声明为 const 类型。如果在编写 const 成员函数时，不慎修改了数据成员，或者调用了其它非 const 成员函数，编译器将指出错误，这无疑会提高程序的健壮性。以下程序中，类 stack 的成员函数 GetCount 仅用于计数，从逻辑上讲 GetCount 应当为 const 函数。编译器将指出 GetCount 函数中的错误。

```c++
class Stack
{
	public:
		void Push(int elem);
		int Pop(void);
		int GetCount(void) const; // const 成员函数
	private:
		int m_num;
		int m_data[100];
};

int Stack::GetCount(void) const
{
	++ m_num; // 编译错误，企图修改数据成员 m_num
	Pop(); // 编译错误，企图调用非 const 函数
	return m_num;
}
```

### 提高程序效率

程序的时间效率是指运行速度，空间效率是指程序占用内存或者外存的状况。全局效率是指站在整个系统的角度上考虑的效率，局部效率是指站在模块或函数角度上考虑的效率。

- 不要一味地追求程序的效率，应当在满足正确性、可靠性、健壮性、可读性等质量因素的前提下，设法提高程序的效率。
- 以提高程序的全局效率为主，提高局部效率为辅。
- 在优化程序的效率时，应当先找出限制效率的“瓶颈”，不要在无关紧要之处优化。
- 先优化数据结构和算法，再优化执行代码。
- 有时候时间效率和空间效率可能对立，此时应当分析那个更重要，作出适当的折衷。例如多花费一些内存来提高性能。
- 不要追求紧凑的代码，因为紧凑的代码并不能产生高效的机器码。

### 一些有益的建议

- 当心那些视觉上不易分辨的操作符发生书写错误。我们经常会把“＝＝”误写成“＝”，象“||”、“&&”、“<=”、“>=”这类符号也很容易发生“丢 1”失误。然而编译器却不一定能自动指出这类错误。
- 变量（指针、数组）被创建之后应当**及时把它们初始化**，以防止把未被初始化的变量当成右值使用。
- 当心变量的**初值、缺省值**错误，或者精度不够
- 当心数据类型转换发生错误，尽量使用**显式的数据类型转换**。
- 当心变量发生上溢或下溢，数组的**下标越界**。
- 当心忘记编写错误处理程序，当心错误处理程序本身有误。
- 当心文件 I/O 有错误
- 避免编写技巧性很高代码
- 不要设计面面俱到、非常灵活的数据结构
- 如果原有的代码质量比较好，尽量复用它。但是不要修补很差劲的代码，应当重新编写
- 尽量使用标准库函数，不要“发明”已经存在的库函数。
- 尽量不要使用与具体硬件或软件环境关系密切的变量
- 把编译器的选择项设置为最严格状态
- 如果可能的话，使用 PC-Lint、LogiScope 等工具进行代码审查。