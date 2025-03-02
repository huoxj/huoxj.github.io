---
title: "STL-容器"
date: 2024-10-12T10:43:24-08:00
categories: 
- "C++拾遗"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "STL 有五大组件： 序列容器实现能按顺序访问的数据结构。 有如下容器： 省略返回类型以及一些无关紧要的参数类型。 没有头部加入和删除元素。 insert，erase 和 clear 复杂度 $O(n..."
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

有如下容器：
- `vector`：动态数组
- `deque`：双端队列
- `list`：双向链表
- `array`：C 风格的固定大小的数组 *C++11*
- `forward_list`：单向链表。性能比 list 略好，基本和 C 中的链表无异 C++11
- `inpalce_vector`：可动态调整大小的固定容量原位连续数组 C++26

#### 一般操作
##### 构造

省略返回类型以及一些无关紧要的参数类型。

- `()`：默认无参数构造
- `(&& other)`：拷贝构造
- `(it1, it2)`：从另一个 vector 的迭代器构造，得到子 vector
- `(count, T element)`：含 count 个 element

##### 赋值

- 赋值号
- `assign` 函数。和构造函数参数是一致的。
- `swap` 交互两个容器。

##### 元素访问

- `operator[]`。数组的访问方式。可以访问任意下标的元素。
- `at(index)`。访问下标 index 处的元素。 

##### 元素操作

- `push_front(T e)` `push_back(T e)`：头尾部加入元素
- `pop_back()` `pop_front()`：尾部删除
- `insert(it, T e) / insert(it, count, T e)`：it 处插入一个或多个 e
- `erase(it) / erase(it1, it2)`：it 处删除 / \[it, it2\) 处删除
- `clear()`：清空

#### 说明

##### vector
没有头部加入和删除元素。
insert，erase 和 clear 复杂度 $O(n)$

##### deque
列出的函数都有。

##### list
- `remove(T e)`：移除和 e 相等的元素

### 关联容器

关联容器实现能快速查找（$O(log n)$ 复杂度）的**有序**数据结构。

有如下容器：
- `set`：集合
- `multiset`：多重集合
- `map`：映射
- `multimap`：多重映射

#### 相关操作

set, map 大部分插入、删除与搜索都是 $O(logn)$ 的。

##### 构造

- `()`：默认无参构造
- `<key, comp>()`：自定义仿函数 comp。默认为 `std::less`

>对 set/multiset
- `(&&)`：移动构造
- `(it1, it2)`：从序列容器构造
- `(int first, int last)`：构造 first 到 last 的数字集合。左闭右开

##### 赋值

只有赋值号和 `swap`。

##### 元素访问

>仅限 map

- `operator[]` 和 `at()`

##### 元素查找

- `find(T& key)`：找到为 key 的第一个元素，返回**迭代器**。
- `count(T& key)`：返回 key 的数目。对于 set 和 map 只会返回 0 和 1。
- `equal_range(T& key)`：找到一个 range，之中每一个元素都等于 key。返回 range 首尾迭代器组成的 pair。
- `lower_bound(T& key)`：找到第一个不小于 key 的元素的迭代器。（也就是**包含**自身） 
- `upper_bound(T& key)`：找到第一个大于 key 的迭代器。（也就是**不包含**自身）

##### 元素操作

- `insert` `emplace`：插入元素
- `clear`：清空
- `erase(T& key)` `erase(it)`：通过元素或者迭代器删除元素

## 参考

[C++ STL 教程 | 菜鸟教程](https://www.runoob.com/cplusplus/cpp-stl-tutorial.html)

CppReference