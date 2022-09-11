<!-- 
# The smallest `#![no_std]` program
-->
# 最小无标准库程序 (`#![no_std]`)

<!-- 
In this section we'll write the smallest `#![no_std]` program that *compiles*.
-->
在这一章里, 我们将编写一个 *能够通过编译* 的最小无标准库程序.

<!-- 
## What does `#![no_std]` mean?
-->
## 无标准库宏 `#![no_std]` 的作用?

<!-- 
`#![no_std]` is a crate level attribute that indicates that the crate will link to the [`core`]
crate instead of the [`std`] crate, but what does this mean for applications?
-->
`#![no_std]` 是一个 crate 层级的宏, 指示链接器把当前 crate 链接到 [`core`] 而不是 [`std`], 但这对于应用来说意味着什么呢?

[`core`]: https://doc.rust-lang.org/core/
[`std`]: https://doc.rust-lang.org/std/

<!-- 
The `std` crate is Rust's standard library. It contains functionality that assumes that the program
will run on an operating system rather than [*directly on the metal*]. `std` also assumes that the
operating system is a general purpose operating system, like the ones one would find in servers and
desktops. For this reason, `std` provides a standard API over functionality one usually finds in
such operating systems: Threads, files, sockets, a filesystem, processes, etc.
-->
`std` crate 是 Rust 的标准库. 它所包含的功能默认程序运行在操作系统而不是[*裸机*][*directly on the metal*]上. `std` 也默认操作系统是一个通用操作系统, 也就是那些服务器和台式机上常见的系统. 因此, `std` 提供的标准 API 建立在大多数操作系统通用的功能上: 线程、文件、socket、文件系统、进程等等.

[*directly on the metal*]: https://en.wikipedia.org/wiki/Bare_machine

<!-- 
On the other hand, the `core` crate is a subset of the `std` crate that makes zero assumptions about
the system the program will run on. As such, it provides APIs for language primitives like floats,
strings and slices, as well as APIs that expose processor features like atomic operations and SIMD
instructions. However it lacks APIs for anything that involves heap memory allocations and I/O.
-->
另一方面, `core` 是 `std` 的一个子集, 它对于程序所运行的系统没有任何假设. 比如, 它面向浮点数、字符串和数组这些编程原语提供了 API, 以及暴露处理器功能的 API 比如原子操作和 SIMD 指令. 但是它缺少涉及堆上内存分配和 I/O 的 API.

<!-- 
For an application, `std` does more than just providing a way to access OS abstractions. `std` also
takes care of, among other things, setting up stack overflow protection, processing command line
arguments and spawning the main thread before a program's `main` function is invoked. A `#![no_std]`
application lacks all that standard runtime, so it must initialize its own runtime, if any is
required.
-->
对于一个应用, `std` 不仅提供了获取系统资源抽象的方法, 同时也做了很多其他事情, 比如设置栈溢出 (stack overflow) 保护、处理命令行参数以及在 `main` 函数被调用前复制主线程. 一个无标准库程序缺少这些标准运行时, 所以如果需要运行时, 它必须自己初始化.

<!-- 
Because of these properties, a `#![no_std]` application can be the first and / or the only code that
runs on a system. It can be many things that a standard Rust application can never be, for example:

- The kernel of an OS.
- Firmware.
- A bootloader.
-->
因为这些特性, 一个无标准库应用可以成为系统上运行的第一段甚至是唯一一段代码. 它可以做到许多标准 Rust 应用做不到的事, 比如:

- 系统内核
- 硬件
- 启动程序

<!-- 
## The code
-->
## 代码

<!-- 
With that out of the way, we can move on to the smallest `#![no_std]` program that compiles:
-->
解答了上面的问题后, 我们继续研究一个能够通过编译的最小无标准库程序:

``` console
$ cargo new --edition 2018 --bin app

$ cd app
```

<!-- 
``` console
$ # modify main.rs so it has these contents
$ cat src/main.rs
```
-->
``` console
$ # 修改 main.rs 中的内容如下
$ cat src/main.rs
```

``` rust
{{#include ../ci/smallest-no-std/src/main.rs}}
```

<!-- 
This program contains some things that you won't see in standard Rust programs:
-->
这个程序包含了一些标准 Rust 程序中没有的部分:

<!-- 
The `#![no_std]` attribute which we have already extensively covered.
-->
我们在前面已经提到的 `#![no_std]`.

