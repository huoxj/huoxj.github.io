---
title: "C++期末复习"
date: 2024-12-24T14:21:51-08:00
categories: 
- "C++拾遗"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "根据上课 PPT 和同学与学长的笔记整理而成。 先跳过。。。 {{}} 副作用，就是一个过程是否会修改参与的变量的性质。 在表达式中，有副作用的运算符：`=` `+=` `++` `<<` 等等 无副..."
---

根据上课 PPT 和同学与学长的笔记整理而成。

## 介绍

### C++ 历史

先跳过。。。

## 结构化编程部分
### 强制类型转换

{{< article link="/posts/C++拾遗/C++-强制类型转换" >}}

### 表达式的副作用

副作用，就是一个过程是否会修改参与的变量的性质。

在表达式中，有副作用的运算符：`=` `+=` `++` `<<` 等等
无副作用的运算符：`+` `&&` 等等 

---

### decltype

和 auto 有相似之处。

auto 和 decltype 都用在声明变量上，都是基于 RTTI 机制实现的。

auto 是对**初始化**的右值进行类型推导，然后给声明的变量确定类型、赋值。

decltype 是对传入的参数类型进行类型推导，然后本身就作为这个类型的名称。

```cpp
int x = 0;
int& ref = x;
auto ref1 = x;
decltype(ref) ref2 = x;  // equals to "int& ref2 = x"
```

既然本身就作为这个类型的名称，我们也可以结合 using 和 typedef：

```cpp
int x = 0;
int& ref = x;
using int_ref = decltype(ref);
```

#### 参考

