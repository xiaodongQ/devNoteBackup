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

### 入门指南

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
        + 执行 `cargo new hello_cargo`，会创建一个hello_cargo目录，包含：
            * 1. 一个src目录，其中有一个main.rs文件(只执行一句`println!("Hello World!")`)
            * 2. .gitignore文件(在有git的目录中new则不会生成git相关文件)
            * 3. Cargo.toml文件
                - 这个文件使用 TOML (Tom's Obvious, Minimal Language) 格式，这是 Cargo 配置文件的格式
                - `[package]` 是一个片段（section）标题，表明下面的语句用来配置一个包
                    + 下面四行设置了 Cargo 编译程序所需的配置：项目的名称、版本、作者以及要使用的 Rust 版本
                    + Cargo 从环境中获取你的名字和 email 信息，若不正确可进行修改
                - `[dependencies]` 罗列项目依赖的片段的开始
                    + 在 Rust 中，代码包被称为 `crates`
                    + 这个项目并不需要其他的 crate
        + Cargo 期望源文件存放在 src 目录中。
            * 项目根目录只存放 README、license 信息、配置文件和其他跟代码无关的文件
        + 如果中间或者结果文件不用加到git中，可以在.gitignore中新增过滤：
            * `**/target`
            * `**/Cargo.lock`
    - 构建
        + 在 hello_cargo 目录下(而不是src)，执行`cargo build`
            * 这个命令会创建一个可执行文件 `target/debug/hello_cargo`，可以cd到目录中`./`执行
            * 也可以使用 `cargo run` 在一个命令中*同时编译并运行*生成的可执行文件
                - 若本次未修改源文件则直接执行二进制文件，
                - 如果修改了源文件的话，Cargo 会在运行之前重新构建项目
        + 首次运行 `cargo build` 时，也会使 Cargo 在项目根目录创建一个新文件：`Cargo.lock`
            * 这个文件记录项目依赖的实际版本
        + `cargo check` 快速检查代码确保其可以编译，但并不产生可执行文件
            * 通常 `cargo check` 要比 `cargo build` 快得多，因为它省略了生成可执行文件的步骤
            * 如果你在编写代码时持续的进行检查，`cargo check` 会加速开发，当准备好使用可执行文件时才运行 `cargo build`
    - 发布(release)构建
        + 当项目最终准备好发布时，可以使用 `cargo build --release` 来优化编译项目
            * 这会在 `target/release` 而不是 `target/debug` 下生成可执行文件
            * 这些优化可以让 Rust 代码运行的更快，不过启用这些优化也需要消耗更长的编译时间
        + 如果你在测试代码的运行时间，请确保运行 `cargo build --release` 并使用 `target/release` 下的可执行文件进行测试
            * `cargo run --release` 运行release版本

* 生成的TOML格式文件：Cargo.toml

```sh
# 生成的Cargo.toml文件

[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["xxxx <xxxxxxxx@mail.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

### 练习demo 猜猜看

* 默认情况下，Rust 将 prelude 模块中少量的类型引入到每个程序的作用域中。如果需要的类型不在 prelude 中，你必须使用 use 语句显式地将其引入作用域
* `String` 是一个标准库提供的字符串类型，它是 UTF-8 编码的可增长文本块
* 如果程序的开头没有 `use std::io` 这一行，可以把函数调用写成 `std::io::stdin`
    - stdin 函数返回一个 `std::io::Stdin` 的实例，这代表终端标准输入句柄的类型
* `.read_line(&mut guess)` 调用 read_line 方法从标准输入句柄获取用户输入
    - `read_line`的工作是，无论用户在标准输入中键入什么内容，都将其存入一个字符串中，因此它需要字符串作为参数
    - 这个字符串参数应该是可变的，以便 `read_line` 将用户输入附加上去
    - `&` 表示这个参数是一个 引用（reference），它允许多处代码访问同一处数据，而无需在内存中多次拷贝
* `.expect("Failed to read line");`
    - 虽然这是单独一行代码，但它是一个逻辑行（虽然换行了但仍是一个语句）的第一部分
    - 也可以写成一行：`io::stdin().read_line(&mut guess).expect("Failed to read line");`
        + 通过换行加缩进来把长行拆开是明智的
        + 过长的行难以阅读，所以最好拆开来写，两个方法调用占两行
    - `read_line`有个返回值，类型为`io::Result`
        + Result 类型是 枚举（enumerations），通常也写作 `enums`
        + Result 的成员是 `Ok` 和 `Err`，Err 成员则意味着操作失败，并且包含失败的前因后果
    - `io::Result` 的实例拥有 `expect` 方法
        + 如果 io::Result 实例的值是 Err，expect 会导致程序崩溃，并显示当做参数传递给 expect 的信息
        + 如果不调用 `expect`，程序也能编译，不过会出现一个警告：警告我们没有使用 read_line 的返回值 Result，说明有一个可能的错误没有处理
            * 消除警告的正确做法是实际编写错误处理代码
* 使用crate
    - Rust 标准库中尚未包含随机数功能，然而，Rust 团队还是提供了一个 rand crate
    - crate 是一个 Rust 代码包
    - 在我们使用 `rand` 编写代码之前，需要修改 Cargo.toml 文件，引入一个 rand 依赖
        + 在底部的 [dependencies] 片段标题之下添加：`rand = "0.5.5"`
        + Cargo 理解语义化版本（Semantic Versioning）（有时也称为 SemVer），这是一种定义版本号的标准。
        + `0.5.5` 事实上是 `^0.5.5` 的简写，它表示 “任何与 0.5.5 版本公有 API 相兼容的版本”
        + Cargo 从 registry 上获取所有包的最新版本信息，这是一份来自 `Crates.io` 的数据拷贝。[Crates.io](https://crates.io/) 是 Rust 生态环境中的开发者们向他人贡献 Rust 开源项目的地方
    - 不修改任何代码，构建项目`cargo build`

```rust
use std::io;

fn main() {
    println!("Guess the number!");
    println!("Please input your guess.");

    let mut guess = String::new();
    io:stdin().read_line(&mut guess)
        .expect("Failed to read line");
    // {}是占位符
    println!("You guessed:{}", guess);
}
```

### 常见编程概念

* 变量与可变性
    - 变量默认是不可改变的（immutable），当变量不可变时，一旦值被绑定一个名称上，你就不能改变这个值
        + e.g. `let x=5; x=6`，编译会报错：error[E0384]: cannot assign twice to immutable variable `x`
        + `rustc --explain E0384` 可以查看该报错类型的说明
    - Rust 编译器保证，如果声明一个值不会变，它就真的不会变。这意味着当阅读和编写代码时，不需要追踪一个值如何和在哪可能会被改变，从而使得代码易于推导。
    - 可以在变量名之前加 `mut` 来使其可变
        + `let mut x=5;`
        + 除了允许改变值外，也表明了其他代码将会改变这个变量值的意图
* 常量
    - 声明常量：`const MAX_POINTS: u32 = 100_000;`
        + Rust 常量的命名规范是使用下划线分隔的大写字母单词，并且可以在数字字面值中插入下划线来提升可读性
    - 在声明它的作用域之中，常量在整个程序生命周期中都有效，这使得常量可以作为多处代码使用的全局范围的值
    - 常量和变量的区别
        + 首先，不允许对常量使用 `mut`。常量不光默认不能变，它总是不能变
            * 声明常量使用 `const` 关键字而不是 `let`，并且**必须**注明值的类型
        + 常量可以在任何作用域中声明，包括全局作用域，这在一个值需要被很多部分的代码用到时很有用
        + 最后一个区别是，常量只能被设置为常量表达式，而不能是函数调用的结果，或任何其他只能在运行时计算出的值。
* 隐藏
    - 定义一个与之前变量同名的新变量，而新变量会 隐藏 之前的变量
    - 这意味着使用这个变量时会看到第二个值。可以用相同变量名称来隐藏一个变量，以及重复使用 let 关键字来多次隐藏
        + `let x = 5; let x = x + 1; let x = x * 2;`
        + 首先将 x 绑定到值 5 上；接着通过 let x = 隐藏 x，获取初始值并加 1，这样 x 的值就变成 6 了；第三个 let 语句也隐藏了 x，将之前的值乘以 2，x 最终的值是 12。
    - 隐藏与将变量标记为 `mut` 是有区别的
        + 对非`mut`变量重新赋值时，如果没有使用 `let` 关键字，就会导致编译时错误
        + 当再次使用 `let` 时，实际上创建了一个新变量，我们可以改变值的类型，但复用这个名字
* 数据类型
    - 两类数据类型子集：`标量`（scalar）和`复合`（compound）