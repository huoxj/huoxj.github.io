---
date: '2024-11-21'
series:
- C++拾遗
title: OO-继承与派生
---

## 前言

考试必考。所以在网上找一些笔记，记下有价值的部分以供复习。

继承和派生这对名词基本是一个意思，只是各自用语习惯不一样：

- 继承 —— 基类 —— 派生类
- 派生 —— 基类 —— 派生类

OO 是 Java 学的，所以我倾向于第一种表达。

## 继承权限

### 继承方式

C++ 很神奇地提供了三种继承方式：public、private 和 protected。

```cpp
class parent{};

class child1: public parent{};
class child2: protected parent{};
class child3: private parent{};
```

继承方式决定了基类成员在派生类中的访问权限，权限高于继承方式的会被限制。

比如 protected 继承会将基类中 public 的成员变成 protected。

### 修改基类成员的访问权限

使用 using 关键字可以修改基类成员在派生类中的访问权限。权限提高和降低都没问题。

```cpp
class parent{
protected:
	int element;
};

class child{
public:
	using parent::element;    // protected -> public
private:
	using parent::element;    // protected -> private
}
```

## 成员覆盖

继承中的成员覆盖问题，本质是作用域的问题。

通过派生类访问成员时，首先会在派生类的作用域下寻找这个名字的成员。如果没找到，再到基类的作用域里找。

主要注意以下情景：

- 派生类同名成员变量访问权限低于基类时
```cpp
parent{
public:
	int a;
};
child{
private:
	int a;
}

child c;
c.a;         // cannot access a.
c.parent.a;  // ok
```

- 派生类与基类不会发生函数重载
```cpp
parent{
	void foo(int a);
};
child{
	void foo(double a);
}

child c;
int a;
c.foo(a);          // double
c.parent::foo(a);  // int
```

## 构造函数

构造函数无法继承。

派生类默认会调用基类默认构造函数。

要手动调用基类的构造函数实现初始化，必须在**成员初始化表**里调用。

```cpp
parent{
public:
	parent(){};
	parent(int a){...};
}

child{
public:
	child(int a, int b): parent(a), ...;
}
```

否则（没有手动调用基类构造函数），派生类会调用基类默认构造函数。

## 析构函数

派生类无法显式调用基类的析构函数。

至于调用顺序：派生类析构 之后 基类析构。与构造的顺序相反。

## 虚函数

基类的引用或指针可以引用或指向派生类对象。当调用这个指针或者引用的虚函数时，会调用派生类中定义的虚函数。

### 前期绑定

没用 virtual 修饰的函数，是前期绑定。

- 编译时绑定
- 根据对象的静态类型
- 效率高、灵活性差

### 动态绑定

用 virtual 修饰的函数，是动态绑定。

- 运行时刻
- 依据对象的实际类型（动态）
- 灵活性高、效率低
- 需要用 `virtual` 显式指出

### 限制

类的成员函数才可以是虚函数
- 静态成员函数不能是虚函数
- 内联成员函数不能是虚函数
- 构造函数不能是虚函数
- 析构函数可以（往往）是虚函数

### 实现

为每一个类的对象添加一个隐藏成员，这个成员是一个指向**虚函数表**的指针。

而虚函数表存放了这个类所有的虚函数。如果虚函数在这个类中重定义了，表里就放重定义的函数，否则放基类的函数。

### 关键字

#### final
这个函数无法被重写。

```cpp
class Base{
	virtual void foo(int) final;
};
class Derived: Base{
	void foo(int);  // wrong! foo is final
};
```

#### override

显式声明这个函数是基类某个虚函数的重写，参数、返回值等属性必须完全一致（访问权限可以不一样）。

是可选的关键字，也就是说，override 删掉也能跑。
但建议加上，以防止错误。

```cpp
struct B {
virtual void f1(int) const ;
virtual void f2 ();
void f3 () ;
};
struct D: B {
void f1(int) const override ;     // correct
void f2(int) override ;           // wrong: parameter dosent match
void f3 () override ;             // wrong: f3 is not virtual
void f4 () override ;             // wrong: no func named f4
```

### 纯虚函数

接口。基类函数只提供声明，不提供实现。派生类必须重写这个函数接口。

在虚函数原型后增加 `=0` 来声明。

```cpp
class Base{
	virtual void foo() = 0;
};
class Derived: Base{
	void foo() override{
		// implementation
	}
}
```

### 缺省参数值

- 不要重新定义继承而来的缺省参数值

缺省参数值是在编译期确定的（静态绑定），不会进行动态绑定。所以派生类在没指定缺省参数时，会根据当前指针的类型来静态确定缺省参数值。

```cpp
class Base {
	virtual void foo(int x = 0) = 0;
};
class Derived: Base{
	void foo(int x = 1) { cout<<x; }
};

Base* ptr = new Derived();
ptr->foo();      // 0
ptr->foo(100);   // 100
Derived* ptr_d = new Derived();
ptr_d->foo();    // 1
```

### 虚函数的小总结

- 纯虚函数
	- 只有函数接口会被继承
	- 派生类**必须**继承函数接口
	- （必须）提供实现代码
- 一般虚函数
	- 函数的接口及缺省实现代码都会被继承
	- 派生类**必须**继承函数接口
	- 可以继承缺省实现代码
- 非虚函数
	- 函数的接口和其实现代码都会被继承
	- 必须同时继承接口和实现代码

### 使用规范

- 确定 public 继承，是真正意义的“is_a”关系
- 不要定义与继承而来的非虚成员函数同名的成员函数
- 明智地运用 private 继承
	- 需要使用基类中的 protected 成员，或重载 virtual function
	- 不希望一个基类被访问派生类的一方使用

## 多继承

```cpp
class child: public parent1, private parent2, protected parent3 {};
```

### 声明顺序

基类的声明次序决定：
- 对基类构造函数/析构函数的调用次序
- 对基类数据成员的存储安排

### 成员访问

对于多继承中同名的成员，访问其成员时要加上类名和域解析符。

```cpp
child: parent1, parent2{};

child c;
c.parent1::a;
c.parent2::a;
```

### 虚继承

菱形继承中，派生类可能同时继承了有相同基类的两个类，这样派生类就会得到这个间接基类的两个副本。

通过虚继承，派生类可以只保留一份间接基类的成员。

- 虚基类的构造函数由最新派生出的类的构造函数调用
- 虚基类的构造函数优先非虚基类的构造函数执行
- 调用非虚基类构造函数时，重复的间接基类就不会被构造了

```cpp
class grand{};
class parent1: public grand{};
class parent2: public grand{};
class child: public parent1, virtual public parent2{};
```

内存模型：

## 参考

[C++ 基础系列——继承与派生 - 锦瑟，无端 - 博客园](https://www.cnblogs.com/cscshi/p/15350328.html)

[图解C++菱形继承、虚继承对象的内存分布_菱形继承 虚函数内存结构图-CSDN博客](https://blog.csdn.net/AgoniAngel/article/details/105893798)