---
title: "Clangd 无法识别 CMake 生成的 CUDA --options-file 标识"
date: 2025-03-13
series: 
- "CUDA"
---

# tl.dr

CMake 生成的 compile_commands.json 中，对 .cu 文件会生成 --options-file 的标识，而 clangd 无法识别，从而导致 clangd 报错无法找到头文件。

解决：在 CMake 中关闭为 cuda 生成 --options-file 的选项

```cmake
# This would work
set(CMAKE_CUDA_USE_RESPONSE_FILE_FOR_INCLUDES 0)
# You can add below if you like
set(CMAKE_CUDA_USE_RESPONSE_FILE_FOR_LIBRARIES 0)
set(CMAKE_CUDA_USE_RESPONSE_FILE_FOR_OBJECTS 0)
```

或者用其他能正确生成 compile_commands.json 的工具。比如 bear 或者 xmake。

# 原因

使用 CMake(3.25.2) 构建工程，某些 .cu 文件里需要 include header目录下的头文件。

```cmake
include_directories(header)
```

生成的 nvcc 编译命令**理应**长这样：

```bash
nvcc *.cu -I/.../header ...
```

但是，CMake 实际生成了：

```bash
nvcc *.cu --options-file .../includes_CUDA.rsp
```

找到 `includes_CUDA.rsp`，里面存放了 -I 的 flags。

```includes_CUDA.rsp
-I/.../header
```

而这个 `--options-file` 选项是 nvcc 独有的，并且 clangd(18.1.3) 似乎无法识别。导致 .cu 在 IDE 中无法识别头文件。

> 这个选项是为了防止 include 和 link 的目录太多，导致命令过长而诞生的
> 大名为 response file
> [How to force CMake to use a response file when linking CUDA on Windows? - Stack Overflow](https://stackoverflow.com/questions/67928865/how-to-force-cmake-to-use-a-response-file-when-linking-cuda-on-windows)

# 解决

搜索了一会没找到从 Clangd 入手的解决方法。只能从 **CMake** 入手。关闭 `--options-file` 的选项即可，CMake 会生成正常的 `-I` 选项。

# 其他

除了 `--options-file`，CMake 还会生成一些 Clangd 无法识别的标识，比如：

- -forward-unknown-to-host-compiler
- --generate-code
- 待发现

稍好的解决方法是在 compile_commands.json 同目录下写一个 **.clangd** 配置，把这些 clangd 无法识别的标识替换掉。

```.clangd
CompileFlags:
  Add:
    - --cuda-path=/usr/local/cuda
    - --cuda-gpu-arch=sm_89
    - -L/usr/local/cuda/lib64
    - -I/usr/local/cuda/include
  Remove:
    - -forward-unknown-to-host-compiler
    - --generate-code*
    - --options-file

```

# 参考

https://github.com/clangd/vscode-clangd/issues/592#issuecomment-2036788981