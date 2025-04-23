---
date: '2025-02-28'
series:
- 有意思的折腾
title: Nvim —— 我的第N代移动开发方案
---

# 起因

本人是台式机的忠实拥趸，对于笔记本持接受不能的态度。不过台式机便携性极低的客观事实无法忽略……这也导致上课时除了坐牢便无事可做。

幸运的是，我能算得上半个 iPad 的爱好者（可能与经历有关，iPad 的确为童年带来了许多快乐的时光）。在 iPad 上搭建一个趁手的开发环境，能让购买时付出高昂成本变得更值得一些。

综上，需求和要求为：
- iPad 开发环境
- 尽可能趁手

# Gen1: vscode on iPad

大名曰：Code App

旨在给 ipad 带来纯粹的 VS Code 体验。Github：[thebaselab/codeapp: Building a full-fledged code editor for iPad](https://github.com/thebaselab/codeapp)

刚上手时感受：
- 轻松上手，毕竟和 VS Code 一个模子
- 可以应对一些简易的 c/c++ 和 python 项目，甚至可以本地编译和运行
- 内置终端有一些基础的命令
- SSH 支持还行

但是，使用了一周之后我便放弃了这个 app，只说几个关键的点：
- 稍大的工程相当不好用。工具链限制太多了
- 没有 extensions
- ipad 上戳触控板很难受
- App 是 testflight 白嫖的。App Store 上 5.99$

我需要的是一整套的、可自定义的开发环境，而不仅仅是一个带有简单终端的 toy IDE。故放弃。

# Gen2: virtual machine

那这样，我直接在 iPad 上搓一个 Linux 虚拟机出来不就好了！？

目光投向了 [UTM](https://getutm.app/)

我不想过于详细地介绍我是怎么折腾地装上 UTM、如何绕过 IOS 的应用证书、最终只得到了一个卡得完全没法用的 Ubuntu 22.04 GNOME 桌面的。这无异于向你展示如何将大象装进冰箱。

我的总结是：至少从开发环境的角度，请不要抱着日常使用的期望在 IpadOS 15.6 以上的版本安装虚拟机。

- IOS 对危险软件（utm 这种虚拟机）及其封闭
- ipad 内存比金子贵、虚拟机架构特殊等等问题必然会导致后续的使用不尽人意

# Gen3: 串流

RT，直接将 Windows 开发环境串流到当前设备即可。相关工具有：

- RD Client
	- 微软原生串流。走的是 RDP 的 3389 端口
	- 配置还算方便，稳定
	- 流畅度、画质一般
- ToDesk
	- 国内流行的远程控制方案
	- 十分方便，还算稳定
	- 流畅度画质和 rd client 差不多，更好的体验得加钱
- Moonlight / Sunshine
	- 游戏首选
	- 配置比较麻烦
	- 基本接近原生的延迟、流畅度和画质

平心而论，串流是一个十分不错的方案。不太用担心网速问题，凭借现在串流的压缩技术，双方有 5~10M 的稳定带宽基本就能有不错的体验了。

但是教室网速比较一般（不稳定），而且我的 iPad 触控板也相当难用，我也无意购入妙控键盘。所以这个方案仅在使用一周后便被抛弃了。

# Gen4: iSH

在 iPad 上存在一个 fancy 的软件 [iSH](https://ish.app/)。基于 x86 模拟器为 IOS 提供一个 linux shell 环境。

乍一看似乎在 iPad 上运行了一个 linux。但其背后的原理比较复杂，自底向上地说：
- iSH 完全运行在 iOS 提供的用户态环境中，没有越狱、不涉及内核扩展。整个应用在用户态。
- iSH 实现了一个用户态的 x86 仿真器，并通过将 Linux 的系统调用转译成等价的 iOS 系统调用，从而让 Linux 用户空间程序得以运行。相当于一个中间层。
- iSH 通过自己实现的中间层，来运行真正的 Linux 镜像。ish 默认使用轻量的 [Alpine Linux](https://www.alpinelinux.org/)
- 你看到的其实是 Alpine 提供的工具链。终端、BusyBox 命令、`apk` 包管理器等，都是 Alpine Linux 用户空间的一部分，而不是 iSH 自己写的。

这很 hack 和 elegant，那么代价是？

- 慢。没有 JIT，iSH 的系统调用转译是解释执行。
- 内存小。JVM 都跑不起来，更别说 docker 了
- 纯软件实现，没有硬件加速
- IOS 限制摆在这，多线程等内核支持完全没有

所以这个 iSH 充其量只是一个“工程妥协的艺术”（ChatGPT 锐评）。

# Gen5: SSH

答案早就甩在脸上了，但这个 ssh 指的是纯命令行开发。都什么年代了还在用命令行？是现代 IDE 不好用吗？

答：妥协+适应之后的豁然开朗。妥协就不说了，iPad 的特殊条件。

适应，一方面得跨过 ssh 的门槛，对于 linux 得足够熟悉和适应。

另一方面，将 ssh 一步步配置成高效的开发工具。走过适应二者的苦痛之路后，其实 ssh 也不是那么难用。甚至发现自己在逐渐抛弃 jetbrains 和 vscode。

目前的 ssh 工作流：t[mux](https://github.com/tmux/tmux) + [nvim](https://neovim.io/) with [LazyVim](https://www.lazyvim.org/)。

有时间写一下我是如何配置全套开发环境的。