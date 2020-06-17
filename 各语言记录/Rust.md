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
* VS Code插件
    - 安装Rust
    - 会提示安装 RLS(Rust Language Server)

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

* Cargo.lock 文件
    - Cargo 有一个机制来确保任何人在任何时候重新构建代码，都会产生相同的结果：
        + Cargo 只会使用你指定的依赖版本，除非你又手动指定了别的
    - 当第一次构建项目时，Cargo 计算出所有符合要求的依赖版本并写入 Cargo.lock 文件
    - 当将来构建项目时，Cargo会发现Cargo.lock已存在并使用其中指定的版本，而不是再次计算所有的版本。这使得你拥有了一个自动化的可重现的构建。
        + 例如，如果下周 rand crate 的 0.5.6版本出来了，它修复了一个重要的bug，同时也含有一个会破坏代码运行的缺陷，这时会发生什么呢？
        + 答案是项目会持续使用 0.5.5 直到你显式升级(因为有Cargo.lock文件)
    - 当你 确实 需要升级 crate 时，Cargo 提供了另一个命令，`cargo update`，它会忽略 Cargo.lock 文件，并计算出所有符合 Cargo.toml 声明的最新版本。
        + 如果成功了，Cargo 会把这些版本写入 Cargo.lock 文件
        + 不过，Cargo 默认只会寻找大于 0.5.5 而小于 0.6.0 的版本。如果 rand crate 发布了两个新版本，0.5.6 和 0.6.0，在运行 cargo update 时会更新到0.5.6
        + 如果想要使用 0.6.0 版本的 rand 或是任何 0.6.x 系列的版本，必须像这样更新 Cargo.toml 文件：`rand = "0.6.0"` ([dependencies]下面)
            * 下一次运行 `cargo build` 时，Cargo 会从 registry 更新可用的 crate，并根据你指定的新版本重新计算。
            * 通过 Cargo 复用库文件非常容易，因此 Rustacean 能够编写出由很多包组装而成的更轻巧的项目

### 练习demo 猜猜看

