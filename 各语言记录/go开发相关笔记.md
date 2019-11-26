## go

### 基础

#### 变量声明

```golang

第一种，指定变量类型，如果没有初始化，则变量默认为零值。
    var v_name v_type

    var b, c int = 1, 2
    var a string = "Runoob"

第二种，根据值自行判定变量类型。
    var v_name = value

    var d = true
    var xdtest = "sldfjk"

第三种，省略 var, 注意 := 左侧如果没有声明新的变量，就产生编译错误(其中有一个新变量也可)
    v_name := value

    var intVal int
    intVal :=1            // 这时候会产生编译错误
    intVal,intVal1 := 1,2 // 此时不会产生编译错误，因为有声明新的变量，因为 := 是一个声明语句

多变量声明
    var vname1, vname2, vname3 type
    vname1, vname2, vname3 = v1, v2, v3

    var vname1, vname2, vname3 = v1, v2, v3

    vname1, vname2, vname3 := v1, v2, v3

    // 这种因式分解关键字的写法一般用于声明全局变量
    var (
        vname1 v_type1
        vname2 v_type2
    )

声明了一个局部变量却没有在相同的代码块中使用它，会得到编译错误 a declared and not used。
全局变量是允许声明但不使用

如果你想要交换两个变量的值，则可以简单地使用 a, b = b, a，两个变量的类型必须是相同。

空白标识符 _ 也被用于抛弃值，如值 5 在：_, b = 5, 7 中被抛弃。

_ 实际上是一个只写变量，你不能得到它的值。
```

#### 值类型

int、float、bool 和 string 这些基本类型都属于值类型，使用这些类型的变量直接指向存在内存中的值

#### 引用类型

Golang中只有三种引用类型：slice(切片)、map(字典)、channel(管道)；

#### map

[Golang教程：（十三）Map](https://blog.csdn.net/u011304970/article/details/75003344)

* 创建和使用

```golang
// 创建
map1 := make(map[string]string)
var map2 map[string]string
// 插入
map1["zhangsan"] = "a"
map1["lisi"] = "b"
// 访问
map1["lisi"] = "assign"             // map是引用类型
fmt.Printf("map:%s", map1["none"])  // 找不到的记录，map会返回零值(对不同类型对应零值是有区别的)
// 遍历
for key,value := range map1 {

}
// 检测一个键是否存在于一个 map 
value, ok := map1["haha"] // 如果 ok 是 true，则键存在，value 被赋值为对应的值。如果 ok 为 false，则表示键不存在
if !ok {
    fmt.Println("not exist")
}
```

**注意：因为 map 是无序的，因此对于程序的每次执行，不能保证使用 range for 遍历 map 的顺序总是一致的。**



#### 接口

接口的实现
自定义类型实现接口，需要实现接口中声明的所有方法

1）接口不能实例化（类似于C++中的抽象类），可以指向一个实现了该接口的自定义类型的变量。
2）只要是自定义数据类型，就可以实现接口，不仅仅是结构体类型
3）一个自定义类型可以实现多个接口。

4）接口之间可以实现继承，利用嵌套匿名接口。

5）interface类型默认是一个指针（引用类型），如果没有对interface初始化，则其为nil。

6）空接口type T interface{}没有任何方法，所有的类型都实现了该空接口，也就可以将任何变量赋给空接口。


类型断言
由于接口是一般类型，不知道具体的数据类型，如果需要转成具体类型，就需要使用类型断言

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

### strings
1. idMemberList := strings.Split(instrumentId, "-")

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
log模块主要提供了3类接口。分别是
    Print 、Panic 、Fatal
对每一类接口其提供了3种调用方式
    Xxxx 、 Xxxxln 、Xxxxf

```go
func main(){
    arr := []int {2,3}
    log.Print("Print array ",arr,"\n")
    log.Println("Println array",arr)
    log.Printf("Printf array with item [%d,%d]\n",arr[0],arr[1])
}
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

### 获取时间 time包

[golang包time用法详解](https://blog.csdn.net/wschq/article/details/80114036)

golang提供以下两种基础类型
    - 时间点(Time)
    - 时间段(Duration)

此外还提供：
    + 时区(Location)
    + Ticker
    + Timer(定时器)

获取当前时间

     (1) currentTime:=time.Now()     //获取当前时间，类型是Go的时间类型Time

时间格式化：

```golang
BeginTime := "20191126 145500"
// 2006-01-02 15:04:05 必须是这个时间点, 记忆方法:6-1-2-3-4-5
// 01/02 03:04:05PM '06 -0700(时区), go语言中使用该递增的格式来解析各时间位
// 此处有个简单的源码分析：[Golang神奇的2006-01-02 15:04:05](https://www.jianshu.com/p/c7f7fbb16932)
formatTime, _ := time.Parse("20060102 150405", BeginTime)
fmt.Printf("ori time:%v, parse:%v\n", BeginTime, formatTime.Format("2006-01-02 15:04:05"))
```

## string



### string、int、int64互相转换

```golang
int,err:=strconv.Atoi(string)
#string到int64
int64, err := strconv.ParseInt(string, 10, 64)
#int到string
string:=strconv.Itoa(int)
#int64到string
string:=strconv.FormatInt(int64,10)
```

