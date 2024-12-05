---
title: "OO-继承与派生"
date: 2024-11-21T12:36:40-08:00
categories: 
- "C++拾遗"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "考试必考。所以在网上找一些笔记，记下有价值的部分以供复习。 继承和派生这对名词基本是一个意思，只是各自用语习惯不一样： OO 是 Java 学的，所以我倾向于第一种表达。 C+神奇地提供了三种继承方式..."
---

## 前言

考试必考。所以在网上找一些笔记，记下有价值的部分以供复习。

继承和派生这对名词基本是一个意思，只是各自用语习惯不一样：

- 继承 —— 父类 —— 子类
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

继承方式决定了父类成员在子类中的访问权限，权限高于继承方式的会被限制。

比如 protected 继承会将父类中 public 的成员变成 protected。

>这个机制很难评。没见过用 public 以外的继承方式的。

### 修改父类成员的访问权限

使用 using 关键字可以修改父类成员在子类中的访问权限。权限提高和降低都没问题。

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

通过子类访问成员时，首先会在子类的作用域下寻找这个名字的成员。如果没找到，再到父类的作用域里找。

主要注意以下情景：

- 子类同名成员变量访问权限低于父类时
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

- 子类与父类不会发生函数重载
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

要手动调用父类构造函数实现初始化，必须在**成员初始化表**里调用。

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

否则（没有手动调用父类构造函数），子类会调用父类默认构造函数。

## 析构函数

子类无法显式调用父类的析构函数。

至于调用顺序：子类析构 之后 父类析构。与构造的顺序相反。

## 多继承

```cpp
class child: public parent1, private parent2, protected parent3 {};
```

### 成员访问

对于多继承中同名的成员，访问其成员时要加上类名和域解析符。

```cpp
child: parent1, parent2{};

child c;
c.parent1::a;
c.parent2::a;
```

### 虚继承

菱形继承中，子类可能同时继承了有相同父类的两个类，这样子类就会得到这个间接父类的两个副本。

通过虚继承，子类可以只保留一份间接父类的成员。

```cpp
class grand{};
class parent1: public grand{};
class parent2: public grand{};
class child: public parent1, virtual public parent2{};
```



## 参考

[C++ 基础系列——继承与派生 - 锦瑟，无端 - 博客园](https://www.cnblogs.com/cscshi/p/15350328.html)