---
layout: post
title: 如何在 Apple Silicon 上使用 Homebrew 安装 x86_64 的库
subtitle: How to install x86_64 librarys via Homebrew on Apple Silicon
categories: Homebrew
tags: [Homebrew, arm64, x86_64]
---

### Circom 生成的 C++代码

在[上一篇文章]({{ site.baseurl }}{% link _posts/2024-01-12-circom_zkproof.md %})中，当执行完如下命令后

```bash
circom AgeProof.circom --r1cs --wasm --sym --c
```

其实会同时生成如下两个目录: `AgeProof_cpp` 和 `AgeProof_js`

![circom 产出的cpp 和 js]({{ "/assets/images/2024-01-14/cpp_dir.png" | absolute url }})

### 用 cpp 代码生成 witness

(如果只关心如何在 M1 上下载 x86_64 的库，可以忽略此段)

上一篇文章中我们只用到了 AgeProof_js 目录下的内容

但其实我们也可以用 cpp 代码来生成 witness

- 安装相关库

  - [nlohmann/json](https://github.com/nlohmann/json)

    https://github.com/nlohmann/json/tree/develop/single_include/nlohmann 目录下的文件拷贝到 AgeProof_cppnlohmann 目录下

  - libgmp

    ```bash
    brew install gmp
    ```

  - nasm

    ```bash
    brew install nasm
    ```

  - 修改 Makefile 为

    ```makefile
    CC=g++

    CFLAGS=-std=c++11 -O3 -I. -I/opt/homebrew/opt/gmp/include
    DEPS_HPP = circom.hpp calcwit.hpp fr.hpp
    DEPS_O = main.o calcwit.o fr.o fr_asm.o


    ifeq ($(shell uname),Darwin)
      NASM=nasm -fmacho64 --prefix _
    endif
    ifeq ($(shell uname),Linux)
    NASM=nasm -felf64
    endif
    all: AgeProof
    %.o: %.cpp $(DEPS_HPP)
    $(CC) -c $< $(CFLAGS)

    fr_asm.o: fr.asm
    $(NASM) fr.asm -o fr_asm.o
    AgeProof: $(DEPS_O) AgeProof.o
    $(CC) -o AgeProof \*.o -lgmp -L/opt/homebrew/opt/gmp/lib

    ```

- 执行 `make`, 编译出 `./AgeProof` 可执行文件

- 运行 `./AgeProof input.json witness.wtns` 来实现和 js 一样的功能

  在 circom 文件比较复杂时，cpp 可执行文件的效率, 会比 js 快一些

### 安装 `x86_64` 的 `Homebrew` 和 库

- 安装 `x86_64` 的 `Homebrew`

  此时在 Apple Silicon 的 cpu 上会遇到一个问题, 因为 circom 导出的 `fr.asm` 都是 `x86_64` 的指令，无法被正确编译为 arm64 的可执行文件。

  而如果用 `arch -x86_64 make` 来进行编译， 会提示说 ` libgmp.a`` 是  `arm64`` 架构的，无法被正确链接(或者提示找不到库)。

  此时可以用如下方式在 M1 上安装 x86_64 的 Homebrew 和相关库

  ```bash
  arch -x86_64 zsh
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```

  此时当前的 terminal 会变成 `x86_64` 的环境, 然后会安装 `x86_64` 的 `Homebrew`

  这个 `x86_64` 的 Homebrew 会安装在 `/usr/local/bin/brew` 的位置

  而 Apple Silicon 的 `arm64 Homebrew` 会安装在 `/opt/homebrew/bin/brew` 的位置

  如果出现如下错误

  ```bash
  Error: Failed to link all completions, docs and manpages:
  Permission denied @ rb_file_s_symlink - (../../../Homebrew/completions/fish/brew.fish, /usr/local/share/fish/vendor_completions.d/brew.fish)
  Failed during: /usr/local/bin/brew update --force --quiet
  ```

  可以执行如下命令后重新运行安装

  ```bash
  sudo chown -R $(whoami) $(brew --prefix)/*
  sudo chown -R $(whoami) /usr/local/*
  ```

- 通过 `Homebrew` 安装 `x86_84` 的 `gmp`

  ```bash
  arch -x86_64 /usr/local/bin/brew reinstall gmp
  ```

  这种情况下 gmp 对应的 include 和 lib 目录分别是

  `/usr/local/opt/gmp/include` 和 `/usr/local/opt/gmp/lib`

  我们也可以用 `lipo` 命令来对 `libgmp.a` 进行确认

  ```bash
  lipo -info /usr/local/opt/gmp/lib/libgmp.a
  Non-fat file: /usr/local/opt/gmp/lib/libgmp.a is architecture: x86_64
  ```

- 安装 `arm64` 的 `Homebrew`库

  如果不太确认当前 terminal 是 `x86_64` 还是 `arm64` 的环境，可以执行如下命令来切换到 arm64

  ```bash
  arch -arm64 zsh
  ```

  然后运行如下命令之一来安装 `gmp``

  ```bash
  /opt/homebrew/bin/brew install gmp
  brew install gmp
  ```

  这种情况下 gmp 对应的 include 和 lib 目录分别是

  `/opt/homebrew/opt/gmp/include` 和 `/opt/homebrew/opt/gmp/lib`

  我们也可以用 `lipo` 命令来对 `libgmp.a` 进行确认

  ```bash
  lipo -info /opt/homebrew/opt/gmp/lib/libgmp.a
  Non-fat file: /opt/homebrew/opt/gmp/lib/libgmp.a is architecture: arm64
  ```
