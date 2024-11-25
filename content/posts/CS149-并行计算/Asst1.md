---
title: "Asst1"
date: 2024-11-08T09:40:53-08:00
categories: 
- "CS149-并行计算"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411080945106.png
summary: "使用 `std::thread` 来绘制分形图像。 默认策略是将图像分成高度相同的几块，每一块分给一个线程画。 这样其实做不到负载均衡，因为每一块画的工作量是不同的。 特别是 view1，很明显发现画..."
---

## Prog1-mandelbrot_threads
使用 `std::thread` 来绘制分形图像。

### 默认策略

默认策略是将图像分成高度相同的几块，每一块分给一个线程画。

这样其实做不到负载均衡，因为每一块画的工作量是不同的。

特别是 view1，很明显发现画中间部分的线程工作时间远高于两边的线程。
![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411080945106.png)

代码实现很朴实，均分然后画就好了。

在 R7 5800x上（原生 8 核心 16 线程），画 view1：

- 8 线程倍率：4.04x
- 16线程倍率：7.55x

都被 view1 中的画中间部分的线程拖后腿了。

```cpp
int startRow = args->height / args->numThreads * args->threadId,
    numRows = args->height / args->numThreads;
mandelbrotSerial(
	args->x0, args->y0,
	args->x1, args->y1,
	args->width, args->height,
	startRow, numRows,
	args->maxIterations,
	args->output
	);
```

### 均衡策略

让线程按行交错绘制图像。

其实这个策略是在模拟随机分配任务。思想是将任务分成尽可能小的部分，然后让大家随机挑任务做。

可以证明对长度为 $n$ 的数列 $a$（工作量），在其中随机取 $n/s$ 项（$s$就是线程数），其和的期望是相同的。

代码实现是按行交错绘制，实现起来比较简单。

对于 view1：

- 8 线程：7.53x
- 16 线程： 11.48x

16线程反而缩水得很严重，主要原因应该是 8 线程时每个线程独享一个核心，基本做到了并行。但是 16 线程时就是两个线程争抢一个核心了，需要并发。

```cpp
int totHeight = args->height;
for(int line = args->threadId; line < totHeight; line += args->numThreads) {
        mandelbrotSerial(
            args->x0, args->y0, 
            args->x1, args->y1, 
            args->width, args->height, 
            line, 1, 
            args->maxIterations, 
            args->output
            );
    }
```

### 核心与线程

问 ChatGPT 得知，CPU 中核心和线程大概是这样的：

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411081100650.png)

但也存在少量架构中线程有独立的 L1 cache。

 从高层次往低层次：

系统级线程 - 逻辑处理器 - 物理内核

## Prog3-mandelbrot_ispc

### ISPC 在 VSCode 中的配置

这部分作业我挪到 WSL 上面做了，因为我的 Linux 机子处理器没有 AVX2 指令集。ISPC 只能在有这个指令集的机器上做。

首先得保证有 ISPC 编译环境。

然后，在 VSCode 上写 ISPC，要安装 ISPC 扩展。

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411212101487.png)


这个扩展还要求安装 C# 环境。

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411212102084.png)


[Download - Stable | Mono](https://www.mono-project.com/download/stable/) 按网页里的引导来配就行。Ubuntu 上一通 apt 就装好哩。

最后，在 ISPC 插件设置中配置平台与 ISPC 编译器路径即可。我只改了蓝色的这两项，改完重启下 VSCode，插件就可以正常工作了。

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411212105190.png)


### ISPC 基础

正式介绍 —— ISPC。

实验文档说，ISPC 程序的每一个实例都是在 SIMD 单元里并行执行的。每一个实例都有两个隐含的变量：`programIndex` 和 `programCount`。前者是此实例的序号，后者是实例数。

对于一个求向量和的代码：

```cpp
const int TOTAL_VALUES = 1024;
float a[TOTAL_VALUES];
float b[TOTAL_VALUES];
float c[TOTAL_VALUES]
 
for(int i = 0; i < TOTAL_VALUES; i++)
	c[i] = a[i] + b[i];
```

对应 ISPC 代码：

```cpp
export sum1(uniform int N, uniform float* a, uniform float* b, uniform float* c) {
	for(int i = 0; i < N; i+= programCount) {
		c[programIndex + i] = a[programIndex + i] + b[programIndex + i];
	}
}
```

>export 使这个函数能直接被 C/C++ 代码调用
>uniform 标识的参数表示对所有实例都相同

可以看到，每个实例负责相邻的 N 个分量的求和。

但是，ISPC 并不希望你详细到指定每个实例干什么工作。不该以“每个实例负责相邻的 N 个分量的和” 设计程序，而是以“求和的工作可以分解成每个分量求和的工作” 来设计程序。如下：

```cpp
export sum2(uniform int N, uniform float* a, uniform float* b, uniform float* c) {
	foreach (i = 0 ... N) {
		c[i] = a[i] + b[i];
	}
}
```

通过 ISPC 的 foreach 来实现。

大任务分解成每个分量求和的小任务时，这些小任务之间是互不影响的，所以执行的顺序也没有要求。我们在 sum1 中将相邻的分量放在一个实例中求和因此没有任何必要。所以通过 foreach 让 ISPC 自己决定小任务执行的顺序就行，程序会变得简洁一些。

## Prog4- sqrt

### Task1

运行一下 sqrt 的代码。

```bash
[sqrt serial]:          [613.685] ms
[sqrt ispc]:            [127.356] ms
[sqrt task ispc]:       [9.783] ms
                                (4.82x speedup from ISPC)
                                (62.73x speedup from task ISPC)
```

和 prog3 类似，ISPC 理论 8 倍加速和 task 的 threads * 8 倍加速都没达到。应该也是负载均衡的问题。我们在 task2 里继续探索。

### Task2

任务2要求通过修改输入的 value 数组，来实现最大的和最小的加速比。

并行任务中，最大的两个开销是 (1) 运算开销 和 (2) 调度开销。前者占比越大加速比越大。后者为 0 时基本可以达到理论加速比。

所以 value 数组设置为 1 和 2.999，可以实现运算开销占比最大化和最小化。

设置为 1：
```bash
[sqrt serial]:          [10.802] ms
[sqrt ispc]:            [5.868] ms
[sqrt task ispc]:       [6.237] ms
                                (1.84x speedup from ISPC)
                                (1.73x speedup from task ISPC)
```

可以看到，加速比小了很多。

设置为 2.999：
```bash
[sqrt serial]:          [1306.762] ms
[sqrt ispc]:            [188.561] ms
[sqrt task ispc]:       [15.588] ms
                                (6.93x speedup from ISPC)
                                (83.83x speedup from task ISPC)
```