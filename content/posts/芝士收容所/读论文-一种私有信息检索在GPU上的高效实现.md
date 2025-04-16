---
title: "读论文-一种私有信息检索在GPU上的高效实现"
date: 2025-01-22
series: 
- "芝士收容所"
---

原文 title：GPU-Based PIR for On-device ML Inference
## 背景

- On-device ML Inference 的 device 有计算与存储限制
    
- 在推荐等场景下，ML 推理需要查 Embedding table
    
    - Embedding table 相当大，不得不存在云端
        
    - 用户直接查表，可能会泄露用户隐私
        
- 使用 naive DPF-PIR 查表
    
    - 通信成本高，表有多长就得传多大的向量
        
    - 计算成本高，加密次数多
        

## 优化 DPF-PIR

### 建模

DPF-PIR 在单次查询中，主要有三个步骤：

1. (client) 生成(gen)密钥
    
2. (server) 评估(eval)密钥，得到秘密分享 secret share
    
3. (server) 计算秘密分享与 Embedding table 矩阵的积，返回给 client
    

其中，gen 步骤不是瓶颈，目前不需要优化。

多个查询之间是独立的，使用 Batch 优化。

### eval 的 GPU 优化

论文里选择了引用 [32] 描述的 DPF 算法。本质上是一个动态规划/dfs。

> 我觉得选择这个 DPF 算法的原因主要是考虑了通信开销。开销是随着 Embedding table 的大小对数增长的。
> 
> 只是文章里没细说，也没有列举其他的算法并进行对比。

递推公式只取和优化相关的部分，大致长这样：

$$  
P(d,j)=f(P(d-1,\lfloor j/2\rfloor))  
$$

其中 f 是一个论文中没有优化的函数。涉及 pseudorandom function，

递推项之间的数据依赖关系是一颗二叉树。

文中提到了优化这类问题，通常有 `branch-parallel` 和 `level-by-level` 两种方法。前者通信开销小，但是会存在重复计算的现象，导致计算开销变大；后者不会重复计算，但是通信开销大，因为每一层的结果要全部存到 buffer 里。

> branch-parallel 最终也是计算出一个长度为 L(table 长度) 的向量，和 level-by-level 一样，那为什么不存在通信瓶颈的问题呢？
> 
> 前者每一个线程可以独立得到叶子节点，即最终向量的一个分量，这个分量计算 table 中对应项的权重，所以不存在通信开销。而后者每一层的计算都要缓存，以便下一层计算，所以导致了严重的通信开销。

通过结合二者思想，文章里提出了 Memory-bounded tree traversal。整体是 dfs 遍历树，但是每一次遍历至多 K 个节点，这些节点内部使用 level-by-level 的思想来并行化。这样能控制单次查询的通信开销为 O(Klog(L)) ，而不是原始的线性复杂度。并且，这样不存在重复计算的情况，因为整体的 dfs 是单线程进行的。

> 用了 SIMD 的思想来优化树的遍历。[trav.pdf](https://ics.uci.edu/~dhirschb/pubs/trav.pdf) 这篇文章里有类似的优化

### table 和 secret share 优化

在经历前面的优化后，我们得到的 secret share 只能是一个长度不大于 K 的，真实 secret share 的子集了。如果要把真实 secret share 拼起来再让二者直接相乘的话，之前的优化就都白做了。

好在 DPF 的性质会让查询表项之外的项，在求和之后结果为 0，所以可以放心大胆地让这个子集求和。反正最终，将两个 server 返回的内容求和后，无用的项会抵消掉。

## Batch-PIR 和 ML

在真实场景中，client 的一次推理往往需要多次查询，并且这些查询有许多的性质。可以从这些性质里下手优化。

### 桶优化

原始的方式是多个查询间 batch，单个查询内 PIR。针对**多次查询**的性质，可以将 embedding table 分成多个桶，每一个桶内都是单独的 PIR。这样就将多次查询的 batch 并行优化到了表级。但是这样会影响结果正确率，当多个查询落到同一个桶里时。

### hot-table split

针对**查询偏好**的性质，一小部分的表项的查询占到了较大的比例。可以考虑对这些 hot 表项建立出一个新 hot-table，这个 hot-table 足够小，能塞进 client 端。

但是，用户查 hot-table 和原 table 的次数会泄露用户隐私。所以要通过将二者限制为同一个数字 Q 来保证隐私安全。这样会导致大于 Q 的查询被丢弃；小于 Q 时需要一些 dummy 占位查询。

> 这种硬分表，可以通过查表次数来得知用户的隐私。那上面提到的桶优化，可以说是软分表策略，有没有可能也有相同的风险呢？

### Co-location

某些表项可能经常**一起出现**。根据一起出现的概率，把这些表项绑在一起。利用时间局部性与空间局部性来优化这类查询