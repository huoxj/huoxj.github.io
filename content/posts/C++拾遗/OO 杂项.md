---
title: "OO 杂项"
date: 2024-11-20T10:34:25-08:00
categories: 
- "C++拾遗"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "```cpp class T {}; class T { T(); // 构造函数 T(const T&); // 拷贝构造 T(const T&&); // 拷贝构造 rval ~T(); // 析..."
---

## 类的内存模型

[What does C++ Object Layout Look Like? | Nimrod's Coding Lab](https://nimrod.blog/posts/what-does-cpp-object-layout-look-like/)

## 空类默认提供的方法

```cpp
class T {};  
class T {  
	T();                            // 构造函数
	T(const T&);                    // 拷贝构造
	T(const T&&);                   // 拷贝构造 rval
	~T();                           // 析构函数
	T& operator=(const T&);         // 拷贝赋值
	T& operator=(const T&&);        // 拷贝赋值 rval
	T *operator &();                // 取地址
	const T* operator &() const;    // 取地址 const 重载版
};
```

最后两个函数比较怪。一个是返回非 const 的指针，另一个返回 const 的。在调用时，编译器如何决定调用哪个呢？

注意函数后的 const 修饰符。这两个函数实际应该长这样：

```cpp
T *operator &(T *this);
const T* operator &(const T *this);
```

是 this 指针的功劳。调用方是哪种就执行哪个。

## const 与 static 成员

### static const 成员变量

对于成员变量，变量会在**编译期**确定，并被放到程序静态区。运行时，这个成员和类与对象在内存上没有关系，但是要通过类名来访问。

static const 只能修饰成员变量。在成员函数前面加 static const 的意义是对函数修饰 static，对**函数返回值**修饰 const。

### static 成员

static 成员属于类，和对象没关系。

#### 初始化

初始化需要在**类的外部**。因为在类的内部初始化实际上是对象的行为，而 static 变量是与对象无关的。

```cpp
class Test{
public:
	static a = 1;  // not allowed
	static constexpr int a = 1;  // C++ 17
};

int Test::a = 1;  // okay
```

#### 使用与修改

对于 private 的 static 变量，除了初始化，在类外无法访问。

对 public 的变量，加上类限定符就可以访问了。

### const 成员

对于 const 成员变量没什么好说的，const 起到限定无法修改的作用。

对 const 成员函数就有意思了。不过严格来说，就不该叫“const 成员函数”。

有些成员函数不会对对象的属性进行修改，对这类函数可以在其声明后加上 const 限制。

```cpp
class Test{
public:
	void get() const;
}
```

其原理很简单，就是将成员函数的 this 指针加上 const 限定：

```cpp
void get();        <=>  void get(Test *this); 
void get() const;  <=>  void get(const Test *this);
```

## 友元

做朋友。破坏类原有的封装性，使外部能够在尽可能少的改动下，从一个类的外部访问这个类的私有成员。

### 友元类

A 指定 类B 是朋友。B 中就可以随意访问 A 的成员了。

注意，友元类要处理声明顺序的问题，看代码第一行。

```cpp
class B; // A 声明友元需要 B 的定义，B 也需要访问 A 的成员。所以先把 B 声明在这里，方便 A 使用。

class A {
	friend class B;
private:
	int x;
	static int y;
	void foo();
};

class B {
	A a;
	void f() {
		a.x;
		A::y;
		foo();
	}
};
```

### 友元函数

> 无论哪种友元函数，都不是这个类的成员函数！！！

首先是友元成员函数。类 A 和 类 B 的一个成员函数 foo 做朋友。函数内部能随意访问 A 的私有成员，但函数之外都不行。

注意几点：
- 依赖顺序，需要前向声明
- foo 的参数只能是 A 的指针或者引用，因为此时 A 只是有了个名字，不知道占空间大小
- friend 定义友元函数语句中，函数前要加类限定符

```cpp
class A;
class B{
	void foo(A &a) {
		a.x;  // okay
	}
	void f(A &a) {
		a.x;  // invalid
	}
}
class A{
	friend void B::foo(A &a);
	int x;
};

```

然后是友元全局函数。类 A 和外部函数 foo 做朋友。foo 能访问 A 所有东西。
- 依赖顺序随意。当然，如果函数参数有类的话还是要先声明，如代码中的例子
- 函数参数包含类对象时，可以不为指针和引用
- 多用于实现**类运算符重载**

```cpp
class A{
	friend void foo(A a);
};
void foo(A a) {

}
```