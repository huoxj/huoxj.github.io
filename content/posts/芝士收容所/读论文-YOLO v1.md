---
title: "读论文-YOLO v1"
date: 2024-11-08T21:56:06-08:00
categories: 
- "芝士收容所"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411101019480.png
summary: "大创要做一个基于两个特征矩阵的信号分割与分类（不知道该不该这么描述），精确度要求不高，但对实时性要求比较高。我想到了借鉴 YOLO 来解决这个问题，所以顺便就来读一下 YOLO 初代的论文，太复杂了猪..."
---

大创要做一个基于两个特征矩阵的信号分割与分类（不知道该不该这么描述），精确度要求不高，但对实时性要求比较高。我想到了借鉴 YOLO 来解决这个问题，所以顺便就来读一下 YOLO 初代的论文，太复杂了猪脑是理解不能的。

## Skimming

### 摘要

YOLO 做的活是 object detection。有如下特点：

- 单 CNN 实现图像分割与分类，而不是两个阶段两个网络
- 推理快，能做到实时 45 fps 以上
- 比传统的 DPM 和 R-CNN 泛化性更好，准确性也是其他模型的大约两倍

佬就是佬，不需要什么花里胡哨的背景介绍，上来就是我们的模型吊打其他模型。

### 介绍

首先，YOLO 的推理过程和人眼很像。这应该是在暗示模型的名字由来（

然后三大点：
- YOLO 很快。
- YOLO 视野更广。推理图片是依靠图像全局信息，而不是滑动窗口那种局部信息。
- YOLO 泛化很强。

### 原理

这里得好好看，所以先跳过。

### 对比其他网络

对比了 DPM 和 R-CNN 等模型。除了这两个模型稍微认识以外，其他的都认不得。

反正很强。

## 原理

### One-stage 如何实现

YOLO 将一张图像分成了 $S$ 行 $S$ 列块方格。

每一块方格负责预测：

- 物体类别
- $B$ 个预测框 (Bounding box)，以及对应置信度

参考下方图片。

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411101019480.png)

## 参考

[目标检测入门论文YOLOV1精读以及pytorch源码复现(yolov1) - 小小猿笔记 - 博客园](https://www.cnblogs.com/xiaoxiaojiea/p/14534513.html)