[C++11特性：decltype关键字 - melonstreet - 博客园](https://www.cnblogs.com/QG-whz/p/4952980.html)

### 聚合初始化

聚合初始化的规则比较复杂，考试应该不涉及。下面简单举几个例子：

```cpp
int[] arr = {1, 2, 3};
int[5] arr1 = {1, 2, 3, 4};
vector<int> arr2{1, 2, 3};
vector<vector<int>> arr3{{1, 2, 3}, {4, 5}, {6}};
```

### Union 实现多态

父类联合体包含所有子类结构体。然后记得给子类加上类型的枚举。

例子中的声明顺序只是为了方便查看，不用在意是否能编译通过。

```cpp
// Base
union FIGURE
{
	FIGURE_TYPE t;
	Line line;
	Rectangle rect;
	Ellipse ellipse;
};
// Derived
struct Line{
	FIGURE_TYPE t;
	int x, y;
};
struct Ellipse
{ 
	FIGURE_TYPE t;
	int x, y, r;
};
struct Rectangle{
	FIGURE_TYPE t;
	int left, top, right, bottom;
};
```

### Struct 对齐

先讨论没有 `#pragma pack`  的情况，如何确定 struct 的对齐与大小。

首先有如下规则：

- 结构体的成员**声明的顺序**就是在**内存中的顺序**
- 某一个成员相较于 struct 开头的偏移量，必须是自身大小的整数倍
- 整个结构体大小必须是最大成员大小的整数倍

第一条规则，确定了成员的顺序就是声明的顺序。

我们按照声明的顺序，一个一个地应用第二条规则。不满足规则的，就补齐。

最后，看满不满足第三条规则。不满足就补齐。

有了 pragma 之后，每个数据成员的对齐，按照#pragma pack指定的数值和自身对齐模数中较小的那个。

---

### cin/cout 输入/输出操纵符

操纵符是令代码能以 operator<< 或 operator>> 控制输入/输出流的帮助函数。

包括的一些感觉比较重要的操纵符如下
输入用：
- `skipws`
- `get_time`
输出用
- `showbase` `setbase`
- `uppercase`
- `dec` `hex` `oct`
- `setprecision`
- `setw`
- `put_time`

可以翻阅 cppreference -> 输入输出库 -> 输入输出操纵符

### Tuple

大名叫 `std::tuple`。元组。能保存任意数量的任意类型的成员变量。可以看成一个类型安全、使用灵活便捷的结构体。

头文件 `<tuple>`

tuple 是静态的，需要在声明的时候就确定要保存哪些类型：
```cpp
std::tuple<int, double, std::string, char> t;
```

初始化：

```cpp
// constructor
std::tuple<int, double, std::string, char> t(1, 2.0, "hello", 'c');
// make_tuple
t = make_tuple(1, 2.0, "hello", 'c');
```

使用 `std::get<>` 获取元素：

```cpp
int first = std::get<0>(t);
int i = 1;
std::get<i>(t); // Wrong!!! Cannot use variable
```

使用 `std::tie` 解包：

```cpp
int a;
double b;
std::string c;
char d;
std::tie(a, b, c, d) = t;
```

使用 `std::tuple_size` 获取 tuple 元素个数：

```cpp
int el_count = std::tuple_size<decltype(t)>::value;
```

#### 参考

[C++ tuple（STL tuple）模板用法详解 - RioTian - 博客园](https://www.cnblogs.com/RioTian/p/14325292.html)

[C++ tuple元组的基本用法(总结)-CSDN博客](https://blog.csdn.net/sevenjoin/article/details/88420885)

### Optional

和 java 的 optional 如出一辙。大致是为了包装函数返回值而生的。

头文件 `<optinal>`

optional 可以理解为包含了两个信息：函数运行信息（成功，失败）、函数返回值

声明 optional：

```cpp
std::optional<T> opt;         // 不包含值的 optional
nullopt;                      // 不包含值的 optional，宏定义好了
std::optional<T> opt(value);  // 包含 value
make_optional<T>(value);      // make 方法
```

函数失败：

```cpp
return nullopt;
```

函数成功，返回 result：

```cpp
return make_optional<T>(result);
```

函数调用方：
使用 `has_value` 检查函数返回是否为空（函数执行是否失败）
如果成功，可以直接将 optional 当成结果来取值。注意，这里 value 类似迭代器，是一个指向结果的指针。

```cpp
auto result = foo();
if(!result.has_value()) {
	cout<< "failed";
}
else {
	result->xxx ...;       // 直接将 optional 对象当成返回结果的指针用
}
```

#### 参考

[C++17 新特性之 std::optional（上） - 知乎](https://zhuanlan.zhihu.com/p/64985296)

[C++三剑客之std::optional(一) : 使用详解_c++ optional-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/136099479)

### Pair

特化的 tuple。只能放两个元素。

头文件 `<utility>`

用 first 和 second 来访问两个元素。

make_pair 来创建一个 pair。通过 tie 来解包。和 tuple 一毛一样。

### Variant

类型安全的 union。但不允许包含引用、数组、void。

头文件 `<variant>`

创建与访问：

```cpp
std::variant<int, float, std::double> var;
var = "abc";  // or var.emplace("abc")
std::get<std::string>(var);  // "abc"
std::get<int>(var);          // exception std::bad_variant_access

var = 0.1f;
const auto intPtr = std::get_if<float>(&var)  // safe get
```

检查当前是否持有某类型：

使用 `std::holds_alternative`

```cpp
if(std::holds_alternative<int>(var)) {
	std::cout<<"var holds int"<<endl;
}
```

空状态：

即 variant 什么都没放的 “无值” 状态。用 `std::monostate` 表示

```cpp
std::variant<std::monostate, int> var;
```

#### 参考

[C++17 std::variant 详解：概念、用法和实现细节 - 非法关键字 - 博客园](https://www.cnblogs.com/linxmouse/p/18436326)

### Any

类型安全的，能存放任意类型数据的容器(void*)。

头文件 `any`

声明与赋值：

```cpp
std::any a;
a = 1;           // store an integer
a.emplace(1);    // same effect as above
a = make_any(1)  // make method
```

取出使用：

```cpp
a = 1;
std::any_cast<int>(a);    // returns 1
std::any_cast<char>(a);   // throw std::bad_any_cast exception
```

检查是否有值：

使用 `has_value` 成员函数。返回 bool。

```cpp
std::any a;
a.has_value();  // false
a = 1;
a.has_value();  // true
```

检查所存储元素类型：

使用 type 成员函数。返回成员类型的 typeid

```cpp
a = 1;
a.type() == typeid(int)  // true
a.type() == typeid(char) // false
```

### new/delete

在使用 new 创建数组时，前 4 个字节会存放声明的数组长度。

方便 delete\[] 时，知道要释放多少空间。

### RAII 与智能指针

{{< article link="/posts/C++拾遗/杂项" >}}

---

### C 数组特性

{{< article link="/posts/C++拾遗/指针、引用与数组" >}}

### 指针作为函数参数

为什么
- 提高传输效率，不用拷贝传递的实参
- 使用函数的副作用，改变传入参数的值。如果不想要副作用，可以加 const

### 函数指针

没啥讲的。方便实现函数的动态调用。然后就是 lambda 函数。

函数指针的解析，看：

{{< article link="/posts/C++拾遗/类型解释" >}}

### 函数执行机制

如下只是大致的步骤，会根据具体的函数调用约定而有些许不同。
- 建立被调用函数的栈空间
- 参数传递：
	- 值传递 (call by value)
	- 引用传递 (call by reference)
- 保存调用函数的运行状态
	- 返回地址
	- 调用者的 ebp
- 将控制转交被调函数
	- 设置新的 ebp 和 esp
- 恢复上下文
	- 恢复调用者 ebp
	- 恢复返回地址

#### 函数调用约定

函数调用约定回答了如下问题：
- 当参数个数多于一个时，按照什么顺序把参数压入堆栈
- 函数调用后，由谁来把堆栈恢复原装

##### stdcall

pascal 风格的调用约定。

```cpp
int __stdcall foo(int a, int b);
```

- 参数从右向左压栈
- 函数自己恢复堆栈
例子中，被调用者 foo 的返回的汇编为 `ret 8`，表示清除 a 和 b 参数占的堆栈
- 函数名的标签变为 `_` + `函数名` + `@` + `参数大小`
例子中，函数的标签为 `_foo@8`

##### cdecl

C 调用约定。是 C 语言默认的调用约定。

```c
int __cdecl foo(int a, int b);
int foo(int a, int b);            // default is cdecl
```

- 参数从右往左压栈
- 调用者恢复堆栈
被调用者直接用 `ret` 返回，不恢复堆栈。
调用者在 `call foo` 之后，使用 `add esp, 8` 来恢复两个 int 参数的堆栈。
- 函数名标签 `_` + `函数名`

这种调用约定能支持可变参数。可变参数在声明里面是放在最右边的，这样保证了左边的参数先入栈，能通过 ebp 确定，从而确认可变参数。并且调用者可以根据自己传的参数大小来恢复堆栈。

##### fastcall

和 stdcall 类似，但是更 fast，因为用到了寄存器。

```cpp
int __fastcall foo(int a, int b);
```

- 第一个和第二个声明的大小小于等于 DWORD 的参数，通过 ecx 和 edx 传
例子中，a 和 b 分别放到 ecx 和 edx 中。然后就没有参数需要压栈了。
- 其他的参数还是按 stdcall 的方式压栈
- 被调用者清理堆栈

##### thiscall

C++ 默认的调用约定。特殊处理了 this 指针。

```cpp
int foo(int a, int b);
int __thiscall foo(int a, int b);  // wrong! no need for '__thiscall'
```

- 参数从右向左压栈
- 参数个数确定的情况下，this 指针通过 ecx 传递；有可变参数，this 指针最后被入栈
- 参数个数确定，被调用者清理堆栈；可变参数，调用者清理堆栈

#### 参考

[C/C++ 函数调用约定（__cdecl、__stdcall、__fastcall）-CSDN博客](https://blog.csdn.net/hellokandy/article/details/54603055)

[关于调用约定 cdecl、stdcall 和 fastcall 的区别 | 拾遗记](https://blog.imkasen.com/calling-conventions/)

[stdcall、cdecl、fastcall、thiscall 、naked call的汇编详解 - findumars - 博客园](https://www.cnblogs.com/findumars/p/5143948.html)

### 格式化串攻击

由于调用约定的存在，不定参数的函数是不知道参数的个数与大小的。这在 scanf 和 printf 上会产生漏洞。

比如，用户在 scanf 输入字符串时，故意包含了格式化字符

```cpp
char str[100];
scanf("%s", str);  // 输入 %x\n%x\n%x\n
printf(str);
```

printf 只知道你传入了 str 格式串。一旦扫描到了 %x，就将对应偏移量的参数给输出出来，但是调用方并没有传对应的参数，这会导致 printf 输出对应地址上的内容。

所以，当 printf 传入参数不足格式化串中字符的个数时，就会导致这个漏洞。
- 泄露地址
- 栈上地址任意写入（通过输入 %n）

---

### 函数重载

原则：
- 名同，参数不同（个数、类型、顺序）
- 返回值类型不作为区别重载函数的依据

特殊情况：传入类型没有重载，但是可以隐式转换到多个重载的，编译无法通过

```cpp
void foo(char a){}
void foo(double a){}

foo(1);  // Call to 'foo' is ambiguous
```

### 默认参数

函数传入的参数可以设置默认参数，在不传默认参数时使用默认参数指定的值。就像 Python 一样。

注意点：
- 默认参数的声明，要在函数原型中给出。如果没有函数原型，就在函数定义中给出。
- 默认参数从右到左声明，不能间断
```cpp
void foo(double, char, int=2);      // okay
void foo(double=1.0, char, int);    // wrong!!!
void foo(double=1.0, char, int=2);  // wrong!!!
```
- 默认函数和函数重载不能出现 ambiguous
```cpp
void f(int);
void f(int, int=2);

f(1); // ambiguous!!!
```

### 内联函数

inline 关键字

目的
- 提高可读性
- 提高效率，因为减少了函数调用的过程

限制
- 不能递归
- 不能作为函数指针

适用：频率高、简单、小段的代码

其他：inline 声明仅仅是请求，编译器可以拒绝

缺点
- 增大目标代码
- 换页抖动
- 降低指令 cache 的命中率

---

### Namespace

命名空间。限制全局标识符的作用域，方便区分同名函数。

声明形式：
- declaration
```cpp
using std::cout;
using std::cin;
```
- directive
不建议同一文件多次用 directive
```cpp
using namespace std;
```
- 默认匿名命名空间
```cpp
using ::variable;
```


- 别名
```cpp
namespace std_alias=std;
```
- 全局
只要在某个文件里声明了 namespace，其他文件可以直接访问
- 开放
namespace 的内容完全公开
- 可嵌套
```cpp
namespace A{
	int a;
	namespace B{
		int b;
	}
}
A::a;
A::B::b;
```
- 重载

### 编译预处理

PPT 上说：与作用域、类型、接口等概念格格不入。

能理解，但理解得不多🫠。

#### include

作用是复制其中的文件内容到此处。

- 包含头文件
	- 防止重定义：用 `#ifdef` 等
- 替换操作
```cpp
printf("#")
```
- `""` 与 `<>`
使用 `<>` 时，预处理器会按如下顺序寻找文件：
1. 到编译选项 -l 指定的目录
2. 到环境变量 INCLUDE 指定的目录
使用 `""` 时：
1. Current Workspace Directory
2. 编译选项 -l 目录
3. 环境变量 INCLUDE 目录

#### define

替换文本宏。`define` 定义替换，`undef` 取消定义

- 空 define
多用于防止重复定义
- 预定义宏
比如 c++ 标准版本的宏
- 功能特性测试宏
比如开优化的宏
- 仿函数宏
	- 含参数的宏，比如之前打 oi 常用的省时间宏
```cpp
#define f(q, w, e) for(int q = w; q < e; q++)

f(i, 0, n) {...}
```
- 还能实现泛型
```cpp
#define add(x, y) (x + y)
```
加括号是必要的，为了保证加法不受外部运算符优先级的影响。
- 对于函数体的替换
用 do while(0) 保证函数体的独立性与完整性
```cpp
#define incAndPrint(a) do{\
	a++;\
	a.print();\
} while(0);
```
如果没有 do while(0)，放在无花括号的 if 里就出问题了。
```cpp
// with out do while(0)
if(a) 
	incAndPrint(a);
// equals to
if(a)
	a++;
	a.print();  // outside of if
```

- `#` 运算符
运算符后跟参数名，将这个参数转换成字符串。
```cpp
#define foo(x) #x

foo(12345);  // "12345"
foo(n);      // "n"
```
- `##` 运算符
运算符后跟参数名，将这个参数的名字作为一个 token 拼接到代码中。
```cpp
#define def_stack(x) typedef stack_##x{}

def_stack(int); // typedef stack_int{}
```

#### pragma

结构体对齐。没研究过，先跳过。

### 泛型

前面说过，可以用宏实现泛型：

```cpp
#define CREATE_STACK(T) \
typedef struct stack_##T { \
	T* array; \
	int top; \
} stack_##T\
\
void __init(stack_##T *stack, int capacity) {...}\
void __push(stack_##T *stack, T data) {...}\
...
```

但是太麻烦了
- 代码可读性查
- 难调试
- 需要显式写出类型参数
- 手动实例化

使用模板实现：

```cpp
template<typename T>
struct Stack{
	T* array;
	int capacity;
	int top;
};
...
```

好写很多。

#### concept

约束类模板和函数模板的模板类型和非类型参数的命名要求。

比如，限制函数模板不能是指针：

```cpp
template<typename T>
concept DataAvailable = !std::is_pointer<T>::value;

template <DataAvailable T>
void function(T t) {...}
```

其他写法：

```cpp
template <typename T>
requires DataAvilable
void function(T t) {...}

template <typename T>
void function(T t) requires DataAvilable<T> {...}

void function(DataAvilable auto v) {...}
```

可以用 `&&` 组合多个约束：

```cpp
```cpp
template <typename T>
concept signed_integral = integral<T> && std::is_signed_v<T>;
```

#### SFINAE

Substitution Failure Is Not An Error

模板的匹配失败不是错误。在匹配类型失败后，编译器还需要尝试其他的可能性

#### 特化

[C++ 模板 全特化与偏特化 - 知乎](https://zhuanlan.zhihu.com/p/346400616)

#### 参考

[C++20: Concept详解以及个人理解 - 知乎](https://zhuanlan.zhihu.com/p/266086040)

[C++模板进阶指南：SFINAE - 知乎](https://zhuanlan.zhihu.com/p/21314708)

---

### 元编程 Meta programming / constexpr

在编译期就计算出运行时需要的东西。

---

## 面向对象部分

### 面向对象概念

### 构造函数

#### 构造函数

构造函数有三种：
- 无参构造函数
- 有参构造函数
- 拷贝构造函数

#### 默认提供

编译器会提供 **默认无参构造函数** 和 **拷贝构造函数**。

```cpp
Empty();
Empty(const Empty&);
```

默认的无参构造函数是空函数，什么都不干。

拷贝构造函数，对所有成员进行浅拷贝。

### 成员初始化表

### 析构函数

编译器默认提供空的析构函数。

### 拷贝构造函数

### 移动构造函数

### 动态内存

### const 成员

### static 成员

### 友元

#### 友元类

#### 友元函数

友元函数不是类的成员函数！！！

### 编译器默认提供的函数

```cpp
class Empty {
	Empty();
	Empty(const Empty&);
	~Empty();
	Empty& operator=(const Empty&);
	Empty* operator &();
	const Empty* operator &() const;
}
```

---

### 继承概念

目的：基于目标代码的复用

思想：对事物进行分类。派生类是基类的具体化。把事物（概念）以层次结构表示出来，有利于描述和解决问题。

可用于：增量开发

--- 
### 多态

同一论域中一个元素可有多种解释
-  提高语言灵活性
-  程序设计语言
	- 一名多用——函数重载
	- 类属——模板
	- OO 程序设计——虚函数

### 运算符重载

- 动机：使用操作符的语义。为自定义数据类型提供类似内置类型的操作方式。
- 作用：提高可读性、可扩充性

{{< article link="/posts/C++拾遗/运算符重载/" >}}

### 对象切片

将派生类对象赋值给基类对象时，会发生对象切片。派生类对象会变成基类，只有其基类部分的成员被保留。

```cpp
class Base{
public:
	virtual void foo() { cout<<"Base"; }
};
class Derived: public Base{
public:  
	void foo() override { cout<<"Derived"; }
}

void function(Base base){
	base.foo();
}

Derived derived;

function(derived);  // Base
```

从结果上来说，把 derived 的数据赋给一个 base 对象的内存是不现实的，因为一般来说 sizeof Derived 比 sizeof Base 大。

避免发生切片的方法是使用引用或者指针。

```cpp
void function(Base& base){  // use Reference
	base.foo();
}

Derived derived;

function(derived);  // Derived
```

---

## 其他

### 异常处理

{{< article link="/posts/C++拾遗/异常处理/" >}}

### 右值引用

{{< article link="/posts/C++拾遗/移动语义与右值引用/" >}}