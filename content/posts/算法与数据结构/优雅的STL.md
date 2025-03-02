---
title: "优雅的STL"
date: 2025-02-28T21:13:36-08:00
categories: 
- "算法与数据结构"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "同属系列。致力于整理优雅、清晰的，以及一些。 本篇记录了使用 C+TL 刷题时的一些心得。 蓦然回首，那迭代器 / 指针已经！等待你的是 `heap-use-after-free`。 ```cpp a..."
---

同属*优雅的*系列。致力于整理优雅、清晰的**处理手法**，以及一些**注意事项**。

本篇记录了使用 C++ STL 刷题时的一些心得。

## 容器导致的迭代器 / 指针失效

蓦然回首，那迭代器 / 指针已经**挂了**！等待你的是 `heap-use-after-free`。

```cpp
auto arr = vector<int>(1, 1);
auto it = arr.begin()
arr.push_back(1);
*it;  // BOOM! Heap use after free
```

容器扩容时偷偷搬家了，房东就不要拿原来的水电气单子去收钱了，会变得不幸（

用索引来记录是一个不太完美的解决方案。

或者，提前预判搬家后迭代器该指向什么位置。仍然不完美，只能处理部分情况。

完美的解决方案？404 not found。