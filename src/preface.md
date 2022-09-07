<!--
# The embedonomicon
-->
# 嵌入式开发指南

<!--
The embedonomicon walks you through the process of creating a `#![no_std]` application from scratch
and through the iterative process of building architecture-specific functionality for Cortex-M
microcontrollers.
-->
这本 *嵌入式开发指南* 描述了从零开发一个无标准库应用 (`#![no_std]`) 的过程, 以及为微处理器 Cortex-M 开发架构特定功能的迭代过程.

<!--
## Objectives
-->
## 目标

<!--
By reading this book you will learn
-->
通过阅读这本书, 你将了解

<!--
- How to build a `#![no_std]` application. This is much more complex than building a `#![no_std]`
  library because the target system may not be running an OS (or you could be aiming to build an
  OS!) and the program could be the only process running in the target (or the first one).
  In that case, the program may need to be customized for the target system.
-->
- 如何构建一个无标准库应用. 这比不使用标准库开发库函数更加复杂, 因为目标机器上可能没有运行操作系统 (也许你正是要开发一个操作系统!) 并且这个程序是机器上运行的唯一一个进程 (或是第一个). 在这种情形下, 程序可能需要适配目标机器.

<!--
- Tricks to finely control the memory layout of a Rust program. You'll learn about linkers, linker
  scripts and about the Rust features that let you control a bit of the ABI of Rust programs.
-->
- 控制 Rust 程序内存布局的技巧. 你将看到链接器、链接脚本以及 Rust 相关特性是如何操控 Rust 程序的二进制应用接口 (ABI).

<!--
- A trick to implement default functionality that can be statically overridden (no runtime cost).
-->
- 实现并静态覆盖 Rust 默认功能的技巧 (没有运行时开销).

<!--
## Target audience
-->
## 目标受众

<!--
This book mainly targets to two audiences:
-->
这本书主要针对两类读者:

<!--
- People that wish to bootstrap bare metal support for an architecture that the ecosystem doesn't
  yet support (e.g. Cortex-R as of Rust 1.28), or for an architecture that Rust just gained support
  for (e.g. maybe Xtensa some time in the future).
-->
- 想要在一个 Rust 社区尚未支持 (比如 Rust 1.28 之于 Cortex-R) 或者刚开始支持 (比如 Xtensa) 的架构上进行裸机开发的读者.

<!--
- People that are curious about the unusual implementation of *runtime* crates like [`cortex-m-rt`],
  [`msp430-rt`] and [`riscv-rt`].
-->
- 对某些 *运行时* crate 的特殊实现 (比如 [`cortex-m-rt`], [`msp430-rt`] 和 [`riscv-rt`]) 感兴趣的读者.

[`cortex-m-rt`]: https://crates.io/crates/cortex-m-rt
[`msp430-rt`]: https://crates.io/crates/msp430-rt
[`riscv-rt`]: https://crates.io/crates/riscv-rt

<!--
## Translations

This book has been translated by generous volunteers. If you would like your
translation listed here, please open a PR to add it.

