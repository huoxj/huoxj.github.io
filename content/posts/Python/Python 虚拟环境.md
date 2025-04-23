---
date: '2024-11-28'
series:
- Python
title: Python 虚拟环境
---

对于 Python 来说，环境管理有着十分甚至九分重要的作用。本文列出一些常用的虚拟环境管理方式，以供参考。

## Conda

最常用的虚拟环境管理器。但是过于 heavy，我个人只会用 conda 配置一些**全局**、**常用**的环境。

conda 的好处就在于：在**任何地方**，只需要知道**环境名**就能启用环境。

## venv

Python 自带的虚拟环境管理器。

- 使用 VSCode

`ctrl + shift + p` 呼出命令面板，搜索 `Python: Create Environment`。

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202411281445582.png)

按步骤来就行。选择 venv 放哪个文件夹、有 requirements.txt 就选。

- 命令行

```bash
python3 -m venv <dir>
```

这会在当前目录创建一个叫 `<dir>` 的目录，存放虚拟环境

激活需要虚拟环境 `bin` 目录下的脚本。在 windows 和 linux 下执行起来有区别：

Linux:
```bash
source venv_dir/bin/activate
```

Windows:

```bash
.\venv_dir\bin\Activate.ps1
```