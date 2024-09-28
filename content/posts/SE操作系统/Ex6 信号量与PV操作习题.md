---
title: "Ex6 信号量与PV操作习题"
date: 2024-05-24T09:53:23-08:00
categories: 
- "SE操作系统"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409271721124.png
summary: "rmutex的理解 : waiting customers"
---

# 信号量与PV操作习题

## 读者写者问题

### v1-读者优先策略

![image-20240524100405216](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409271721124.png)

rmutex的理解

- 可以理解为更新临界资源readcount的坑位，同一时间只允许一个reader蹲在那里更新readcount

## 理发师问题

> 一个理发师、一把理发椅、等候区容量N
>
> 理发师：等候区有顾客就理发，没顾客睡觉
>
> 顾客：等候区不满就进等候区，在等候区如果理发师睡觉就叫醒；等候区满就不理发了

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409271721753.png)

### 重要思想

**协同信号量**: waiting <-> customers

- waiting变量和customers信号量是一对协同量，二者意义相同，数值也相同(除了customer可能为-1)。
- 所以要用mutex信号量去保证二者在**有顾客的时候**是同步变化的，这样去理解mutex就容易的多了
- **为什么**？因为我们要判断if(waiting<CHAIRS)，假如没有waiting，我们就得判断if(customers<CHAIRS)，但是用户代码直接访问信号量的操作是不被允许的