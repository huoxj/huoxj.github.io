---
title: "OO 杂项"
date: 2024-11-20T10:34:25-08:00
categories: 
- "C++拾遗"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "```cpp class T {}; class T { T(); // 构造函数 T(const T&); // 拷贝构造 ~T(); // 析构函数 T& operator=(const T&);..."
---

## 空类默认提供的方法

```cpp
class T {};  
class T {  
	T();                            // 构造函数
	T(const T&);                    // 拷贝构造
	~T();                           // 析构函数
	T& operator=(const T&);         // 拷贝赋值
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

