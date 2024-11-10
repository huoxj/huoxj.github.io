---
title: "小米路由器 4Av2 刷 Openwrt 并开启 NAT6"
date: 2024-11-03T23:09:37-08:00
categories: 
- "有意思的折腾"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411032345328.png
summary: "网络环境是`南京大学鼓楼校区校园网`。仙林应该是差不多的。 设备是`小米路由器 4A v2`。这个设备导致我装 Openwrt 的过程相当坎坷 TAT，再也不贪便宜买低端设备了。 Breed 是一种 ..."
---


## 需求
- 校园网 128 位前缀的 IPV6 地址做 NAT6
- IPV6 访问 PT 站
- 其他功能，比如魔法

网络环境是`南京大学鼓楼校区校园网`。仙林应该是差不多的。

设备是`小米路由器 4A v2`。这个设备导致我装 Openwrt 的过程相当坎坷 TAT，再也不贪便宜买低端设备了。

## 刷 Openwrt
### Breed
Breed 是一种 bootloader，路由器会先引导至 Breed 后再引导你刷的其他系统。这样就不怕系统刷坏变砖了。

刷 Breed 的教程很多，这里不再赘述。（刷一次基本一劳永逸了，所以不太记得怎么刷）

### 自编译 Openwrt
你也可以选择在线编译 Openwrt 的方案，比如 Github Action 和 [openwrt.ai](https://openwrt.ai)。

我之前直接使用了别人编译好的系统，但存在许许多多的问题导致 IPV6 配不起来，故尝试了自己编译，也还算有趣。

编译的环境最好选 Linux 或者 WSL。常用的包和依赖得备齐，并且最好全程有魔法，否则很难编译成功。

首先拉取 openwrt 源码。

```bash
git clone https://github.com/openwrt/openwrt
```

这样拉取到的是最新版。写作本文时最新版为 23.05.5，下文以此为准。

如果你想从其他版本构建，比如 LEDE 或者老版本 Openwrt，可以参考[小米4A千兆版V2刷自己编译的OpenWRT以及IPV6设置（包括中继与NAT6） - 哔哩哔哩](https://www.bilibili.com/read/cv23234832/)

在拉取下来的目录中运行：

```bash
git clone https://github.com/kenzok8/small-package package/small-package
```

这条命令把一些常用的软件包拉了下来，便于我们编译时选择。

然后在仓库目录中运行下面两条命令，可能会花一点时间，取决于你的网络：

```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

然后我们开始配置 config：

```bash
make menuconfig
```

会弹出以下界面：

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411032345328.png)

前三项选成图中选项。其他的就是自由发挥了，根据你的需求来。



### 刷入路由器

拔掉路由器电源，按住 reset 键插上电，等三秒以上再松，访问 `192.168.1.1` 就能进入 Breed 页面。

我们在固件备份中把系统和 eeprom.bin 备份一下。

（图片引用自[小米路由R4A千兆版安装breed+OpenWRT教程以及救砖（全脚本无需硬改） - 哔哩哔哩](https://www.bilibili.com/read/cv25114361/)）
![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411032348344.png)

然后就是刷入固件了。

但这一步中，小米路由器 4A v2 有一个大坑。不能直接用 breed 的`固件更新`刷固件。因为这个路由器的闪存布局比较独特，要把系统写到 `0x180000` 地址才能正常启动。如果你用 breed 的固件更新里刷固件，并且闪存布局是 `0x50000` 的话，路由器会无限重启。

参考[小米路由器 4A千兆版 V2 新版硬件安装OpenWRT - 知乎](https://zhuanlan.zhihu.com/p/680602138)就好。十分感谢这位仁兄。

## 开启 NAT6



## 参考
[小米路由R4A千兆版安装breed+OpenWRT教程以及救砖（全脚本无需硬改） - 哔哩哔哩](https://www.bilibili.com/read/cv25114361/)

[小米4A千兆版V2刷自己编译的OpenWRT以及IPV6设置（包括中继与NAT6） - 哔哩哔哩](https://www.bilibili.com/read/cv23234832/)

[小米路由器 4A千兆版 V2 新版硬件安装OpenWRT - 知乎](https://zhuanlan.zhihu.com/p/680602138)