* [demo子链接](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html)
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
* 使用crate来增加更多功能
    - crate 是一个 Rust 代码包
    - Rust 标准库中尚未包含随机数功能，然而，Rust 团队还是提供了一个 rand crate
    - 在我们使用 `rand` 编写代码之前，需要修改 Cargo.toml 文件，引入一个 rand 依赖
        + 在底部的 [dependencies] 片段标题之下添加：`rand = "0.5.5"`
        + Cargo 理解语义化版本（Semantic Versioning）（有时也称为 SemVer），这是一种定义版本号的标准。
        + `0.5.5` 事实上是 `^0.5.5` 的简写，它表示 “任何与 0.5.5 版本公有 API 相兼容的版本”
        + Cargo 从 registry 上获取所有包的最新版本信息，这是一份来自 `Crates.io` 的数据拷贝。[Crates.io](https://crates.io/) 是 Rust 生态环境中的开发者们向他人贡献 Rust 开源项目的地方
    - 不修改任何代码，构建项目`cargo build`
        + 更新完 registry 后，Cargo 会检查 [dependencies] 片段并下载缺失的 crate
        + 本例虽然只声明了 rand 一个依赖，然而 Cargo 还是额外获取了 libc(libc v0.2.71) 和 `rand_core(rand_core v0.4.2)` 的拷贝，因为 rand 依赖 libc 来正常工作。下载完成后，Rust 编译依赖，然后使用这些依赖编译项目
    - `cargo doc --open` 可以构建所有本地依赖提供的文档(crate 的使用说明位于其文档中)，并在浏览器中打开
        + 例如，假设你对 `rand` crate中的其他功能感兴趣，你可以运行 `cargo doc --open`并点击左侧导航栏中的rand
        + 类似于Go中的`godoc`
    - 使用`std::cmp::Ordering`来进行比较
        + `use std::cmp::Ordering;` 从标准库引入了一个叫做 `std::cmp::Ordering` 的类型。同 Result 一样， `Ordering`也是一个枚举，不过它的成员是`Less`、`Greater` 和 `Equal`
        + 使用一个`match`表达式，对两个数调用`cmp`返回的`Ordering`枚举值进行判断，进行分支处理
            * 一个 `match` 表达式由 `分支`（arms） 构成
            * 一个分支包含一个 `模式`（pattern）和 表达式开头的值 与 分支模式 相匹配时应该执行的代码
            * Rust 获取提供给 `match` 的值并挨个检查每个分支的模式
            * match 结构和模式是 Rust 中强大的功能，它体现了代码可能遇到的多种情形，并帮助你确保没有遗漏处理
    - 根据string变量获取数字类型的值
        + `let guess: u32 = guess.trim().parse().expect("Please type a number!");`
            * guess原来的定义是String：`let mut guess = String::new();`
            * String 实例的 `trim` 方法会去除字符串开头和结尾的空白字符
            * 字符串的 `parse` 方法 将字符串解析成数字
            * parse 方法返回一个 Result 类型，用 `expect` 方法处理

* 完整程序：

```rust
// src/main.rs
use std::io;
use rand::Rng;
use std::cmp::Ordering;

fn main() {
    println!("Guess the number!");

    // Rust 默认使用 i32
    let secret_number = rand::thread_rng().gen_range(1,101);
    println!("secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");
        let mut guess = String::new();
        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        // 对上面的guess变量进行了隐藏，通过let guess:u32告诉parse解析具体数字类型
        // let guess:u32 = match guess.trim().parse().expect("Please type a number!);
        let guess:u32 = match guess.trim().parse() {
            // 能转换成功则返回一个包含数字的Ok，num1用于接收转换结果
            Ok(num1) => num1,
            Err(_) => {
                // 若此处不用trim()则回车也会算在guess变量中
                println!("guess:[{}] invalid! please type a num!", guess.trim());
                continue;
            }
        };
        println!("You guessed:{}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
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
    - Rust 是 静态类型（statically typed）语言，也就是说在编译时就必须知道所有变量的类型。
        + 根据值及其使用方式，编译器通常可以推断出我们想要用的类型。当多种类型均有可能时，必须增加类型注解
        + e.g. `let guess: u32 = "42".parse().expect("Not a number!");` 中的`: u32`
    - 标量(scalar)
        + 标量（scalar）类型代表一个单独的值
        + Rust 有四种基本的标量类型：整型、浮点型、布尔类型和字符类型
        + 整型
            * `i8/u8、i16/u16、i32/u32、i64/u64、i128/u128、isize/usize`(`isize` 和 `usize` 类型依赖运行程序的计算机架构：64 位架构上它们是 64 位的， 32 位架构上它们是 32 位的)
            * 整型字面值
                - Decimal，e.g. `98_222`
                - Hex，e.g. `0xff`
                - Octal，e.g. `0o77`
                - Binary，e.g. `0b1111_0000`
                - Byte (u8 only)，e.g. `b'A'`
                - 注意除 `byte` 以外的所有数字字面值允许使用类型后缀，例如 `57u8`，同时也允许使用 `_` 做为分隔符以方便读数，例如`1_000`
            * 如果拿不定主意，Rust 的默认类型通常就很好，数字类型默认是 `i32`：它通常是最快的，甚至在 64 位系统上也是。`isize` 或 `usize` 主要作为某些集合的索引
        + 浮点型
            * `f32` 和 `f64`，默认类型是 `f64`，因为在现代 CPU 中，它与 `f32` 速度几乎一样，不过精度更高
        + 布尔型
            * 用`bool`表示，可能值：`true` 和 `false`
        + 字符类型
            * Rust 的 `char` 类型是语言中最原生的字母类型
                - `let c = 'z';`
                - `let z = 'ℤ';`
                - `let heart_eyed_cat = '😻';`
            * Rust 的 `char` 类型的大小为**四个字节**(four bytes)，并代表了一个 `Unicode` 标量值（Unicode Scalar Value），这意味着它可以比 ASCII 表示更多内容
            * 在 Rust 中，拼音字母（Accented letters），中文、日文、韩文等字符，emoji（绘文字）以及零长度的空白字符都是有效的 char 值
    - 复合类型
        + Rust 有两个原生的复合类型：`元组`（tuple）和`数组`（array）
        + `元组`是一个将多个其他类型的值组合进一个复合类型的主要方式
            * 元组长度固定：一旦声明，其长度不会增大或缩小
            * 使用包含在圆括号中的逗号分隔的值列表来创建一个元组，元组中的每一个位置都有一个类型，而且这些不同值的类型也不必是相同的
                - e.g. `let tup: (i32, f64, u8) = (500, 6.4, 1);`
            * 为了从元组中获取单个值，可以使用模式匹配（pattern matching）来解构（destructure）元组值
                - `let tup = (500, 6.4, 1);` 先创建了一个元组并绑定到 tup 变量上
                - `let (x, y, z) = tup;` 接着使用了`let`和一个模式将 tup 分成了三个不同的变量，x、y 和 z。这叫做 **解构（destructuring）**
            * 除了使用模式匹配解构外，也可以使用点号（`.`）后跟值的索引来直接访问它们
                - 元组的第一个索引值是 0
                - 对于上面定义的元组tup，`tup.1`为6.4
        + `数组`（array）与元组不同，数组中的每个元素的类型必须相同
            * Rust 中的数组是固定长度的：一旦声明，它们的长度不能增长或缩小
            * 当你想要在栈（stack）而不是在堆（heap）上为数据分配空间（第四章将讨论栈与堆的更多内容），或者是想要确保总是有固定数量的元素时，数组非常有用
                - 但是数组并不如 `vector` 类型灵活。`vector` 类型是标准库提供的一个 允许 增长和缩小长度的类似数组的集合类型
                - 当不确定是应该使用数组还是 `vector` 的时候，你可能应该使用 `vector`
            * `let a = [1, 2, 3, 4, 5];`
            * 可以像这样编写数组的类型：在方括号中包含每个元素的类型，后跟分号，再后跟数组元素的数量
                - `let a: [i32; 5] = [1, 2, 3, 4, 5];`
                - i32 是每个元素的类型。分号之后，数字 5 表明该数组包含五个元素
            * 另一个初始化数组的语法：如果你希望创建一个每个元素都相同的数组，可以在中括号内指定其初始值，后跟分号，再后跟数组的长度
                - `let a = [3; 5];` 5个元素，值都为3
            * 访问数组元素 `a[0]`
* 函数
    - Rust 代码中的函数和变量名使用 snake case 规范风格
        + 在 snake case 中，所有字母都是小写并使用下划线分隔单词
        + e.g. `fn another_function() {}`
    - 源码中，函数可以定义在`main`函数之后，也可以定义在之前。Rust不关心函数定义于何处，只要定义了就行
    - 函数参数：
        + 在函数签名中，必须 声明每个参数的类型
        + e.g. `fn another_function(x: i32) {}`
        + e.g. `fn another_function(x: i32, y: i32) {}`
    - 语句和表达式
        + `语句`（Statements）是执行一些操作但不返回值的指令
            * 语句不返回值，因此，不能把 let 语句赋值给另一个变量
                - `let x = (let y = 6);` 是*错误*的
                - 报错信息 error: expected expression, found statement (`let`)，期望一个表达式
                - 这与其他语言不同，例如 C 和 Ruby，它们的赋值语句会返回所赋的值，如`x = y = 6`，Rust中不能这样写
        + `表达式`（Expressions）计算并产生一个值
            * `let y = 6;`中，`6`是一个表达式，而`let y = 6;`并不是
            * 函数调用是一个表达式
            * 宏调用是一个表达式
            * `{}`也是一个表达式
        + 下面的示例(表达式示例1)，注意结尾没有分号的那一行 `x+1`，与你见过的大部分代码行不同。
            * 表达式的结尾没有分号。
            * 如果在表达式的结尾加上分号，它就变成了语句，而语句不会返回值。
            * 在接下来探索具有返回值的函数和表达式时要谨记这一点
    - 具有返回值的函数
        + 我们并不对返回值命名，但要在箭头（`->`）后声明它的类型
        + 在 Rust 中，函数的返回值等同于函数体最后一个表达式的值。
            * 使用 `return` 关键字和指定值，可从函数中提前返回；
            * 但大部分函数隐式的返回最后的表达式。
            * e.g. `fn five() -> i32 { 5 }` (注意`5`后面没有分号`;`)
                - 若加了`;`，把它从表达式变成语句，我们将看到一个错误:expected i32, found ()
                - 使用空元组 `()` 表示不返回值，因为不返回值与函数定义相矛盾，从而出现一个错误
                - 在输出中，Rust 提供了一条信息，可能有助于纠正这个错误：它建议删除分号，这会修复这个错误：- help: consider removing this semicolon

```rust
// 表达式示例1
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

* 控制流
    - `if`表达式(注意是表达式，不是语句)
        + `if number < 5 {}`，`else`，`else if`
        + 代码中的条件必须是`bool`值。如果条件不是`bool`值，我们将得到一个错误
            * e.g. `let num=3; if num {}`会报错：note: expected type `bool`
        + Rust 并不会尝试自动地将非布尔值转换为布尔值。必须总是显式地使用布尔值作为 if 的条件
        + 使用过多的 `else if` 表达式会使代码显得杂乱无章，所以如果有多于一个 else if 表达式，最好重构代码
            * 使用`match`
        + 因为 `if` 是一个表达式，我们可以在 `let` 语句的右侧使用它
            * `let number= if condition1 {5} else {6};`
            * if 的每个分支的可能的返回值都必须是相同类型
                - Rust 需要在编译时就确切的知道 number 变量的类型，这样它就可以在编译时验证在每处使用的 number 变量的类型是有效的
                - Rust 并不能够在 number 的类型只能在运行时确定的情况下工作；
                - 这样会使编译器变得更复杂而且只能为代码提供更少的保障，因为它不得不记录所有变量的多种可能的类型。
    - 循环
        + Rust 有三种循环：`loop`、`while` 和 `for`
        + 可以用`break` 表达式停止循环，并返回返回值，e.g. `break x+2;`
        + 使用`for`遍历集合
            * `let a = [10, 20, 30, 40, 50];`
            * `for element in a.iter() { println!("value:{}", element);}`
        + `for number in (1..4).rev() {print!("{}!", number);}`
            * `rev()`用来反转range，上面打印 3! 2! 1! (注意不包含4)

### 所有权

* `所有权`（系统）是 Rust 最为与众不同的特性，它让 Rust 无需垃圾回收（garbage collector）即可保障内存安全
* Rust 的核心功能（之一）是 所有权（ownership）
    - 一些语言中具有垃圾回收机制，在程序运行时不断地寻找不再使用的内存； Java、Go
    - 在另一些语言中，程序员必须亲自分配和释放内存 C/C++
    - Rust 则选择了第三种方式：通过所有权系统管理内存，编译器在编译时会根据一系列的规则进行检查
        + 在运行时，所有权系统的任何功能都不会减慢程序
* 所有权的规则
    - Rust 中的每一个值都有一个被称为其 `所有者`（owner）的变量。
    - 值有且只有一个所有者。
    - 当所有者（变量）离开作用域，这个值将被丢弃。
* `String`
    - 这个类型被分配到堆上
    - 可以使用`from`函数基于字符串字面值来创建`String`：
        + `let s = String::from("hello");`
    - `s.push_str(", world!");`
        + `push_str()` 在字符串后追加字面值
        + 将打印 `hello, world!`
* 内存和分配
    - 就字符串字面值来说，我们在编译时就知道其内容，所以文本被直接硬编码进最终的可执行文件中
    - `String::from`向操作系统请求所需的内存，内存在拥有它的变量离开作用域后就被自动释放
    - 当变量离开作用域，Rust 为我们调用一个特殊的函数。这个函数叫做 `drop` (类似于C++中的RAII)
    - 移动
        + `String`由三部分组成：一个指向存放字符串内容内存的指针，一个长度，和一个容量(类似于Go里的slice)
        + `let s1 = String::from("hello");`， `let s2 = s1;`
            * 将 s1 赋值给 s2，String 的数据被复制了，这意味着我们从栈上拷贝了它的指针、长度和容量
            * 并没有复制指针指向的堆上数据
        + 不同于其他语言(如C++)中的`浅拷贝`(shallow copy)和`深拷贝`(deep copy)， Rust 同时使第一个变量无效了，这个操作被称为 `移动`(move)，而不是浅拷贝
            * 上面的例子可以解读为 s1 被 移动 到了 s2 中
            * 这样就不会出现：当 s2 和 s1 都离开作用域时，二次释放（double free）的错误
    - 克隆
        + 如果确实需要深度复制String中堆上的数据，而不仅仅是栈上的数据，可以使用`clone`函数
            * `let s1 = String::from("hello");`
            * `let s2 = s1.clone();`
    - 拷贝
        + 只在栈上的数据
        + Rust 有一个叫做 `Copy trait` 的特殊注解
            * 可以用在类似整型这样的存储在栈上的类型上
            * 如果一个类型拥有 Copy trait，一个旧的变量在将其赋值给其他变量后仍然可用
                - e.g. `let x = 5;` `let y = x;`，x仍然可用，而不像String一样
            * Rust 不允许自身或其任何部分实现了 `Drop trait` 的类型使用 `Copy trait`
            * 什么类型是`Copy`的可以查看给定类型的文档来确认。不过作为一个通用的规则，任何简单标量值的组合可以是`Copy`的，不需要分配内存或某种形式资源的类型是`Copy`的
                - 所有整数类型
                - 布尔类型
                - 所有浮点数类型
                - 字符类型
                - 元组，当且仅当其包含的类型也都是 Copy 的时候，如`(i32, i32)`，而`(i32, String)`就不是
* 所有权和函数
    - 对于函数调用：向函数传递值可能会移动或者复制，就像赋值语句一样
        + `let s = String::from("hello");`  // s 进入作用域
        + `takes_ownership(s);` // s 的值移动到函数里，在该函数之后访问s就不再有效(所有权转移)
        + 在函数中，最后会移出作用域并调用`drop`释放内存
    - 对于返回值：返回值也可以转移所有权
        + `let s1 = gives_ownership();` gives_ownership 将返回值移给 s1
    - 变量的所有权总是遵循相同的模式：将值赋给另一个变量时移动它
        + 当持有堆中数据值的变量离开作用域时，其值将通过 drop 被清理掉，除非数据被移动为另一个变量所有
    - 如果我们想要函数使用一个值但不获取所有权
        + Rust 对此提供了一个功能，叫做 `引用`（references）
* 引用和借用
    - `&` 符号就是 `引用`，它们允许你使用值但不获取其所有权
        + e.g. `let s1 = String::from("hello");`，`let len = calculate_length(&s1);`
    - 将获取引用作为函数参数称为 `借用`（borrowing）
        + **注意：**（默认）不允许修改引用的值
    - 可变引用
        + 创建一个可变引用 `&mut s` 和接受一个可变引用
            * `let mut s = String::from("hello");`
            * `change(&mut s);`
            * 函数定义：`fn change(some_string: &mut String) {}`
            * 注意定义String时也需要定义为`mut`，形参加`&mut`，实参也加`&mut`
        + 不过可变引用有一个很大的限制：在特定作用域中的特定数据有且只有一个可变引用
            * 这个限制的好处是 Rust 可以在编译时就避免数据竞争（data race）
            * 数据竞争会导致未定义行为，难以在运行时追踪，并且难以诊断和修复；
            * Rust 避免了这种情况的发生，因为它甚至不会编译存在数据竞争的代码
    - 不能在拥有不可变引用的同时拥有可变引用(`&mut`)
        + 场景
            * `let mut s3 = String::from("xyz");`，`let r1 = &mut s3;`
            * 下面的表达式只能同时用一个
                - `println!("s3:{}", s3);` 非mut借用，^^ immutable borrow occurs here
                - `println!("r1:{}", r1);` mut借用， -- mutable borrow later used here
        + 同时拥有多个不可变引用是可以的
        + 注意一个`引用的作用域`从声明的地方开始一直持续到最后一次使用为止
    - 悬垂引用(dangling references)
        + 在 Rust中编译器确保引用永远也不会变成悬垂状态：
        + C/C++中悬垂指针（dangling pointer）：指向已经销毁的对象或已经回收的地址
        + Rust中当你拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域
        + 当尝试创建一个悬垂引用时，Rust 会通过一个编译时错误来避免
        + e.g.
            * `fn dangle() -> &String { let s = String::from("hello"); return &s}` 返回字符串 s 的引用
                - `}`之后，s 离开作用域并被丢弃。其内存被释放。这意味着这个引用会指向一个无效的 String，这可不对！Rust 不会允许我们这么做
            * 这里的解决方法是直接返回 String：
                - `fn no_dangle() -> String { let s = String::from("hello"); s}`
                - 这样就没有任何错误了。所有权被移动出去，所以没有值被释放
    - 引用的规则
        + 在任意给定时间，要么 只能有一个可变引用，要么 只能有多个不可变引用
        + 引用必须总是有效的

```rust
// 可以使用大括号来创建一个新的作用域，以允许拥有多个可变引用，只是不能 同时 拥有
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用

let r2 = &mut s;
```

* 字符串slice
    - `字符串 slice`（string slice）是 String 中一部分值的引用
        + `let s = String::from("hello world");`
        + `let hello = &s[0..5];`
        + `let world = &s[6..11];`