# Rust

* [官网](https://www.rust-lang.org/zh-CN/)
* 为什么选择 Rust?
    - 高性能
        + Rust 速度惊人且内存利用率极高
        + 由于没有运行时和垃圾回收，它能够胜任对性能要求特别高的服务，可以在嵌入式设备上运行，还能轻松和其他语言集成
    - 可靠性
        + Rust 丰富的类型系统和所有权模型保证了内存安全和线程安全，让您在编译期就能够消除各种各样的错误。
    - 生产力
        + Rust 拥有出色的文档、友好的编译器和清晰的错误提示信息， 还集成了一流的工具——包管理器和构建工具， 智能地自动补全和类型检验的多编辑器支持， 以及自动格式化代码等等

## 《Rust 程序设计语言》

* 《Rust 程序设计语言》
    - [The Rust Programming Language](https://doc.rust-lang.org/book/title-page.html)
    - [Rust 程序设计语言 简体中文版](https://kaisery.github.io/trpl-zh-cn/#rust-%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E8%AF%AD%E8%A8%80)
* 介绍
    - 在 Rust 中，编译器充当了守门员的角色，它拒绝编译存在这些难以捕获的 bug 的代码，这其中包括并发 bug
    - 开发工具
        + Cargo
            * 内置的依赖管理器和构建工具，它能轻松增加、编译和管理依赖，并使其在 Rust 生态系统中保持一致
            * go: go module
        + Rustfmt
            * 确保开发者遵循一致的代码风格
            * go: gofmt
        + Rust Language Server
            * 为集成开发环境（IDE）提供了强大的代码补全和内联错误信息功能
            * go: gopls
    - 学习 Rust 的过程中一个重要的部分是学习如何阅读编译器提供的错误信息：它们会指导你编写出能工作的代码
* 安装
    - 通过 `rustup` 下载 Rust，这是一个管理 Rust 版本和相关工具的命令行工具
    - Linux/Mac：`curl https://sh.rustup.rs -sSf | sh`
        + 会下载官方Rust编译器和包管理工具Cargo
        + 会添加`cargo`, `rustc`, `rustup` 和其他命令到Cargo的bin目录
            * 可通过`CARGO_HOME`修改目录，我的环境CentOS安装默认放在:`/root/.cargo/bin`
        + Rustup的元数据和工具链安装在`/root/.rustup`，可通过`RUSTUP_HOME`环境变量修改
        + 自己CentOS的环境安装的版本(20200611)：`rustc 1.44.0`
            * 安装后，添加一下`$HOME/.cargo/bin`路径到`PATH`环境变量
            * 另外，你需要一个某种类型的链接器（linker）。很有可能已经安装，不过当你尝试编译 Rust 程序时，却有错误指出无法执行链接器，这意味着你的系统上没有安装链接器，你需要自行安装一个
                - C 编译器通常带有正确的链接器
            * 检查是否成功安装`rustc --version`
                - 结果：rustc 1.44.0 (49cae5576 2020-06-01)
    - Windows: 下载安装包，参考[install](https://www.rust-lang.org/tools/install)
        + `cargo --version`查看版本，cargo 1.44.0 (05d080faa 2020-05-06)
        + 不用再手动添加环境变量，默认在`用户目录/.cargo/bin/`
    - 更新
        + 通过 rustup 安装了 Rust 之后，很容易更新到最新版本，`rustup update`
    - 若要卸载，可随时通过rustup的自卸载功能卸载
        + `rustup self uninstall`
    - 文档
        + 安装程序也自带一份文档的本地拷贝，可以离线阅读。
        + 运行 `rustup doc`在浏览器中查看本地文档(CentOS环境要装可视化工具)
* Hello World
    - 新建一个源文件，命名为 `main.rs`。Rust 源文件总是以 `.rs` 扩展名结尾
    - 如果文件名包含多个单词，使用下划线分隔它们。例如命名为 `hello_world.rs`，而不是 helloworld.rs
    - 一般来说，将左花括号与函数声明置于同一行并以空格分隔，是良好的代码风格
    - 这里有四个重要的细节需要注意
        + 首先 Rust 的缩进风格使用 4 个空格，而不是 1 个制表符（tab）
        + 第二，`println!` 调用了一个 Rust 宏（macro）
            * 如果是调用函数，则应输入 `println`（没有`!`）
            * 当看到符号 `!` 的时候，就意味着调用的是宏而不是普通函数
        + 第三，"Hello, world!" 是一个字符串
        + 第四，该行以分号结尾（;），这代表一个表达式的结束和下一个表达式的开始
            * 大部分 Rust 代码行以分号结尾
    - 编译
        + 编译和运行是彼此独立的步骤
        + 编译：`rustc main.rs`
            * 编译后生成了两个文件(Windows下)：`main.exe`(可执行文件) 和 `main.pdb`(包含调试信息)
        + Rust 是一种 预编译静态类型（ahead-of-time compiled）语言
            * 这意味着你可以编译程序，并将可执行文件送给其他人，他们甚至不需要安装 Rust 就可以运行。
            * 如果你给他人一个 .rb、.py 或 .js 文件，他们需要先分别安装 Ruby，Python，JavaScript 实现（运行时环境，VM）。不过在这些语言中，只需要一句命令就可以编译和运行程序。这一切都是语言设计上的权衡取舍。
    - 仅仅使用 `rustc` 编译简单程序是没问题的，不过随着项目的增长，你可能需要管理你项目的方方面面，并让代码易于分享。接下来，我们要介绍一个叫做 `Cargo` 的工具，它会帮助你编写真实世界中的 Rust 程序

```rust
// main.rs

fn main() {
    println!("Hello, world!");
}
```

* Cargo
    - Cargo 是 Rust 的构建系统和包管理器
        + 它可以为你处理很多任务，比如构建代码、下载依赖库并编译这些库
        + 如果使用 “安装” 部分介绍的官方安装包的话，则自带了 Cargo
        + `cargo --version` 查看是否安装
            * Windows: cargo 1.44.0 (05d080faa 2020-05-06)
    - 使用Cargo创建项目
        + `cargo new hello_cargo`
            * 会创建一个hello_cargo目录，包含：
            * 1. 一个src目录，其中有一个main.rs文件(只执行一句`println!("Hello World!")`)
            * 2. .gitignore文件
            * 3. Cargo.toml文件
                - 这个文件使用 TOML (Tom's Obvious, Minimal Language) 格式，这是 Cargo 配置文件的格式

* TOML格式
    - `[package]` 是一个片段（section）标题，表明下面的语句用来配置一个包

```
# 生成的Cargo.toml文件

[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["xxxx <xxxxxxxx@mail.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```