<!-- 
The `#![no_main]` attribute which means that the program won't use the standard `main` function as
its entry point. At the time of writing, Rust's `main` interface makes some assumptions about the
environment the program executes in: For example, it assumes the existence of command line
arguments, so in general, it's not appropriate for `#![no_std]` programs.
-->
`#![no_main]` 注解意味着程序不使用标准的 `main` 函数作为程序入口. 在本书写作时, Rust 的 `main` 接口对程序运行环境做了一些假定: 比如, 它假设命令行参数是存在的, 所以通常来说它不适用于 `#![no_std]` 程序.

<!-- 
The `#[panic_handler]` attribute. The function marked with this attribute defines the behavior of
panics, both library level panics (`core::panic!`) and language level panics (out of bounds
indexing).
-->
`#![panic_handler]` 注解. 该注解函数定义了如何处理 panic, 无论是函数库级别的 panic (`core::panic!`) 还是语言级别的 panic (数据越界).

<!--
This program doesn't produce anything useful. In fact, it will produce an empty binary.
-->
这个程序没有产生任何作用. 实际上, 它编译出了一个空二进制文件.

<!--
``` console
$ # equivalent to `size target/thumbv7m-none-eabi/debug/app`
$ cargo size --target thumbv7m-none-eabi --bin app
```
-->
``` console
$ # 等价于 `size target/thumbv7m-none-eabi/debug/app`
$ cargo size --target thumbv7m-none-eabi --bin app
```

``` text
{{#include ../ci/smallest-no-std/app.size}}
```

<!--
Before linking, the crate contains the panicking symbol.
-->
在链接前, 这个 crate 包含了程序崩溃标志.

``` console
$ cargo rustc --target thumbv7m-none-eabi -- --emit=obj

$ cargo nm -- target/thumbv7m-none-eabi/debug/deps/app-*.o | grep '[0-9]* [^N] '
```

``` text
{{#include ../ci/smallest-no-std/app.o.nm}}
```

<!--
However, it's our starting point. In the next section, we'll build something useful. But before
continuing, let's set a default build target to avoid having to pass the `--target` flag to every
Cargo invocation.
-->
但是, 这是我们的起点. 接下来, 我们将构建一些有用的东西. 但是在开始之前, 让我们设置默认的编译目标以避免每次都要传 `--target` 参数.

<!--
``` console
$ mkdir .cargo

$ # modify .cargo/config so it has these contents
$ cat .cargo/config
```
-->
``` console
$ mkdir .cargo

$ # 修改 .cargo/config 中的内容如下
$ cat .cargo/config
```

``` toml
{{#include ../ci/smallest-no-std/.cargo/config}}
```

<!--
## eh_personality
-->
## 自定义错误处理 eh_personality

<!--
If your configuration does not unconditionally abort on panic, which most targets for full operating
systems don't (or if your [custom target][custom-target] does not contain
`"panic-strategy": "abort"`), then you must tell Cargo to do so or add an `eh_personality` function,
which requires a nightly compiler. [Here is Rust's documentation about it][more-about-lang-items],
and [here is some discussion about it][til-why-eh-personality].
-->
大部分完整操作系统的目标机器在遇到 panic 不会无条件停机, 如果你的配置也是如此 (或者你的[自定义目标机器][custom-target]不包含停机策略 `"panic-strategy": "abort"`), 那么你必须告诉 Cargo 这么做或是添加一个 `eh_personality` 函数, 后者需要一个 nightly 版本的编译器. 这里有 [相关的 Rust 文档][more-about-lang-items] 和 [相关的讨论][til-why-eh-personality].

<!--
In your Cargo.toml, add:
-->
在你的 Cargo.toml 中添加:

``` toml
[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

<!--
Alternatively, declare the `eh_personality` function. A simple implementation that does not do
anything special when unwinding is as follows:
-->
除此之外, 也可以声明一个 `eh_personality` 函数. 一个没有做任何处理的实现如下:

``` rust
#![feature(lang_items)]

#[lang = "eh_personality"]
extern "C" fn eh_personality() {}
```

<!--
You will receive the error `language item required, but not found: 'eh_personality'` if not
included.
-->
如果没有前一行, 你会看到一个 `language item required, but not found: 'eh_personality'` 的报错.

[custom-target]: ./custom-target.md
[more-about-lang-items]:
  https://doc.rust-lang.org/unstable-book/language-features/lang-items.html#more-about-the-language-items
[til-why-eh-personality]:
  https://www.reddit.com/r/rust/comments/estvau/til_why_the_eh_personality_language_item_is/