* [Japanese](https://tomoyuki-nakabayashi.github.io/embedonomicon/)
  ([repository](https://github.com/tomoyuki-nakabayashi/embedonomicon))
-->

<!--
## Requirements
-->
## 环境要求

<!--
This book is self contained. The reader doesn't need to be familiar with the
Cortex-M architecture, nor is access to a Cortex-M microcontroller needed -- all
the examples included in this book can be tested in QEMU. You will, however,
need to install the following tools to run and inspect the examples in this
book:
-->
这本书是自包含的. 读者不需要熟悉 Cortex-M 架构, 也不需要拥有一个 Cortex-M 微控制器 -- 书中的所有例子都可以在 QEMU 上测试. 当然, 读者可以安装下列工具运行和调试书中的例子:

<!--
- All the code in this book uses the 2018 edition. If you are not familiar with
  the 2018 features and idioms check the [`edition guide`].
-->
- 书中的所有代码都是 2018 版本的. 如果你不熟悉 2018 版本的特性和术语, 请查阅[`版本指引`].

<!--
- Rust 1.31 or a newer toolchain PLUS ARM Cortex-M compilation support.
-->
- Rust 1.31 或者更新的工具链**加上** ARM Cortex-M 编译支持.

<!--
- [`cargo-binutils`](https://github.com/japaric/cargo-binutils). v0.1.4 or newer.
-->
- [`cargo-binutils`](https://github.com/japaric/cargo-binutils). v0.1.4 或者更新版本.

- [`cargo-edit`](https://crates.io/crates/cargo-edit).

<!--
- QEMU with support for ARM emulation. The `qemu-system-arm` program must be
  installed on your computer.
-->
- 支持 ARM 模拟的 QEMU . 你的电脑上必须安装 `qemu-system-arm`.

<!--
- GDB with ARM support.
-->
- 支持 ARM 的 GDB.

[`版本指引`]: https://rust-lang-nursery.github.io/edition-guide/

<!--
### Example setup
-->
### 配置示例

<!--
Instructions common to all OSes
-->
所有操作系统共同指令

<!--
``` console
$ # Rust toolchain
$ # If you start from scratch, get rustup from https://rustup.rs/
$ rustup default stable

$ # toolchain should be newer than this one
$ rustc -V
rustc 1.31.0 (abe02cefd 2018-12-04)

$ rustup target add thumbv7m-none-eabi

$ # cargo-binutils
$ cargo install cargo-binutils

$ rustup component add llvm-tools-preview

```
-->
``` console
$ # Rust 工具链
$ # 如果你从零开始, 从 https://rustup.rs/ 下载 rustup
$ rustup default stable

$ # 工具链版本应该高于 1.31.0
$ rustc -V
rustc 1.31.0 (abe02cefd 2018-12-04)

$ rustup target add thumbv7m-none-eabi

$ # cargo-binutils
$ cargo install cargo-binutils

$ rustup component add llvm-tools-preview

```

#### macOS

<!-- 
``` console
$ # arm-none-eabi-gdb
$ # you may need to run `brew tap Caskroom/tap` first
$ brew install --cask gcc-arm-embedded

$ # QEMU
$ brew install qemu
```
-->
``` console
$ # arm-none-eabi-gdb
$ # 你也许要先运行 `brew tap Caskroom/tap`
$ brew install --cask gcc-arm-embedded

$ # QEMU
$ brew install qemu
```

#### Ubuntu 16.04

``` console
$ # arm-none-eabi-gdb
$ sudo apt install gdb-arm-none-eabi

$ # QEMU
$ sudo apt install qemu-system-arm
```

<!-- 
#### Ubuntu 18.04 or Debian
 -->
#### Ubuntu 18.04 或 Debian

<!-- 
``` console
$ # gdb-multiarch -- use `gdb-multiarch` when you wish to invoke gdb
$ sudo apt install gdb-multiarch

$ # QEMU
$ sudo apt install qemu-system-arm
```
-->
``` console
$ # gdb-multiarch -- 用 `gdb-multiarch` 替换 gdb 命令
$ sudo apt install gdb-multiarch

$ # QEMU
$ sudo apt install qemu-system-arm
```

#### Windows

<!-- 
- [arm-none-eabi-gdb](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads).
  The GNU Arm Embedded Toolchain includes GDB.
-->
- [arm-none-eabi-gdb](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads).
  包含 GDB 的 GNU Arm 嵌入式工具链.

- [QEMU](https://www.qemu.org/download/#windows)

## Installing a toolchain bundle from ARM (optional step) (tested on Ubuntu 18.04)
- With the late 2018 switch from
[GCC's linker to LLD](https://rust-embedded.github.io/blog/2018-08-2x-psa-cortex-m-breakage/) for Cortex-M 
microcontrollers, [gcc-arm-none-eabi][1] is no longer 
required.  But for those wishing to use the toolchain 
anyway, install from [here][1] and follow the steps outlined below:
``` console
$ tar xvjf gcc-arm-none-eabi-8-2018-q4-major-linux.tar.bz2
$ mv gcc-arm-none-eabi-<version_downloaded> <your_desired_path> # optional
$ export PATH=${PATH}:<path_to_arm_none_eabi_folder>/bin # add this line to .bashrc to make persistent
```
[1]: https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads
