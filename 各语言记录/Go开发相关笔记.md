## go

## Go文档

### 如何使用 Go 编程

* 官网入门文档 [How to Write Go Code](https://golang.org/doc/code.html)
    - 中文版：[如何使用 Go 编程](https://go-zh.org/doc/code.html)
* 安装
    - [Install the Go tools](https://golang.org/doc/install#install)
    - 下载并解压到/usr/local，`tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz`
    - 添加环境变量(~/.bashrc或用zsh时：~/.zshrc)，`export PATH=$PATH:/usr/local/go/bin`
    - 测试`go version` 查看版本得到 `go version go1.13.5 linux/amd64`
* `GOPATH`环境变量
    - [The GOPATH environment variable](https://golang.org/doc/code.html#Workspaces)
    - `go help gopath` 查看说明信息
    - `GOPATH`指定工作空间(`workspace`)的位置，Unix上默认`$HOME/go`，可以通过`go env GOPATH`查看
        + 可以指定一个不同的位置
        + 注意：`GOPATH`不能和go的安装路径一样
    - 通过环境变量指定`GOPATH`，可以指定多个，用`:`隔开(Unix`:`，而Windows上用`;`)
        + e.g. `export GOPATH=$GOPATH:/home/xd/go_path`
        + 为了方便起见，可以把工作空间的`bin`路径添加到`PATH`中
        + `go get` 时默认安装到**第一个**GOPATH路径
    - 每个`GOPATH`下的目录结构需要满足指定的结构
        + `src` 放源码
        + `pkg` 放安装包
            * 存放的目录结构为`pkg/GOOS_GOARCH`，`GOOS_GOARCH`用来区分不同系统架构，如Windows下为`windows_amd64`
        + `bin` 放编译好的命令，每个命令都以`源码目录的最后层级`命名
            * e.g. DIR在GOPATH中，源码目录：DIR/src/foo/quux，则编译的命令quux路径为：DIR/bin/quux，而不是 DIR/bin/foo/quux
            * 如果设置了`GOBIN`变量，则命令放置位置会由`DIR/bin`替换为环境变量设置的路径(需要设置为绝对路径)
    - Go搜索源代码时会搜索每个在`GOPATH`中列出的目录，不过注意：新的包只会下载在列表的`第一个`目录
        + 若使用`modules`，则`GOPATH`不再用于解析`import`，不过还是会用于存放源码和编译后的命令
* `import path`
    - [Import paths](https://golang.org/doc/code.html#ImportPaths)
    - `import path`唯一标识一个包，一个包的import path和它在工作空间或者远程仓库中的位置是对应的
* `go install` 生成执行文件到`$GOPATH/bin`中
    - 可以在任何地方执行，`/home/xd/workspace/go_path/src/hello`目录下，新建了`hello.go`文件
        + 则可以cd到目录中执行`go install`
        + 或者 `go install /home/xd/workspace/go_path/src/hello`
* `go build` 使用库
    - [Your first library](https://golang.org/doc/code.html#Library)
    - 示例见链接
        + 新建要包含函数的.go文件，`package stringutil`
        + `func Reverse(s string) string {}`
        + 测试package编译，执行`go bulid`
            * 此步骤不会生成输出文件，而是将编译的包保存在本地编译缓存中
        + 在其他函数中调用上述函数(使用`import 相对$GOPATH的路径`导入其所在的包)，`go install` 生成执行文件，执行即可
            * `import ("stringutil")`
* `Package names`
    - 在Go源文件中，第一个语句必须是 `package name`
    - `name`是`import`时默认的package包名
    - 所有在一个package中的文件必须使用相同的`name`
    - Go的约定是package名是导入路径的最后一级的名字，如导入`import "crypto/rot13"`，则导入的包应该是rot13
    - **可执行的命令必须使用包`main`**，e.g. `package main`
        + 链接到一个二进制文件的所有包中，包名不必需唯一，但导入的全路径必须唯一
* `Testing`
    - [Testing](https://golang.org/doc/code.html#Testing)
    - Go包含一个轻量级框架 `testing` 包，使用`go test`命令使用
        + `import "testing"` 来使用
    - 使用要求
        + 文件命名格式：`_test.go`
        + 要测试的函数命名格式：`TestXXX`
        + 签名：`func (t *testing.T)`
            * e.g. `func TestReverse(t *testing.T) {...}`
        + 调用`t.Error` 或者 `t.Fail`则当做测试失败
    - `go test` 进行用例测试
* `Remote packages` 远程package
    - `go get`
        + 该命令将会自动 `fetch`、`build`、`install`
        + 如果本地已存在该package，则`go get`会跳过`fetch`(和`go install`表现得一致)
        + e.g. `go get github.com/golang/example/hello`，生成执行文件：`$GOPATH/bin/hello`，可直接使用


### 《Effective Go》

* [《Effective Go》中英双语版](https://bingohuang.gitbooks.io/effective-go-zh-en/content/)
    - 本文档就如何编写清晰、地道的 Go 代码提供了一些技巧。它是对 `语言规范`、`Go 语言之旅` 以及 `如何使用 Go 编程` 的补充说明，因此我们建议您先阅读这些文档。
        + [Go编程语言规范](https://go-zh.org/ref/spec)
            * 英文版：[The Go Programming Language Specification](https://golang.org/ref/spec)
            * 阅读笔记查章节: `### [Go编程语言规范]`
        + [Go 语言之旅](https://tour.golang.org/)
        + [如何使用 Go 编程](https://go-zh.org/doc/code.html)
            * 英文版：[How to Write Go Code](https://golang.org/doc/code.html)
            * 阅读笔记查章节: `### 如何使用 Go 编程`
    - 格式化
        + 在 Go 中让机器来处理大部分的格式化问题。gofmt 程序（也可用 go fmt，它以包为处理对象而非源文件）将 Go 程序按照标准风格缩进、 对齐，保留注释并在需要时重新格式化
        + 你无需花时间将结构体中的字段注释对齐，gofmt 将为你代劳
        + 标准包中所有的 Go 代码都已经用 gofmt 格式化过了
        + 部分格式化细节
            * 缩进：使用制表符(tab)缩进，gofmt 默认也使用它。
            * 比起 C 和 Java，Go 所需的括号更少：控制结构（if、for 和 switch）在语法上并不需要圆括号。
    - 命名(Names)
        + 包名(Package names)
            * 当一个包被导入后，包名就会成了内容的访问器 e.g. `import "bytes"`
            * 按照惯例，包应当以小写的单个单词来命名，且不应使用下划线或驼峰记法。
            * 另一个约定就是包名应为其源码目录的基本名称
                - 在 src/pkg/encoding/base64 中的包应作为 "encoding/base64" 导入，其包名应为 base64， 而非 encoding_base64 或 encodingBase64
            * 包的导入者可通过包名来引用其内容，因此包中的可导出名称可以此来避免冲突
                - bufio包中的缓存读取器类型叫做Reader而非BufReader，长命名并不会使其更具可读性。一份有用的说明文档通常比额外的长名更有价值
                - bufio.Reader 不会与 io.Reader 发生冲突
        - 获取器(Getters)
            * Go 并不对获取器（getter）和设置器（setter）提供自动支持， 你应当自己提供获取器和设置器，通常很值得这样做
                - 若你有个名为 owner（小写，未导出）的字段，其获取器应当名为 `Owner`（大写，可导出）而非 `GetOwner`
                - 若要提供设置器方法，`SetOwner` 是个不错的选择
                - e.g. `owner := obj.Owner()`, `obj.SetOwner(user)`
        - 接口名(Interface names)
            * 按照约定，只包含一个方法的接口应当以该方法的名称加上 `er` 后缀来命名，如 Reader、Writer、 Formatter、CloseNotifier 等。
            * 请将字符串转换方法命名为 String 而非 ToString
        - `驼峰记法`
            * Go 中约定使用驼峰记法 `MixedCaps` 或 `mixedCaps` 而非下划线的方式来对多单词名称进行命名
    - 分号(Semicolons)
        + 和 C 一样，Go 的正式语法使用分号来结束语句；*和 C 不同的是，这些分号并不在源码中出现*。 取而代之，词法分析器会使用一条简单的规则来*自动插入分号*，因此源码中基本就不用分号了
        + 警告：无论如何，你都不应将一个控制结构（if、for、switch 或 select）的左大括号放在下一行。如果这样做，就会在大括号前面插入一个分号，这可能引起不需要的效果。 你应该这样写
            * `if i < f() {` (而不是另起一行写`{`)
    - 控制结构
        + 循环 统一了 for 和 while，不再有 do-while 了
        + Go 没有逗号操作符 (逗号用于平行赋值)
            * `a, b = b, a` 反转赋值
        + Go 的 switch 比 C 的更通用。其表达式无需为常量或整数
            * break 语句可以使 switch 提前终止
            * 还可以用break打破层层的循环。在 Go 中，我们只需将标签放置到循环外，然后 “蹦” 到那里即可`break Loop`
    - 函数
        + Go 函数的返回值或结果 “形参” 可被命名，并作为常规变量使用，就像传入的形参一样
            * e.g. `func nextInt(b []byte, pos int) (value, nextPos int) {`
            * 命名后，函数开始执行时其被初始化为与其类型相应的零值
            * 若该函数执行了一条不带实参的 return 语句，则结果形参的当前值将被返回
            * 此名称不是强制性的，但它们能使代码更加简短清晰：它们就是文档
        + defer 语句
            * 被推迟函数的实参（如果该函数为方法则还包括接收者）在**推迟执行时就会被求值**，而不是在调用`执行时`才求值。
                - 这样不仅无需担心变量值在函数执行时被改变， 同时还意味着单个被推迟的调用可推迟多个函数的执行
            * 被推迟的函数按照`后进先出（LIFO）`的顺序执行
    - 数据分配
        + `new`
            * `new(T)` 会为类型为 T 的新项分配*已置零*的内存空间，并返回它的地址，也就是一个类型为 `*T` 的值(指针)
            * 请注意，返回一个局部变量的地址完全没有问题，这点与 C 不同。 该局部变量对应的数据 在函数返回后依然有效。
            * 每当获取一个复合字面量的地址时，都将为一个新的实例分配内存
                - `return &File{fd, name, nil, 0}`
            * 表达式 `new(File)` 和 `&File{}` 是等价的
        + `make`
            * 内建函数 `make(T, args)` 的目的不同于 `new(T)`。
            * 它只用于创建切片、映射和信道，并返回类型为 T（而非`*T`）的一个已初始化 （而非置零）的值
                - 出现这种用差异的原因在于，这三种类型本质上为引用数据类型，它们在使用前必须初始化
                - 切片是一个具有三项内容的描述符，包含一个指向（数组内部）数据的指针、长度以及容量， 在这三项被初始化之前，该切片为 nil
                - make 用于初始化其内部的数据结构并准备好将要使用的值
            * 对比
                - `make([]int, 10, 100)` 会分配一个具有 100 个 int 的数组空间，接着创建一个长度为 10，容量为 100 并指向该数组中前 10 个元素的*切片结构*
                - 而 `new([]int)` 会返回一个指向新分配的，已置零的切片结构，即一个*指向 nil 切片值的指针*
            * 记住：make 只适用于映射map、切片slice和信道chan且不返回指针。
                - 若要获得明确的指针，使用 `new` 分配内存或显式地获取一个变量的地址。
    - 数组
        + 在Go 和 C 中的区别
            * 数组是值。将一个数组赋予另一个数组会复制其所有元素
            * 若将某个数组传入某个函数，它将接收到该数组的一份**副本**而非指针
                - 若想要 C 那样的行为和效率，可以传递一个指向该数组的指针。(不过这并不是 Go 的习惯用法，切片才是)
            * 组的大小是其类型的一部分。类型 `[10]int` 和 `[20]int`是不同的
    - 切片
        + 切片通过对数组进行封装，为数据序列提供了更通用、强大而方便的接口
        + 除了矩阵变换这类需要明确维度的情况外，Go 中的大部分数组编程都是通过切片来完成的。
        + 切片保存了对底层数组的引用，若将某个切片赋予另一个切片，它们会引用同一个数组
    - 映射(map)
        + 若试图通过映射中不存在的键来取值，就会返回与该映射中项的类型对应的零值
        + `seconds, ok := timeZone[tz]` 区分某项是不存在还是其值为零值，使用`comma ok`惯用法，存在则ok为ture
        + 要删除映射中的某项，可使用内建函数 `delete`，它以映射及要被删除的键为实参
            * 即便对应的键不在该映射中，此操作也是安全的
            * `delete(timeZone, "PDT")`
    - `init`
        + golang的init函数
        + [五分钟理解golang的init函数](https://zhuanlan.zhihu.com/p/34211611)
        + `init`函数
            * `init`函数先于`main`函数自动执行，不能被其他函数调用(*注意*)；
            * `init`函数没有输入参数、返回值；
            * 每个包可以有多个`init`函数；
            * 包的每个源文件也可以有多个`init`函数(一个文件中可有多个init)，这点比较特殊；
            * 同一个包的多个init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序
                - 链接中有个示例，猜测是按源文件名称的字典序，注意没找到根据
            * *不同包*的init函数按照包导入的`依赖关系`决定执行顺序
        + Golang程序初始化
            * golang程序初始化先于main函数执行，由runtime进行初始化，初始化顺序如下：
                - 1. 初始化`导入的包`(不一定从上到下顺序)，runtime需要解析包依赖关系，没有依赖的包最先被初始化，与变量初始化依赖关系类似
                    + 链接中演示了(全局/局部)变量初始化顺序：[golang变量的初始化](https://mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)
                    + 函数作用域内的局部变量，初始化顺序：从左到右、从上到下，所以对于*局部*的`var a int=b+1`, `var b int=1`，会报错(b未定义)
                    + 但对于*全局*定义:`var a int=b+1`, `var b int=1`，可先于b定义就使用b，使用正常
                        * 对于package级别的变量，初始化顺序与初始化依赖有关
                        * 若依赖关系出现嵌套循环(a依赖b,b依赖a)，则会报错：`initialization loop`
                - 2. 初始化包作用域的`变量`(不一定按从上到下、从左到右顺序)，runtime解析变量依赖关系，没有依赖的变量最先被初始化
                - 3. 执行包的`init`函数
            * import包根据依赖关系初始化的示例，见链接
        + golang对没有使用的导入包会编译报错，但是有时我们只想调用该包的`init`函数，而不使用包导出的变量或者方法
            * 可以导入时用`_`操作符：`import _ "net/http/pprof"`
        + init函数的主要用途：初始化不能使用初始化表达式初始化的变量，如下示例：

```golang
var initArg [20]int
func init() {
   initArg[0] = 10
   for i := 1; i < len(initArg); i++ {
       initArg[i] = initArg[i-1] * 2
   }
}
```

* 方法
    - 查看本笔记中的 "method`方法`声明" 章节
* 空白标识符 `_`
    - 空白标识符可被赋予或声明为任何类型的任何值，而其值会被无害地丢弃，可用于for-range 循环和 map
    - for range 循环中对空白标识符的用法是一种具体情况，更一般的情况即为多重赋值
        + `fi, _ := os.Stat(path)` 丢弃错误值(不过忽略错误是种糟糕的实践)
    - 未使用的导入和变量
        + 若导入某个包或声明某个变量而不使用它就会产生错误。未使用的包会让程序膨胀并拖慢编译速度， 而已初始化但未使用的变量不仅会浪费计算能力，还有可能暗藏着更大的 Bug。 然而在程序开发过程中，经常会产生未使用的导入和变量
        + a. `import "fmt"`但是未使用, 可用`var _ = fmt.Printf` 来让编译器停止报错
        + b. `fd, err := os.Open("test.go")`，fd暂时未用到，则可 `_ = fd`来停止报错，建议加上注释提醒`// TODO: use fd`
    - 为次要作用而导入
        +  导入某个包只是为了其次要的作用，没有其它使用该包的可能
        +  `import _ "net/http/pprof"` 如net/http/pprof包的init函数中记录了HTTP处理程序的调试信息，部分客户端只需要这些信息
    - 若只需要判断某个类型是否是实现了某个接口，而不需要实际使用接口本身（可能是错误检查部分），就使用空白标识符来忽略类型断言的值：
        + `if _, ok := val.(json.Marshaler); ok {`
    - 若某个类型（例如 `json.RawMessage`） 需要一种定制的 JSON 表现时，它应当实现 `json.Marshaler`， 不过现在没有静态转换可以让编译器去自动验证它，可在该包中用空白标识符声明一个全局变量：
        + `var _ json.Marshaler = (*RawMessage)(nil)`
        + 在此声明中，我们调用了一个 *RawMessage 转换并将其赋予了 Marshaler，以此来要求 *RawMessage 实现 Marshaler，这时其属性就会在编译时被检测
        + 在这种结构中出现空白标识符，即表示该声明的存在只是为了类型检查。不过请不要为满足接口就将它用于任何类型。作为约定， 仅当代码中不存在静态类型转换(`.(具体类型)`)时才能这种声明，毕竟这是种罕见的情况。
            * 静态类型转换的要求：该类型必须为该接口所拥有的具体类型， 或者该值可转换成的第二种接口类型，可参考本笔记中的`* 类型断言`章节
* 并发
    - 链接中的示例，循环创建goroutine时存在的循环变量req共享的问题，两种解决方式：
        + a. 将 req 的值作为实参传入到该 goroutine 的闭包中
        + b. 用相同的名字获得了该变量的一个新的版本， 以此来局部地刻意屏蔽循环变量，使它对每个 goroutine 保持唯一
            * `req := req` 它的写法看起来有点奇怪，但在 Go 中这样做是合法且惯用的。用相同的名字获得了该变量的一个新的版本， 以此来局部地刻意屏蔽循环变量，使它对每个 goroutine 保持唯一
    - 可参考`* 并发编程`章节的笔记


### [Go语言圣经（中文版）](https://books.studygolang.com/gopl-zh/index.html)


### [Go编程语言规范](https://go-zh.org/ref/spec)

* [Go编程语言规范](https://go-zh.org/ref/spec)
    - 源码为采用 `UTF-8` 编码的 Unicode 文本
* 数值类型(Numeric types)
    - **注意**
        + `int` 和 `uint`位数一样，或者32位或者64位，取决于系统
        + `byte`为`uint8`的别名，8位
        + `rune`为`int32`的别名，32位
        + `uintptr` 一个足够大的无符号整型，用来表示任意地址
            * an unsigned integer large enough to store the uninterpreted bits of a pointer value
            * 并没有明确说位数，最大应该为64位
    - [Numeric types](https://go-zh.org/ref/spec#Numeric_types)
* `struct类型`
    - 允许有声明类型而没有明确字段域名字的域(成员)，这种成员称为匿名域(也叫嵌入式域)
    - 不过在一个struct中，匿名域必须是独一无二的，下面示例的情况是不允许的(三者都会冲突)
    - 一个域声明的时候可以接一个可选的tag标签，参考本笔记中的`### struct json标签`记录，序列化时会将结构体和标签关联，像json的标签，序列化为json时就会成为key
    - 示例：

```golang
// 不合法的匿名域
struct {
    T     // 会和*T、*P.T冲突
    *T    // 会和T、*P.T冲突
    *P.T  // 会和T、*T冲突
}

// 合法的匿名域
struct {
    T1    // 匿名域
    *T2   // 匿名域
    P.T3  // 匿名域
    x int
}
```

* 指针类型，未初始化的指针是nil
    - `*Point`
    - `*[4]int`  数组的指针
    - 对结构体指针的赋值(对一个复合类型取地址会生成一个指向其类型的指针)：`var pointer *Point3D = &Point3D{y: 1000}`
    - 数组/切片的成员为复合类型T时，赋值可省略成员的类型，也可省略&T
        + e.g. `[...]*Point{{1.5, -3.5}, {0, 0}}`，和 `[...]*Point{&Point{1.5, -3.5}, &Point{0, 0}}`一样
* 常量声明

```golang
const Pi float64 = 3.1415
// 可以省略类型
const zero = 0.0 //表示浮点型常量

// 可以同时定义多个常量
const (
    xdname string = "XD"
    age = -1 // 声明时可无类型，此处整型常量
)

// 也可以这样一条语句同时定义多个，常量类型由右边的表达式推导
const a, b, c = 3, 4, "foo" //a=3,b=4,c="foo"

// 相同类型的多个常量
const u, v float32 = 0, 3    // u = 0.0, v = 3.0
```

* iota
    - 在括号内的常量声明中，`iota`标识符表示连续的无类型整数常量(从0开始)
        + 括号表达式中，若声明的变量省略了表达式列表，那此时这个变量和前一个变量的表达式列表和类型(如果有的话)一样
    - iota每次出现时，被重置为0
    - iota在每个表达式后新增一次，若在同一个const语句中，iota多次出现，则只递增一次
        + `bit0, mask0 = 1 << iota, 1<<iota - 1`, bit0=1(1<<0), mask0=0(1<<0 - 1)

```golang
// 从0连续递增的整数常量
const (
    c0 = iota // c0=0
    c1 = iota // c1=1
    c2 = iota // c2=2
)
// 配合运算符使用
const (
    a = 1<<iota // a=1, 1<<0左移0位
    b = 1<<iota // a=2, 1<<1
    c = 1<<iota // a=4, 1<<2
    d float64 = iota * 42 // 指定常量类型不进行省略，d=3*42=126.0
)

// 非括号内，每次出现iota会重置
const x=iota //x=0, iota重置为0
const y=iota //y=0, iota重置为0

// 如果括号内声明变量时省略了表达式列表和类型，那么跟前一个变量一样
const (
    v1 = iota  //v1=0
    v2         //v2=1
    v3=iota    //v3=2
    v4 float32 = 1*iota //v4=3.0
    v5  //v5=4.0，省略表达式和类型时跟前一个变量一样
)

// iota在每个表达式后新增一次，若在同一个const语句中，iota多次出现，则只递增一次
const (
    bit0, mask0 = 1 << iota, 1<<iota - 1  // bit0 == 1(1<<0), mask0 == 0(1<<0 - 1)
    bit1, mask1                           // bit1 == 2(1<<1), mask1 == 1(1<<1 - 1)
    _, _                                  // skips iota == 2 (丢弃值，该const表达式执行后iota还是加1)
    bit3, mask3                           // bit3 == 8(1<<3), mask3 == 7(1<<3 - 1)
)
```

* `func`函数类型
    - 函数声明(function declarations)
    - 参数列表的变量名称必须同时有，或者同时没有
        + `func(x int, y int) (float64, int)`
        + `func(int, int) (float64, int)`
    - 返回值通常都用括号`()`括起来
        + 只有当返回明确只有**一个**、**未命名**的返回值时，可以不用括号
            * e.g. 函数类型：`func(a int, b float32) int`
            * 一般：`func(a int, b float32) (int)` 或 `func(a int, b float32) (res int)`
    - 最后一个参数可能在类型前加上`...`前缀，表示可变参数，可能有0个也可能多个
        + e.g. `func(prefix string, values ...int)`
    - 未初始化的函数变量，值是nil
* method`方法`声明
    - method declarations
    - 一个方法(method)是指带有一个接收者的函数(function)，接收者是`T`或者`*T`形式(可以用括号括起来)，T是`type`名
        + `func (p *Point) Calc(in int32) float64 {xxx实现}` (此处T为通过type定义的Point类型)
        + 此时`Calc方法`的类型为：`func(p *Point, in int32) (float64)`(*不过注意*：若一个函数用这种方式声明，其并不是方法)
    - 一个方法可以看作接收者类型作为函数第一个参数的函数(如上示例)
        + e.g. `T`为struct类型，接收方法：`func (tv T) Mv(a int) int {xxx}`和`func (tp *T) Mp(f float32) float32 {xxx}`
        + `T.Mv`表示一个签名为：`func(tv T, a int) int` 的函数(第一个参数为接收者自身的类型，若为指针则第一个参数为指针)
            * 函数调用可以有以下几种形式：
            * `t.Mv(7)`
            * `T.Mv(t, 7)` (t为一个`T`类型的变量)
            * `(T).Mv(t, 7)`
            * `f1 := T.Mv; f1(t, 7)` 定义一个函数f1，然后用函数调用
            * `f2 := (T).Mv; f2(t, 7)` 类似f1
        + 类似地，`(*T).Mp` 表示函数签名：`func(tp *T, float32) float32`
            * 对于一个接收者是值类型(而不是指针类型)，可用显式指针接收者来派生一个函数
                - e.g.`(*T).Mv` (`Mv`定义时接收者是非指针)
                - 其表示的函数签名为：`func(tv *T, a int) int`
                    + 若`tp`为`*T`类型(结构体指针)，`Mv`方法接收者为`T`，则会自动对指针解引用：`tp.Mv` 和 `(*tp).Mv`是一样的
                - 函数体内并不会改变传入地址(指针)对应的值
            * 对于接收者是指针类型时，**不能**用值类型(非指针类型)来调用函数
                - 对接收者为值的方法的调用，会自动取地址，t为`T`类型，`f := t.Mp; f(7)`，和`(&t).Mp(7)`效果一样
                    + 此处先取方法`t.Mp`再调用定义的函数合法，而`t.Mp(7)`非法 (Mp的接收者是`*T`类型)
                - 指针类型接收者的方法，并不在值类型的方法集中
            * 以指针或值为接收者的区别在于：值方法可通过指针和值调用，而指针方法只能通过指针来调用
                - 之所以会有这条规则是因为指针方法可以修改接收者；
                - 而通过值调用它们会导致方法接收到该值的副本， 因此任何修改都将被丢弃，因此该语言不允许这种错误
                - 若该值是可寻址的， 那么该语言就会自动插入取址操作符来对付一般的通过值调用的指针方法。在上面`f := t.Mp`的例子中，变量 t 是可寻址的，因此只需通过 t.Mp 来调用它的 Mp 方法，编译器会将它重写为 (&t).Mp
                - 参考[Effective Go](https://bingohuang.gitbooks.io/effective-go-zh-en/content/10_Methods.html)
    - 类型`T`被称作接收者基本类型，`T`不能是指针或者接口类型，且必须和方法(method)定义在同一个包(package)中
    - 方法被称作绑定到类型`T`，并且方法名仅在这个类型的使用时可见
* `interface`
    - interface接口类型指定了一个方法集
        + 接口T里面定义的成员可以是另一个接口成员E，叫做(E为在T中的)嵌入接口，但是不能有同名的方法
        + 接口不能嵌入自身，也不能变相嵌入自身(e.g. A包含嵌入接口B，同时B包含接口A)
    - 一个interface类型的变量能够赋值任何 带有该接口方法集超集 的类型(叫做 类型实现了该接口)
        + 若类型实现了某个接口，则同时也实现了任何由该接口子集组成的接口
    - 未初始化的接口类型，值是nil
    - 所有类型实现了空接口 `interface{}`
    - 接口`interface`的实现

```
自定义类型(type)实现接口，需要实现接口中声明的所有方法

1）接口不能实例化（类似于C++中的抽象类），可以指向一个实现了该接口的自定义类型的变量。
2）只要是自定义数据类型，就可以实现接口，不仅仅是结构体类型
3）一个自定义类型可以实现多个接口。

4）接口之间可以实现继承，利用嵌套匿名接口。

5）interface类型默认是一个指针（引用类型），如果没有对interface初始化，则其为nil。

6）空接口type T interface{}没有任何方法，所有的类型都实现了该空接口，也就可以将任何变量赋给空接口。
```

* `map`
    - map是由一组无序的元素组成的类型
    - `make(map[string]int)`, `make(map[string]int, 100)` 使用make新建一个空map，可同时指定容量
    - `len()`获取元素个数
    - nil map和空map，除了nil map不允许添加元素外，两者一样
    - [Golang教程：（十三）Map](https://blog.csdn.net/u011304970/article/details/75003344)
        + `cap()`不能用于求map
        + 创建和使用(*注意需要创建后才能使用*)

```golang
// 创建
map1 := make(map[string]string)
var map2 map[string]string{"lisi":"xxx"}  // 定义时初始化
// 插入
map1["zhangsan"] = "a"
map1["lisi"] = "b"
```

* 访问(key不存在时返回零值)

```golang
map1["lisi"] = "assign"             // map是引用类型
fmt.Printf("map:%s", map1["none"])  // 找不到的记录，map会返回零值(对不同类型对应零值是有区别的)
```

*  遍历

```golang
for key,value := range map1 {

}
```

* 检测是否存在(不存在时map[unexistkey]会返回零值，不能通过nil来判断是否存在)

```golang
// 检测一个键是否存在于一个 map
value, ok := map1["haha"] // 如果 ok 是 true，则键存在，value 被赋值为对应的值。如果 ok 为 false，则表示键不存在
if !ok {
    fmt.Println("not exist")
}
```

**注意：因为 map 是无序的，因此对于程序的每次执行，不能保证使用 for range 遍历 map 的顺序总是一致的。**

* 使用map实现set (go中没有set基本类型)
    - map[anytype]bool

* 利用value为函数时实现工厂模式

```
func TestMapWithFuncValue(t *testing.T) {
    m := map[int]func(op int) int{} //定义一个map, key为int,value为函数(函数是一等公民)
    m[1] = func(op int) int { return op }
    m[2] = func(op int) int { return op * op }
    m[3] = func(op int) int { return op * op * op }
    t.Log(m[1](3), m[2](3), m[3](3))
}
```

* `chan`
    - 通道(channel)提供了并发执行函数来发送和读取的一种机制
    - 未初始化的通道值为nil
    - `<-` 指定通道的方向，是发送还是接收; 如果没有指定方向，则通道是双向的
    - 使用make定义chan:
        + `make(chan int, 100)` 能发送和接收int类型的数据，容量可选
            * 容量指定了通道中的缓冲区大小(元素的个数)，当还有缓冲区(发送时非满/读取时非空)时，能够不阻塞地成功通信
            * 如果容量不设置或者设置为0，则通道是无缓冲的，通信时发送和接收方都准备好才能成功
            * nil通道永远不会为通信做好准备
            * 使用内置的`close(c)`关闭通道
                - close一个已经`关闭的通道`和`nil通道`，会*引发运行时panic*
            * 接收操作符 `x, ok := <-ch`(从ch通道接收数据)，**读channel返回两个值**，会指示通道是否关闭
                - 如果正常接收，则ok为true
                - 从一个`关闭的chan`接收，会*立即返回*(注意此时不是panic)，ok值置为false，x置为对应类型的零值
                    + bool:false; integer:0; float:0.0; string:""; pointer:nil; interface:nil;
                    + slice:nil; map:nil; channel:nil
                    + 对于复合类型，go会自动递归地将每个元素初始化为对应类型的零值
                    + `var i int` 会自动置为零值，i=0
                - 从一个`nil通道`接收，会*永远阻塞*
            * 发送语句
                - `ch <- 3`表示发送语句向通道ch发送3
                - 向一个非缓冲chan中发送数据会阻塞到接收者能够处理，向缓冲chan中发送数据会阻塞到缓冲区有空间
                - 向一个`关闭的chan`中发送数据，会*引发panic*
                - 向一个`nil通道`发送数据，会*永远阻塞*
        + `make(chan<- float64, 10)` 只能发送float64数据(给通道)
        + `make(<-chan int)` 只能(从通道)接收int数据
    - `<-`可以和最左边的chan通道关联
        + `make(chan<- chan int)` 或 `make(chan<- (chan int))` 定义一个写通道，发送的是(可读写的通道chan int)类型
        + `make(chan<- <-chan int)` 或 `make(chan<- (<-chan int))` 写通道，发送的是(读通道，<-chan int)类型
        + `make(<-chan <-chan int)` 或 `make(<-chan (<-chan int))` 读通道，读取的是(写通道，<-chan int)类型
        + `make(chan (<-chan int))` 读写通道，读取的是(写通道，<-chan int)类型
* type声明
    - type声明的类型并不继承绑定到已有类型(将其定义为新类型的原类型)的方法，如下示例
        + `type Mutex struct{xxx结构体成员定义}`，`func (m *Mutex) Lock() {xxx实现}`，Lock()方法绑定到Mutex类型上了
        + `type NewMutex Mutex`，NewMutex和Mutex有相同的结构，但是NewMutex的方法集是空的
        + `type PtrMutex *Mutex`，PtrMutex的方法集是空的
    - 但是一个接口类型的方法集，和组合类型的成员是保持不变的，如下示例
        + `type PrintableMutex struct { Mutex }`，新的PrintableMutex类型包含Lock()方法(Mutex绑定到其匿名域了)
        + `type MyBlockInter BlockInter`，(BlockInter是一个interface)新的MyBlockInter类型包含原接口中的方法集
    - type声明可以给基本类型其别名，然后向它绑定一些方法
        + `type TimeZone int`, `func (tz TimeZone) String() string {return fmt.Sprintf("GMT+%dh", tz)}`
        + 传入的是时区值，返回格式化的string，String()方法绑定到了TimeZone类型上(内层函数引用了 外层函数中的变量/自由变量 的函数，闭包)

```golang
type IntArray [16]int

// 同时声明多个
type (
    Point struct{ x, y float64 }
    Polar Point
)

// 声明结构体
type TreeNode struct {
    left, right *TreeNode
    value *Comparable
}

// 声明接口
type Block interface {
    BlockSize() int
    Encrypt(src, dst []byte)
    Decrypt(src, dst []byte)
}
```

* 变量声明
    - 指定变量类型，如果没有初始化，则变量默认为零值。
        + `var v_name v_type   // 默认为零值`
        + `var b, c int = 1, 2` 可声明多个变量
    - 根据值自行判定变量类型。
        + 如果没有指定明确的类型，则首先转换为其默认类型，float64/int/bool，nil不能用来初始化一个无明确类型的变量
        + `var v_name = value  // 自动会判断类型`
        + `var d = true`
        + `var xdtest = "sldfjk"`
        + `var n = nil`，这个是**非法**的(nil不能用来初始化一个无明确类型的变量)
    - 省略 var, 注意 := 左侧如果没有声明新的变量，就产生编译错误(其中有一个新变量也可)
        + `v_name := value`
        + `var intVal int`
            * `intVal :=1` 这时候会产生编译错误;
            * `intVal,intVal1 := 1,2` 此时不会产生编译错误，因为有声明新的变量，因为 := 是一个声明语句
    - 注意
        + 声明了一个局部变量却没有在相同的代码块中使用它，会得到编译错误 a declared and not used
        + 全局变量允许声明但不使用
        + `_` 是一个只写变量, `v, _ = 1, 5`, 只能读取到v=1，5被丢弃

```golang
 // 同时定义多个变量，这种因式分解关键字的写法一般用于声明全局变量
    var (
        vname1 v_type1
        vname2 v_type2
    )
```

* 选择器(访问类型中的成员)
    - 若`x`是指针且`(*x).f`表示x中的一个域，则`x.f`和`(*x).f`效果一样，是其的简写
    - 假设`T2`struct类型包含struct指针`T0`，T0包含`x`成员
        + 若`var t T2`，则`t.x`是合法的，`(*t.T0).x`(对指针成员解引用)也是合法的
        + 若`var p *T2`，则`p.x`是合法的，`(*(*p).T0).x`也是合法的
        + 若`type Q *T2`，`var q Q`，则`(*(*q).T0).x`来访问x是合法的，`(*q).x`也是合法的，但是`q.x`是**非法**的
            * *需要注意这种情况*
            * 和method方法类似：type声明的类型并不继承绑定到已有类型(将其定义为新类型的原类型)的方法
    - 关于接收者的说明，可参考上面的章节："method`方法`声明"
* 索引表达式`a[x]`
    - 若`a`是一个数组array的指针，则`a[x]`是`(*a)[x]`的简写
    - 若`a`是string类型，则`a[x]`的类型为byte，且`a[x]`不能被赋值
    - 若`a`是map类型，`a[x]`对应的value不存在时，返回是对应类型的零值
        + 通过 `v, ok := a[x]` 来判断是否存在key x对应的成员，若存在则ok为true，v为对应的value
        + 向一个nil map赋值，会引发运行时panic
* slice
    - 初始化方式
        + `make([]T, length)` 或 `make([]T, length, capacity)`同时指定容量
            * e.g. `make([]int, 3, 8)`
            * 若未指定则cap和len一样，
            * 注意长度为length，即使没有append成员，其长度也是length
                - `arr := make([]int, 5)`，则`len(arr)`为5
                - 遍历打印切片，会打印5个成员(默认零值)
                - `arr = append(arr, 1)`之后，`len(arr)`变为6了
                - 超出容量后，会自动扩展成一个新的更大的空间(2倍)，成员拷贝到新空间(比较耗费时间)
        + `var sl []int`
            * `cap`和`len`都是0
        + 对于slice的自动扩容
            * 很多copy-paste的博客说的并不完全准确(只是覆盖到部分分支)：
                - slice扩容，cap不够1024的，直接翻倍；cap超过1024的，新cap变为老cap的1.25倍
            * 扩容后的slice容量和成员类型也有关系，`[]int32` 和 `[]int`，从原来3个成员`append`一个成员后，
                - `e := []int32{1,2,3}`，`e = append(e,4)`，cap从3到8
                - `f := []int{1,2,3}`，`f = append(f,4)`，cap从3到6
            * 链接中gdb单步并跟踪源码，可参考跟踪的过程。结论：
                - append的时候发生扩容的动作：
                    + append单个元素，或者append少量的多个元素，这里的少量指double之后的容量能容纳，这样就会走以下扩容流程，不足1024，双倍扩容，超过1024的，1.25倍扩容
                    + 若是append多个元素，且double后的容量不能容纳，直接使用预估的容量
                - 此外，以上两个分支得到新容量后，均需要根据slice的类型size，算出新的容量所需的内存情况`capmem`，然后再进行`capmem`向上取整(`roundupsize`)，得到新的所需内存，除上类型size，得到真正的最终容量,作为新的slice的容量
                    + 内存对齐，涉及Golang的内存管理
            * [Go slice扩容分析之 不是double或1.25那么简单](https://www.jianshu.com/p/303daad705a3)
        + 对于strint/array/array指针/slice，`a[low:high]`构造一个string子串或slice切片
            * 得到的结果中，索引从`0`开始，长度为`high-low`
            * e.g. 数组 `a := [5]int{1, 2, 3, 4, 5}`
                - `s := a[1:4]` 得到一个`[]int`类型的slice切片，长度为3，容量为4，成员`{2,3,4}`
            * 可以省略任何索引(另外如上面索引表达式中所述，若pa为数组指针，则`pa[l:h]`相当于`(*pa)[l:h]`)
                - `a[2:]` 相当于 `a[2:len(a)]`
                - `a[:3]` 相当于 `a[0:3]`
                - `a[:]` 相当于 `a[0:len(a)]`
            * 对于string和数组，索引不超过(`<=`) `len(a)`，否则超出范围
            * 对于slice，索引上限为`cap(a)`，而不是`len(a)`
    - 切片的结果和原数组共享空间，即slice切片截取赋值时，修改新变量成员会影响之前的切片(共享存储空间)
        + `year := []string{"Jan", "Feb", "Mar", "Apr", "May", "6", "7", "8", "9", "10", "11", "12"}`
        + `Q2 := year[3:6]`      // 下标从3到5(注意不包含6, [3,6))，结果：[Apr May 6]
        + `summer := year[5:8]`  // 结果[6 7 8]
        + `summer[0] = "Unknow"` // 修改summer成员，Q2和year都会受影响，都变成 "Unknow"
        + 切片的传值和指针问题，参考本笔记章节：`* slice作为形参`
    - 完整的切片表达式：`a[low : high : max]`
        + 和`a[low:high]`有一样的长度`(high-low)`，另外设置结果slice的容量为`(max-low)`，第一个索引可省略(默认0)
        + e.g. `a := [5]int{1, 2, 3, 4, 5}`，`t := a[1:3:5]`，结果t为`{2,3}`，len为2，容量为4(而不是5，5-1)
    - 遍历
        + `for index, v := range arr {}`
    - append
        + `arr = append(arr, 3)`  // 把值3添加到[]int，len()新增，cap()按2^n可能受影响
            * 注意append之后，len加1，而不是把之前的空填上了
        + 若要从slice中*删除元素*，则可以截取后赋值
            * `a = a[:len(a)-1]` 删除最后一个元素，删除后n个则`len(a)-n`
            * `a = a[1:]` 删除第一个元素
            * `a = append(a[:i], a[i+1:]...)` 删除第i个元素(i不为最后元素)，通过append来截取拼接，注意拼接一个slice时需要`...`
        + append的用法有两种：
            * `slice = append(slice, elem1, elem2)`，拼接元素到slice中
            * `slice = append(slice, anotherSlice...)`，拼接slice到slice后面
    - 清空
        + `a = a[0:0]`
    - 循环(三种形式，使用for，没有while关键字)
        + `for i:=1; i<10; i++ {}`, 类似C的for(i=0; i<10; i++){}
        + `for {}`, 无限循环，类似while(1){}或for(;;){}
        + `for condition {}`, 类似C的while(condition){}
* 类型断言
    - `x.(T)`
    - 由于接口是一般类型，不知道具体的数据类型，如果需要转成具体类型，就需要使用类型断言
    - `var x interface{} = 7`，x有动态类型int，值为7，,`i := x.(int)`则新增变量i为int类型，值为7
    - `type I interface { m() }`，`var y I`，则
        + `s := y.(string)`，非法，string类型并没有实现I接口
        + `r := y.(io.Reader)`，新增r类型可为 io.Reader，若要使该表达式成立，则y必须实现I和io.Reader接口
    - 为了防止上面的非法情况，类型断言赋值时可以使用如下方式：
        + `v, ok := x.(T)`，如果断言成立，则ok为true，如果不成立则ok为false，v为零值，该情况不产生panic
    - `.(type)`和`.(具体类型)`可参考：[Effective Go接口转换与类型断言](https://bingohuang.gitbooks.io/effective-go-zh-en/content/11_Interfaces_and_other_types.html)
        + `switch str := value.(type) {`，然后用`case`子句依次判断类型，`case string:` `case Stringer:`
        + `str, ok := value.(string)` ()中的类型必须为该接口所拥有的具体类型，或者该值可转换成的第二种接口类型
            * 下面if...else if组合起来，和对`.(type)`判断switch...case等价
            * `if str, ok := value.(string); ok {`
            * `else if str, ok := value.(Stringer); ok {`
* 函数传参
    - `func Greeting(prefix string, who ...string)`
        + 可以用 `参数名 ...T`表示传入一个`[]T`类型的切片(slice)，调用函数时不送值则为nil
            * `Greeting("nobody")`，第二个参数为nil
            * `Greeting("hello:", "Joe", "Anna", "Eileen")`，第二个参数为`[]string{"Joe", "Anna", "Eileen"}`
* 操作符优先级
    - `*p++` 和 `(*p)++` 效果一样，取值后，指针++ ?
* 转换表达式 (使用`T(x)`形式，T是要转换为的类型，x是值，必要时须使用括号来避免歧义)
    - `*Point(p)` 表示构造一个 Point类型，值为p，再解引用取值，同`*(Point(p))`
    - `(*Point)(p)` 表示构造一个 `*Point`类型(Point的指针)，其中的值为p
    - `<-chan int(c)` 和 `<-(chan int(c))`一样： c为`chan int`类型，再进行c进行读取
    - `(<-chan int)(c)` 表示构造一个`<-chan int`类型，赋值为c
    - `func()(x)` 表示函数签名 `func() x`(单个返回值可不用括号)
    - `(func())(x)` x被转换为 `func()` 函数(无返回值)
    - `(func() int)(x)` x被转换为`func() int`函数(一个int返回值)
    - `func() int(x)` x被转换为 `func() int` 函数，注意没有二义性，和上面的表示是一致的
    - 注意
        + string(65.0) 非法
            * string(-1)合法，会将整数代表的utf表示的字符串(若数字超出unicode限制，则转换为"\uFFFD"，此处-1即表示"\ufffd")
            * string('a') 合法，代表"a"
            * string(0xf8)      // "\u00f8" == "ø" == "\xc3\xb8"
            * 可以用`[]byte{}`转换string
                - e.g. `string([]byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'})` 表示`"hellø"`
                    + 注意最后字符 "\u00f8" == "ø" == "\xc3\xb8"
                - `string([]byte{})`表示 `""`
                - `string([]byte(nil))` 表示`""`
            * 可以用`[]rune{}`转换string
                - rune类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 `rune` 类型。`rune` 类型等价于 `int32` 类型
                - e.g. `string([]rune{0x767d, 0x9d6c, 0x7fd4})`表示 "\u767d\u9d6c\u7fd4" == "白鵬翔"
                - `string([]rune{})` 表示`""`
                - `string([]rune(nil))`表示`""`
        + int(1.2) 非法
            * float32(5)合法
* 语句(statement)
    - 几个单词
        + expression 表达式
        + statement 语句 (控制语句 control statements)
* 程序初始化
    - 存储一个分配的变量时(无论通过声明还是new)，变量都会有一个默认值(初始化为对应类型的零值)
        + `var i int` 和 `var i int = 0` 是相同的
        + 结构体类型的变量定义后，其成员各自初始化为零值
* `unsafe`包(内置)
    - 内置的unsafe包 告诉编译器，为低级别的编程提供了便利，包括违反类型系统的操作
    - 为了安全起见，使用unsafe包必须经过手动审核，并且可能不可移植。
    - `unsafe.Pointer` 所有基础类型的指针都可以转换为`Pointer`类型
        + `var f float64`, `bits = *(*uint64)(unsafe.Pointer(&f))`

* slice作为形参
    - 参考下面的实例
    - 切片实现结构：数组指针ptr+切片大小len+切片容量cap
    - `func test1(s []int) ([]int)` (操作s后再return s)
        + `sliceA`作为参数传入时，会新建一个`sliceB`并将其复制为`sliceA`的内容传入(即传值)，其指向的数组和`sliceA`指向的数组(假设为`arrPA`)一样(取slice元素成员的地址，是一样的)
        + 函数内修改`s`的成员时，切片指向的数组内容会被修改
            * 表现是`sliceA`和`sliceB`的元素成员都被修改了
        + `test1()`中操作导致传入slice扩容时，slice的地址不变，但是指向的数组地址会改变，假设为`arrPB`(不再是`arrPA`)
            * 此时原来的`sliceA`指向的数组并不会改变(`arrPA`)
            * 只是新建的`sliceB`指向的数组变成了`arrPB`
        + return返回时，会新建一个新的切片`sliceC`，这个sliceC和传入的`s`就没有关系了
            * 其指向的地址也是扩容时的新数组地址(`arrPB`)
            * 若不扩容，则指向的地址还是和`sliceA`、`sliceB`指向的数组地址一样(均为`arrPA`)
        + 向`s`新增成员的问题
            * 新增成员时，只是新建传入的`sliceB`新增了成员，原来的`sliceA`并无变化(其指向的数组地址也不会变化，只是值随test1变化)
    - `func test2(s *[]int)([]int)`
        + 传slice指针时，传入`sliceA的地址`，则修改都会体现到`sliceA`上
        + 若`test2()`中导致扩容，则`sliceA`指向的数组地址会改变
    - 结论：若要改变原有的slice，则设计函数时，传入slice的指针
        + 若不想改变原有的成员值，则新建一个slice(`make`或者`var`来定义)，再进行复制，而不是用`[:]`直接来截取
    - 参考：[Golang 切片与函数参数"陷阱"](https://www.yuque.com/docs/share/6c07ca60-44d0-49ef-aff4-06ff20d57818)

```golang
package main

import (
    "log"
)

func changeSlice(s []int) []int {
    s[1] = 12345
    log.Printf("sptr:%p, m1p:%p\n", &s, &s[1])
    s = append(s, 1)
    log.Printf("after append, sptr:%p, m1p:%p\n", &s, &s[1])
    return s
}

func changeSlicePtr(s *[]int) []int {
    (*s)[1] = 12345
    log.Printf("sptr:%p,  m1p:%p\n", s, &(*s)[1])
    *s = append(*s, 1)
    log.Printf("after append, sptr:%p, m1p:%p\n", s, &(*s)[1])
    return *s
}

func main() {
    log.SetFlags(log.Lshortfile)
    sli := []int{1, 2, 3}
    log.Printf("sli:%v, ptr:%p, m1p:%p\n", sli, &sli, &(sli[1]))

    sli2 := changeSlice(sli)
    log.Printf("\nsli:%v,ptr:%p,m1p:%p \nsli2:%v,ptr:%p,m1p:%p\n",
        sli, &sli, &(sli[1]), sli2, &sli2, &(sli2[1]))
    sli2[1] = 456
    log.Printf("sli:%v, sli2:%v", sli, sli2)

    log.Println("\n==================\n")
    slip := []int{1, 2, 3}
    log.Printf("slip:%v, ptr:%p, m1p:%p\n", slip, &slip, &(slip[1]))
    slip2 := changeSlicePtr(&slip)
    log.Printf("slip:%v, slip2:%v", slip, slip2)
}

// 结果
slice_param.go:26: sli:[1 2 3], ptr:0xc000090020, m1p:0xc0000a6008
slice_param.go:9: sptr:0xc000090080, m1p:0xc0000a6008
slice_param.go:11: after append, sptr:0xc000090080, m1p:0xc000096098
slice_param.go:29: 
sli:[1 12345 3],ptr:0xc000090020,m1p:0xc0000a6008 
sli2:[1 12345 3 1],ptr:0xc000090060,m1p:0xc000096098
slice_param.go:32: sli:[1 12345 3], sli2:[1 456 3 1]
slice_param.go:34: 
==================

slice_param.go:36: slip:[1 2 3], ptr:0xc000090120, m1p:0xc0000a6088
slice_param.go:17: sptr:0xc000090120,  m1p:0xc0000a6088
slice_param.go:19: after append, sptr:0xc000090120, m1p:0xc000096128
slice_param.go:38: slip:[1 12345 3 1], slip2:[1 12345 3 1]
```


### 值类型

int、float、bool 和 string 这些基本类型都属于值类型，使用这些类型的变量直接指向存在内存中的值

### 引用类型

Golang中只有三种引用类型：slice(切片)、map(字典)、channel(管道)；

### 包安装相关问题

####  go版本更新，下载安装包：

https://studygolang.com/dl

#### golang.org/x/sys/unix: unrecognized问题：

golang.org/x/sys/unix: unrecognized，go get不成功(由于被墙), 

通过手动的方式clone包：

```sh
cd golang.org/x
git clone https://github.com/golang/sys.git
```

参考：
[golang.org/x/sys/unix: unrecognized](https://blog.csdn.net/wzygis/article/details/89030353)


### struct json标签

Golang解析JSON之Tag篇
[golang json用法讲解](https://www.cnblogs.com/williamjie/p/9927281.html)

* 一个结构体正常序列化

struct成员间不用,

```go
// Product 商品信息
type Product struct {
    Name      string
    ProductID int64
    Number    int
    Price     float64
    IsOnSale  bool
}

func main() {
    p := &Product{}
    p.Name = "Xiao mi 6"
    p.IsOnSale = true
    p.Number = 10000
    p.Price = 2499.00
    p.ProductID = 1
    data, _ := json.Marshal(p)
    fmt.Println(string(data))
}

//结果
{"Name":"Xiao mi 6","ProductID":1,"Number":10000,"Price":2499,"IsOnSale":true}
```

* 何为Tag，tag就是标签，给结构体的每个字段打上一个标签，标签冒号前是类型，后面是标签名
打标签后，序列化得key与标签一致

```go
// Product _
type Product struct {
    Name      string  `json:"name"`
    ProductID int64   `json:"-"` // 表示不进行序列化
    Number    int     `json:"number"`
    Price     float64 `json:"price"`
    IsOnSale  bool    `json:"is_on_sale,string"`
}

// 序列化过后，可以看见
   {"name":"Xiao mi 6","number":10000,"price":2499,"is_on_sale":"false"}
```

* omitempty，tag里面加上omitempy，可以在序列化的时候忽略0值或者空值

```go
// Product _
type Product struct {
    Name      string  `json:"name"`
    ProductID int64   `json:"product_id,omitempty"`
    Number    int     `json:"number"`
    Price     float64 `json:"price"`
    IsOnSale  bool    `json:"is_on_sale,omitempty"`
}

func main() {
    p := &Product{}
    p.Name = "Xiao mi 6"
    p.IsOnSale = false
    p.Number = 10000
    p.Price = 2499.00
    p.ProductID = 0

    data, _ := json.Marshal(p)
    fmt.Println(string(data))
}
// 结果
{"name":"Xiao mi 6","number":10000,"price":2499}
```

* type，有些时候，我们在序列化或者反序列化的时候，可能结构体类型和需要的类型不一致，这个时候可以指定,支持string,number和boolean

```go
// Product _
type Product struct {
    Name      string  `json:"name"`
    ProductID int64   `json:"product_id,string"`
    Number    int     `json:"number,string"`
    Price     float64 `json:"price,string"`
    IsOnSale  bool    `json:"is_on_sale,string"`
}

func main() {
    var data = `{"name":"Xiao mi 6","product_id":"10","number":"10000","price":"2499","is_on_sale":"true"}`
    p := &Product{}
    err := json.Unmarshal([]byte(data), p)
    fmt.Println(err)
    fmt.Println(*p)
}
// 结果
<nil>
{Xiao mi 6 10 10000 2499 true}
```

### fmt

```go
func Print(a ...interface{}) (n int, err error) {
	return Fprint(os.Stdout, a...)
}
// Println formats using the default formats for its operands and writes to standard output.
// Spaces are always added between operands and a newline is appended.
// It returns the number of bytes written and any write error encountered.
func Println(a ...interface{}) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
func Printf(format string, a ...interface{}) (n int, err error) {
	return Fprintf(os.Stdout, format, a...)
}
```

```go
// Sprint formats using the default formats for its operands and returns the resulting string.
// Spaces are added between operands when neither is a string.
func Sprint(a ...interface{}) string {
}
// Sprintln formats using the default formats for its operands and returns the resulting string.
// Spaces are always added between operands and a newline is appended.
func Sprintln(a ...interface{}) string {
}
// Sprintf formats according to a format specifier and returns the resulting string.
func Sprintf(format string, a ...interface{}) string {
}
```

```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error) 
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
```

```go
func Errorf(format string, a ...interface{}) error {
	return errors.New(Sprintf(format, a...))
}
```

**真正的项目中一定要使用log或者封装更高层的自定义log，不要直接使用fmt**

原因有2:
（1）最重要的一点，log包是并发goroutine安全的，而fmt不是，这点特别重要
如果要保证并发安全，日志请用log包
(多线程访问时，采用了加锁机制，当一个线程访问该类的某个数据时，进行保护，不会出现数据不一致或者数据污染)

（2）log包会打印时间信息，fmt不会

#### 占位符

[占位符](https://studygolang.com/articles/21039)

占位符	说明	举例	输出
%v	相应值的默认格式。	Printf("%v", people)	{zhangsan}
%+v	打印结构体时，会添加字段名	Printf("%+v", people)	{Name:zhangsan}
%#v	相应值的Go语法表示	Printf("#v", people)	main.Human{Name:"zhangsan"}
%T	相应值的类型的Go语法表示	Printf("%T", people)	main.Human

占位符	说明	举例	输出
%t	true 或 false	Printf("%t", true)	true 布尔占位符
%d 整数
%s 字符串
%p 指针

其他占位符见链接


### log包
* log模块主要提供了3类接口。分别是
    - Print 、Panic 、Fatal
        + `Fatal`相当于调用`Print`之后`os.Exit(1)`退出
        + `Panic`相当于调用`Print`之后`panic()`
* 对每一类接口其提供了3种调用方式
    - Xxxx 、 Xxxxln 、Xxxxf
* 可以设置log的打印格式(默认不带文件名和行号，默认标志为`Ldate | Ltime`)
    - 使用 `log.SetFlags()` 设置，设置多个标志以`|`分隔
        + `Ldate` 显示日期标志
        + `Ltime` 显示时间
        + `Lshortfile` 显示文件名和行号，`Llongfile` 以绝对路径显示文件名
            * 两个选项设置时，打印信息中会显示一个`:`
        + 个人习惯：`log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)`，可以将其放到`init()`初始化
            * 打印内容形式为：`2020/04/04 10:54:28 linkedlist_palindromic.go:87: node value:b, ptr:&{b 0xc00005a420}`
    - `SetPrefix(prefix string)` 设置打印前缀
        + e.g. `log.SetPrefix("[xd] ")`
    - `SetOutput(w io.Writer)` 设置输出目的地
    - 设置不同等级需要定制创建，Info,Warning,Error
        + 可以使用第三方log框架，如`logrus`
            * 完全兼容golang标准库日志模块：logrus日志框架有六种日志级别，debug、info、warn、error、fatal和panic
            * 可扩展的Hook机制：允许使用者通过hook的方式将日志分发到任意地方，如本地文件系统、标准输出、logstash、elasticsearch或者mq等，或者通过hook定义日志内容和格式等
                - 初始化前添加相应自定义的hook即可，`log.AddHook(hook)` (hook变量对应类型要实现`Hook`接口，可自己实现，也有一些第三方HOOK供使用)
            * 可选的日志输出格式
                - logrus(1.2.0之前，现已提供)没有提供文件名和行号(出于性能考虑)，这在大型项目中通过日志定位问题时有诸多不便，若要支持可：自己实现一个hook；或通过装饰器包装`logrus.Entry`
                    + 貌似从logrus 1.2.0开始已经支持在日志中写入 文件名，行号，函数等定位信息
                    + 搜v1.2.0(版本发布时间2018.11.1)，新增了`SetReportCaller`，[logrus releases](https://github.com/Sirupsen/logrus/releases)
                        * 使用 `logrus.SetReportCaller(true)`开启
            * Field机制：logrus鼓励通过Field机制进行精细化的、结构化的日志记录，而不是通过冗长的消息来记录日志
            * logrus是一个可插拔的、结构化的日志框架
            * [golang日志框架之logrus](https://blog.csdn.net/wslyk606/article/details/81670713)
            * [logrus Github](https://github.com/sirupsen/logrus)
                - 对于文件名、函数名和行号，可选择第三方formatter: powerful-logrus-formatter

```go
func main(){
    arr := []int {2,3}
    log.Print("Print array ",arr,"\n")
    log.Println("Println array",arr)
    log.Printf("Printf array with item [%d,%d]\n",arr[0],arr[1])
}
```

```golang
// logrus鼓励使用第二种方式替代方式1
// 方式1
log.Fatalf("Failed to send event %s to topic %s with key %d", event, topic, key)

// 方式2，Fields形式
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```


### bytes.NewReader



### sync.WaitGroup

golang中有2种方式同步程序，一种使用channel，另一种使用锁机制。
这里要涉及的是锁机制，更具体的是sync.WaitGroup

sync.WaitGroup只有3个方法，Add()，Done()，Wait()

```go
var wg sync.WaitGroup
wg.Add(1)
go func(){
	wg.Done()
}()
wg.Wait()
```

### channel

通过channel来同步goroutine

```go
done := make(chan bool)

Channel类型的定义格式如下：

ChannelType= ("chan"|"chan""<-"|"<-""chan")ElementType.
它包括三种类型的定义。可选的 <- 代表channel的方向。如果没有指定方向，那么Channel就是双向的，既可以接收数据，也可以发送数据。

e.g.
chanT// 可以接收和发送类型为 T 的数据
chan<-float64// 只可以用来发送 float64 类型的数据
<-chanint// 只可以用来接收 int 类型的数据
```

### select

[Go并发编程—select的使用](https://blog.csdn.net/zg_hover/article/details/81453379)

通过select可以监听多个channel的读写事件
Go中的select和channel配合使用，监控的是channel的读写状态。


### 注释

struct结构体注释，使用注释方式如下：
`// 结构体名 说明`

### 代码组织和模块化

model      放数据库操作;

handler    把具体的handler函数封装到api包中，因为handler函数要操作数据库，所以会引用model包


router.go  把路由抽离出来，修改router.go，我们在路由文件中封装路由函数

main函数   app入口

文件拆分后，不能像之前简单的使用go run main.go，需要运行`go run *.go`命令编译运行
如果是最终编译二进制项目，则运行go build -o app

### 方法值和方法表达式

```go
func (XdStruct) Get() int{}
func (*XdStruct) Set(val *int) {}
```

方法值调用：
定义一个变量t:=Xdstruct{}，使用变量t.Get, t.Set访问方法

方法表达式调用：

```go
t:=Xdstruct{}
XdStruct.Get(t)
(*XdStruct).Set(&t, t)
```

**注意: 方法表达式调用，需要新增一个参数，该参数为结构体实例或其指针**

### Comma-ok 断言

Comma-ok断言的语法是：value, ok := element.(T)
element必须是接口类型的变量

Go语言里面有一个语法，可以直接判断是否是该类型的变量： value, ok = element.(T)，这里value就是变量的值，ok是一个bool类型，element是interface变量，T是断言的类型。

如果element里面确实存储了T类型的数值，那么ok返回true，否则返回false。

func (s *Ethereum) StartMining(local bool) error {
    ...
    if clique, ok := s.engine.(*clique.Clique); ok {
        ...
    }
    ...
}

### 匿名字段

[面向对象编程](https://www.jianshu.com/p/f29ff2f2de3c)

```go
//人
type Person struct {
    name string
    sex  byte
    age  int
}
//学生
type Student struct {
    Person // 匿名字段，那么默认Student就包含了Person的所有字段
    id     int
    addr   string
    name   string
}

//正确
s1 := Student{Person{"mike", 'm', 18}, 1, "sz"}
//错误
//s2 := Student{"mike", 'm', 18, 1, "sz"} //err

```

```go
type mystr string //自定义类型
//学生
type Student struct {
    Person // 匿名字段，那么默认Student就包含了Person的所有字段
    int
    mystr
    addr   string
    name   string
}

基本类型和自定义类型也可作为匿名字段，访问时，s.int 则会访问到

//初始化
    s1 := Student{Person{"mike", 'm', 18}, 1, "bj"}
//成员的操作，打印结果：mike, m, 18, 1, bj
    fmt.Printf("%s, %c, %d, %d, %s\n", s1.name, s1.sex, s1.age, s1.int, s1.mystr)

    注意s1.int 和 s1.mystr
```

### 同名字段
同上结构
有同名字段时，s1.name 访问为最外部字段；若没有同名字段，则可以访问到Person中的name


### go testing

1. 文件名必须以xx_test.go命名
2. 方法必须是Test[^a-z]开头
3. 方法参数必须 `t *testing.T`

加-v 可打印testing.T.Log()内容

`go test -v -run=TestGetSpotOrders`

```go
func TestOKWSAgent_Login(t *testing.T){
    tests := []struct {
    name    string        //用例名
    a       *OKWSAgent    //测试接口的receiver或入参
    wantErr bool
    }{
        {},{}   //有多少组用例参数则定义多少个数组成员
    }

    for _, tt := range tests {
        t.Run(tt.name, func (t *testing.T) {       //Run的第二个参数，定义一个匿名函数，执行具体操作
            if err := tt.a.Login(); (err != nil) != wantErr {      //wantErr是否故意预期要执行错误
                t.Errorf("OKWSAgent.Login() error = %v, wantErr %v", err, wantErr)
            }
        })
    }
}
```


### go lint

golint校验常见的问题:

[Golint代码规范检测](https://blog.csdn.net/chenguolinblog/article/details/90665161)


不能使用下划线命名法，使用驼峰命名法
外部可见程序结构体、变量、函数都需要注释

通用名词要求大写
iD/Id -> ID
Http -> HTTP
Json -> JSON
Url -> URL
Ip -> IP
Sql -> SQL

包命名统一小写不使用驼峰和下划线
注释第一个单词要求是注释程序主体的名称，注释可选不是必须的
外部可见程序实体不建议再加包名前缀
if语句包含return时，后续代码不能包含在else里面
errors.New(fmt.Sprintf(…)) 建议写成 fmt.Errorf(…)
receiver名称不能为this或self
错误变量命名需以 Err/err 开头
a+=1应该改成a++，a-=1应该改成a–

[Go语言规范-命名篇](https://www.cnblogs.com/Survivalist/articles/10596115.html)
[Go语言规范汇总](https://www.cnblogs.com/Survivalist/articles/10596152.html)

### go 调试

dlv
[GO语言调试利器——dlv](https://www.jianshu.com/p/7373042bba83)

按推荐的go get安装下载很慢，老是失败。
直接下载https://github.com/go-delve/delve，放到$GO_PATH/github.com/go-delve/delve，
进行编译安装即得到执行文件，并自动放到了$GO_PATH/bin

```
cd delve/cmd/dlv/
go build
go install
```

### Golang工程结构和编译

go build : 编译出可执行文件

go install : go build + 把编译后的可执行文件放到GOPATH/bin目录下

go get : git clone + go install

### time包 获取时间

* [golang timestamp time 时间戳小结](https://pylist.com/t/1438769640)
    - 将utc时间戳(秒数)转换为Time类型
        + `tm := time.Unix(timestamp, 0)`，返回Time类型，原型：`func Unix(sec int64, nsec int64) Time {`
        + `tm.Format("2006-01-02 15:04:05")`，返回格式化的文本，原型：`func (t Time) Format(layout string) string {`
    - 将Time类型转换为utc时间戳：
        + `t := time.Now()` 获取当前时间的Time结构体
        + `secs := t.Unix()`秒，`t.UnixNano()`纳秒，根据纳秒/1000000获得毫秒，可获取Time结构秒数
        + `t.Year(), t.Month(), t.Day()` 等分别获取`Time`结构的年月日
    - 当前日期加时间段
        + `time.AddData(1,1,1)` 当前日期加1年1月1日
        + `time.Add(d Duration)` 按Duration添加
        + 均在 `time.go` 对应的package time包中
* 比较时间，`Time`的比较是使用`Before`,`After`和`Equal`方法
    - `t1.Before(t2)`
    - `Sub`方法返回的是两个时间点之间的时间距离
    - `Add`方法和Sub方法是相反的，获取t0和t1的时间距离d是使用Sub，将t0加d获取t1就是使用Add方法
    - `IsZero`方法：Time的zero时间点是January 1, year 1, 00:00:00 UTC，这个函数判断一个时间是否是zero时间点
    - `Local`，`UTC`，`Ln`是用来显示和计算地区时间的
    - [Go语言_时间篇](https://www.cnblogs.com/yjf512/archive/2012/06/12/2546243.html)
* [golang包time用法详解](https://blog.csdn.net/wschq/article/details/80114036)
* golang提供以下两种基础类型
    - 时间点(`Time`)
    - 时间段(`Duration`)
* 此外还提供：
    + 时区(`Location`)
    + `Ticker`
    + `Timer`(定时器)
* 时间格式化：
    - `tm2, _ := time.Parse("01/02/2006", "02/08/2015")`
        + 从字符串转为时间戳，第一个参数是格式，第二个是要转换的时间字符串
    - `time.ParseInLocation()`
        + 按指定时区格式化，`time.Parse()`是按UTC格式化
        + `func ParseInLocation(layout, value string, loc *Location) (Time, error)`
        + `*Location`可以送`time.Local`系统本地时区

```golang
BeginTime := "20191126 145500"
// 2006-01-02 15:04:05 必须是这个时间点, 记忆方法:6-1-2-3-4-5
// 01/02 03:04:05PM '06 -0700(时区), go语言中使用该递增的格式来解析各时间位
// 此处有个简单的源码分析：[Golang神奇的2006-01-02 15:04:05](https://www.jianshu.com/p/c7f7fbb16932)
formatTime, _ := time.Parse("20060102 150405", BeginTime)
fmt.Printf("ori time:%v, parse:%v\n", BeginTime, formatTime.Format("2006-01-02 15:04:05"))
```

* Sleep
    - `time.Sleep(10 * time.Second)`，sleep十秒
    - 函数如下：`func Sleep(d Duration)`
        + `Duration` 结构：`type Duration int64`，表示持续时间，int64表示的`纳秒`(最长290年左右)
        + 定义的一些时间相关的const变量
            * `const Second = 1000 * Millisecond`

```golang
const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```

## string

* 是一个不可变的byte切片，初始化后不能修改成员
* Go语言的字符有以下两种：
    - 一种是 `uint8` 类型，或者叫 `byte` 型，代表了 ASCII 码的一个字符
    - 另一种是 `rune` 类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 `rune` 类型。`rune` 类型等价于 `int32` 类型。(注意`int`和系统位数有关==，64位系统上为8字节，笔记中搜索：`* 数值类型(Numeric types)`章节)
    - **互转**
        + string转[]byte：`var data []byte = []byte(str)`
        + []byte转string：`var str string = string(data[:])`
* `unicode`和`utf-8`
    - unicode是一种编码，utf-8是unicode的一种存储实现
    - golang中string底层是通过`byte数组`实现的。中文字符在unicode下占`2`个字节，在utf-8编码下占`3`个字节，而golang默认编码正好是`utf-8`。
        + 所以 `var str = "hello 你好"`，`len(str)`是12(5+1+3*2)，而不是8
        + `len([]rune(str))` 则是8

```golang
var s string = "中"   // len(s):3
c := []rune(s)        // len(c):1
t.Logf("中 unicode %x", c[0])  // 4e2d
t.Logf("中 UTF8 %x", s)        // e4b8ad
```

* utf8和unicode关系

由上面的例子

```
字符           "中"
Unicode        0x4E2D
UTF-8          0xE4B8AD (在string中分成了3个byte，如下)
string/[]byte  [0xE4, 0xB8, 0xAD]
```

* 一种格式化打印用法：
    - t.Logf("%[1]x, %[1]s", s)  // 打印："e4b8ad, 中"，[1]指定第一个参数

* 常用字符串函数(包)
    - strings 包
    - strconv 包

* 中文编码问题，gbk转utf
    - `go get github.com/axgle/mahonia` BSD协议
    - [golang 字符串编码转换 gbk转utf8](https://blog.csdn.net/jeffrey11223/article/details/79287010)

```golang
import ("github.com/axgle/mahonia")

//src为要转换的字符串，srcCode为待转换的编码格式，targetCode为要转换的编码格式
func ConvertToByte(src string, srcCode string, targetCode string) []byte {
    srcCoder := mahonia.NewDecoder(srcCode)
    srcResult := srcCoder.ConvertString(src)
    tagCoder := mahonia.NewDecoder(targetCode)
    _, cdata, _ := tagCoder.Translate([]byte(srcResult), true)
    return cdata
}
```

### strings 包 和 strconv 包

* strings
    - 切割
        + strings.Split(s, ",")
            * s := "A,B,C"
            * parts := strings.Split(s, ",")
    - 连接
        + strings.Join(parts, "-")

* strconv
    - 转换
        + s := strconv.Itoa(10)       // int到string, 转成字符串
        + i,err := strconv.Atoi("10") // string到int，注意还会返回error
    - 若要进行拼接，则可用`str += fmt.Sprintf("movie=%s","xx")`

* string、int、int64互相转换示例：

```golang
int,err:=strconv.Atoi(string)
#string到int64
int64, err := strconv.ParseInt(string, 10, 64)
#int到string
string:=strconv.Itoa(int)
#int64到string
string:=strconv.FormatInt(int64,10)
```


## net 包

* net 包
    - net包提供编写一个网络客户端或者服务器程序的基本组件，无论两者间通信是使用TCP，UDP或者Unix domain sockets
        + net/http包里的方法，也算是net包的一部分

# Go语言标准库

* [《Go语言标准库》The Golang Standard Library by Example](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/)

## flag 包

* flag 包
    - 参考：[1. 13.1 flag - 命令行参数解析](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter13/13.1.html#131-flag---%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0%E8%A7%A3%E6%9E%90)
    - flag 包实现了命令行参数的解析(使用命令行执行程序，传入参数)
        + 示例见参考链接，实现了一个执行`nginx -h`时，对-h参数的解析返回，可以指定其他种类参数(-h为是否存在`h`的bool类型)
    - 定义 flags 有两种方式
        + `flag.Xxx()`，其中 Xxx 可以是 Int、String 等；返回一个相应类型的指针
            * `func Int(name string(标志名称), value int(默认值), usage string(使用提示)) *int(存放标志值) {xxx}`
            * e.g. `var ip = flag.Int("flagname", 1234, "help message for flagname")`
        + `flag.XxxVar()`，将 flag 绑定到一个变量上
            * `func IntVar(p *int(存放标志flag的值), name string, value int, usage string) {xxx}` 无返回值
            * e.g. `var flagvar int`, `flag.IntVar(&flagvar, "flagname", 1234, "help message for flagname")`
    - 在所有的 flag 定义完成之后，可以通过调用 `flag.Parse()` 进行*解析*，命令行flag的语法有如下三种形式
        + `-flag` // 只支持 bool 类型
        + `-flag=x`
        + `-flag x` // 只支持非 bool 类型

## govendor

* [govendor](https://github.com/kardianos/govendor)
    - `go get -u github.com/kardianos/govendor`
        + The -u flag instructs get to use the network to update the named packages
and their dependencies
    - Go modules 是Go1.11开始引入的(本机安装版本为go1.13.1)，并在Go1.11.2中修复提升，在Go1.12中有更好的表现
    - Go modules作为发布和构建工具，已经准备就绪可以立即使用。推荐在小项目和个人项目中使用Go modules

## 项目demo

* [xiaodongQ/douban-movie](https://github.com/xiaodongQ/douban-movie)
    - fork自：[go-crawler/douban-movie](https://github.com/go-crawler/douban-movie)


## 极客时间Go专栏

* main()
    - 不支持返回值
        + 可通过`os.Exit()`来返回(可指定返回错误码： `os.Exit(-1)`)
    - 不支持传入参数`func main(str []string)`是错误的
        + 若要获取命令行参数，使用`os.Args`来获取(是个数组，`len(os.Args)`, if len>1则第一个参数：`os.Args[1]`)
* 并发编程
    - 协程访问共享内存
        + 互斥`var mtx sync.Mutex`, `mtx.Lock()`, `mtx.Unlock()` 互斥锁，Unlock可以放到defer里去执行
        + 等待完成(而不用sleep去模拟)`var wg sync.WaitGroup`, `wg.Add(1)`, `wg.Done()`, `wg.Wait()` 
    - CSP并发机制
        + CSP：Communication Sequential Process （简称CSP）是著名计算机科学家C.A.R.Hoare为解决并发现象而提出的代数理论，是一个专为描述并发系统中通过消息交换进行交互通信实体行为而设计的一种抽象语言
        + 使用channel通道 `chan`类型
    - 多路选择和超时
        + `select {case xxx: xxx}`，结合 `<-time.After(时间)`类型的读channel，可以实现超时退出功能(加一个检查该channel的case)
    - `close(ch)`关闭channel，
        + 对读取channel的语句`xxx,ok:=<-ch1`判断返回值来确认是否已结束
        + 对写channel的语句，可用一个新的channel来辅助判断channel是否已`close`(向所有读取channel的地方广播)，用`select`先判断这个辅助channel是否关闭
            * 对各种情况的发送端和接收端(1发多收/多发1收/多发多收)，可参考下面链接中的示例
                - [有关Golang channel关闭的优雅方式](https://blog.csdn.net/studyhard232/article/details/88996434?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)
            * 示例中有使用到对channel的`for...range`语法：
                - Go提供了range关键字，将其使用在channel上时，会自动等待channel的动作一直到channel被关闭
                - for range 可以遍历通道（channel），但是通道在遍历时，只输出一个值，即管道内的类型对应的数据
        + 对于已关闭的channel进行操作的结果，搜索本笔记中的 "* `chan`" 章节(读取没事，发送和再次关闭会panic)
* 性能工具
    - 准备工作
        + 安装`graphviz`
            * [Welcome to Graphviz](http://graphviz.org/)
            * Graphviz是一个开源的图形可视化软件
            * 进入pprof后执行`svg`命令生成图表需要该工具
        + 下载并复制火焰图的`flamegraph.pl`到$GOPATH/bin
            * [火焰图](https://github.com/brendangregg/FlameGraph)
        + 安装`go-troch`：`go get github.com/uber/go-torch`
            * 会生成一个go-torch.exe(windows下)到$GOPATH/bin
    - 文件方式输出profile
        + 手动调用`runtime/pprof`的API
        + `go tool pprof [binary] [binary.prof]`

* disruptor 框架
    - [高性能队列——Disruptor](https://tech.meituan.com/2016/11/18/disruptor.html)
        + Disruptor是一个高性能的线程间通信的框架，即在同一个JVM进程中的多线程间消息传递,由LMAX开发
    - [高性能的消息框架 go-disruptor](https://colobu.com/2016/07/22/using-go-disruptor/)