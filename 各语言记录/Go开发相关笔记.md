## go

## Go文档

### 如何使用 Go 编程

* 官网FAQ：[Frequently Asked Questions (FAQ)](https://golang.org/doc/faq#closures_and_goroutines)
* 官网入门文档 [How to Write Go Code](https://golang.org/doc/code.html)
    - 中文版：[如何使用 Go 编程](https://go-zh.org/doc/code.html)
* 安装
    - [Install the Go tools](https://golang.org/doc/install#install)
    - 下载并解压到/usr/local，`tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz`
    - 添加环境变量(~/.bashrc或用zsh时：~/.zshrc)，`export PATH=$PATH:/usr/local/go/bin`
    - 测试`go version` 查看版本得到 `go version go1.13.5 linux/amd64`
    - CentOS使用yum安装
        + [使用yum安装Go(Golang)](https://cloud.tencent.com/developer/article/1478660)
        + 直接`yum install golang`，发现找不到包，则配置下源
            * `rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO`
            * `curl -s https://mirror.go-repo.io/centos/go-repo.repo | tee /etc/yum.repos.d/go-repo.repo`
        + `yum install golang`
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
        + 项目的目录结构
            * 参考：[目录结构](https://draveness.me/golang-101/)
            * 官方并没有给出一个推荐的目录划分方式，很多项目对于目录结构的划分也非常随意，这其实也是没有什么问题的，但是社区中还是有一些比较常见的约定
            * `/pkg`
                - Go语言项目中非常常见的目录，我们几乎能够在所有知名的开源项目（非框架）中找到它的身影
                - 这个目录中存放的就是项目中可以被外部应用使用的代码库，其他的项目可以直接通过 import 引入这里的代码，所以当我们将代码放入 pkg 时一定要慎重
                - 遵循公有和私有代码划分是非常好的做法，建议对项目中公有和私有的代码进行妥善的划分
                - 私有代码推荐放到 `/internal` 目录中，真正的项目代码应该写在 `/internal/app` 里，同时这些内部应用依赖的代码库应该在 `/internal/pkg` 子目录和 `/pkg` 中
                - 在*其他项目*引入包含 `internal` 的依赖时，Go 语言会在编译时报错
            * `/src`
                - 在 Go 语言的项目最不应该有的目录结构其实就是 `/src` 了
                - 最重要的原因其实是 Go 语言的项目在默认情况下都会被放置到 `$GOPATH/src` 目录下，这个目录中存储着我们开发和依赖的全部项目代码，如果我们在自己的项目中使用 `/src` 目录，该项目的 PATH 中就会出现两个 src(使用go modules没有代码须放`$GOPATH/src`下的限制)
                - 将代码平铺在根目录下也很正常，如要引用github.com/draveness/project/pkg/somepkg包，则在根目录使用目录层次：`github.com/draveness/project`
            * `/cmd`
                - 存储的都是当前项目中的可执行文件
                - 该目录下的每一个子目录都应该包含我们希望有的可执行文件，如我们的项目是一个 grpc 服务的话，/cmd/server/main.go 中就包含了启动服务进程的代码，编译后生成的可执行文件就是 server
                - 我们不应该在 /cmd 目录中放置太多的代码，我们应该将公有代码放置到 /pkg 中并将私有代码放置到 /internal 中并在 /cmd 中引入这些包，保证 main 函数中的代码尽可能简单和少
            * `/api`
                - 存放的就是当前项目对外提供的各种不同类型的 API 接口定义文件
                - 其中可能包含类似 /api/protobuf-spec、/api/thrift-spec 或者 /api/http-spec 的目录，这些目录中包含了当前项目对外提供的和依赖的所有 API 文件
            * Makefile 文件
                - 在任何一个项目中都会存在一些需要运行的脚本，这些脚本文件应该被放到 `/scripts` 目录中并由 Makefile 触发
            * 模块拆分
                - 按职责拆分。相对于Java和Ruby里按MVC框架划分为models、views 和 controllers形式的目录，Go语言在拆分模块时就使用了完全不同的思路，往往都按照职责对模块进行拆分
                - 如对于一个比较常见的博客系统，使用 Go 语言的项目会按照不同的职责将其纵向拆分成 post、user、comment 三个模块，每一个模块都对外提供相应的功能，post 模块中就包含相关的模型和视图定义以及用于处理 API 请求的控制器（或者服务）
                - Go 语言项目中的每一个文件目录都代表着一个独立的命名空间，也就是一个单独的包，当我们想要引用其他文件夹的目录时，首先需要使用 import 关键字引入相应的文件目录
                - 如果我们在Go语言中使用model、view和controller来划分层级，你会在其他的模块中看到非常多的 model.Post、model.Comment 和 view.PostView。这种划分层级的方法在Go语言中会显得非常冗余，并且如果对项目依赖包的管理不够谨慎时，很容易发生引用循环
            * [Standard Go Project Layout](https://github.com/golang-standards/project-layout)
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
                - 不需要提前担心冲突，在所有源码中不必要保持唯一，少数情况有冲突可以在导入时换个不同的名字
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
            * 带`defer`的函数返回，执行步骤
                - 执行return的值拷贝，将return语句返回的值拷贝到函数返回值的栈区(若return不返回值，则无操作)
                - 执行defer语句(LIFO/FILO)
                - 执行RET指令
    - 数据分配
        + `new`
            * `new(T)` 会为类型为 T 的新项分配*已置零*的内存空间，并返回它的地址，也就是一个类型为 `*T` 的值(指针)
            * 请注意，返回一个局部变量的地址完全没有问题，这点与 C 不同。 该局部变量对应的数据 在函数返回后依然有效。
            * 每当获取一个复合字面量的地址时，都将为一个新的实例分配内存
                - `return &File{fd, name, nil, 0}`
            * 表达式 `new(File)` 和 `&File{}` 是等价的
            * 指针对象复制
                - `T1 := &TestS{1}` 定义一个指针变量T1
                    + `T2 := &*T1` 貌似还是不行，还是会指向T1的存储位置，跟`T2:=T1`表现一样
                    + `T2 := T1` 并不定义一个和T1内容一样的新空间，而只是定义和T1指针值相等的指针，对T2成员修改也会影响T1
                    + 用这种方式：`T2 := &TestS{1}`，`*T2 = *T1`
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

* mutex
    - [Go语言互斥锁（sync.Mutex）和读写互斥锁（sync.RWMutex）](http://c.biancheng.net/view/107.html)
    - `sync.Mutex` 互斥锁
        + `countGuard.Lock()`
        + `countGuard.Unlock()`
    - `sync.RWMutex` 读写互斥锁
        + 读`RLock()` 和 `RUnlock()` 注意配套
        + 写`Lock()` 和 `Unlock()`


### go lint

* [Golint代码规范检测](https://blog.csdn.net/chenguolinblog/article/details/90665161)
* Golint 是一个源码检测工具用于检测代码规范
    - Golint 不同于gofmt, Gofmt用于代码格式化
* Golint会对代码做以下几个方面检查
    - package注释 必须按照 `Package xxx` 开头
    - package命名 不能有大写字母、下划线等特殊字符(上面的《Effective Go》中也做了该说明，搜`包名(Package names)`)
    - 不能使用下划线命名法，使用驼峰命名法
    - 外部可见程序结构体、变量、函数都需要注释
    - 通用名词要求大写
        + iD/Id -> ID
        + Http -> HTTP
        + Json -> JSON
        + Url -> URL
        + Ip -> IP
        + Sql -> SQL
    - 包命名统一小写不使用驼峰和下划线
    - 注释第一个单词要求是注释程序主体的名称，注释可选不是必须的
    - 外部可见程序实体不建议再加包名前缀
    - if语句包含return时，后续代码不能包含在else里面
    - `errors.New(fmt.Sprintf(…))` 建议写成 `fmt.Errorf(…)`
    - receiver名称不能为this或self
    - 错误变量命名需以 `Err/err` 开头
    - a+=1应该改成`a++`，a-=1应该改成`a--`

* [Go语言规范汇总](https://www.cnblogs.com/Survivalist/articles/10596152.html)
* 目录名
    - 规则：目录名必须为全小写单词，允许加中划线`-`组合方式，但是头尾不能为中划线
        + 建议：虽然允许出现中划线，但是尽量避免或少加中划线
* 文件名
    - 规则：文件名必须为小写单词，允许加下划线`_`组合方式，但是头尾不能为下划线
        + 建议：虽然允许出现下划线，但是尽量避免
        + 建议：文件名以功能为指引，名字中不需再出现模块名或者组件名
* 常量
    - 规则：常量、枚举名采用大小写混排的驼峰模式（Golang官方要求），不允许出现下划线
* 变量
    - 规则：变量名称一般遵循驼峰法，并且不允许出现下划线，当遇到特有名词时，需要遵循以下规则
        + 如果变量为私有，且特有名词为首个单词，则使用小写，如：`apiClient`
        + 其它情况都应当使用该名词原有的写法，如 APIClient、repoID、UserID
    - 在函数外部申明必须使用`var`,不要采用`:=`，容易踩到变量的作用域的问题
* 返回值
    - 规则：返回值如果是命名的，则必须大小写混排，首字母小写
        + 函数的返回值应避免使用命名的参数

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

```golang
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
    - `make(map[string]int)`
        + `make(map[string]int, 100)` 使用make新建一个空map，可同时指定容量
    - `len()`获取元素个数
    - nil map和空map，除了nil map不允许添加元素外，两者一样
    - [Golang教程：（十三）Map](https://blog.csdn.net/u011304970/article/details/75003344)
        + `cap()`不能用于求map
        + 创建和使用(*注意需要创建后才能使用*)

```golang
// 创建
map1 := make(map[string]string)
map2 := map[string]string{"lisi":"xxx"}  // 定义时初始化
// var map2 map[string]string{"lisi":"xxx"}  // 这一句的语法是不允许的!!!
// 插入
map1["zhangsan"] = "a"
map1["lisi"] = "b"
```

* 访问(key不存在时返回零值)

```golang
map1["lisi"] = "assign"             // map是引用类型
fmt.Printf("map:%s", map1["none"])  // 找不到的记录，map会返回零值(对不同类型对应零值是有区别的)
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

*  遍历

```golang
for key,value := range map1 {

}
```

* 比较
    - map 不能通过 `==` 操作符比较是否相等。`==` 操作符只能用来检测 map 是否为 `nil`
    - `if map1==map2` 是不允许的，只能比较它们的元素是否相等
* 使用map实现set (go中没有set基本类型)
    - map[anytype]bool
* 删除
    - `delete(map, key)` 如果 map为nil或者key不存在，该操作是一个空操作，不会产生错误
        + `delete`为内建操作
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

* [golang map源码详解](https://juejin.im/entry/5a1e4bcd6fb9a045090942d8)
    - go1.14版本，map实现文件：`runtime/map.go`
        + `runtime/map_fast64.go`/`runtime/map_fast32.go` 提供了一些快速操作map的函数
    - map的底层结构是`hmap`（即hashmap的缩写），核心元素是一个由若干个桶（bucket，结构为`bmap`）组成的数组，每个bucket可以存放若干元素（通常是8个），
        + key通过哈希算法被归入不同的bucket中
        + 当超过8个元素需要存入某个bucket时，hmap会使用extra中的overflow来拓展该bucket

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
        + `type PrintableMutex struct { Mutex }`，新的PrintableMutex类型包含`Lock()`方法(Mutex绑定到其匿名域了)
        + `type MyBlockInter BlockInter`，(BlockInter是一个interface)新的MyBlockInter类型包含原接口中的方法集
    - type声明可以给基本类型其别名，然后向它绑定一些方法
        + `type TimeZone int`, `func (tz TimeZone) String() string {return fmt.Sprintf("GMT+%dh", tz)}`
        + 传入的是时区值，返回格式化的string，String()方法绑定到了TimeZone类型上(内层函数引用了 外层函数中的变量/自由变量 的函数，闭包)
    - 注意区分 类型别名 和 类型定义
        + `type NewInt int` 将NewInt定义为int类型，定义了一个新类型
        + `type TypeAlias = Type` 将int取一个别名叫IntAlias，类型还是int
        + 在结构体成员嵌入时使用别名时，若同时存在原类型和类型别名，调用方法时若省略字段名，会提示冲突
        + [Go语言type关键字（类型别名）](http://c.biancheng.net/view/25.html)

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
            * 易错场景：
                - 错误方式演示(伪代码)：
                    + `var sList [][]*InfoList`，然后在一个循环中每次执行(总次数N)：
                        * 用append填充slice：`sList = append(sList, mem)`
                        * 起一个goroutine，传入地址 `go func(mem *[]*InfoList) { 对传入slice进行操作xxx }(&sList[i])`
                    + 对于原来的`sList`，append之后可能重新分配内存，成员地址可能变化了，导致最后sList中每个成员内容和预期并不一致
                - 正确方式：
                    + `make`的方式指定长度，`var sList = make([][]*InfoList, N)`，然后循环起goroutine
                    + 或者在循环外，先append N次，再循环内起goroutine
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
            * [Go 每日一库之 logrus](https://darjun.github.io/2020/02/07/godailylib/logrus/)
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

* logrus 使用
    - [sirupsen/logrus](https://github.com/Sirupsen/logrus)
    - Logrus是一个结构化的Go日志记录器(golang)，完全使用标准库日志记录器兼容的API
    - `go get github.com/sirupsen/logrus`
    - 打印示例`logrus.Printf("hellsdj, %d", 123)`：
        + `INFO[2020/06/29 16:26:52]F:/work/workspace/go_path/src/plate_calc_server/main.go:76 main.main() hellsdj, 123`
        + 记录方法名(包括文件名)会增加开销，可以通过benchmarks测试`go test -bench=.*CallerTracing`
        + 默认打印出来的文件名是绝对路径，太长了，可设置自定义的回调函数来修改路径，见下面的`CallerPrettyfier`字段
            * 变为：`INFO[2020/07/02 11:00:27]config.go:59 plate_calc_server/config.initLogger() hellsdj, 123`
    - 设置钩子hook(可扩展性及功能增强)
        + e.g. `logrus.AddHook(airbrake.NewHook(123, "xyz", "production"))`
        + 钩子需要实现logrus.Hook接口
            * `Levels() []Level` Levels()方法返回感兴趣的日志级别，输出其他日志时不会触发钩子
            * `Fire(*Entry) error` Fire是日志输出前调用的钩子方法
        + logrus的第三方 Hook 很多
            * mgorus：将日志发送到 mongodb；
            * logrus-redis-hook：将日志发送到 redis
            * logrus-amqp：将日志发送到 ActiveMQ
            * 可以用内置的hooks(syslog/writer)
                - [Writer Hooks for Logrus](https://github.com/sirupsen/logrus/tree/master/hooks/writer)
                - syslog是Unix/Linux才有
    - 实现日志切分(logrotate)
        + 日志输出到支持rotate的日志中(如syslog)，利用Linux系统的`logrotate`命令切分
        + 使用logrus的hook来加载输出模块，模块实现了日志rotate的`io.Writer`，如lumberjack
            * lumberjack是一个写入日志到滚动文件的Go包
                - [natefinch/lumberjack](https://github.com/natefinch/lumberjack)
            * `import "gopkg.in/natefinch/lumberjack.v2"`
            * 使用：`log.SetOutput(&lumberjack.Logger{xxx}`，见下面的`lumberjack使用示例`

```golang
logrus.SetFormatter(&logrus.TextFormatter{
    FullTimestamp:   true,
    TimestampFormat: "2006/01/02 15:04:05",
    ForceQuote:      true,
    ForceColors:     true, // 设置该选项会改变日志的格式，会打印普通日志而不是key-value的形式
    CallerPrettyfier: func(f *runtime.Frame) (string, string) {
        filename := path.Base(f.File)
        return fmt.Sprintf("%s()", f.Function), fmt.Sprintf("%s:%d", filename, f.Line)
    },
})

// logrus.SetFormatter(&logrus.JSONFormatter{
//     // 缩进显示json
//     PrettyPrint: true,
// })
// json格式时，可以加自定义的key(logrus.WithFields会创建一个*logrus.Entry，也可以创建一个全局Entry定义默认key)
// logrus.WithFields(logrus.Fields{
//     "xdk1": 123,
//     "xdk2": "abc",
// }).Errorf("测试: %s", "test only3")

// 保存到文件中
// file, err := os.OpenFile("logrus.log", os.O_CREATE|os.O_APPEND|os.O_RDWR, 0666)
// if err != nil {
//     log.Fatal(err)
// }
// logrus.SetOutput(file)

// 若不支持记录方法名，则更新一下logrus：go get github.com/sirupsen/logrus
log.SetReportCaller(true)

// 使用内置的hook，可同时记录到标准输出/错误等(还可设置成不同等级输出到不同途径)
logrus.AddHook(&writer.Hook{
    Writer:    os.Stdout,
    LogLevels: logrus.AllLevels,
})
```
* lumberjack使用示例

```golang
import lumberjack "gopkg.in/natefinch/lumberjack.v2"
// 切割文件格式:logrus-2020-07-02T14-44-37.485.log

logrus.SetOutput(&lumberjack.Logger{
    Filename:   "/var/log/myapp/foo.log",
    MaxSize:    500, // megabytes
    MaxBackups: 3,
    MaxAge:     28, //days
    Compress:   true, // disabled by default
    LocalTime:  true,
})
```

### viper

* [spf13/viper](https://github.com/spf13/viper)
    - `go get github.com/spf13/viper`
    - viper是一个完整的配置解决方案，用于Go应用程序，包括12个要素。它被设计为在应用程序中工作，并且可以处理所有类型的配置需求和格式。
    - 支持：
        + 设置默认值
            * `viper.SetDefault("ContentDir", "content")` 配置文件中没有时
        + 支持配置文件格式：JSON, TOML, YAML, HCL, envfile文件 和 Java properties配置文件
            * Viper搜索和读取配置文件：
                - `viper.SetConfigName("config")` 配置文件名称(不需要扩展名)
                    + `viper.SetConfigType("yaml")` 当配置文件名中没有扩展名时才需要设置
                - `viper.AddConfigPath("/etc/appname/")` 查找路径(可添加多次来搜索多个路径)
                    + 目前只支持一份配置文件(多个路径里仅包含一个配置文件)
                - `err := viper.ReadInConfig()` 读取配置文件
            * 写配置文件
                - `viper.WriteConfig()` 当前配置写到预定义读取到的路径文件中，没有预定义则报错。
                    + 若已存在配置文件，则覆盖。
                    + 如果不存在文件，则会在程序执行路径生成`SetConfigName`指定名称的文件(无后缀)，格式由`SetConfigType`指定
                - `viper.SafeWriteConfig()` 和上面一样，区别是若存在配置文件不会覆盖
                - `viper.WriteConfigAs("/path/to/my/.config")` 保存到指定文件，若存在则覆盖
                - `viper.SafeWriteConfigAs("/path/to/my/.config")` 和上面一样，区别是存在则不覆盖(报错)
        + 实时监测和重新读取配置文件(可选)
            * 程序运行中监测配置文件修改，开启功能，调用：`viper.WatchConfig()`
            * 可指定检测到修改时进行操作：`viper.OnConfigChange( func(e fsnotify.Event) {xxx} )`
            * 参考下面的示例：`监控配置文件实现日志运行时切换等级和打印格式`
        + 从环境变量读取变量
            * 大小写敏感
            * `viper.SetEnvPrefix("spf")` 设置环境变量前缀
                - `viper.BindEnv("id")`
                - `os.Setenv("SPF_ID", "13")` 外部设置一个环境变量
                - `id := viper.Get("id")` // 13 获取环境变量
        + 从远程配置系统(etcd/consul)读取，并且监测修改
            * 要开启远程支持，进入如下导入：`import _ "github.com/spf13/viper/remote"`
            * etcd示例
                - `viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001","/config/hugo.json")`
                - `viper.SetConfigType("json")`
                - `err := viper.ReadRemoteConfig()`
            * Consul示例
                - `viper.AddRemoteProvider("consul", "localhost:8500", "MY_CONSUL_KEY")`
                - `viper.SetConfigType("json")`
                - `err := viper.ReadRemoteConfig()`
                - `fmt.Println(viper.Get("port"))`
            * 监测
                - `var runtime_viper = viper.New()`
                - `runtime_viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001", "/config/hugo.yml")`
                - `runtime_viper.SetConfigType("yaml")`
                - 定期执行：
                    + `err := runtime_viper.WatchRemoteConfig()`
                    + `runtime_viper.Unmarshal(&runtime_conf)`
        + 从命令行参数读取
            * 可配合`Cobra`包使用，
                - github: [spf13/cobra](https://github.com/spf13/cobra)
                    + 很多Go项目在使用Cobra，包括Kubernetes, Docker, Hugo, and Github CLI等等
                - 它提供了简单的接口来创建命令行程序，同时，Cobra 也是一个应用程序，用来生成应用框架，从而开发以 Cobra 为基础的应用
                - 可了解：[Golang : cobra 包简介](https://www.cnblogs.com/sparkdev/p/10856077.html)
        + 从buffer中读取
            * `viper.ReadConfig(bytes.NewBuffer(yamlExample))`
                - `var yamlExample []byte(xxx)`，此处yamlExample是定义好的一串yaml格式的文本
                - `viper.SetConfigType("yaml")` 调用`viper.ReadConfig`前设置一下类型，此处`yaml`可替换为`YAML`
            * 读取：e.g. `viper.Get("name")`
            * 设置：e.g. `viper.Set("Verbose", true)`
            * 设置别名：e.g. `viper.RegisterAlias("loud", "verbose")`，后续操作
        + 设置明确的值
    - 读取
        + `Get(key string) : interface{}`
        + `GetBool(key string) : bool`
        + `GetFloat64(key string) : float64`
        + `GetInt(key string) : int`
        + `GetIntSlice(key string) : []int`
        + `GetString(key string) : string`
        + `GetStringMap(key string) : map[string]interface{}`
        + `GetStringMapString(key string) : map[string]string`
        + `GetStringSlice(key string) : []string`
        + `GetTime(key string) : time.Time`
        + `GetDuration(key string) : time.Duration`
        + `IsSet(key string) : bool`
        + `AllSettings() : map[string]interface{}`
    - 可读取嵌套的域：`GetString("datastore.metric.host")`
        + 若层级中同时有该域，则优先`"datastore.metric.host"`，层级中的域会被隐藏
    - 截取子域：`subv := viper.Sub("app.cache1")`
    - `err := viper.Unmarshal(&C)` 全部或指定值解码转换为struct、map等
    - `bs, err := yaml.Marshal(c)` 编码为指定格式的string
        + 导入对应格式的包，此处为`import yaml "gopkg.in/yaml.v2"`

```golang
// 监控配置文件实现日志运行时切换等级和打印格式
viper.WatchConfig()
viper.OnConfigChange(func(e fsnotify.Event) {
    logrus.Printf("config file[%s] changed, op:%s", e.Name, e.Op.String())
    // 当前只监控日志等级
    // 日志等级
    loglevel := logrus.InfoLevel
    switch viper.GetString("log.level") {
    case "error":
        loglevel = logrus.ErrorLevel
    case "info":
        loglevel = logrus.InfoLevel
    case "debug":
        loglevel = logrus.DebugLevel
    case "trace":
        loglevel = logrus.TraceLevel
    default:
        loglevel = logrus.InfoLevel
    }
    if logrus.GetLevel() != loglevel {
        logrus.SetLevel(loglevel)
    }
    // 是否开启文件名和函数名记录
    logrus.SetReportCaller(viper.GetBool("log.PrintFileFunc"))
})
```

### os包

* os包
    - [golang中os包用法](https://blog.csdn.net/chenbaoke/article/details/42494851)
    - os包中实现了平台无关的接口，设计向Unix风格，但是错误处理是go风格
    - 部分函数
        + `func Chdir(dir string) error` 设置dir目录为当前目录
        + `func Getwd() (dir string, err error)` 类似`pwd`
        + `func Chown(name string, uid, gid int) error` 修改文件所有者
        + `func Getpid() int` 获取进程id
        + `func Getgid() int` 组id
        + `func Hostname() (name string, err error)` 获取主机名
        + `func Link(oldname, newname string) error` 创建硬连接
            * `func Symlink(oldname, newname string) error` 创建软连接
        + `func IsPathSeparator(c uint8) bool` 查看是否为路径分隔符号
            * 在不同系统上，路径分隔符可能有差异(linux:`/`, windows:`\`)
            * `if os.IsPathSeparator('\\') {...}` 在该系统下是路径分隔符则返回true
        + `func Mkdir(name string, perm FileMode) error` 创建目录
            * FileMode参数指定权限
            * 创建一个已经存在的目录时会报错
            * `func MkdirAll(path string, perm FileMode) error` 存在目录时，该函数创建不会报错
        + `func Remove(name string) error` 删除文件或目录
            * `func RemoveAll(path string) error` 删除目录及子目录和文件
        + 文件操作
            * `func Create(name string) (file *File, err error)` 
                - 创建一个文件，文件mode是0666(读写权限)，
                - 如果文件已经存在，则重新创建一个，原文件被覆盖，创建的新文件具有读写权限，其相当于`OpenFile`的快捷操作，
                    + 等同于`OpenFile(name string, O_CREATE, 0666)`
            * `func Open(name string) (file *File, err error)`
                - 打开一个文件，返回文件描述符，该文件描述符只有只读权限．
                - 相当于`OpenFile(name string, O_RDWR, 0)`
            * `func OpenFile(name string, flag int, perm FileMode) (file *File, err error)`
                - 指定文件权限和打开方式打开name文件或者create文件
                - flag标志如下:
                    + O_RDONLY：只读模式(read-only)
                    + O_WRONLY：只写模式(write-only)
                    + O_RDWR：读写模式(read-write)
                    + O_APPEND：追加模式(append)
                    + O_CREATE：文件不存在就创建(create a new file if none exists.)
                    + `O_EXCL`：与 `O_CREATE` 一起用，构成一个新建文件的功能，它要求文件必须不存在(used with O_CREATE, file must not exist)
                    + O_SYNC：同步方式打开，即不使用缓存，直接写入硬盘
                    + O_TRUNC：打开并清空文件
                - 至于操作权限perm，除非创建文件时才需要指定，不需要创建新文件时可以将其设定为0
                    + 虽然go语言给perm权限设定了很多的常量，但是习惯上也可以直接使用数字，如0666(具体含义和Unix系统的一致)
            * `func (f *File) Close() error` 关闭文件，使其不能够再进行i/o操作，其经常和defer一起使用
            * `func (f *File) Fd() uintptr` 返回系统文件描述符，也叫做文件句柄
            * `func (f *File) Name() string` 返回文件名字，与`file.Stat().Name()`等价
            * `func (f *File) Read(b []byte) (n int, err error)` 
                - 从文件f中读取len(b)长度的数据到b中，如果无错，则返回n和nil,否则返回读取的字节数n以及响应的错误
                - `func (f *File) ReadAt(b []byte, off int64) (n int, err error)` 和`Read()`类似，可指定读取的开始位置
            * `func (f *File) Write(b []byte) (n int, err error)` 将b中的数据写入f文件
                - `func (f *File) WriteAt(b []byte, off int64) (n int, err error)` 从指定位置写入
            * `func (f *File) WriteString(s string) (ret int, err error)` 将字符串s写入文件
        + 进程操作
            * `func FindProcess(pid int) (p *Process, err error)` 通过进程pid查找运行的进程，返回相关进程信息及error
            * `StartProcess(name string, argv []string, attr *ProcAttr) (*Process, error)` 启动一个新的进程
                - StartProcess是一个低层次的接口。os/exec包提供了高层次的接口
            * `func (p *Process) Kill() error` 杀死进程并直接退出
            * `func (p *Process) Release() error` 释放进程p的所有资源
            * `func (p *Process) Signal(sig Signal) error` 发送一个信号给进程p
            * `func (p *Process) Wait() (*ProcessState, error)` 等待进程退出，并返回进程状态和错误

### bytes包

* `bytes.Buffer`
    - [Golang bytes.Buffer 用法](https://blog.csdn.net/k346k346/article/details/94456479)
    - `bytes.Buffer` 是 Golang 标准库中的缓冲区，具有读写方法和可变大小的字节存储功能。
    - 常用方法
        + 声明
            * `var b bytes.Buffer`
            * `b := new(bytes.Buffer)`
            * `b := bytes.NewBuffer(s []byte)` 从一个`[]byte`切片，构造一个Buffer
            * `b := bytes.NewBufferString(s string)` 从一个string变量，构造一个Buffer
        + 往 Buffer 中写入数据
            * `b.Write(d []byte) (n int, err error)` 将切片写入buffer，返回d切片长度，err总是nil，若buffer过长则会产生panic
            * `b.WriteString(s string) (n int, err error)` 返回string长度，err总是nil
            * `b.WriteByte(c byte) error` 将字符c写入Buffer尾部
            * `b.WriteRune(r rune) (n int, err error)` 将一个rune类型的数据放到缓冲区的尾部
            * `b.ReadFrom(r io.Reader) (n int64, err error)` 从`r`里面读取数据放到buffer尾部
        + 从 Buffer 中读取数据
            * `b.Next(n int) []byte` 读取 n 个字节数据并返回，如果 buffer 不足 n 字节，则读取全部
            * `b.Read(p []byte) (n int, err error)`
                - 一次读取 len(p) 个 byte 到 p 中，每次读取新的内容将覆盖p中原来的内容。
                - 成功则返回实际读取的字节数，off 向后偏移 n，buffer 没有数据返回错误 io.EOF
            * `b.ReadByte() (byte, error)` 读取第一个byte并返回，off 向后偏移 n
            * `b.ReadRune() (r rune, size int, err error)` 读取第一个 UTF8 编码的字符并返回该字符和该字符的字节数
            * `b.ReadBytes(delimiter byte) (line []byte, err error)` 读取缓冲区第一个分隔符前面的内容以及分隔符并返回
            * `b.ReadString(delimiter byte) (line string, err error)` 读取缓冲区第一个分隔符前面的内容以及分隔符
            * `b.WriteTo(w io.Writer) (n int64, err error)` 将 Buffer 中的内容输出到`w`可写对象
        + `b.Bytes() []byte` 返回字节切片，`b.Len() int` 等于len(b.Bytes())
        + `b.Cap() int` 返回 buffer 内部字节切片的容量
        + `b.Grow(n int)` 为 buffer 内部字节切片的容量增加 n 字节
        + `b.Reset()` 清空数据
        + `b.String() string` 字符串化
        + `b.Truncate(n int)` 丢弃缓冲区中除前n个未读字节以外的所有字节，n为负或超出缓冲区长度，则panic
        + `b.UnreadByte() error` 将最后一次读取操作中被成功读取的字节设为未被读取的状态，即将已读取的偏移 off 减 1
        + `b.UnreadRune() error` 将最后一次ReadRune()读取操作返回的 UTF8 字符 rune设为未被读取的状态，off减去字符 rune 的字节数

* `bytes.NewReader`

### sync包

#### sync.WaitGroup

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


#### sync.Map

* [Go语言sync.Map（在并发环境中使用的map）](http://c.biancheng.net/view/34.html)
    - Go语言中的 map 在并发情况下，只读是线程安全的，同时读写是线程不安全的
    - 需要并发读写时，一般的做法是加锁，但这样性能并不高，Go语言在 1.9 版本中提供了一种效率较高的并发安全的 `sync.Map`，sync.Map和map不同，不是以语言原生形态提供，而是在 sync 包下的特殊结构
    - `sync.Map` 有以下特性：
        + 无须初始化，直接声明即可
        + `sync.Map` 不能使用 map 的方式进行取值和设置等操作，而是使用 sync.Map 的方法进行调用
            * `Store` 表示存储，`Load` 表示获取，`Delete` 表示删除
        + 使用 `Range` 配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值，Range 参数中回调函数的返回值在需要继续迭代遍历时，返回 true，终止迭代遍历时，返回 false
    - sync.Map 没有提供获取 map 数量的方法，替代方法是在获取 sync.Map 时遍历自行计算数量，sync.Map 为了保证并发安全有一些性能损失，因此在非并发情况下，使用 map 相比使用 sync.Map 会有更好的性能

```golang
package main
import (
    "fmt"
    "sync"
)

func main() {
    var scene sync.Map
    // 将键值对保存到sync.Map
    scene.Store("greece", 97)
    scene.Store("london", 100)
    scene.Store("egypt", 200)
    // 从sync.Map中根据键取值
    fmt.Println(scene.Load("london"))
    // 根据键删除对应的键值对
    scene.Delete("london")
    // 遍历所有sync.Map中的键值对
    scene.Range(func(k, v interface{}) bool {
        fmt.Println("iterate:", k, v)
        return true
    })
}
```

#### sync.Pool

* [深入分析Golang sync.pool](http://myself659.github.io/post/golang-sync-pool-1/)
    - sync.Pool是一个可以存或取的临时对象池。对外提供New、Get、Put等API，利用mutex支持多线程并发
    - sync.Pool解决以下问题：
        + 增加临时对象的用复用率，减少GC负担
        + 通过对象的复用，减少内存申请开销，有利于提高一部分性能
    - 操作
        + `New` 定义一个函数并赋值给`New`
            * 当对象池中为空，则会通过指定的函数来分配对象
            * e.g. `var pool = sync.Pool{  New: func() interface{} { return new(Object) }, }`
        + `Get`
            * Get解决了如何从具体sync.Pool中获取对象的问题
            * 获取对象有三个来源：
                - private池
                - shared池
                - 系统的Heap内存
            * 获取对象顺序是先从private池获取，如果不成功则从shared池获取，如果继续不成功，则从Heap中申请一个对象。
                - 先从本P绑定的poolLocal获取对象：先从本poolLocal的private池获取对象，再从本poolLocal的shared池获取对象
                - 上一步没有成功获取对象，再从其他P的shared池获取对象
                - 上一步没有成功获取对象，则从Heap申请对象
        + `Put`
            * Put完成将对象放回对象池
                - 如果poolLocalInternal的private为空，则将回收的对象放到private池中
                - 如果poolLocalInternal的private非空，则将回收的对象放到shared池中
        + `CleanUp`
            * pool.go里的`init()`会注册`poolCleanup`函数
            * CleanUp时机：在gc开始前，进行CleanUp回收对象池
                - `func gcStart(trigger gcTrigger)`中调用到的`clearpools()`就是清理sync.Pools
            * `clearpools()`
                - 如果`poolcleanup`不为空，调用`poolcleanup`函数，`poolcleanup`即`init`中注册的清理函数
    - 应用
        + sync.Pool并不是万能药。要根据具体情境而定是否使用sync.Pool
        + 总结不适合使用sync.Pool的情境
            * 对象中分配的系统资源如socket，buffer
            * 对象需要进行异步处理
            * 对象是组合对象，如存在指针指向其他的对象
            * 批量对象需要并发处理
            * 复用对象大小存在的波动，如对象结构成员存在slice
        + 在排除上面情境下，适合使用的sync.Pool应满足以下条件
            * 对象是buffer或非组合类型如buffer reader, json decode, bufio writer
            * 对象内存可以重复使用
        + 同时在使用应该注意问题：
            * Put对象之前完成初始化，避免数据污染带来问题, 这可能带来各种各样的问题
            * 写代码时要满足one Get， one Put的要求
            * 注意获取对象后是否存在修改对象内存布局的代码
            * 关注应用场景是否容易出现Pool竞争的情况
            * sync.Pool不是万能药，不要拿着锤子，看什么都是钉子

```golang
// sync/pool.go
// go.1.14(而1.13之前的版本如1.12.7，poolLocalInternal中还定义了一个Mutex，且无poolChain)
type Pool struct {
    // 防止sync.Pool被复制
    noCopy noCopy

    // poolLocal数组的指针
    local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal
    // poolLocal数组大小
    localSize uintptr        // size of the local array

    // go1.12中没有，go1.13之后有
    // 前一个周期的局部数据
    victim     unsafe.Pointer // local from previous cycle
    victimSize uintptr        // size of victims array

    // New optionally specifies a function to generate
    // a value when Get would otherwise return nil.
    // It may not be changed concurrently with calls to Get.
    // 函数指针申请具体的对象，便于用户定制各种类型的对象
    New func() interface{}
}

// Local per-P Pool appendix.
type poolLocalInternal struct {
    // private私有池，只能被对应P使用
    private interface{} // Can be used only by the respective P.
    // shared共享池，能被任何P使用
    shared  poolChain   // Local P can pushHead/popHead; any P can popTail.
}

type poolLocal struct {
    poolLocalInternal

    // Prevents false sharing on widespread platforms with
    // 128 mod (cache line size) = 0 .
    // 防止伪共享，组成cpu cache line的倍数(x86环境一般为64)
    pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte
}
```

```golang
// sync/pool.go
// go.1.12.7
type poolLocalInternal struct {
    private interface{}   // Can be used only by the respective P.
    shared  []interface{} // Can be used by any P.
    // 保护shared共享池
    Mutex                 // Protects shared.
}
```

#### sync.Once

* [sync.Once惰性初始化](https://books.studygolang.com/gopl-zh/ch9/ch9-05.html)
    - 如果初始化成本比较大的话，那么将初始化延迟到需要的时候再去做就是一个比较好的选择。如果在程序启动的时候就去做这类初始化的话，会增加程序的启动时间，并且因为执行的时候可能也并不需要这些变量，所以实际上有一些浪费。
    - 使用方式
        + `var once sync.Once`
        + `once.Do(func() {xxx})`，传入一个初始化函数给`once.Do`即可

```golang
// sync/once.go
type Once struct {
    // done indicates whether the action has been performed.
    // It is first in the struct because it is used in the hot path.
    // The hot path is inlined at every call site.
    // Placing done first allows more compact instructions on some architectures (amd64/x86),
    // and fewer instructions (to calculate offset) on other architectures.
    done uint32
    m    Mutex
}

// 调用Do时的逻辑
func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 0 {
        // Outlined slow-path to allow inlining of the fast-path.
        o.doSlow(f)
    }
}

// 加锁检查变量 done 是否为0，是则执行传入的函数，并最后置1。后续再调用就不会再执行函数了
func (o *Once) doSlow(f func()) {
    o.m.Lock()
    defer o.m.Unlock()
    if o.done == 0 {
        defer atomic.StoreUint32(&o.done, 1)
        f()
    }
}
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

```golang
// 判断通道是否关闭1
func isCancelled(cancelChan chan struct{}) bool {
    select {
    case <-cancelChan:
        return true
    default:
        return false
    }
}

// 判断通道是否关闭2，推荐
func isCancelled(ctx context.Context) bool {
    select {
    case <-ctx.Done():
        return true
    default:
        return false
    }
}
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

* `go test`
    - [go test命令（Go语言测试命令）完全攻略](http://c.biancheng.net/view/124.html)
    - Go语言拥有一套单元测试和性能测试系统，仅需要添加很少的代码就可以快速测试一段需求代码
        + 单元测试（unit testing），是指对软件中的最小可测试单元进行检查和验证
    - **单元测试**
        + `func TestXxx (t *testing.T)`
        + 要求：
            * 1. 文件名必须以`xxx_test.go`形式命名(`_test.go`结尾)
            * 2. 方法必须是`Test`开头(`Test[^a-z]`，即Test后面接的首字母不能是小写)
            * 3. 方法参数必须 `t *testing.T`
        + 参数
            * 默认的情况下，go test命令不需要任何的参数，它会自动把你源码包下面所有 test 文件测试完毕，当然也可以带上参数，列出几个常用参数：
            * `-bench regexp` 执行相应的 benchmarks，例如 `-bench=.`；
            * `-cover` 开启测试覆盖率；
            * `-run regexp` 只运行 regexp 匹配的函数，例如 -run=Array 那么就执行包含有 Array 开头的函数；
            * `-v` 显示测试的详细命令
        + 测试用例文件不会参与正常源码编译，不会被包含到可执行文件中
        + 测试用例文件使用`go test`指令来执行，没有也不需要 main() 作为函数入口
            * `go test helloworld_test.go`
            * `go test -run=TestPrintInfo`
                - `-run=Array` 也会执行包含有 Array 开头的函数，若要完全匹配，则用`-run=Array$`
            * `go test -run TestA`
                - 注意此处会匹配`TestA`开头的所有单元测试用例，支持正则表达式，`TestA$`则只测试`TestA`
                - 也可以同时指定哪个文件：`go test -run TestA func_test.go`，
                    + 如果`_test.go`文件会调用别的文件则需要一并带上，放`_test.go`文件后面
            * 所有在以_test结尾的源码内以Test开头的函数会自动被执行
        + 每个测试用例可能*并发执行*，使用 `testing.T` 提供的日志输出可以保证日志跟随这个测试上下文一起打印输出
            * `t.Log/t.Logf/t.Error/t.Errorf/t.Fatal/t.Fatalf`
        + 测试用例时，可以不传入 *testing.T 参数(不是指定义时不传)
    - **基准测试**
        + 可以测试一段程序的运行性能及耗费 CPU 的程度
        + Go语言中提供了基准测试框架，使用方法类似于单元测试，使用者无须准备高精度的计时器和各种分析工具，基准测试本身即可以打印出非常标准的测试报告
        + `func BenchmarkXXX(b *testing.B)`
        + 要求：
            * 文件名以 `_test.go`结尾
            * 方法名以 `Benchmark` 开头，后面的首字母不能小写(和单元测试类似)
            * 方法参数为 `b *testing.B`
        + `go test -v -bench=. benchmark_test.go` 开启基准测试
            * `-bench=.` 表示文件里的所有基准测试，类似单元测试的`-run`参数
                - `go test`不会默认执行基准/压力测试的函数，如要要执行，加上`-bench`参数
                - **注意**：Windows下使用时，写为`-bench="."`，否则测试不到用例
                - 和`-run`类似，`-bench 正则表达式`可进行方法名匹配
            * 结果示例：`BenchmarkAdd-12         20000000               130 ns/op`
                - 各列分别为：
                - `BenchmarkAdd-12` 表示基准测试名称、
                - `20000000` 表示测试的次数(即testing.B 结构中提供给程序使用的 N)、
                - `130 ns/op` 表示每一个操作耗费多少时间（纳秒）
            * 测试代码需要保证函数可重入性及无状态，也就是说，测试代码不使用全局变量等带有记忆性质的数据结构。
                - 避免多次运行同一段代码时的环境不一致，不能假设`b.N`(框架提供，`b *testing.B`为入参)值范围
        + 测试时间
            * 基准测试框架对一个测试用例的默认测试时间是 1 秒，开始测试时，当以 Benchmark 开头的基准测试用例函数返回时还不到 1 秒，那么 testing.B 中的 N 值将按 1、2、5、10、20、50……递增，同时以递增后的值重新调用基准测试用例函数
            * 可以用`-benchtime`参数自定义指定基准测试的时间
        + 内存分配情况
            * 加上`-benchmem`参数
            * `go test -v -bench=Alloc -benchmem benchmark_test.go`
                - `-bench`后添加了 Alloc，指定只测试`Benchmark_Alloc()`函数(也可以跟上面一样不指定函数和文件)
            * 结果示例：`BenchmarkAdd-12    10000000    111 ns/op   126 B/op  1 allocs/op`
                - `126 B/op` 表示一次分配126个字节
                - `1 allocs/op` 表示每次调用有一次内存分配
            * 开发者根据这些信息可以迅速找到可能的分配点，进行优化和调整
        + 控制计时开始和结束
            * 如果从 `Benchmark()` 函数开始计时会很大程度上影响测试结果的精准性。testing.B 提供了一系列的方法可以方便地控制计时器，从而让计时器只在需要的区间进行测试
                - `b.ResetTimer()` 重置计时器
                - `b.StopTimer()` 停止计时器
                - `b.StartTimer()` 开始计时器
            * 没搞清楚作用，执行前reset然后开始，测出的时间比不调用这些设置还长 *TODO*
    - gin里面集成pprof
        + `go get github.com/DeanThompson/ginpprof`，集成时直接`ginpprof.Wrap(router)`即可
            * 方法签名为：`func Wrap(router *gin.Engine)`
            * 打开链接即可看到pprof相关信息：`http://localhost:port/debug/pprof/`
        + [使用google的pprof工具以及在gin中集成pprof](https://www.cnblogs.com/TimLiuDream/p/10038239.html)

* go test单元测试用例及执行 示例：`go test -v -run=TestOKWSAgent_Login`

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

```golang
// net/http/pprof包的init已经定义了几个http endpoint页面
func init() {
    http.HandleFunc("/debug/pprof/", Index)
    http.HandleFunc("/debug/pprof/cmdline", Cmdline)
    http.HandleFunc("/debug/pprof/profile", Profile)
    http.HandleFunc("/debug/pprof/symbol", Symbol)
    http.HandleFunc("/debug/pprof/trace", Trace)
}
```

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

* 计时统计
    - `start := time.Now()`
    - `cost := time.Since(start)`
        + 签名：`func Since(t Time) Duration`

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
    - strings包
    - strconv包

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

### strings包 和 strconv包

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
// string到int
int,err:=strconv.Atoi(s)

// string到int64
int64, err := strconv.ParseInt(s, 10, 64)

// int到string
string:=strconv.Itoa(i)

// int64到string
string:=strconv.FormatInt(i, 10)

// string到float
v1, err := strconv.ParseFloat(f, 32)
```


## net包

* net 包
    - net包提供编写一个网络客户端或者服务器程序的基本组件，无论两者间通信是使用TCP，UDP或者Unix domain sockets
        + net/http包里的方法，也算是net包的一部分

# Go语言标准库

* [《Go语言标准库》The Golang Standard Library by Example](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/)

## flag包

* flag 包
    - 参考：[1. 13.1 flag - 命令行参数解析](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter13/13.1.html#131-flag---%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0%E8%A7%A3%E6%9E%90)
    - flag 包实现了命令行参数的解析(使用命令行执行程序，传入参数)
        + 示例见参考链接，实现了一个执行`nginx -h`时，对-h参数的解析返回，可以指定其他种类参数(-h为是否存在`h`的bool类型)
    - 定义 flags 有两种方式
        + `flag.Xxx()`，其中 Xxx 可以是 Int、String 等；返回一个相应类型的指针
            * `func Int(name string(标志名称), value int(默认值), usage string(使用提示)) *int(返回存放标志值) {xxx}`
            * e.g. `var ip = flag.Int("flagname", 1234, "help message for flagname")`
        + `flag.XxxVar()`，将 flag 绑定到一个变量上
            * `func IntVar(p *int(存放标志flag的值), name string(名称), value int(默认值), usage string(提示)) {xxx}` 无返回值
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
* Go Modules
    - [Go Modules 终极入门](https://mp.weixin.qq.com/s/fNMXfpBhBC3UWTbYCnwIMg)
    - Go modules 是 Go 语言中正式官宣的项目依赖解决方案
        + Go modules（前身为vgo）于 Go1.11 正式发布，在 Go1.14 已经准备好，并且可以用在生产上（ready for production）了，Go 官方也鼓励所有用户从其他依赖项管理工具迁移到 Go modules
    - Go1.11 后就开始逐步建议使用 Go modules，不再推荐 GOPATH 的模式
        + GOPATH 模式下没有版本控制的概念，具有致命的缺陷，至少会造成以下问题
            * 在执行go get的时候，你无法传达任何的版本信息的期望，也就是说你也无法知道自己当前更新的是哪一个版本，也无法通过指定来拉取自己所期望的具体版本
            * 在运行 Go 应用程序的时候，你无法保证其它人与你所期望依赖的第三方库是相同的版本，也就是说在项目依赖库的管理上，你无法保证所有人的依赖版本都一致
            * 你没办法处理 v1、v2、v3 等等不同版本的引用问题，因为 GOPATH 模式下的导入路径都是一样的，都是github.com/foo/bar
        + Go 语言官方从 Go1.11 起开始推进 Go modules（前身vgo），Go1.13 起不再推荐使用 GOPATH 的使用模式，Go modules 也渐趋稳定，因此新项目也没有必要继续使用GOPATH模式
    - go get下载不了包问题的解决方式
        + GOPROXY 的默认值是：https://proxy.golang.org,direct，
        + 这有一个很严重的问题，就是 proxy.golang.org 在国内是无法访问的，因此这会直接卡住你的第一步，所以你必须在开启 Go modules 的时，同时设置国内的 Go 模块代理，执行如下命令：
            * `go env -w GOPROXY=https://goproxy.cn,direct`
                - `https://goproxy.io`代理也可以
            * `GOSUMDB`
                - `GOSUMDB`用于在拉取模块版本时（无论是从源站拉取还是通过 Go module proxy 拉取）保证拉取到的模块版本数据未经过篡改，若发现不一致，也就是可能存在篡改，将会立即中止
                - 官方默认是GOSUMDB="sum.golang.org"，goproxy.cn 支持代理 sum.golang.org
                - 所以在设置 `GOPROXY` 后，可以不用关心该值
            * go 1.13中建议Go相关的环境变量由`go env -w`(Go1.13新增)来管理，设置后，会在`$HOME/.config/go/env`文件中新增指定的配置项
                - `go env -w`不会覆盖系统环境变量，如果环境变量中设置了某个选项(e.g. GOPATH)，则执行`go env -w GOPATH=xxx`时会报错：warning: go env -w GOPATH=... does not override conflicting OS environment variable
                - 建议将环境变量中关于GO的设置去掉，统一调整为`go env -w`来设置
            * 下载的包会在GOPATH路径(不过和原来的目录组织方式差别很大，只是包存放在该路径)，所以可以按需设置一下GOPATH
        + 了解goproxy.cn，可参考：[Go Modules 和 goproxy.cn](https://blog.csdn.net/EDDYCJY/article/details/102426277)
            * 迁移项目至Go Modules
                - 在项目路径初始化Modules，`go mod init xxx`，然后`go get -u`更新现有依赖即可(如果要加单元测试则加`-t`选项)
                    + 可参考下面章节里的选项说明： 搜 "* `go get` 拉取依赖"
                - 其他命令见下面的说明
    - Go Modules基本使用
        + 命令
            * `go mod init` 生成 go.mod 文件
            * `go mod download` 下载 go.mod 文件中指明的所有依赖
            * `go mod graph` 查看现有的依赖结构
            * `go mod edit` 编辑 go.mod 文件
            * `go mod tidy` `go mod vendor` `go mod verify` `go mod why`
        + 所提供的环境变量
            * `GO111MODULE` 作为 Go modules 的开关
                - `auto` 只要项目包含go.mod就启用Go modules，目前在 Go1.11 至 Go1.14 中仍然是默认值
                - `on` 启用 Go modules，推荐设置，将会是未来版本中的默认值
                - `off` 禁用 Go modules
                - `go env -w GO111MODULE=on`
            * `GOPROXY` `GOSUMDB` `GONOPROXY`/`GONOSUMDB`/`GOPRIVATE`
        + 参考链接中从头开始创建一个 Go modules 的项目，原则上所创建的目录应该不要放在 GOPATH 之中
    - 某个包不使用go module指定的版本或包，可进行降级或指定本地已有包的路径(老代码使用新版本出现目录等不兼容问题时)
        + `go.mod`文件中修改，e.g.
            * require中新增：`github.com/gogf/gf v1.13.1`
            * require外面新增replace，并指定本地包路径：`replace github.com/gogf/gf => F:/work/workspace/go_path/src/github.com/gogf/gf`
    - 示例(在一个非`$GOPATH/src`路径进行)
        + github上随便找的一个string工具包，[github.com/ozgio/strutil](https://github.com/ozgio/strutil)
            * `Words (Docs)`函数，返回文本中的单词
            * e.g. `strutil.Words("Lorem ipsum, dolor sit amet") //-> []string{"Lorem", "ipsum", "dolor", "sit", "amet"}`
        + a. 初始化项目 `go mod init xddemo/strutil` (注意这个路径不是要导入的package路径，下面的模块即指package)
            * 会生成一个go.mod文件，内容为(两行)："module xddemo/strutil", "go 1.14"
            * 指定了模块导入路径为 `xddemo/strutil`，若要import的包包含本项目的模块，则import的时候要带上该路径，e.g.`import "xddemo/strutil/utils"`
        + b. 在该项目根目录下创建 main.go，import之前导入的模块
            * `import ("github.com/ozgio/strutil")`
        + c. 获取要import的包，查看go.mod及go.sum文件
            * 项目根目录执行命令拉取模块：`go get github.com/ozgio/strutil`
                - 拉取的结果缓存在：`$GOPATH/pkg/mod`和`$GOPATH/pkg/sumdb`
                - 在mod目录下会以 github.com/foo/bar 的格式进行存放
                    + 目录名为 `模块名@版本`的形式，本示例中路径为`$GOPATH\pkg\mod\github.com\ozgio\strutil@v0.3.0`
                    + 需要注意的是同一个模块版本的数据只缓存一份，所有其它模块共享使用
                    + 若希望清理所有已缓存的模块版本数据，可以执行 `go clean -modcache` 命令
                    + `go get`没有指定任何的版本信息，则由 Go modules 自行按照内部规则进行选择
                        * 规则
                            - 所拉取的模块有发布 tags，如果只有单个模块，那么就取主版本号最大的那个tag；如果有多个模块，则推算相应的模块路径，取主版本号最大的那个tag
                            - 所拉取的模块没有发布过 tags，默认取主分支最新一次 commit 的 commithash
                        * 所拉取的模块有发布 tags，此处获取到最新的版本tag: v0.3.0
                        * `go get` 拉取依赖，会进行指定性拉取（更新），并不会更新所依赖的其它模块
                        * `go get -u` 更新现有的依赖，会强制更新它所依赖的其它全部模块，不包括自身
                        * `go get -u -t ./...` 更新所有直接依赖和间接依赖的模块版本，包括单元测试中用到的
                    + 选择具体版本
                        * `go get golang.org/x/text@latest` 拉取最新的版本，若存在tag，则优先使用
                        * `go get golang.org/x/text@master` 拉取 master 分支的最新 commit
                        * `go get golang.org/x/text@v0.3.2` 拉取 tag 为 v0.3.2 的 commit
                        * `go get golang.org/x/text@342b2e` 拉取 hash 为 342b231 的 commit，最终会被转换为 v0.3.2
            * 查看go.mod文件，多了一行：`require github.com/ozgio/strutil v0.3.0 // indirect`
                - 其详细罗列了当前项目直接或间接依赖的所有模块版本，并写明了那些模块版本的 SHA-256 哈希值以备 Go 在今后的操作中保证项目所依赖的那些模块版本不会被篡改
            * 并且生成了一个 go.sum 文件(两行内容如下)
                - `github.com/ozgio/strutil v0.3.0 h1:60gAF16tu4v0jv8lG++6BSRjtEAUZHWCUdBVnP9sPV8=`
                    + h1+hash形式
                - `github.com/ozgio/strutil v0.3.0/go.mod h1:KwdrJYF4j4XOlgE0RGzXCa5wRvbuYXpfy3379Yl3Ric=`
                    + gomod+hash形式
                - h1+hash 和 go.mod+hash 两者，要不就是同时存在，要不就是只存在 go.mod+hash
                    + 当 Go 认为肯定用不到某个模块版本的时候就会省略它的 h1+hash
            * Go modules的版本规则，参考链接中的说明
        + 这就实现了一个不需要在$GOPATH路径创建的项目，下载的包也不必在$GOPATH/src中，并提供模块多版本支持
    - 理论上 go.mod 和 go.sum 文件都应该提交到你的 Git 仓库中去

* main.go demo内容

```golang
package main

import (
    "log"

    "github.com/ozgio/strutil"
)

func main() {
    log.SetFlags(log.Lshortfile)
    // strutil.Words("Lorem ipsum, dolor sit amet") //-> []string{"Lorem", "ipsum", "dolor", "sit", "amet"}`

    res := strutil.Words("Lorem ipsum, dolor sit amet")
    log.Printf("get res:%v\n", res)
}
```

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
        + 手动调用`runtime/pprof`的API，结束后会生成一个文件
        + `go tool pprof [binary] [binary.prof]`
    - 具体用法示例，见下面的章节：`## Go性能工具`

* Disruptor 框架
    - Disruptor是一个高性能的线程间通信的框架，即在同一个JVM进程中的多线程间消息传递,由LMAX开发
    - [高性能队列——Disruptor](https://tech.meituan.com/2016/11/18/disruptor.html)
        + Disruptor是英国外汇交易公司LMAX开发的一个高性能队列，研发的初衷是解决内存队列的延迟问题（在性能测试中发现竟然与I/O操作处于同样的数量级）。
        + 基于Disruptor开发的系统单线程能支撑每秒600万订单，2010年在QCon演讲后，获得了业界关注。2011年，企业应用软件专家Martin Fowler专门撰写长文介绍。同年它还获得了Oracle官方的Duke大奖。目前，包括Apache Storm、Camel、Log4j 2在内的很多知名项目都应用了Disruptor以获取高性能
        + Java内置队列，链接中列出了数组、链表、堆实现的多种类型的线程安全的队列类
            * 基于数组的线程安全的队列：`ArrayBlockingQueue`，通过加锁来保证线程安全
            * 基于链表的线程安全的队列，分为：`LinkedBlockingQueue` 和 `ConcurrentLinkedQueue`，前者也通过加锁、后者通过原子变量compare and swap(`CAS`)这种不加锁的方式实现线程安全
                - 关于`CAS`之前笔记中有记录(搜`并发无锁队列`)：[并发编程.md](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%90%84%E8%AF%AD%E8%A8%80%E8%AE%B0%E5%BD%95/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B.md)
        + 通过不加锁的方式实现的队列都是无界的（无法保证队列的长度在确定的范围内）；
            * 而加锁的方式，可以实现有界队列
        + 在稳定性要求特别高的系统中，为防止生产者速度过快而导致内存溢出，只能选择有界队列；同时，为减少Java垃圾回收对系统性能的影响，尽量选择array/heap数据结构(链表结构需要另外的指针)，符合条件的队列就只有`ArrayBlockingQueue`(有界、加锁、arraylist结构)
            * 但`ArrayBlockingQueue`使用过程中，会因为*加锁*和*伪共享*等出现严重的性能问题
        + `加锁`
            * 现实编程过程中，加锁通常会严重地影响性能。
                - 线程会因为竞争不到锁而被挂起，等锁被释放的时候，线程又会被恢复，这个过程中存在着很大的开销，并且通常会有较长时间的中断，因为当一个线程正在等待锁时，它不能做任何其他事情
                - 如果一个线程在持有锁的情况下被延迟执行，例如发生了缺页错误、调度延迟或者其它类似情况，那么所有需要这个锁的线程都无法执行下去。如果被阻塞线程的优先级较高，而持有锁的线程优先级较低，就会发生优先级反转。
            * Disruptor论文中讲述了一个实验，结果：CAS操作比单线程无锁慢了1个数量级；有锁且多线程并发的情况下，速度比单线程无锁慢3个数量级。可见无锁速度最快。
                - 单线程情况下，不加锁的性能 > CAS操作的性能 > 加锁的性能
                - 多线程情况下，为了保证线程安全，必须使用CAS或锁，这种情况下，CAS的性能超过锁的性能，前者大约是后者的8倍
            * CAS是CPU的一个指令，由CPU保证原子性。
                - e.g. 两线程对变量Entry加1，CAS会先把变量Entry现在的value跟线程当初读出的值相比较，若相同，则赋值；若不相同，则赋值执行失败。一般会通过while/for循环来重新执行，直到赋值成功。
                - 在高度竞争的情况下，锁的性能将超过原子变量的性能，但是更真实的竞争情况下，原子变量的性能将超过锁的性能。
                - 同时原子变量不会有死锁等活跃性问题。
        + `伪共享`
            * 访问共享数据的速度：寄存器 > 一级缓存L1 > 二级缓存L2 > 三级缓存L3 > 内存(主存)
                - 越靠近CPU的缓存，速度越快，容量也越小。所以L1缓存很小但很快，并且紧靠着在使用它的CPU内核；L2大一些，也慢一些，并且仍然只能被一个单独的CPU核使用；L3更大、更慢，并且被单个插槽上的所有CPU核共享；最后是主存，由全部插槽上的所有CPU核共享
            * 缓存行(cache line)
                - Cache是由很多个cache line组成的。每个cache line通常是`64`字节，并且它有效地引用主内存中的一块地址。
                    + 一个Java的long类型变量是8字节，因此在一个缓存行中可以存8个long类型的变量。
                - CPU每次从主存中拉取数据时，会把相邻的数据也存入同一个cache line。
                - 在访问一个long数组的时候，若数组中的一个值被加载到缓存中，会自动加载另外7个。因此能非常快的遍历这个数组。事实上，可以非常快速的遍历在连续内存块中分配的任意数据结构。
                    + e.g. `arr[i][j]`两层循环遍历，同样用`[0,8)`，第二层循环`a[i][j]`(`j`做下标)，比第一层循环用`a[j][i]`(`i`做下标)要快很多(链接中的示例描述在2G Hz、2核、8G内存的运行环境中测试，速度差一倍)
            * 无法充分使用缓存行特性的现象，称为伪共享
                - `ArrayBlockingQueue`中三个成员变量放到一个缓存行中，每次修改，都会使之前缓存的数据失效，从而不能完全达到共享的效果，这种现象即为伪共享
    - [LMAX Disruptor简介](https://www.jianshu.com/p/a44b779c22cb)
        + LMAX
            * LMAX是一个英国外汇黄金交易所，它是第一家也是唯一一家采用多边交易设施Multilateral Trading Facility(MTF)，拥有交易所牌照和经纪商牌照的欧洲顶级金融公司。
            * 而LMAX所用的Disruptor技术，在一个线程每秒处理6百万订单。
        + Disruptor只是LMAX平台一部分，LMAX是一个新型零售金融交易平台，它能够达到低延迟、高吞吐量(大量交易)。
            * 这个系统建立在JVM平台上，核心是一个逻辑处理器，每秒能够处理600百万订单
            * 业务逻辑处理器完全运行在内存中(in-memory)，使用事件源驱动方式(event sourcing)
            * 而业务逻辑处理器核心是Disruptor，这是一个并发组件，能够在无锁情况下实现网络并发查询操作。
            * Disruptor实现了队列的功能，而且是一个有界的队列。应用场景是"生产者-消费者"模型
        + Disruptor中的核心概念
            * Ring Buffer
                - 环形缓冲区，曾经是Disruptor中的核心对象，不过从3.0版本开始，只负责对通过Disruptor进行交换的数据(事件)进行存储和更新
            * Sequence
                - 通过递增的序号管理进行交换的数据(事件)，对数据(事件)的处理过程总是沿着序号逐个递增处理的。
            * xxx
    - [高性能的消息框架 go-disruptor](https://colobu.com/2016/07/22/using-go-disruptor/)

## Go性能工具

* pprof
    - [google/pprof](https://github.com/google/pprof)
    - [Golang 大杀器之性能剖析 PProf](https://www.jianshu.com/p/4e4ff6be6af9)
        + pprof 是用于可视化和分析性能分析数据的工具
        + pprof以 [profile.proto](https://github.com/google/pprof/blob/master/proto/profile.proto) 中定义的协议读取分析样本的集合，并生成报告以可视化并帮助分析数据（支持文本和图形报告）
    - [Go pprof性能调优](https://www.cnblogs.com/Dr-wei/p/11742414.html)
    - Go自带profiling的库
        + 在计算机性能调试领域里，profiling 是指对应用程序的画像，画像就是应用程序使用 CPU 和内存的情况
        + Go语言内置了获取程序的运行数据的工具，包括以下两个标准库：
            * `runtime/pprof`  采集工具型应用运行数据进行分析
            * `net/http/pprof` 采集服务型应用运行时数据进行分析
                - 文件路径在: Go\src\net\http\pprof.go
                - 用的时候只要`import _ "net/http/pprof"`就会附带一个可视化的web统计，HTTP 服务会多出`/debug/pprof`的endpoint(即`ip:port/debug/pprof`形式的url)，可观察应用程序的情况(可以查看`Golang 大杀器之性能剖析 PProf`链接中的示例)
        + pprof开启后，每隔一段时间（10ms）就会收集下当前的堆栈信息，获取各个函数占用的CPU以及内存资源；最后通过对这些采样数据进行分析，形成一个性能分析报告。
            * 只应该在性能测试的时候才在代码中引入pprof
    - Go语言项目中的性能优化主要有以下几个方面
        + CPU profile
            * 报告程序的 CPU 使用情况，按照一定频率去采集应用程序在 CPU 和寄存器上面的数据
        + Memory Profile（Heap Profile）
            * 报告程序的内存使用情况
        + Block Profiling
            * 报告 goroutines 不在运行状态的情况，可以用来分析和查找死锁等性能瓶颈
        + Goroutine Profiling
            * 报告 goroutines 的使用情况，有哪些 goroutine，它们的调用关系是怎样的
    - 使用(runtime/pprof)
        + CPU性能：`func StartCPUProfile(w io.Writer) error`
            * 在测试结束的地方`pprof.StopCPUProfile()`，或者`defer pprof.StopCPUProfile()`测试整个函数
            * 实现了`io.Writer`接口的结构体均可以作为入参，该interface中只有一个方法：`Write(p []byte) (n int, err error)`
                - w可送一个文件句柄,`os.Create(filename)`创建
        + Mem性能：`func WriteHeapProfile(w io.Writer) error`
            * 写入前进行垃圾回收来获取最新统计 `runtime.GC()`
    - 分析profile文件(使用`go tool pprof`)
        + `go tool pprof profile文件(可以是本地文件，也可以是http地址)`
            * 查看下面的`分析结果(可视化)`章节：
                - `go tool pprof -http=:8080 cpu.prof` 会在浏览器里显示调用图
            * http地址(配合net/http/pprof)
                - `go tool pprof http://localhost:6060/debug/pprof/profile?seconds=60` 可以查看上面net/http/pprof参考链接中的示例
                    + 上面的url会指定CPU Profiling的时间段，结束后将默认进入 pprof 的交互式命令模式
                - 也可以分析内存：`go tool pprof http://localhost:6060/debug/pprof/heap` (也可`/block`对阻塞同步的堆栈跟踪；`/mutex`对互斥锁持有情况堆栈跟踪)
                    + -inuse_space：分析应用程序的常驻内存占用情况，可用`list -inuse_space`方式查看
                    + -alloc_objects：分析应用程序的内存临时分配情况
            * 会进入一个交互式界面，输入各命令来查看相关统计信息
            * `top`
                - 查看程序中占用CPU前n位(默认10)的函数，也可指定数量(`top3`/`top 3`)
                - top的结果：`flat  flat%   sum%        cum   cum%`
                    + flat：当前函数占用CPU的耗时
                    + flat%:当前函数占用CPU的耗时百分比
                    + sum%：函数占用CPU的耗时累计百分比
                    + cum：当前函数加上调用当前函数的函数占用CPU的总耗时
                    + cum%：当前函数加上调用当前函数的函数占用CPU的总耗时百分比
            * `list 函数名` 查看函数详情，可查看函数各部分的消耗
                - 可以直接查看自己代码里的函数
    - pprof与性能测试结合(`go test -bench`)
        + 关于`go test`使用，可查看`### go testing`章节
        + `go test`命令有两个参数和 pprof 相关，它们分别指定生成的 CPU 和 Memory profiling 保存的文件
            * `-cpuprofile`：cpu profiling 数据要保存的文件地址
            * `-memprofile`：memory profiling 数据要保存的文件地址
        + 性能测试时结合pprof：
            * `go test -bench="." -cpuprofile=./cpu.prof` 执行 CPU profiling并保存结果到cpu.prof
                - `-bench=.`表示所有方法，Windows下要加引号`"."`
            * `go test -bench="." -memprofile=./mem.prof` 执行 Mem profiling并保存结果到mem.prof
            * 分析结果(可视化)：
                - 方法1：`go tool pprof -http=:8080 cpu.prof` 会在浏览器里显示调用图
                    + 除了调用关系图外，还有很丰富的选项来查看各种报表及统计
                    + 选项查看包含：top/调用图/火焰图/源码分析(包含汇编和底层调用的耗时分析)
                    + 通过 PProf 的可视化界面，我们能够更方便、更直观的看到 Go 应用程序的调用链、使用情况等，并且在 View 菜单栏中，还支持如上多种方式的切换
                - 方法2：`go tool pprof cpu.prof`，进入交互界面后，执行`web`，就会在浏览器展示调用图
                    + 没有第一种方法牛逼，只展示调用图
            * 提前装了`graphviz`、`go-torch`、`火焰图`，如果提示缺少组件，则安装一下并加好路径(如图形化展示里所述)
        + Profiling 一般和性能测试一起使用，因为一般只有应用在负载高的情况下 Profiling 才有意义
    - 图形化展示
        + 安装`graphviz`
            * 想要查看图形化的界面首先需要安装graphviz图形化工具
            * 若windows下，将graphviz安装目录下的bin文件夹添加到Path环境变量中，`dot -version`验证是否安装ok
        + 火焰图和`go-torch`
            * go-torch是 uber 开源的一个工具，可以直接读取 golang profiling 数据，并生成一个火焰图的 svg 文件
                - go-torch安装：`go get -v github.com/uber/go-torch`
            * 火焰图 svg 文件可以通过浏览器打开，它对于调用图的最优点是它是动态的：可以通过点击每个方块来 zoom in 分析它上面的内容
                - `git clone https://github.com/brendangregg/FlameGraph.git`下载，并把目录加到PATH
* 使用示例

```golang
package main

import (
    "log"
    "math"
    "os"
    "runtime/pprof"
    "time"
)

func main() {
    log.Println("start profiling...")
    file, err := os.Create("./cpu.profile")
    if err != nil {
        log.Printf("create file err![%v]\n", err)
        return
    }
    defer file.Close()

    pprof.StartCPUProfile(file)
    defer pprof.StopCPUProfile()

    for i := 0; i < 1000; i++ {
        log.Printf("test:%v\n", FuncTestA())
        log.Print("=============")
    }

    heapfile, err := os.Create("./mem.profile")
    if err != nil {
        log.Fatal("create error!\n")
    }
    defer heapfile.Close()
    pprof.WriteHeapProfile(heapfile)

}
```

## NSQ

* [NSQ官网文档](https://nsq.io/overview/design.html)
* NSQ由三个守护进程组成
    - [INTERNALS](https://nsq.io/overview/internals.html)
    - `nsqd` (4151 http、4150 tcp)
        + [nsqd](https://nsq.io/components/nsqd.html)
        + `nsqd`是接收、排队并将消息传递给客户端的守护进程
        + 可以单机使用，但是一般配置成集群(通过`nsqlookupd`的实例，这种情况下它会广播topic和channel用于发现)
        + 一个单独的`nsqd`可以有多个`topic`，一个topic可以有多个`channel` (topic和channel都通过一个string标识)
            * 一个`topic`是一个独立的数据流
            * 一个`channel`是用户订阅指定topic的一个逻辑分组
            * 每个`channel`接收指定topic上的所有消息的副本，当channel上的每个消息分布在其订阅者之间(多个订阅者订阅用同一个channel)时，支持多播风格的传递，从而支持负载均衡
        + 命令行选项
            * 启动nsqd时，监听了两个TCP端口(4150用于tcp客户端、4151用于HTTP API)，可选择监听第三个端口用于HTTPS
            * `-http-address`
                - 默认`"0.0.0.0:4151"`，用于监听接收`http`客户端请求
                - 如：`curl -d 'hello' 'http://127.0.0.1:4151/pub?topic=test'`，`-d`通过`POST`指定请求数据
            * `-tcp-address`
                - 默认`"0.0.0.0:4150"`，监听接收`tcp`客户端请求
            * `-https-address string`
                - 默认`"0.0.0.0:4152"`，监听接收`https`客户端请求
            * `-lookupd-tcp-address`
                - `lookupd`服务的tcp地址(ip:4160形式)，可以指定多个(通过多次指定)
        + HTTP API
            * [nsqd http-api](https://nsq.io/components/nsqd.html#http-api)
            * `GET /stats` 返回内部统计数据
                - `curl http://192.168.50.207:4151/stats`
            * `curl http://192.168.50.207:4151/ping` 检查连接，正常返回OK
            * `curl http://192.168.50.207:4151/info` 版本
            * `GET /debug/pprof` 返回一个debug信息的地址
                - 访问浏览器 `http://192.168.50.207:4151/debug/pprof`
            * `GET /debug/pprof/profile` 启动一个cpu `pprof` 30秒，并输出结果
                - 结果是二进制，保存成文件再`go tool pprof`分析：
                    + `curl -o nsqd.prof http://192.168.50.207:4151/debug/pprof/profile`
                - *推荐*: `go tool`支持url，可以直接`go tool pprof -http=:8080 http://192.168.50.207:4151/debug/pprof/profile`
                - 注意，这个连接不在`/debug/pprof`的索引页上，考虑到运行时的性能影响
            * `GET /debug/pprof/goroutine` 返回当前所有运行协程的堆栈跟踪
                - `go tool pprof -http=:8080 http://192.168.50.207:4151/debug/pprof/goroutine` 直接起一个web页查看
            * `GET /debug/pprof/heap` 堆和内存状态切片
            * `GET /debug/pprof/block` goroutine阻塞情况切片
            * `GET /debug/pprof/threadcreate` 创建系统线程的goroutine堆栈跟踪
            * `GET /config/nsqlookupd_tcp_addresses` 查询连接的`nsqlookupd` tcp地址
            * `PUT /config/nsqlookupd_tcp_addresses` 修改要连接的`nsqlookupd` tcp地址
                - `curl -X PUT http://127.0.0.1:4151/config/nsqlookupd_tcp_addresses -d '["127.0.0.1:4160", "127.0.0.2:4160"]'`
                - 数据格式是一个json数组的形式
    - `nsqlookupd` (4161 http、4160 tcp)
        + [nsqlookupd](https://nsq.io/components/nsqlookupd.html)
        + `nsqlookupd`是管理拓扑信息并提供最终一致的发现服务的守护进程
        + 客户端查询`nsqlookupd`来发现指定topic的`nsqd`生产者，`nsqd`节点通过其广播topic和channel信息
        + 包含两套接口：
            * 用于给`nsqd`广播的TCP接口
            * 用于客户端查询和管理的HTTP接口
        + 选项
            * `-broadcast-address`
                - 将会注册到`lookupd`中的地址，默认是当前系统的hostname，如`"PROSNAKES.local"`
            * `-http-address`
                - 默认`"0.0.0.0:4161"`，监听接收`http`客户端请求
            * `-tcp-address`
                - 默认`"0.0.0.0:4160"`，监听接收`tcp`客户端请求
        + HTTP API
            * [nsqlookupd HTTP Interface](https://nsq.io/components/nsqlookupd.html#http-interface)
            * `GET /lookup`，获取指定topic的生产者列表
                - e.g. `curl http://192.168.50.207:4161/lookup?topic=test`
            * `GET /topics` 获取topic列表
                - `curl http://192.168.50.207:4161/topics`
            * `GET /channels` 获取指定topic的channel列表
                - `curl http://192.168.50.207:4161/channels?topic=test`
            * `GET /nodes` 获取`nsqd`列表
                - `curl http://192.168.50.207:4161/nodes`
            * `POST /topic/create` 向`nsqlookupd`的注册表添加一个topic
                - `curl -d '' http://192.168.50.207:4161/topic/create?topic=xdtp1`
            * `POST /topic/delete` 删除一个存在的topic
                - `curl -d '' http://192.168.50.207:4161/topic/delete?topic=xdtp1`
                - 删除不存在的topic不会报错
            * `POST /channel/create` 向`nsqlookupd`的注册表添加一个channel
                - `curl -d '' http://192.168.50.207:4161/channel/create?topic=xdtp2&channel=xdch1`
                - 若topic不存在则会同时创建topic
            * `POST /channel/delete` 删除一个已存在topic的已存在channel
                - `curl -d '' http://192.168.50.207:4161/channel/delete?topic=xdtp2&channel=xdch1`
                - 删除不存在的channel会报错：`{"message":"CHANNEL_NOT_FOUND"}`
            * `GET /ping` 检查连接，返回ok
                - `curl http://192.168.50.207:4161/ping`
            * `GET /info` 返回版本
                - `curl http://192.168.50.207:4161/info`
            * `POST /topic/tombstone` 用于从集群清理topic(直接删除topic的方式保证不了所有消费者的订阅已不再使用)
    - `nsqadmin`
        + [nsqadmin](https://nsq.io/components/nsqadmin.html)
        + 一个实时查看集群状态，并能执行各种管理任务的web UI
        + 命令行选项
            * `-http-address`
                - 默认`"0.0.0.0:4171"`，监听接收`http`客户端请求
                - 浏览器访问用的就是该端口，e.g. 浏览器输入地址`http://192.168.50.207:4171/`
            * `-lookupd-http-address`
                - `lookupd`服务的HTTP地址(ip:4161形式)，可以指定多个(通过多次指定)
            * `-nsqd-http-address`
                - `nsqd`的HTTP地址，可指定多个
                - 这个指定得干嘛不知道，*mark*
        + 列出了展示的各项指标(包含 topic、channel、client连接 相关的各项信息)，可登录管理界面查看
* NSQ是`simplequeue`(`simplehttp`的一部分)的继承者
    - `simplehttp`是构建在`libevent`之上的一系列库和守护进程，它们使高性能HTTP服务器编写起来简单而直接
        + [bitly/simplehttp](https://github.com/bitly/simplehttp)
        + C语言实现
        + `simplequeue`部分：[simplequeue](https://github.com/bitly/simplehttp/tree/master/simplequeue)
    - `SPOF`(Singal Point Of Failure) 单点故障
* 安装
    - DOCKER方式
        + 有一个单独的、最小的nsq镜像，它包含所有的NSQ二进制文件
            * 每个二进制文件可以通过如下命令运行：
            * `docker run nsqio/nsq /<command>`
            * 注意前面有一个`/`，e.g. `docker run nsqio/nsq /nsqlookupd`
        + 拉取镜像：`docker pull nsqio/nsq`
* 使用
    - 运行`nsqlookupd`
        + `docker run --name lookupd -p 4160:4160 -p 4161:4161 nsqio/nsq /nsqlookupd` (没有设置后台执行)
        + 启动之后，即可通过网络访问，e.g. `curl`或者浏览器输入`http://192.168.50.207:4161/ping`会有应答
    - 运行`nsqd`(要添加进集群的节点上)
        + 如下命令(`nsqd启动命令示例`)，指定容器主机ip(假设为192.168.50.207)
            * 类似容器中执行 `nsqd --lookupd-tcp-address=127.0.0.1:4160`
        + 若要使用`TLS`，需要包含证书文件、私钥、根CA文件，docker镜像有一个卷挂载`/etc/ssl/certs/`用于此用途
        + `docker exec -it nsqd /bin/sh` 进入容器(注意用`/bin/sh`，镜像中无`bash`)
    - 持久化NSQ数据
        + 存储`nsqd`的数据到主机磁盘，使用`/data`卷或者挂载到主机其他目录
        + `docker run nsqio/nsq /nsqd --data-path=/data`
    - 使用`docker-compose`(需要另外安装)同时启动`nsqd`、`nsqlookupd`、`nsqadmin`
        + 创建一个`docker-compose.yml`文件
        + 使用`docker-compose up -d`启动
        + 会创建一个私有的网络，三个容器使用这个网络启动
    - 运行`nsqadmin` (webUI管理界面)
        + `docker run --name nsqadmin -p 4171:4171 nsqio/nsq /nsqadmin --lookupd-http-address=192.168.50.207:4161`
        + 启动`nsqadmin`的那台机器(只要指定lookupd所在地址)，可以浏览器访问一个管理界面`http://192.168.50.217:4171/`
    - (起了`nsqd`的节点上)发布一个初始消息(同时会创建topic)
        + `curl -d 'hello' 'http://127.0.0.1:4151/pub?topic=test'`
        + `curl -d 't2' 'http://127.0.0.1:4151/pub?topic=tp2'`
        + `curl -d 't3' 'http://127.0.0.1:4151/pub?topic=tp3'`
        + 会在请求的那台节点上创建topic
    - 另一个终端执行`nsq_to_file`，保存到磁盘(会一直运行，一来消息就持久化到文件中)
        + `nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=192.168.50.207:4161`
        + 容器环境则需要进入容器后执行(`docker exec -it nsqd /bin/sh`)，或者直接启动容器方式运行：
            * `docker run nsqio/nsq /nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=192.168.50.207:4161`

* nsqd启动命令示例

```sh
# 运行nsqd
# 如nsqlookupd运行的主机ip为192.168.50.207，节点217，命令如下
docker run --name nsqd -idt -p 4150:4150 -p 4151:4151 \
    nsqio/nsq /nsqd \
    --broadcast-address=192.168.50.217 \
    --lookupd-tcp-address=192.168.50.207:4160

# 若使用TLS启动：
docker run -p 4150:4150 -p 4151:4151 -p 4152:4152 -v /home/docker/certs:/etc/ssl/certs \
    nsqio/nsq /nsqd \
    --tls-root-ca-file=/etc/ssl/certs/certs.crt \
    --tls-cert=/etc/ssl/certs/cert.pem \
    --tls-key=/etc/ssl/certs/key.pem \
    --tls-required=true \
    --tls-client-auth-policy=require-verify
```


## 优雅关闭

* 捕捉信号
    - 场景：后台起一个grpc服务，同时会初始化一些数据库连接
        + `main()`里面`defer`关闭数据库连接
        + 但是每次`ctrl+c`打断程序时，程序是直接关闭的，不会执行到`defer`
        + 因此需要捕获`ctrl+c`(SIGINT)和`kill`(SIGTERM)的信号
    - [[Go] 捕捉信号以优雅地关闭服务器进程](https://blog.twofei.com/782/)
        + 改造：grpc服务启动放到goroutine中
        + `func Notify(c chan<- os.Signal, sig ...os.Signal)` 捕获信号(让信号中转给`c`)
            * `quit := make(chan os.Signal)`，quit作为参数
            * `sig := <-quit` 等待信号

```golang
// main()
quit := make(chan os.Signal)
// 让信号转给quit
signal.Notify(quit, syscall.SIGINT)
signal.Notify(quit, syscall.SIGTERM)
// 等待信号
sig := <-quit
log.Printf("received signal: %v\n", sig)
log.Println("server shutted down")
```

## Go Code Review Comments 官方列出的常见错误清单

* [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
    - 可看做是《Effective Go》的补充，可查看前面`《Effective Go》`章节
* `context.Context`
    - 里面包含安全认证信息、跟踪信息、截止时间、API的取消信号、进程边界
    - 绝大多数使用`Context`的函数，应该将其作为第一个参数
        + `func F(ctx context.Context, /* other arguments */) {}`
        + 不指定请求参数，则可用`context.Background()`
    - 不要将`Context`作为struct的成员，取而代之的是给每个方法添加一个`Context`的参数
* 复制
    - 一般来说，若类型`T`的方法和`*T`关联则不要对该类型进行复制，可能引起非预期的结果
* `rand`
    - 不要使用`math/rand`来生成秘钥，即便是一次性秘钥，没有初始化种子则其结果是可预测的，
    - 使用`crypto/rand`的`Read`来生成随机串，`rand.Read(buf)`
* 定义空的slice
    - 推荐使用`var t []string`，而不是`t := []string{}`，前者为nil，而后者非nil，只是两者的len、cap都是0
    - 有限的场景下，推荐使用非nil形式(`[]string{}`)。如编码JSON对象时，`[]string{}`被编码成`[]`，而nil形式slice编码为`null`
* `Example` 测试
    - 添加一个新的package时，在里面包含一个预期要如何使用的示例：一个可运行的Example 或 测试简单的成功调用流程
    - `Example`会作为包的测试套件的一部分进行编译
    - `Example`函数要求
        + `Example`用例和典型测试一样，存在于`_test.go`文件中
        + 和典型测试不同，`Example`没有任何参数，并且以`Example`开头，而不是`Test`
        + `// Output: olleh` 注释，标明预期输出
            * `Output:`后面的内容若和`Example`用例执行返回的信息一致，则PASS；否则会打印预期结果和实际结果，且结果为FAIL
            * `:`后有无空格没关系，该行注释后面若有其他注释行，则会依次匹配前面每次`fmt.Println`的结果
            * 若移除整个`// Output:`注释，则`go test -v`只会编译而不执行该用例
            * 没有`Output`注释的`Example`对于演示不能作为单元测试用例执行的代码非常有用，如访问网络
    - 若要`godoc`显示`Example (SortMultiKeys)`形式，则可以用下划线加小写开头的函数名称形式，如：`func Example_sortMultiKeys()`
    - 多个`Example`用例的场景：整个文件example
        + 同一份数据，想用不同的函数实现来测试比较，则可以写一个example文件，其中`Example_`形式的用例函数只有一个，里面可以调用多个实现
        + godoc会从文件内容读取并生成格式化的文档，如sort包：[sort package](https://golang.org/pkg/sort/)
    - `godoc示例`是编写和维护代码文档的非常好的方法
        + `godoc -http=:6060`
    - 参考：[Testable Examples in Go](https://blog.golang.org/examples)

```golang
func ExampleReverse() {
    fmt.Println(stringutil.Reverse("hello"))
    // Output: olleh
}
```

* goroutine生命周期
    - 当生成goroutine时，弄清楚它什么时候存在或者是否存在
    - goroutine可能通过channel阻塞发送或者接收而泄露，即使阻塞它的通道不可用，gc也不会终止goroutine
* `import`
    - 避免对import的包重命名来避免冲突，好的包名不需要重命名。若冲突了则尽量选择最本地或项目特定的导入
    - `import`时，对包进行组织，用空行区分不同的组，把标准库的包放到最前面一组
    - `import _` 仅应该在`package main`或者测试程序中使用
    - `import .` 测试文件(要测试的包为foo)里(测试代码的package为foo_test)，把要测试的包当做一部分，则可以用`.`
* 接收类型(Receiver Type)
    - 选择方法的接收类型为值还是指针并不容易，如果不确定，则传指针，但是有一些情况用值更清晰且高效
    - 一些有用的指导
        + 如果接收者是`map`、`func`、`map`，则不要使用指针；若接收者是`slice`且方法并不用reslice或重新分配slice，则不要使用指针
        + 若方法要改变接收者，则必须要用指针
        + 若接收者是一个包含`sync.Mutex`或者类似用于同步的域，则必须使用指针来避免拷贝
        + 若接收者是一个很大的`struct`或数组，则指针更高效
* 变量名称
    - 变量名称应该简短，尤其是范围有限的局部变量，使用`c`而不是`lineCount`，使用`i`而不是`sliceIndex`
    - 基本准则是：
        + 使用的名称离声明越远，名称的描述性就越强
        + 对于方法的接收者，一个或两个字母就足够了


* [如何写出优雅的 Go 语言代码](https://draveness.me/golang-101/)
    - 面向接口：面向接口是Go语言鼓励的开发方式，也能够为我们写单元测试提供方便，我们应该遵循固定的模式对外提供功能；
        + 使用大写的 Service 对外暴露方法；
        + 使用小写的 service 实现接口中定义的方法；
        + 通过 func NewService(...) (Service, error) 函数初始化 Service 接口；
    - 下述代码其实就不是一个设计良好的代码，
        + 它不仅在 init 函数中隐式地初始化了 grpc 连接这种全局变量，
        + 而且没有将 ListPosts 通过接口的方式暴露出去，这会让依赖 ListPosts 的上层模块难以测试

```golang
package post

var client *grpc.ClientConn

func init() {
    var err error
    client, err = grpc.Dial(...)
    if err != nil {
        panic(err)
    }
}

func ListPosts() ([]*Post, error) {
    posts, err := client.ListPosts(...)
    if err != nil {
        return []*Post{}, err
    }

    return posts, nil
}
```

* 重构：

```golang
package post

type Service interface {
    ListPosts() ([]*Post, error)
}

type service struct {
    conn *grpc.ClientConn
}

func NewService(conn *grpc.ClientConn) Service {
    return &service{
        conn: conn,
    }
}

func (s *service) ListPosts() ([]*Post, error) {
    posts, err := s.conn.ListPosts(...)
    if err != nil {
        return []*Post{}, err
    }

    return posts, nil
}
```

* 当我们使用这种方式重构代码之后，就可以在 `main` 函数中显式的初始化 grpc 连接、创建 Service 接口的实现并调用 `ListPosts` 方法

```golang
func main() {
    conn, err = grpc.Dial(...)
    if err != nil {
        panic(err)
    }

    svc := post.NewService(conn)
    posts, err := svc.ListPosts()
    if err != nil {
        panic(err)
    }

    fmt.Println(posts)
}
```

## go run -race

* `-race` 竞争
    - golang在1.1之后引入了竞争检测的概念
    - 使用`go run -race` 或者 `go build -race` 来进行竞争检测(数据竞争)

```golang
// race1.go
1 package main
2
3 import (
4     "log"
5     // "time"
6 )
7
8 func main() {
9     a := 1
10     go func() {
11         a = 2
12     }()
13     a = 3
14     log.Printf("a:%v", a)
15 }

```

* 运行`go run -race .\race1.go`，结果如下(可以看到运行的同时检测出了data race)：

```sh
==================
WARNING: DATA RACE
# 在goroutine 7中，代码行11行，对地址0x00c000122068有写入
Write at 0x00c000122068 by goroutine 7:
  main.main.func1()
      F:/work/workspace/go_path/src/github.com/xiaodongQ/go_learning/nsq_learn/race1.go:11 +0x3f

# 在13行对同一个地址也有写入
Previous write at 0x00c000122068 by main goroutine:
  main.main()
      F:/work/workspace/go_path/src/github.com/xiaodongQ/go_learning/nsq_learn/race1.go:13 +0x8f

# goroutine 7在代码中创建的位置
Goroutine 7 (running) created at:
  main.main()
      F:/work/workspace/go_path/src/github.com/xiaodongQ/go_learning/nsq_learn/race1.go:10 +0x81
==================
2020/06/18 17:12:12 a:3
# 找到一个data race
Found 1 data race(s)
exit status 66
```

* [Any race is a bug](https://ms2008.github.io/2019/05/12/golang-data-race/)
    - 在 Go（甚至是大部分语言）中，一条普通的赋值语句其实并不是一个原子操作（语言规范同样没有定义 `i++` 是原子操作, 任何变量的赋值都不是原子操作）。
        + 例如，在 32 位机器上写 int64 类型的变量是有中间状态的，它会被拆成两次写操作 MOV —— 写低 32 位和写高 32 位
        + 如果一个线程刚写完低 32 位，还没来得及写高 32 位时，另一个线程读取了这个变量，那它得到的就是一个毫无逻辑的中间变量，这很有可能使我们的程序出现诡异的 Bug
    - 在 Go 的内存模型中，有 race 的 Go 程序的行为是未定义行为，理论上出现什么情况都是正常的
    - 解决 race 的问题
        + 上锁，`sync.Mutex`
        + 无锁队列，用`atomic`包(sync/atomic)
    - mutex 由操作系统实现，而 atomic 包中的原子操作则由底层硬件直接提供支持
        + 在 CPU 实现的指令集里，有一些指令被封装进了 atomic 包，这些指令在执行的过程中是不允许中断（interrupt）的，因此原子操作可以在 lock-free 的情况下保证并发安全，并且它的性能也能做到随 CPU 个数的增多而线性扩展
        + 若实现相同的功能，后者通常会更有效率，并且更能利用计算机多核的优势。所以，以后当我们想并发安全的更新一些变量的时候，我们应该优先选择用 `atomic` 来实现

## go框架

* 分布式框架
    - [Sentinel](https://github.com/alibaba/sentinel-golang/wiki/%E4%BB%8B%E7%BB%8D)
        + 阿里开源
        + Sentinel 是面向分布式服务架构的流量控制组件，主要以流量为切入点，从限流、流量整形、熔断降级、系统负载保护等多个维度来帮助您保障微服务的稳定性
        + [Sentinel 与 Hystrix 的对比](https://yq.aliyun.com/articles/623424)
            * Hystrix 的关注点在于以 隔离 和 熔断 为主的容错机制，超时或被熔断的调用将会快速失败，并可以提供 fallback 机制
            * Sentinel 的侧重点在于：多样化的流量控制；熔断降级；系统负载保护；实时监控和控制台
* web框架
    - Gin
    - Revel
    - Echo
    - Beego
    - Buffao
    - Martini
    - Goji
    - go-rest
    - Circuit
    - macaron
    - g0-iris
    - [golang十大主流web框架](https://www.bilibili.com/read/cv5680879/)
* 存储服务器(Storage Server)
    - [有哪些值得学习的 Go 语言开源项目？](https://www.zhihu.com/question/20801814)
    - Minio 是一个与 Amazon S3 APIs 兼容的开源对象存储服务器，分布式存储方案
    - rclone 用于云存储的 Rsync
    - Camlistore 是你的个人存储系统：一种存储、同步、共享、建模和备份内容的方式
    - torus CoreOS 的现代分布式存储系统

* nsq
* skynet

* ginbro
    - Gin脚手架工具，生成app代码
    - [SQL+RESTful开源GO脚手架工具ginbro(gin and gorm's brother) 详解](https://mojotv.cn/2019/05/22/golang-felix-ginbro)


## timer 定时器

* 先看一段下面的代码(使用`Timer`)
    - 时间在150100之后和92900之前，不执行后续的功能逻辑，但是需要定期检查grpc客户端是否关闭了流
    - 实现
        + 可以简单地在for循环中用，`time.Sleep(time.Second * 10)`
        + 不过看到别处使用过定时器，于是有了下面的修改，由此引入对`time.Ticker`和`time.Timer`源码的跟踪

```golang
ticker := time.NewTicker(time.Second * 10)
defer ticker.Stop()

for{
    if curtime > 150100 || curtime < 92900 {
        select {
        case <-ticker.C:
            if isClientCancelled(stream.Context()) {
                log.Printf("client cancled, exit loop")
                break
            }
            log.Printf("curtime:%v", curtime)
        }
        continue
    }

    // xxx其他逻辑
}
```

* `isClientCancelled`函数定义如下(此处该函数是作为变量定义的)：

```golang
isClientCancelled := func(ctx context.Context) bool {
    select {
    case <-ctx.Done():
        return true
    default:
        return false
    }
}
```

* 源码跟踪
    - `time.Ticker`和`time.Timer`都包含一个`C <-chan Time`通道 和 一个`r runtimeTimer`类型
        + 两个类型分别通过`time.NewTicker(d Duration)`和`time.NewTimer(d Duration)`创建
        + 读类型的chan成员`C`由一个双向的chan赋值，`c := make(chan Time, 1)`
    - `time.Ticker`或`time.Timer`创建好后，通过调用`startTimer`并传入已经定义好的`runtimeTimer`来实现功能
        + `runtimeTimer`结构中包含上面创建的`chan`和一个将当前时间传给该`chan`的回调函数
        + 在VS Code里跳转到`startTimer`函数，会跳转到文件`src/time/sleep.go`，其中只有一个定义而没有函数体
            * 仅有定义：`func startTimer(*runtimeTimer)`
            * 它的实现在：`src/runtime/time.go`(`$GOROOT`目录)，两者通过 `go:linkname` 指令关联
    - 计时功能主要在于`startTimer`，所以关注它的实现
        + 打开`src/runtime/time.go`文件，查看实现，`startTimer`调用了`addtimer`
        + 1.14对timer做了优化，所以前后源码并不一样，分别看下两个实现
        + 1.14之前的实现
            * 可参考：[9.3.1 实现原理](https://rainbowmango.gitbook.io/go/chapter09/9.3-foreword/9.3.1-timersproc_principle)
    - 定时器在go 1.13 和 go1.14中的区别
        + 参考：[go1.14基于netpoll优化timer定时器实现原理](http://xiaorui.cc/archives/6483)
        + 要理解源码里面结构定义(`type g struct{xxx}`, `runtime/runtime2.go`文件)，先了解协程，查看本笔记中的`## 协程(coroutine)`章节
        + 1.13前
            * golang在1.10版本之前是由一个独立的`timerproc`通过`小顶堆`和`futexsleep`来管理定时任务
            * 1.10之后采用的方案是把独立的timerproc和小顶堆分成最多64个timerproc协程和`四叉堆`，用来休眠就近时间的方法还是依赖futex timeout机制
            * 默认timerproc数量会跟GOMAXPROCS一致的，但最大也就64个，因为会被64取模
            * ps: 关于`小顶堆`和`四叉堆`，等数据结构get到再放链接。。。
        + 简单的过一遍go1.13版定时器的实现
            * 不管是NewTimer、NewTicker、After等其实调用的都是`addTimer`来新增定时任务
                - `addTimer`中调用`assignBucket`，给当前协程分配一个`timerBucket`
                - go初始化时会预先实例化长度64的timers数组，通过协程的`p`跟64取摸来分配timerBucket
                - 如果新的定时任务较新，那么使用notewakeup来激活唤醒timerproc的futex等待。如果发现没有实例化timerproc，则启动
            * `timerproc`协程运行时会从堆顶拿`timer`，然后判断是否到期，到期则直接执行，当`bucket`无任务时，调用`runtime.goparkunlock`来休眠该协程
        + go1.14版的timer
            * 首先把存放定时事件的四叉堆放到`p`结构中，另外取消了`timerproc`协程，转而使用`netpoll`的`epoll wait`来做就近时间的休眠等待
            * 在每次`runtime.schedule`调度时都检查运行到期的定时器
        + 源码分析go1.14 timer
            * 在`struct p`中定义了`timer`相关字段，`timers`数组用来做四叉堆数据结构
                - 源码文件`src/runtime/runtime2.go`
                - 成员：`timers []*timer`

## 协程(coroutine)

* [Go语言的协程，系统线程以及CPU管理](https://www.pengrl.com/p/29953/)
    - M，P，G 模型
        + Go拥有一个将协程调度到系统线程执行的调度器。这个调度器定义了三个核心概念
            * `G` - goroutinue. 协程
            * `M` - worker thread, or machine. 工作线程
            * `P` - processor, 执行Go代码时所必须的一种资源
        + `M`必须有一个相关联的`P`才能执行Go代码
        + 每个协程（`G`）在一个分配给逻辑processor（`P`）的系统线程（`M`）上运行
        + 首先，Go会根据当前机器的逻辑CPU个数来创建相应数量的`P`，并将它们存放在一张空闲`P`列表中
        + 然后，新创建并等待被运行的协程会唤醒一个`P`来执行这个任务，这个`P`会创建一个和系统线程相关联的`M`
            * 和`P`一样，如果一个`M`没有工作可做了，该`M`会被放入空闲`M`链表中
            * 在程序启动时，Go会预先创建一些系统线程以及相关联的`M`
        + 上面的参考链接中演示了一个多协程调度的示例
            * 第一个协程使用主协程，第二个协程会从空闲列表中获取一个`M`和`P`
    - 进一步看看Go在什么情况下会使用更多的`M`和`P`，以及调用系统调用时协程是如何被管理的
        + Go对系统调用做了优化，具体做法是在运行时对系统调用做了封装（不管系统调用是否会造成阻塞）。
        + 该部分封装代码会自动将`P`与线程`M`解除绑定，使得另一个线程`M`可以在这个`P`上运行
        + 当系统调用结束之后，Go顺序执行如下流程直到其中一条规则被满足：
            * 试图获取同一个`P`，如果获取到，则恢复执行
            * 试图在空闲列表中获取一个`P`，如果获取到，则恢复执行
            * 将协程放入全局队列中，将相关的`M`放入空闲列表中

* 阅读上面的文章链接，发现博主的博客主页有不少好质量的文章:
* [利用CPU cache特性优化Go程序](https://pengrl.com/p/9125/)
    - `CPU cache`
        + CPU cache位于CPU和内存之间，CPU读取数据时，并不是直接从内存读取，而是先从CPU cache中读，读不到再从内存读
        + 显然，CPU cache命中率越高，程序执行速度就越快。但是受限于制造工艺以及成本，CPU cache的容量远小于内存。所以必须要有一种缓存策略，用于决定哪些数据缓存在CPU cache中。
        + CPU cache的缓存策略是基于局部性原理设计的。局部性分两点：
            * 时间局部性，即最近刚被访问的数据大概率会被再次访问；
            * 空间局部性，即最近刚被访问的数据，相邻的数据大概率会被访问
    - `CPU cache line` (CPU缓存行)
        + 根据以上原理，CPU cache在缓存数据时，并不是以单个字节为单位缓存的，而是以`CPU cache line`大小为单位缓存：
            * `CPU cache line`在一般的x86环境下为`64`字节。
            * 也就是说，即使从内存读取`1`个字节的数据，也会将邻近的`64`个字节*一并*缓存至CPU cache中
        + linux下，可以通过`getconf -a | grep CACHE`命令获取cache line大小
            * (在自己的CentOS上执行)，执行结果形式：`LEVEL1_ICACHE_LINESIZE  64`，可以看到L1、L2、L3缓存linesize都是64
        + 这也是访问数组通常比链表快的原因之一
    - `false sharing` (伪共享)
        + 本笔记中之前的`* Disruptor 框架`章节中的`伪共享`也提到过(不过没有此处这篇文章讲得清晰)
            * *访问共享数据的速度：寄存器 > 一级缓存L1 > 二级缓存L2 > 三级缓存L3 > 内存(主存)* (但大小增大)
            * `L1`紧靠着在使用它的*CPU核*；
            * `L2`只能被一个单独的*CPU核*使用；
            * `L3`被*单个插槽上的所有CPU核*共享；
            * 最后是主存，由*全部插槽上的所有CPU核*共享
        + 一个CPU核在读取一个变量时，以`cache line`的方式将后续的变量也读取进来，缓存在自己这个核的cache中，而后续的变量也可能被其他CPU核并行缓存。
            * 当前面的CPU对前面的变量进行写入时(要写入变量不一定是cache line的大小)，该变量同样是以`cache line`为单位写回内存
            * 此时，在其他核上，尽管缓存的是该变量之后的变量，但是由于没法区分自身变量是否被修改，所以它只能认为自己的缓存失效，重新从内存中读取
            * 这种现象叫做`false sharing`(伪共享)
    - `cache line padding` (缓存行填充)
        + 在高性能系统编程场景下，一般解决false sharing的方法是，在变量间添加无意义的填充数据（cache line padding）。使得我们真正需要高频并发读写的不同变量，不出现在一个cache line中(尽量在不同核缓存)
    - Go标准库中使用cache line padding的两个例子
        + `Timer`定时器(此处讲得代码是Go1.14之前的定时器，1.14源码做了优化)
            * (关于定时器原理，之前的章节：`## timer 定时器`也有学习笔记)
            * 博主的另一篇讲解更详细：[golang源码阅读之定时器以及避坑指南](https://pengrl.com/p/62835/)
                - 避坑需要的操作：
                - `Ticker`对象不再使用后，显式调用`Stop`方法
                - `Timer`对象不再使用后，在高性能场景下，也应该显式调用`Stop`方法，及时释放资源
                    + 对已超时的Timer调用`Stop`方法内部有变量保护，是安全的。但是这种保护需要拿一次桶内的互斥锁，高性能场景下也需要考虑这个消耗
                - `Stop()`源码调用关系：`time.Stop()` -> `time.stopTimer()` -> `runtime.deltimer()`
                    + `runtime.deltimer()`里面是循环执行switch-case，利用`atomic`包进行原子操作：操作`Timer`对象中的`P`，(加锁后)把其成员`deletedTimers`加1
                    + 加锁操作的实现是，获取本协程(`G`)对应的`M`，然后进行上面的+1操作(`atomic.Cas`原子操作)，而后释放`M`
            * 在`timers`的定义中，`var timers [timersLen]struct{xxx}`，定义了一个结构体数组
                - 数组元素类型是一个匿名结构体，结构体包含成员：
                    + 真正与业务逻辑功能相关的`timersBucket`
                    + 和 `pad [cpu.CacheLinePadSize - unsafe.Sizeof(timersBucket{})%cpu.CacheLinePadSize]byte`，用于做`cache line padding`优化
                - 如果去掉cache line padding的优化，上面的匿名结构体数组等价于`var timers [timersLen]timersBucket`
                    + 匿名结构体对`timersBucket`的封装，相当于在原本一个接一个的`timersBucket`数组元素之间，插入了`pad`。从而保证不同的`timersBucket`对象不会出现在同一个cache line中
                - 对于`pad`数据成员
                    + 其中的`cpu.CacheLinePadSize`变量定义在`src/internal/cpu`下，它通过*构建标签*的方式 在不同环境下定义了不同的值，比如在cpu_x86.go文件下被定义为64
                    + 关于*构建标签*，博主有一篇译文介绍：[Go语言如何使用条件编译](https://pengrl.com/p/41852/)
                        * 类比C/C++中的宏定义和预处理实现的`条件编译`，根据依赖的底层平台或处理器体系特性的不同，来进行不同的处理
                        * `构建规则`（Build Constraints）和 `构建标签`（Build tags）
        + 另外一个Go标准库中的例子，来自内存管理模块
            * `type mheap struct {xxx}`结构，包含一个结构体数组成员：`central [numSpanClasses]struct {xxx}`
            * `central`匿名结构体数组的定义，和计时器例子中的定义类似，包含两个成员：
                - `mcentral mcentral` 和 `pad [cpu.CacheLinePadSize - unsafe.Sizeof(mcentral{})%cpu.CacheLinePadSize]byte`
    - 总结
        + cache line padding适用于多个相邻的变量频繁被并发读写的场景
        + 但也存在其缺点
            * 首先，内容无实际意义的padding增加了内存使用量开销
            * 更重要的是，在某些场景下增加padding，意味着你放弃了CPU cache提供给你的空间局部性相关的预读取的奖励
