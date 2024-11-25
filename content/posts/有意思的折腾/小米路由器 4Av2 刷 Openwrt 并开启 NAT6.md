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
你也可以选择在线编译 Openwrt 的方案，比如 Github Action 、[OpenWrt Firmware Selector](https://firmware-selector.openwrt.org/) 和 [openwrt.ai](https://openwrt.ai)。

我之前直接使用了别人编译好的系统，但存在许许多多的问题导致 IPV6 配不起来，故尝试了自己编译，也还算有趣。抖m指数++。

编译的环境最好选 Linux 或者 WSL。常用的包和依赖得备齐，并且最好全程有魔法，否则很难编译成功。

首先拉取 openwrt 源码。

```bash
git clone https://github.com/openwrt/openwrt
```

拉取下来的 `main` 分支是 snapshot 版本，建议还是选择稳定版，否则有些软件包装不了。写作本文时最新版为 23.05.5，对应 branch 为 `openwrt-23.05`，下文以此为准。

```bash
git checkout openwrt-23.05
```

如果你想从其他版本构建，比如 LEDE 或者老版本 Openwrt，可以参考[小米4A千兆版V2刷自己编译的OpenWRT以及IPV6设置（包括中继与NAT6） - 哔哩哔哩](https://www.bilibili.com/read/cv23234832/)

cd 到拉取下来的目录中运行：

```bash
git clone https://github.com/kenzok8/small-package package/small-package
```

这条命令把一些常用的软件包拉了下来，便于我们编译时选择。

然后在仓库目录中运行下面两条命令，可能会花一点时间，取决于你的网络：

```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

过程中可能会提示缺少依赖，注意看下脚本 log，把缺的东西装上。

然后我们开始配置 config：

```bash
make menuconfig
```

会弹出以下界面：

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411032345328.png)

前三项选成图中选项。其他的就是自由发挥了，根据你的需求来。这里有 LuCI 软件包的插件对照表[rk3568 m68s openwrt插件对照表（自己备用）-OPENWRT专版-恩山无线论坛](https://www.right.com.cn/forum/thread-8406571-1-1.html)

menuconfig 在使用上只要注意有些地方可以用 `/` 键来搜索，和 vim 一样。

并且注意一下选择软件包时前面的 `< >` , `<*>` 和 `<M>`：
- `< >` 表示不安装这个包
- `<*>` 表示将这个包以系统核心组件安装。会直接将软件包安装到固件里面。
- `<M>` 表示将这个包编译成模块，需要后续在 openwrt 中安装。

建议直接安装到固件中，后续是可以删的，并且免得装了系统还要装一大堆包。

顺带一提，我的 config 如下：

- Base system
	- `dnsmasq-full`：完全版 dnsmasq，因为要用 IPV6。记得把基础版 dnsmasq 删了。
- Kernel modules
	- Network Support
		- `kmod-tcp-bbr`：TCP 阻塞控制
- LuCI
	- Collections
		- `luci`：luci 图形界面整合包
	- Applications
		- `luci-app-adguardhome`： 去广告 + DNS
		- `luci-app-npc`：内网穿透，类似 frpc
		- `luci-app-passwall2`：富强民主文明和谐
		- `luci-app-smartdns`：聪明dns
- Network
	- `UDPspeeder`
- Utilities
	- Editors
		- `vim`：ssh 改东西的时候方便

config 配置好后，选到底下那排 Save，以 `.config` 保存。

接着正式开始编译。建议使用特殊手段来优化网络，能快不少。

```bash
make download -j8
make V=s -j1
```

第一个 make 是下载对应软件包，直接上八核伺候。

第二个是编译固件，第一次编译建议只用一个核心，否则并行起来日志顺序都是乱的，不方便看报错。

编译的时间一般要半个小时以上。5800x 第一次编译大概花了 20 多分钟。但是增量编译就很快了，几分钟。

编译完成后，在 `target/`

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

[rk3568 m68s openwrt插件对照表（自己备用）-OPENWRT专版-恩山无线论坛](https://www.right.com.cn/forum/thread-8406571-1-1.html)