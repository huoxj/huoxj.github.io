---
title: "Asst3"
date: 2025-02-17
series: 
- "CS149-并行计算"
---

## Part 1: SAXPY

使用 CUDA 实现 SAXPY。

实现很简单，跟着实验文档和 CUDA 文档做下来就行。

Question 2：为什么观测到的带宽约为 5.3 GB/s，远不及 PCIe 3.0 的理论带宽上限？
- 主板芯片组性能限制
- **Pinned memory** 机制
	- GPU 通过 DMA 访存
	- Pinned memory 是物理内存上一块固定的区域，不会被换出，能通过 DMA 加速通信
	- CPU 内存(host data)上的数据是虚拟内存上的可分页数据，可能存在于物理内存上或者硬盘上（页被换出物理内存了）
	- GPU 直接通过物理地址访存，host data 需要先拷贝到临时的 pinned memory 区上，再拷贝到 GPU (device memory)
	- 用 `cudaHostAlloc` 或 `cudaMallocHost` 分配 pinned memory
	- [CUDA:页锁定内存(pinned memory)和按页分配内存(pageable memory ) - 牛犁heart - 博客园](https://www.cnblogs.com/whiteBear/p/17842246.html)

## Part 2

### Parallel Prefix-Sum

使用 CUDA 实现一个并行的前缀和算法。

[并行算法科普向 系列之二：前缀和，fork-join 和矩阵乘法 - 知乎](https://zhuanlan.zhihu.com/p/91089093)

这篇文章里讲得很清晰，比课件里直接把算法抛出来好多了。

### Find Repeats

找出数组中 $arr_i = arr_{i+1}$ 的元素，并返回这些元素的下标组成的数组。

并行地找这样的元素相当容易，但是找到之后如何统计下标就比较困难了。

一般的思路：结果数组配上互斥锁。不过这样并行就变成串行了，而且也没法保证顺序。

参考课件中的 gather 算子的思路，只要能计算出 repeat 元素的下标在结果数组中的位置，并行化就不成问题。

一种实现是：
- 使用 map 算子，将满足 repeat 元素映射到 1，不满足的映射到 0
- 对上一步的 1-0 数组做 exclusive-scan
- scan 结果中，所有 $scan_i \neq scan_{i+1}$ 的元素都对应了原数组中 repeat 的元素（可自行验证），并且 scan 的值就是应在结果数组中的下标
- scan 最后一个值就是总 repeat 元素的数量
- gather 一下所有 $scan_i \neq scan_{i+1}$  元素得到结果。具体来说，$output[scan[i]]:=i$

```cpp
__global__  
void repeats_mask(const int *arr, int *mask, const int N) {  
    size_t idx = blockIdx.x * blockDim.x + threadIdx.x;  
    if (idx >= N) return;  
    if (idx == N - 1) mask[idx] = 0;  
    mask[idx] = (arr[idx] == arr[idx + 1]);  
}  
  
__global__  
void repeats_gather(const int *mask_sum, int *output, const int N) {  
    size_t idx = blockIdx.x * blockDim.x + threadIdx.x;  
    if (idx >= N - 1) return;  
    if (mask_sum[idx] != mask_sum[idx + 1]) output[mask_sum[idx]] = idx;  
}  
  
// find_repeats --  
int find_repeats(int* device_input, int length, int* device_output) {  
  
    int *mask;  
    int roundN = nextPow2(length);  
    cudaMalloc((int **) &mask, roundN * sizeof(int));  
  
    int num_block = (length + THREADS_PER_BLOCK - 1) / THREADS_PER_BLOCK;  
  
    repeats_mask<<<num_block, THREADS_PER_BLOCK>>>(device_input, mask, length);  
  
    exclusive_scan(nullptr, roundN, mask);  
    cudaDeviceSynchronize();  
  
    int output_length;  
    cudaMemcpy(&output_length, mask + length - 1, sizeof(int), cudaMemcpyDeviceToHost);  
  
    repeats_gather<<<num_block, THREADS_PER_BLOCK>>>(mask, device_output, length);  
  
    return output_length;  
}
```

Benchmark 结果：
```
-------------------------
Find_repeats Score Table:
-------------------------
--------------------------------------------------------------------
| Element Count  | Ref Time      | Student Time   | Score          |
--------------------------------------------------------------------
| 1000000        | 2.165         | 1.638          | 1.25           |
| 10000000       | 13.549        | 9.119          | 1.25           |
| 20000000       | 25.574        | 17.033         | 1.25           |
| 40000000       | 50.006        | 32.268         | 1.25           |
--------------------------------------------------------------------
|                                | Total score:   | 5.0/5.0        |
--------------------------------------------------------------------

```

