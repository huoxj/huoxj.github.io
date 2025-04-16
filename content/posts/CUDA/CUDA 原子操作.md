---
title: "CUDA 原子操作"
date: 2025-03-05
series: 
- "CUDA"
---

CUDA 原子操作和 C++ 的原子操作概念基本是一样的。

给出定义：An atomic function performs a read-modify-write atomic operation on one 32-bit, 64-bit, or 128-bit word residing in global or shared memory.

> 128 bit 的支持似乎是新版本加的，网上部分资料还仅限于 32 和 64 bit

本文记录一些特殊点和坑点。

## 使用

CUDA 原子操作被包装成了函数。

比如原子加法 `atomicAdd(int* addr, int val)`。向 addr 地址加上 val。

对于操作的对象，如定义所言，只支持 32, 64, 128 bit 的类型。

## 返回旧值

原子函数的返回值是参数地址被修改前的值。

利用这一点，可以实现 filter。

```cpp
__global__ void myCudaFilter(int *dest, const int *arr, int len){
	int idx = ...; // linear index of current thread
	__shared__ int sum = 0;
	// filter element that greater than 114
	if (arr[idx] > 114) {
		// loc is the value before increment
		int loc = atomicAdd(&sum, 1);  
		dest[loc] = arr[idx];
	}
}
```

## 实现任意原子操作

### atomicCAS

先介绍 atomicCAS。全称：atomic Compare And Swap

```cpp
int atomicCAS(int *addr, int cmp, int val)
```

查看 addr 里的值是否等于 cmp，如果等于，将 addr 的值置为 val。

### 任意原子操作

```cpp
__device__ __inline__ int myAtomicAdd(int *dest, int src) {
	int old = *dst, expect;
	do {
		expect = old;
		old = atomicCAS(dest, expect, expect + src);
	} while (expect != old);
	return old;
}
```

## 访存性能优化

如果很多线程在一小段时间内对同一使用原子操作，这些线程会串行执行而损失并行度。

可以使用线程局部变量作为缓存，最后再将局部变量使用原子操作同步到目标地址。

