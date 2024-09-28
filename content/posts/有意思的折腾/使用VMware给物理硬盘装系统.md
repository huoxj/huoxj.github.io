---
title: "使用VMware给物理硬盘装系统"
date: 2024-09-27T17:06:04-08:00
categories: 
- "有意思的折腾"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913151634.png
summary: "由于手贱不小心删了Ubuntu自带的Python，服务器上的Ubuntu Server炸了。。。只能重新装一遍系统。但是我懒得做启动U盘了，所以索性直接在VMware里把系统给装起来。 只需要先下好你..."
---

## 起因
由于手贱不小心删了Ubuntu自带的Python，服务器上的Ubuntu Server炸了。。。只能重新装一遍系统。但是我懒得做启动U盘了，所以索性直接在VMware里把系统给装起来。

## 创建虚拟机并挂载物理磁盘
只需要先下好你要装的系统的映象就行。然后把要装系统的硬盘装到你的电脑上。
还有很重要的一点，VMware一点要以**管理员身份**启动！不然Windows不让加物理磁盘！

首先随便创建一个虚拟机，配置选自定义。

![Pasted image 20240913151634](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913151634.png)

在安装来源这里选稍后安装操作系统，或者直接选系统映象。

![Pasted image 20240913151818](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913151818.png)

安装位置随便选一个地方就行，反正等会就给他删掉了。我直接放桌面。

![Pasted image 20240913151919](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913151919.png)

内存、网络正常选就行。
I/O控制器选VMware推荐的`LSI Logic`，其他的应该也没差。

![Pasted image 20240913152323](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913152323.png)

虚拟磁盘类型选SATA或者NVMe，取决于你的物理磁盘是哪种类型的。我装在机械硬盘上，所以选SATA。

![Pasted image 20240913155440](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913155440.png)

然后重要的来了，使用哪个磁盘这里选择使用物理磁盘。

![Pasted image 20240913152548](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913152548.png)

这里如果没有用管理员身份启动VMware就会出问题：

![Pasted image 20240913152716](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913152716.png)

然后就是选哪个PhysicalDrive的问题。这个只需要右键`此电脑`-`管理`-`磁盘管理`里面看就行。

![Pasted image 20240913153011](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913153011.png)

可以看到磁盘0、磁盘1和磁盘2。这就对应着PhysicalDrive的编号。然后选择`使用整个磁盘`。
磁盘分区信息放桌面那里就行了。

## 安装Ubuntu Server
创建好虚拟机之后，如果你之前创建虚拟机的时候没有选择映象的话，在虚拟机设置里面的`CD/DVD`中把映象选上去。然后启动时连接记得打开，不然

![Pasted image 20240913155651](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913155651.png)

然后在硬盘一项中点击`高级`，模式选择`独立`-`永久`

![Pasted image 20240913161135](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/Pasted%20image%2020240913161135.png)

然后就是正常的安装流程了。基本会一点英文就能顺利通过啦~