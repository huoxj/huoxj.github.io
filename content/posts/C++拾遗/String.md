---
title: "String"
date: 2024-10-12
series: 
- "C++拾遗"
---

## 前言

在以前打竞赛的时候，基本都是用字符数组配合`string.h`里的函数去处理字符串。现在开始系统地接触C++，所以来记录一下`string`类的知识。

## 简介

`string` 是 C++ 风格的字符串类，也是一种STL容器(根据CppReference说法，string是唯一的`伪容器`)。相比于 C 风格的字符串 `char[]` ，string 支持自动内存管理，并且拥有很多方便的函数。

头文件：`string`

## 构造

### 一般构造

- `std::string(const char* s)`
- `std::string(const string& s)`

传入一个 C 字符串或者字符串类，变成 C++ string类。没什么好说的。

### 特殊构造

- 直接赋值`string str = "..."`

直接将 C 字符串赋值或者类型转换。

- `std::string(size_t n, char c)`

构造一个 n 个 c 的字符串。也就是将 c 重复 n 次。

## 属性

### 长度

优先 `string.length()`。

其他诸如 `string.size()` 和 `strlen(const &string)`。

### 是否空

和其他 STL 容器一样，判断 string 是否为空：

`string.empty()`

## 访问与遍历

### 索引

`str[i]` 直接得到对应字符。

### 迭代器

`string.begin()`, `string.end()` 得到头尾 iterator。

### Foreach

和 Java 一毛一样。你乐意用 `auto` 也行，反正元素就是char。

```cpp
for(char c : str)
```

## 常用

### 修改

#### 修改元素

修改元素用索引、迭代器这些都行，或者下面的 replace。

`str.insert(pos, str2)`

在索引为pos的地方插入str2。

就是说，插入的str2的第一个元素在修改后的str的pos处。

#### 修改子串

`str.replace(pos, len, str2)`

pos 索引(0开始)，长度为 len 的子串替换成 str2。

`str.erase(pos, len)`

删除 pos 开始长度为 len 的子串。

### 连接

`+`

直接+号运算就行。

`str.append(str2)`

这个不用说了。

### 查找

`str.find(str2)`

找 str 中 str2 的索引。找不到返回 -1。

`str.rfind(str2)`

反着找。

### 截取子串

`str.substr(pos, len)`

注意！！！和 Java 不一样！！！

截取 pos 开始，长度为 len 的子串。

### 转数字

- `stoi(str)`：int
- `stof(str)`：float
- `stod(str)`：double
还有很多，按需取用。

### 大小写

`toupper(str)` 和 `tolower(str)`。