---
title: "STL"
date: 2024-10-12T10:43:24-08:00
categories: 
- "C++拾遗"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "STL 有五大组件： 序列容器实现能按顺序访问的数据结构。 动态数组。 双端队列。 双向链表。 C 风格的固定大小的数组。 单向链表。性能比 `list` 略好，基本和 C 中的链表无异。 可动态调整..."
---

## 前言

STL 有五大组件：

- 容器（Container），数据结构；
- 迭代器（Iterator），提供了一种顺序访问容器中对象的方法。
- 算法（Algorithm），是用来操作容器中的数据的模板函数。
- 仿函数（Functor），就是使一个类的使用看上去象一个函数，就是类中实现一个operator()。
- 适配器（Adaptor），对原有的容器进行包装，给换一套接口。

## 容器

### 序列容器

序列容器实现能按顺序访问的数据结构。

#### vector

动态数组。

#### deque

双端队列。

#### list

双向链表。

#### array
> C++11

C 风格的固定大小的数组。

#### forward_list
> C++11

单向链表。性能比 `list` 略好，基本和 C 中的链表无异。

#### inplace_vector
>C++26

可动态调整大小的固定容量原位连续数组。

太新了，先不管。

### 关联容器

关联容器实现能快速查找（$O(log n)$ 复杂度）的有序数据结构。

#### set



#### multiset

#### map

#### multimap

## 参考

[C++ STL 教程 | 菜鸟教程](https://www.runoob.com/cplusplus/cpp-stl-tutorial.html)

CppReference