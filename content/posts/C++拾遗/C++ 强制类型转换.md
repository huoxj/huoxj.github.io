---
date: '2024-12-24'
series:
- C++拾遗
title: C++ 强制类型转换
---

## Why
C 中的强制类型转换是通过在表达式前面加类型名实现的。

C++ 中，强制类型转换被细分成了四个运算符：
- static_cast
- reinterpret_cast
- const_cast
- dynamic_cast

原因如下：
- 不同场景下的强制转换，风险是不同的，需要细分来体现
比如，int 转成 double 是风险不大的；但是将 const 指针转换成非 const 指针是十分危险的。
- 在继承场景下，将基类转换成派生类需要检查是否成功
C 中的强制转换是无法知道基类指针转换成派生类是否成功的。但是 dynamic_cast 可以以返回 nullptr 的方式告诉你转换是否成功。
- 程序中强制类型转换难以定位
新方法中，你只需要找 cast 单词出现的地方，就能知道所有进行了显示强制类型转换的地方。
- 类型转换出错时，cast 会报错，方便定位错误

## static_cast

static_cast 负责风险低、自然的转换。允许可以隐式转换的类型间的转换。

具体来说：
- 允许**基本类型**之间的转换。比如 int、float、double 之间的转换
- 允许对象的重载了强制类型转换运算符的转换
- 其他场景尽量不要用 static_cast

## dynamic_cast

dynamic_cast 负责基类到派生类的引用和指针的转换。

- 目标类型是源类型的派生。即父转子。
- 被转换的类需要有虚函数，否则会报错：
`bad_dynamic_cast_not_polymorphic: 'xxx' is not polymorphic`

## reinterpret_cast

reinterpret_cast 负责指针的不安全的转换。转换时，逐 bit 复制数据，也印证了 reinterpret 的含义，就是对相同的 01 bit 流进行不同的解释。

- 负责不同类型的指针、引用之间的转换
- 指针到指针数值（里面存的地址）的转换。

## const_cast

const_cast 负责去除 const 的转换。

- 将 const 的只读变量转换成普通的可读可写的变量


## 参考

[C++强制类型转换运算符（static_cast、reinterpret_cast、const_cast和dynamic_cast） - C语言中文网](https://c.biancheng.net/view/410.html)

[C++ 类型转换详解](https://blog.csdn.net/test1280/article/details/72763451)