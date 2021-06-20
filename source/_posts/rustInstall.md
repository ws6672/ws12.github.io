---
title: rust安装与初试
date: 2021-06-16 16:43:10
tags: [rust]
---

# 一、 安装

官方文档：https://doc.rust-lang.org/book/


1.编译环境选择

rust底层是依赖C环境，所以需要先安装C/C++编译环境, 有两种选择:

+	安装微软的[C++ build tools](https://visualstudio.microsoft.com/zh-hans/downloads/)，即 Visual Studio 2019 生成工具；
+	安装[mingw](https://sourceforge.net/projects/mingw-w64/files/)/cygwin（模拟linux） ，需要配置 gnu toolchain开发环境（stable-x86_64-pc-windows-gnu）：
	+	Cygwin 提供了Windows下的类Unix环境（Window）；
	+	MinGW 是提供用于开发Window应用的开发环境（Linux）。


# 二、配置Cygwin

1. 初始化与更新 Cygwin 包

为 64 位版本的 Windows 安装和更新 Cygwin：当您想要更新或安装适用于 64 位 Windows 的 [Cygwin 软件包](https://cygwin.com/setup-x86_64.exe) 时，请运行setup-x86_64.exe。该签名的设置，x86_64.exe可以用来验证这个二进制文件的有效性。


注意事项：
+	首次安装软件包时，安装程​​序 不会安装每个软件包。 默认情况下只安装 Cygwin 发行版中的最小基础包，大约占用 100 MB。
单击安装程序包安装屏幕中的类别和包，您可以选择安装或更新的内容。像bash、gcc、less等单独的包是独立于 Cygwin DLL 发布的，因此 Cygwin DLL 版本作为通用的 Cygwin 版本号没有用处。安装程序会跟踪所有已安装组件的版本，并提供用于安装或更新此站点上 Cygwin 可用所有内容的机制。

不提供apt, yum：
+	不使用功能更全的包管理器的原因是这样的程序需要完全访问 Cygwin 的所有 POSIX 功能。然而，这在没有 Cygwin 的环境中很难提供，例如需要第一次安装时就存在。此外，Windows 不允许覆盖正在使用的可执行文件，因此在包管理器使用 DLL 时安装新版本的 Cygwin DLL 是有问题的。


验证安装程序的签名（例子）：
+	```
	gpg --recv-key 1A698DE9E2E56300
	gpg --keyid-format=long --with-fingerprint --verify setup-x86_64.exe.sig setup-x86_64.exe
	```

通过Cygwin Setup 可以更新包，为了能够通过命令安装软件，可以先使用该程序添加GIT。
+	打开后，直接下一步即可；
+	到了包管理页面找到GIT选择版本号即可；
+	再通过GIT安装apt-cyg。
	+	```
	git clone https://github.com/transcode-open/apt-cyg.git
	cd apt-cyg/
	chmod +x apt-cyg
	cp apt-cyg /bin
	apt-cyg install wget
	```

注：wget安装会提示缺少lynx，需要通过Cygwin安装程序安装。

2. 在windows命令下使用cygwin

需要添加环境变量，Path 添加 `D:\cygwin64\bin`,刷新环境变量`set PATH=C`。

在Window下通过以下命令可以进入类unix环境：
```bash
D:\Environment\Rust>bash
	86178@LAPTOP-KFPIFLPS /cygdrive/d/Environment/Rust
	86178@LAPTOP-KFPIFLPS /cygdrive/d/Environment/Rust exit
```

3. 为了通过 Cygwin配置Rustup，需要通过Cygwin安装程序安装以下包：

+	httpd
+	make
+	ls
+	curl

# 三、Rustup

Rustup是Rust 安装程序和版本管理工具，[相关资源](https://www.rust-lang.org/learn/get-started)

1. 配置环境变量

```
CARGO_HOME=D:\Environment\Rust\.cargo
RUSTUP_HOME=D:\Environment\Rust\.rustup
RUSTUP_UPDATE_ROOT=http://mirrors.ustc.edu.cn/rust-static/rustup
RUSTUP_DIST_SERVER=http://mirrors.ustc.edu.cn/rust-static
path 添加%CARGO_HOME%\bin

set PATH=C
```
 
2. 安装程序

打开rustup-init.exe文件，输入2，自定义安装选项，如下：
```
host triple：x86_64-pc-windows-gnu 版本
default toolchain：stable 稳定版
profile: complete 表示完全安装
modify PATH variable：y 表示按照环境变量定义的路径安装
```

查看安装结果：
```
C:\Users\86178>cargo version
cargo 1.52.0 (69767412a 2021-04-21)
```


查看版本：
```
C:\Users\86178>rustup show
Default host: x86_64-pc-windows-gnu
rustup home:  D:\Environment\Rust\.rustup

installed toolchains
--------------------

stable-x86_64-pc-windows-gnu (default)
stable-x86_64-pc-windows-msvc

active toolchain
----------------

stable-x86_64-pc-windows-gnu (default)
rustc 1.52.1 (9bc8c42bb 2021-05-09)
```

3. Hello World

一个Hello World的例子。

3.1 配置

在windows配置项目：
```
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

创建一个新的源文件并将其命名为main.rs。Rust 文件总是以.rs扩展名结尾。如果您在文件名中使用多个单词，请使用下划线将它们分开。例如，使用hello_world.rs而不是 helloworld.rs。

```rust
fn main() {
    println!("Hello, world!");
}
```


注：将左大括号与函数声明放在同一行是一种很好的风格，中间加一个空格。如果你想在 Rust 项目中坚持标准风格，你可以使用一个自动格式化工具`rustfmt`来以特定的风格格式化你的代码.
+	Rust 风格是缩进四个空格，而不是一个制表符;
+	`println!`调用一个 Rust 宏。如果它改为调用函数，则输入为println（不带!）
+	大多数 Rust 代码行都以分号结尾。

3.2 编译与运行

在运行 Rust 程序之前，您必须使用 Rust 编译器通过输入`rustc`命令并将源文件的名称传递给它来编译它

运行方式如下：
```
-- Linux
$ rustc main.rs  # 编译
$ ./main  # 运行
Hello, world!

-- Window
> rustc main.rs
> .\main.exe
Hello, world!
```

Rust 是一种提前编译语言，这意味着您可以编译程序并将可执行文件提供给其他人，他们甚至可以在没有安装 Rust 的情况下运行它。如果你给某人一个.rb、.py或 .js文件，他们需要安装一个 Ruby、Python 或 JavaScript 实现（分别）。但是在这些语言中，您只需要一个命令来编译和运行您的程序。一切都是语言设计的权衡。

# 四、Cargo

Cargo 是 Rust 的构建系统和包管理器。大多数 Rustaceans 使用这个工具来管理他们的 Rust 项目，因为 Cargo 为你处理了很多任务，比如构建代码、下载代码依赖的库以及构建这些库。（我们将您的代码需要的库称为依赖项。）


1. Cargo创建项目

查看是否安装cargo：
```rust
cargo --version
```

使用 Cargo 创建项目：
```rust
$ cargo new hello_cargo
$ cd hello_cargo

-- 使用现成的git存储库创建项目
cargo new --vcs=git.
```

2. 文件结构

生成以下文件
+	src
	+	main.rs
+	Cargo.toml
+	.gitignore
+	.git  Git存储库


Cargo.toml（`Tom's Obvious, Minimal Language`）的内容如下：
```
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["ws6672 <ws6672s@gmail.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

```

Cargo 希望您的源文件位于src目录中。顶级项目目录仅用于 README 文件、许可证信息、配置文件以及与您的代码无关的任何其他内容。使用 Cargo 可以帮助您组织项目。一切都有一个地方，一切都在它的位置。

3. 编译与运行项目

```
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
$ cargo run

// 此命令会快速检查您的代码以确保它可以编译但不生成可执行文件
$ cargo check 

// 为发布而构建
cargo build --release
```


4. 相关实例

```rust
use std::io; //使用use 语句将该类型显式引入作用域

fn main() {
    println!("Hello, world!");
    println!("Guess Number");
    // 存储用户输入
    let mut guess = String::new();
    // stdin函数返回 的一个实例std::io::Stdin，它是一种表示终端标准输入句柄的类型。
    // read_line 表示读取一行到缓冲区guess
    // expect 表示异常处理
    io::stdin()
        .read_line(&mut guess)
        .expect("faild");
    
    println!("You guessed: {}", guess);
}
```

read_line函数源码如下：
```rust
    #[stable(feature = "rust1", since = "1.0.0")]
    pub fn read_line(&self, buf: &mut String) -> io::Result<usize> {
        self.lock().read_line(buf)
    }
```

read_line将用户输入的内容放入我们传递给它的字符串中，但它也返回一个值Result，表示函数运行结果，这个模块有不同的实现，例如`io::Result`。Result类型是enum，通常被称为枚举。枚举是一种可以具有一组固定值的类型，这些值称为枚举的变体。对于Result，变体是Ok或Err。所述Ok变体指示操作是成功的，并且内部Ok是成功生成值。该Err变种意味着操作失败，并Err包含有关操作如何或为何失败的信息。

如果您不调用expect，程序将编译，但会收到警告。


# 导读

> [Cygwin安装新的软件或者程序](https://www.zhangtiefei.cn/211.html)
[Windows 下不污染环境安装 Rust 编译器](https://icedream2linxi.github.io/blog/2018/09/29/Windows%E4%B8%8B%E4%B8%8D%E6%B1%A1%E6%9F%93%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85Rust%E7%BC%96%E8%AF%91%E5%99%A8)