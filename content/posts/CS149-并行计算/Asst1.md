---
title: "Asst1"
date: 2024-11-08T09:40:53-08:00
categories: 
- "CS149-并行计算"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411080945106.png
summary: "使用 `std::thread` 来绘制分形图像。 默认策略是将图像分成高度相同的几块，每一块分给一个线程画。 这样其实做不到负载均衡，因为每一块画的工作量是不同的。 特别是 view1，很明显发现画..."
---

## Prog1
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