# 1. Rust relearn

之前学过基本使用，间隔时间有点长且实践较少，遗忘了很多，重新学习一下。

实践环境说明：

* Mac：原来是`rustc 1.44.0`，升级一下：`rustup update`，升级后当前版本为`rustc 1.81.0`（2024-09-04）
* CentOS8：之前重装过系统，这次安装下rust
    * `curl https://sh.rustup.rs -sSf | sh`，安装的版本也是当前最新的stable版本：`rust version 1.81.0 (eeb90cda1 2024-09-04)`

## 1.1. 学习参考资料

**网站：**

* [官网](https://www.rust-lang.org/zh-CN/)
* [官方GitHub](https://github.com/rust-lang)
* [学习Rust](https://www.rust-lang.org/zh-CN/learn) 里面推荐了一些开源学习资料，包括下面的一些开源书籍
    * 核心文档
        * [标准库](https://doc.rust-lang.org/std/index.html)：详尽的 Rust 标准库 API 手册
        * [Rust 版本指南](https://doc.rust-lang.org/edition-guide/index.html)：介绍各版本特性及兼容性说明
        * [Cargo手册](https://doc.rust-lang.org/cargo/index.html)：Rust包管理器使用指南
        * [rustdoc手册](https://doc.rust-lang.org/rustdoc/index.html)：编写规范的Rust项目文档
        * [rustc手册](https://doc.rust-lang.org/rustc/index.html)：Rust编译器使用指南，理解各选项含义
        * [编译错误索引表](https://doc.rust-lang.org/error_codes/error-index.html)：可能会遇到的编译错误

一、基础内容

* [The Rust Programming Language](https://doc.rust-lang.org/book/)
    * 中文翻译版：[Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/)
        * `Rustacean`中文意思为 Rust 开发者，Rust 用户，Rust 爱好者。注意，Rust开发者**不要**写成 `Ruster`，另外 Rustacean 一般第一个字母为大写形式，就和 Rust 一样，[参考](https://rustwiki.org/wiki/translate/other-translation/#the-rust-programing-language)
    * 《Rust 程序设计语言》被亲切地称为“圣经”，其中文出版书名为《Rust 权威指南》
* [Rust开源教程：Rust Course](https://course.rs/about-book.html)
* [Rust Cookbook](https://rust-lang-nursery.github.io/rust-cookbook/)
    * [中文版](https://rustwiki.org/zh-CN/rust-cookbook/)

* 极客时间：[陈天 · Rust 编程第一课](https://time.geekbang.org/column/article/408400)
* [rust常用的crates](https://github.com/daheige/rs-cookbook?tab=readme-ov-file#rust%E5%B8%B8%E7%94%A8%E7%9A%84crates)

二、进阶

* [The Rustonomicon](https://doc.rust-lang.org/nomicon/)
    * 中文版：[Rust 秘典（死灵书）](https://nomicon.purewhite.io/intro.html)
    * 《Rust 秘典》是 Unsafe Rust 的黑魔法指南。它有时被称作“死灵书”
* [Rust官方RFC文档](https://github.com/rust-lang/rfcs)
* [Rust设计模式](https://rust-unofficial.github.io/patterns/)
    * [中文版](https://chuxiuhong.com/chuxiuhong-rust-patterns-zh/)

## 1.2. 语言基础

前置说明：

* 1、基于《Rust 程序设计语言》（中文版：[Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/)）大概过一下，之前的初步学习笔记在：[Rust.md](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%90%84%E5%88%86%E7%B1%BB%E8%AE%B0%E5%BD%95/Rust/Rust.md)
* 2、上述资料大概看了一下，[Rust开源教程：Rust Course](https://course.rs/about-book.html) 的内容比较贴合自己当前的偏好，先基于该教程学习梳理，其他作为辅助。
* 3、代码练习还是复用之前的仓库：[rust_learning](https://github.com/xiaodongQ/rust_learning)
* 4、VSCode插件（《Rust编程第一课》中推荐的插件）
    + `rust-analyzer`：它会实时编译和分析你的 Rust 代码，提示代码中的错误，并对类型进行标注。你也可以使用官方的 Rust 插件取代。
        + 官方的`Rust`插件已经不维护了
    + ~~`crates`~~ `Dependi`：帮助你分析当前项目的依赖是否是最新的版本。
        + crates插件已经不维护了，主页中推荐切换为`Dependi`，支持多种语言的依赖管理：Rust, Go, JavaScript, TypeScript, Python and PHP
    + ~~`better toml`~~ `Even Better TOML`：Rust 使用 toml 做项目的配置管理。该插件可以帮你语法高亮，并展示 toml 文件中的错误。
        + better toml插件也不维护了，其主页推荐切换为`Even Better TOML`
    + 其他插件，暂不安装
        + `rust syntax`：为代码提供语法高亮（有必要性？前面插件会提供语法高亮）
        + `rust test lens`：rust test lens：可以帮你快速运行某个 Rust 测试（也不维护了）
        + `Tabnine`：基于 AI 的自动补全，可以帮助你更快地撰写代码（暂时用的`CodeGeeX`，当作`Copilot`平替）

### 1.2.1. 基础语法

* 变量绑定：`let a = "hello world"`，其他语言里叫赋值
    * Rust 的变量在默认情况下是不可变的，通过 `mut` 关键字让变量变为可变
    * 创建了一个变量却不在任何地方使用它，Rust通常会给一个警告，可以用下划线作为变量名的开头来消除警告：`let _x = 5;`

* 整型：`i8`/`u8`、`i16`/`u16`、`i32`/`u32`、`i64`/`u64`、`i128`/`u128`、`isize`/`usize`
    * 不指定则默认`i32`
* 浮点型：`f32`/`f64`
* 序列(Range)：`1..5`表示`1, 2, 3, 4`，`1..=5`表示`1, 2, 3, 4, 5`
    * `for i in 1..5`：`i`是`1..5`的迭代器
    * 序列只允许用于`数字`或`字符`类型
* 单元类型：只有一个，唯一的值就是 `()`，是一个零长度的元组
    * 例如常见的 `println!()` 的返回值也是单元类型 `()`
    * 比如，可以用 `()` 作为 `map` 的值，表示我们不关注具体的值，只关注 `key`。 这种用法和 Go 语言的 `struct{}` 类似，可以作为一个值用来占位，但是完全不占用任何内存。
* 语句（`statement`）和表达式（`expression`）
    * 语句会执行一些操作但是不会返回一个值，而表达式会在求值后**总有一个返回值**
    * 表达式不能包含分号，否则就变成一条语句
    * 表达式如果不返回任何值，会隐式地返回一个 `()` 单元类型

关于语句和表达式，需要能区分开。示例：使用一个语句块表达式将值赋给`y`变量

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

**注意**：`x + 1`不能以分号结尾，否则就会从表达式变成语句， **表达式不能包含分号**。这一点非常重要，一旦在表达式后加上分号，它就会变成一条语句，再也不会返回一个值。

* 函数
    * 函数命名规则：蛇形命名法(snake case)，即小写字母，单词之间用**下划线**连接，比如`fn add_two() -> i32 {}`
    * 函数的位置可以随便放，Rust 不关心我们在哪里定义了函数，只要有定义即可（不像C/C++，需要声明在前）
    * 每个函数参数都需要标注类型
    * 返回值：
        * 表达一个函数没有返回值：`fn test(i: i32) {}` 或者 `fn test(i: i32) -> () {}`，返回值类型都是单元类型 `()`
        * `fn dead_end() -> ! { xxx }`，`!` 是一个特殊的类型，表示函数永不返回(diverge function)，这种语法往往用做会导致程序崩溃的函数：
            * `fn dead_end() -> ! { panic!("This call never returns") }`
            * `fn forever() -> ! { loop {} }`

示例：

```rust
fn add(i: i32, j: i32) -> i32 {
    // 这里没有分号，是一个表达式，返回值就是 i + j；也可以用return语句显式返回
    i + j
}
```

Rust函数构成示意图：  
![Rust函数构成示意图](/images/2024-09-18-rust-function.png)

在这里进行找错练习，直接网页上可修改运行：[Rust By Practice](https://practice-zh.course.rs/basic-types/functions.html)

### 1.2.2. 流程控制

* 分支控制（`if`/`else if`/`else`）
    * if 语句块是表达式，因此可以赋值给变量：`let number = if condition { 5 } else { 6 };`

```rust
fn main() {
    let n = 6;

    if n % 4 == 0 {
        println!("number is divisible by 4");
    } else if n % 3 == 0 {
        println!("number is divisible by 3");
    } else if n % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

* 3种循环控制方式：`for`、`while` 和 `loop`
    * break 可以单独使用，也可以带一个返回值，有些类似 return
    * loop 是一个表达式，因此可以返回一个值
    * 相对而言，`for`比`while`和`loop`更常用，因为`for`可以遍历序列，而`while`和`loop`需要手动控制循环条件

```rust
fn test_for() {
    // 从1到5，包含5
    for i in 1..=5 {
        println!("{}", i);
    }

    // 若不需要使用控制变量，可以用 _ 来忽略变量，否则定义了不使用会有编译警告
    for _ in 1..=5 {
        println!("loop again");
    }
}

fn test_while() {
    let mut n = 0;

    while n <= 5  {
        println!("{}!", n);

        n = n + 1;
    }

    println!("test_while end！");
}

fn test_loop() {
    let mut n = 0;

    loop {
        if n > 5 {
            // 加不加分号都可以，因为loop是一个表达式，可以返回值
            break
        }
        println!("{}", n);
        n+=1;
    }

    println!("test_loop end！");
}
```

### 1.2.3. 所有权和借用

所有权（`ownership`）的规则：

* Rust 中每一个值都被一个变量所拥有，该变量被称为值的`所有者`
* 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
* 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

基本类型（比如整型）在编译时是已知大小的，会被存储在栈上，不会发生`所有权转移（transfer ownership）`。比如如下示例：

```rust
fn test_owner_ship() {
    let s = String::from("hello");
    // 所有权转移，会影响s的生命周期；若需要拷贝，则可以使用 s.clone()
    // let s2 = s;
    println!("{}", s);
    task_ownership(s);
    // 这里所有权转移到task_ownership后，已经释放了s，所以这里会报错
    // println!("{}", s);

    println!("===============");
    let n = 888;
    // 这里不影响n的所有权，基本类型不会发生所有权转移
    let n2 = n;
    println!("n: {}, n2: {}", n, n2);
}

fn task_ownership( s : String ) {
    println!("input string: {}", s);
    // 调用结束后，s移出作用域，被释放
}
```

Rust有一个叫做`Copy`的特征，可以用在类似整型这样在`栈`中存储的类型，不会发生所有权转移。`Copy`特征判断规则：任何**基本类型**的组合可以`Copy`，不需要分配内存或某种形式资源的类型是可以`Copy`的。

下面都是具有`Copy`特征的类型：

* 所有整数类型、bool、浮点数类型、字符类型`char`
* 元组且其成员都是`Copy`的，比如：`(i32, i32)` 是 Copy 的，但 `(i32, String)` 就不是
* 不可变引用 `&T`
    * 比如：`let x: &str = "hello, world";`，而后`let y = x;`，此处的`x`只是引用，所以不会发生所有权转移，此时 `y` 和 `x` 都引用了同一个字符串

上面的`let x: &str = "hello, world";`示例，此处的`&str`其实是`借用`。

**借用(Borrowing)**：获取变量的引用，称之为`借用(borrowing)`。

* `&`表示借用，`*`表示解引用（dereference）。
    * 比如：`let x = 5; let y = &x`，`*y`就是5
* 引用指向的值**默认是不可变（immutable）**的
    * `fn change(some_string: &String) { some_string.push_str(", world"); }`，这里`some_string`是借用，不能修改，**会报错**

**可变引用（mutable reference）**：

* `可变引用`可以解决上述问题，可修改引用指向的值
    * `fn change(some_string: &mut String) { some_string.push_str(", world"); }`，这里`some_string`是可变借用，可以修改
    * 需要在定义时指定`mut`，传参时也指定`mut`。定义：`let mut s = String::from("hello");`，调用：`change(&mut s);`，否则会报错
* 同一**作用域**，特定数据只能有一个可变引用（脱离作用域后，引用失效，再进行可变引用不会报错）。编译器会进行借用检查（borrow checker），确保引用有效性，在**编译器**就避免数据竞争（data race）
    * 数据竞争可能由下述行为造成
        * 两个或更多的指针同时访问同一数据
        * 至少有一个指针被用来写入数据
        * 没有同步数据访问的机制
    * 可通过大括号`{}`手动限定变量的作用域，从而解决编译器借用检查的问题
* `可变引用`与`不可变引用`不能同时存在
    * Rust 的编译器一直在优化，早期的（`Rust 1.31 前`）编译器，引用的作用域跟`变量作用域`是一致的，这对日常使用带来了很大的困扰
    * 但是在新的编译器中，引用作用域的结束位置从花括号变成`最后一次使用的位置`
    * 对于这种编译器优化行为，Rust 专门起了一个名字 —— `Non-Lexical Lifetimes(NLL)`，专门用于找到某个引用在作用域(`}`)结束前就不再被使用的代码位置。

**悬垂引用（dangling reference）**：

`悬垂引用`也叫做`悬垂指针`：指针指向某个值后，这个值被释放掉了，而指针仍然存在，其`指向的内存`可能 `不存在任何值` 或 `已被其它变量重新使用`。

Rust编译器中可以确保引用**永远也不会变成悬垂状态**。
