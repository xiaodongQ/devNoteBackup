## grpc

### grpc介绍
[Go使用grpc+http打造高性能微服务](https://blog.csdn.net/RA681t58CJxsgCkJ31/article/details/78601747)

gRPC是由Google主导开发的RPC框架，使用HTTP/2协议并用Protobuf作为序列化工具

与REST不同的是，REST是基于HTTP1.1 JOSN格式的一种轻量级服务模式

#### JSON和Protobuf比较：

* 首先可以从JOSN与Protobuf之间的差别入手进行对比，Protobuf很难读，它是面向机器的文字格式，而JSON则是面向人的；
* Protobuf相对于JSON而言编解码速度都非常快；
* 最后就是兼容性，现在基本所有浏览器都支持JOSN格式，而Protobuf目前仅部分语言支持。

#### http1.1与http2比较

* http1.1是文本方式，因此http的请求我们都可以很清楚的读懂；
* http1.1中每一个请求会有一个新的connection，但是http2 是一个持续的http connection；
* http1是持续repetitive，http2是compressed；
* http1.1所有浏览器都默认支持，而http2只是部分支持。

HTTP/1的主要问题
[深度解析gRPC以及京东分布式服务框架跨语言实战](http://www.sohu.com/a/126977118_494947)

Head-of-line blocking，新请求的发起必须等待服务器对前一个请求的回应，无法同时发起多个请求，导致很难充分利用TCP连接。
* 头部冗余

HTTP头部包含大量重复数据，比如cookies，多个请求的cookie可能完全一样


### gRPC环境和protoc.exe、protoc-gen-go.exe

**gRPC调用时数据错位，可能是客户端pb协议和服务端不一致**

[gRPC-go protoc](https://www.jianshu.com/p/ec3e75e5aad1)

1、下载protobuf的编译器protoc
https://github.com/google/protobuf/releases
window：
  下载: protoc-3.3.0-win32.zip (本人protoc-3.9.0-win64.zip)
  解压，把bin目录下的protoc.exe复制到GOPATH/bin下，GOPATH/bin加入环境变量。
当然也可放在其他目录，需加入环境变量，能让系统找到protoc.exe (本人放到了C:\Go\bin)

linux：
    下载：protoc-3.3.0-linux-x86_64.zip 或 protoc-3.3.0-linux-x86_32.zip
解压，把bin目录下的protoc复制到GOPATH/bin下，GOPATH/bin加入环境变量。
如果喜欢编译安装的，也可下载源码自行安装，最后将可执行文件加入环境变量。

2、获取protobuf的编译器插件protoc-gen-go
  进入GOPATH目录
  运行
> go get -u github.com/golang/protobuf/protoc-gen-go
  如果成功，会在GOPATH/bin下生成protoc-gen-go.exe文件 (在path中加下该环境变量)


### 生成
$ protoc --go_out=./ *.proto #不加gRPC插件               **不使用**
$ protoc --go_out=plugins=gRPC:./ *.proto #添加gRPC插件  **使用**
对比发现内容增加
得到 helloServer.pb.go文件

* `protoc` 命令
    - 该命令工具用于解析proto文件并根据选项生成输出各语言对应的grpc文件
    - (linux)直接键入`protoc`，会提示各个选项
        + `-I PATH` 或者 `--proto_path=PATH` 指定查找import的路径，可以指定多次，会按顺序搜索
        + `-o FILE` 指定生成的文件名
        + `--plugin=EXECUTABLE` 指定一个可执行插件
            * 一般来说`protoc`会查找PATH目录来获取插件，但是也可以使用这个标志来指定额外的可执行插件。
            * 此外，`EXECUTABLE`可能是`NAME=PATH`的形式， 在这种情况下，给定的插件名被映射到给定的可执行文件，即使可执行文件本身的名称不同
            * e.g. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin`
        + `--cpp_out=OUT_DIR` 生成C++头文件和源文件，`OUT_DIR`指定生成文件放置的目录
        + `--java_out/--js_out/--python_out=OUT_DIR` 生成Java/JavaScript/Python等

## Go示例流程和源码结构

* Go `protoc --go_out=plugins=gRPC:./ *.proto`
    - [gRPC Basics - Go](https://grpc.io/docs/tutorials/basic/go/)

## C++示例流程和源码结构

- 官方示例：[Generating client and server code](https://grpc.io/docs/tutorials/basic/cpp/)
- 执行下面的生成命令，生成客户端和服务端代码
    + protoc -I ../../protos --grpc_out=. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` ../../protos/route_guide.proto
        * 生成 `.grpc.pb.h` 和 `.grpc.pb.cc`，声明和实现了.protoc中的`service`定义的服务类
    + protoc -I ../../protos --cpp_out=. ../../protos/route_guide.proto
        * 生成 `.pb.h` 和 `.pb.cc`，声明和实现了`message`定义的类
- 会生成四个文件
    + route_guide.pb.h，该头文件声明了`message`定义的类
    + route_guide.pb.cc，包含了`message`定义的类的实现
    + route_guide.grpc.pb.h，该头文件声明了`service`定义的类
    + route_guide.grpc.pb.cc，包含了`service`定义的类的实现
- 创建服务端，包含两部分
    + 实现根据定义的`service`生成的服务接口，这里是服务中实际要做的工作
        * `class RouteGuideImpl`继承实现`RouteGuide::Service`，这里是同步服务，也可以继承实现异步服务`RouteGuide::AsyncService`
        * 需要实现所有`service`中定义的方法(成员函数)
            - e.g. 其中一个：`Status GetFeature(ServerContext* context, const Point* point, Feature* feature) override {}`
            - 传入一个用于RPC的`ServerContext* context`上下文对象
        * 注意所有方法都可能被多线程调用，所以方法实现中需要保证线程安全
    + 运行gRPC服务来监听客户端请求
        * 启动服务需要用到一个 `ServerBuilder` 工厂类，需要以下步骤
            - 1. 根据上一步的实现的类`RouteGuideImpl`创建一个实例
            - 2. 创建一个 `ServerBuilder` 工厂类
            - 3. 调用工厂类的 `AddListeningPort()`方法 来指定要监听的地址和端口
            - 4. 调用工厂类的 `RegisterService()`方法 注册 定义的实现类
            - 5. 调用工厂类的 `BuildAndStart()`方法 为服务service创建和启动一个RPC服务端server
            - 6. 调用server的 `wait()` 方法 进行阻塞等待处理。 当调用`Shutdown()`时程序被kill
- 创建客户端并调用服务方法
    + 为了调用服务端的方法，需要创建一个`stub`存根
        * 首先，为`stub`创建一个gRPC `channel`，`channel`指定服务端的地址和端口
            - `grpc::CreateChannel("localhost:50051", grpc::InsecureChannelCredentials());`
            - grpc::InsecureChannelCredentials() 指定 非SSL
        * 然后，调用根据.proto生成的类`RouteGuide`中的 `NewStub` 方法，使用上一步的`channel`来创建`stub`
            - `RouteGuideClient(std::shared_ptr<ChannelInterface> channel, const std::string& db): stub_(RouteGuide::NewStub(channel)) {}`
    + 调用服务的方法(示例中只演示同步阻塞方法)
        * 创建一个客户端上下文 `ClientContext`对象(服务端创建时也会类似传入一个`ServerContext* context`对象)，可以给这个对象设置一些选项
        * 类似本地方法一样调用方法：`Status status = stub_->GetFeature(&context, point, feature);`
        * service定义：`rpc GetFeature(Point) returns (Feature) {}`
        * 调用：`Status status = stub_->GetFeature(&context, point, feature);`
    + 流式的RPC
        * [这几种类型的service示例](https://grpc.io/docs/tutorials/basic/cpp/)
        * rpc service 定义：[server](https://grpc.io/docs/tutorials/basic/cpp/#server)
        * 服务端的流(server-side streaming)
            - service定义：`rpc ListFeatures(Rectangle) returns (stream Feature) {}`
            - 实际调用：`std::unique_ptr<ClientReader<Feature> > reader(stub_->ListFeatures(&context, rect));`
            - 跟之前的传入`context, request`、`response`不一样的是：现在传入`context, request`，返回一个`ClientReader`对象
                + 客户端可以使用这个`ClientReader`来多次通过`Read()`服务端的应答
                + `Read()`的返回值为true则可以继续读取，false则结束
            - 最后通过`ClientReader`对象的 `Finish()` 方法来完成调用并获取RPC状态
        * 客户端的流(client-side streaming)
            - service定义：`rpc RecordRoute(stream Point) returns (RouteSummary) {}`
            - 实际调用：`std::unique_ptr<ClientWriter<Point> > writer(stub_->RecordRoute(&context, &stats));`
            - 和上面类似，除了传入`context`、`response`外，返回的是一个`ClientWriter`对象
                + 向流里面使用这个`ClientWriter`对象的`Write()`方法写入客户端的请求
                + 所有都发送完则使用 `WritesDone()` 方法告诉gRPC已经完成了写入
                + 然后使用 `Finish()` 完成调用并获取RPC状态
        * 双向流(bidirectional streaming) (英 /ˌbaɪdəˈrekʃənl/ 双向的)
            - service定义：`rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}`
            - 调用：`std::shared_ptr<ClientReaderWriter<RouteNote, RouteNote> > stream(stub_->RouteChat(&context));`
            - 传入只有一个 `context`，返回一个 `ClientReaderWriter` 对象
                + `ClientReaderWriter`对象既可以读也可以写消息，读写方式和服务端`Read()`和客户端流`Write()`一样
                + 读写操作的先后顺序可以任意，它们之间的操作也是独立的

### CopyFrom

- message定义的类中，每个类都会有两个`CopyFrom`实现(重载关系)
    + `void XdTest::CopyFrom(const ::PROTOBUF_NAMESPACE_ID::Message& from)`
    + `void XdTest::CopyFrom(const XdTest& from)`
    + 自定义类:`XdTest` 继承自类:`::PROTOBUF_NAMESPACE_ID::Message`，由于用的是引用`&`而不是指针，所以定成两个方法(若传入指针则可只有一个方法用基类指针来实现多态)

#### `void XdTest::CopyFrom(const ::PROTOBUF_NAMESPACE_ID::Message& from)`

* 在.pb.h(如上面的说明所说，该头文件声明了`message`定义的类)中的声明为：`void CopyFrom(const ::PROTOBUF_NAMESPACE_ID::Message& from) final;`
    - 函数使用`final`关键字说明该函数不能被派生类重写
* 逻辑：
    - 1. 判断 `if (&from == this)`，是则直接return，不需要再copy
    - 2. 执行`.Clear();`
        + 对于类中有repeated成员时，会执行`Clear()`方法。
            - gRPC会基于成员生成模板类：`::PROTOBUF_NAMESPACE_ID::RepeatedPtrField<成员类>`
* `RepeatedPtrField` 类
    - 定义在 `repeated_field.h`中，继承自 `RepeatedPtrFieldBase`类：`class RepeatedPtrField final : private internal::RepeatedPtrFieldBase{}`
    - `RepeatedPtrFieldBase` 类
        + 定义在 `repeated_field.h`中，其成员函数部分作为内联函数实现在.h中，部分实现在.cc中
        + 成员变量
            * static const int kInitialSize = 0;

#### `void XdTest::CopyFrom(const XdTest& from)`

在.pb.h中的声明为：`void CopyFrom(const XdTest& from);`


### protocol buffer

[Potocol Buffer详解](https://www.cnblogs.com/oumyye/p/4652780.html)

Protobuf 编译器会将 .proto 文件编译生成对应的数据访问类以对 Protobuf 数据进行序列化、反序列化操作

#### 定义第一个Protocol Buffer消息
1. 创建扩展名为.proto的文件
2. 将以下内容存入该文件中。
      message LogonReqMessage {
          required int64 acctID = 1;
          required string passwd = 2;
      }

    说明:
    message是消息定义的关键字
    LogonReqMessage为消息的名字，等同于结构体名或类名。
    required前缀表示该字段为必要字段，既在序列化和反序列化之前该字段必须已经被赋值
      (另外两个类似的关键字:optional和repeated)
    int64和string分别表示长整型和字符串型的消息字段
      (在Protocol Buffer中存在一张类型对照表，该对照表中还将给出在不同的数据场景下，哪种类型更为高效。)
    acctID和passwd分别表示消息字段名
    标签数字1和2则表示不同的字段在序列化后的二进制数据中的布局位置。
      (该值在同一message中不能重复，对于Protocol Buffer而言，标签值为1到15的字段在编码时可以得到优化，既标签值和类型信息仅占有一个byte，标签范围是16到2047的将占有两个bytes；在设计消息结构时，可以尽可能考虑让repeated类型的字段标签位于1到15之间，这样便可以有效的节省编码后的字节数量)

#### 定义第二个（含有枚举字段）Protocol Buffer消息。

enum UserStatus {
    OFFLINE = 0;  //表示处于离线状态的用户
    ONLINE = 1;   //表示处于在线状态的用户
}
message UserInfo {
    required int64 acctID = 1;
    required string name = 2;
    required UserStatus status = 3;
}

   以上消息定义的关键性说明:
   enum是枚举类型定义的关键字
   UserStatus为枚举的名字
   枚举值之间的分隔符是分号，而不是逗号。
   OFFLINE/ONLINE为枚举值
   0和1表示枚举值所对应的实际整型值，可以为枚举值指定任意整型值，而无需总是从0开始定义

#### 定义第三个（含有嵌套消息字段）Protocol Buffer消息

可以在同一个.proto文件中定义多个message

enum UserStatus {
    OFFLINE = 0;
    ONLINE = 1;
}
message UserInfo {
    required int64 acctID = 1;
    required string name = 2;
    required UserStatus status = 3;
}
message LogonRespMessage {
    required LoginResult logonResult = 1;
    required UserInfo userInfo = 2;
}

    说明：
    LogonRespMessage消息的定义中包含另外一个消息类型作为其字段，如UserInfo userInfo
    Protocol Buffer提供了另外一个关键字import，这样我们便可以将很多通用的message定义在同一个.proto文件中，而其他消息定义文件可以通过import的方式将该文件中定义的消息包含进来，如：
    import "myproject/CommonMessages.proto"

> Potocol Buffer部分概念原则

#### 限定符(required/optional/repeated)的基本规则
1. 在每个消息中必须至少留有一个required类型的字段。
2. 每个消息中可以包含0个或多个optional类型的字段。
3. repeated表示的字段可以包含0个或多个数据。需要说明的是，这一点有别于C++/Java中的数组，因为后两者中的数组必须包含至少一个元素。
4. 如果打算在原有消息协议中添加新的字段，同时还要保证老版本的程序能够正常读取或写入，那么对于新添加的字段必须是optional或repeated。

proto3中，去掉了required 和 optional

>We dropped required fields in proto3 because required fields are generally considered harmful and violating protobuf's compatibility semantics.

使用repeated时, protoc生成struct字段为指针数组，即成员都是指针
e.g. MsgList              []*HelloRequest

访问时，使用import包名加结构类型, e.g. 给结构体成员赋初值时: MsgList: []*(pt.HelloRequest){&(pt.HelloRequest{Name: "helloname"})}  (结构体变量的地址赋值给指针数组作为一个成员)

#### 类型对照表
见链接 [](https://www.cnblogs.com/oumyye/p/4652780.html)
| .proto Type | Notes | C++ Type | Java Type  |
| ----------- | ----- | -------- | ---------- |
| double      |       |double    |double      |
| float       |       |float     |float       |
| int32       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead.|  int32   |int|
| int64       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead.|  int64   |long|
| uint32      |  Uses variable-length encoding.|   uint32  |int|
| uint64      |  Uses variable-length encoding.|   uint64  |long|
| sint32      |  Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s.|   int32   |int|
| sint64      |  Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s.|   int64   |long|
| fixed32     | Always four bytes. More efficient than uint32 if values are often greater than 228.|    uint32  |int|
| fixed64     | Always eight bytes. More efficient than uint64 if values are often greater than 256.|   uint64  |long|
| sfixed32    |  Always four bytes.|   int32   |int|
| sfixed64    |  Always eight bytes.|  int64   |long|
| bool        || bool  |boolean|
| string      |A string must always contain UTF-8 encoded or 7-bit ASCII text.| string  |String|
| bytes       |May contain any arbitrary sequence of bytes.|  string  |ByteString|


#### Protocol Buffer消息升级原则

在实际的开发中会存在这样一种应用场景，消息格式因为某些需求的变化而不得不进行必要的升级，但又需要基于新老消息格式的新老程序同时运行，一般遵循如下规则：

1. 不要修改已经存在字段的标签号。
2. 任何新添加的字段必须是optional和repeated限定符，否则无法保证新老程序在互相传递消息时的消息兼容性。
3. 在原有的消息中，不能移除已经存在的required字段，optional和repeated类型的字段可以被移除，但是他们**之前使用的标签号必须被保留**，不能被新的字段重用。
4. int32、uint32、int64、uint64和bool等类型之间是兼容的，sint32和sint64是兼容的，string和bytes是兼容的，fixed32和sfixed32，以及fixed64和sfixed64之间是兼容的，这意味着如果想修改原有字段的类型时，为了保证兼容性，**只能将其修改为与其原有类型兼容的类型**，否则就将打破新老消息格式的兼容性。
5. optional和repeated限定符也是相互兼容的。

#### packages

我们可以在.proto文件中定义包名，如：
      package ourproject.lyphone;
      该包名在生成对应的C++文件时，将被替换为名字空间名称，既namespace ourproject { namespace lyphone。而在生成的Java代码文件中将成为包名。

#### gRPC c++

* 结构体成员问题

参考：
[对set_allocated_和mutable_的使用](https://blog.csdn.net/wujunokay/article/details/51287312)

[gRPC Basics - C++](https://grpc.io/docs/tutorials/basic/cpp/)

对于grpc中的结构体成员，通过：
`request->mutable_B类型成员名b()` 的方式访问，

查看分析grpc生成的.cpp和.h文件代码，生成的成员函数为`mutable_`、`set_allocated_`，通过这种方式访问和设置成员。

下面代码演示(另外分析函数入参被限定为const时的访问情况)：

* 程序伪代码：

```cpp

// 假设 B b; 是A的成员变量
void func(const &B)
{
}
...(const A *request) 
{
    // A类中的成员b作为入参，调用func函数，表面上需要解引用作为入参，但直接解引用会报错
    func(*request->mutable_b());
}

// grpc访问结构体成员
若request为const修饰的变量，要调用如下func函数的话，需要解引用，会出现const的this调用非const函数，编译报类似下面错误。
```

* 编译报错：

```sh
# const访问问题
错误：将‘const A’作为‘B* A::mutable_Bstructfield()’的‘this’实参时丢弃了类型限定 [-fpermissive]
```

* 使用方式：

可通过新增变量方式解引用：

```cpp
    auto a = new A;  //注意资源的释放，可以改成智能指针防止忘记
    a->CopyFrom(*request)
    func(*a->mutable_b)
```

* 定义返回为 stream时，成员中返回错误码问题
    - 

### CopyFrom 源码逻辑

* CopyFrom 原码如下
    - 调用CopyFrom时传入本身

```cpp
void TestStruct::CopyFrom(const TestStruct& from) {
// @@protoc_insertion_point(class_specific_copy_from_start:testxd.TestStruct)
  if (&from == this) return;
  Clear();
  MergeFrom(from);
}
```

## gRPC go

### 使用流程

服务端:

```go
//监听端口
    lis, err := net.Listen("tcp", ":8000")
    if err != nil {
        return
    }
    //创建一个gRPC 服务器
    s := gRPC.NewServer()
    //注册事件
    log.Println("RegisterGreeterServer...")
    pro.RegisterGreeterServer(s, &server{})
    //处理链接
    s.Serve(lis)
```

### 几种通信方式

gRPC主要有4种请求／响应模式

(1) 简单模式（Simple RPC）

这种模式最为传统，即客户端发起一次请求，服务端响应一个数据，这和大家平时熟悉的RPC没有什么大的区别，所以不再详细介绍。

(2) 服务端数据流模式（Server-side streaming RPC）

这种模式是客户端发起一次请求，服务端返回一段连续的数据流。典型的例子是客户端向服务端发送一个股票代码，服务端就把该股票的实时数据源源不断的返回给客户端。

(3) 客户端数据流模式（Client-side streaming RPC）

与服务端数据流模式相反，这次是客户端源源不断的向服务端发送数据流，而在发送结束后，由服务端返回一个响应。典型的例子是物联网终端向服务器报送数据。

(4) 双向数据流模式（Bidirectional streaming RPC）

顾名思义，这是客户端和服务端都可以向对方发送数据流，这个时候双方的数据可以同时互相发送，也就是可以实现实时交互。典型的例子是聊天机器人。

使用场景：
在项目中如果是提供接口这种，比如查询账号是否存在之类的可以使用简单模式。
但是如果需要长连接、相互通信那么简单模式就不行了。


### gRPC stream

srteam 顾名思义就是一种 流，可以源源不断的 推送数据
很适合传输一些大数据，或者服务端和客户端*长时间*数据交互，比如客户端可以向服务端订阅 一个数据，服务端就可以利用stream，源源不断地推送数据。

stream的种类:
* 客户端推送服务端 rpc GetStream (StreamReqData) returns (stream StreamResData){}

>客户端向服务器发送请求并获取流以读取消息序列；客户端从返回的流中读取，直到没有更多消息； gRPC 保证单个 RPC 调用中的消息排序。

* 服务端推送客户端 rpc PutStream (stream StreamReqData) returns (StreamResData){}

>客户端再次使用提供的流写入一系列消息并将其发送到服务器；一旦客户端写完消息，它就等待服务器读取它们并返回它的响应； gRPC 保证在单个 RPC 调用中的消息排序。

* 客户端与服务端 互相 推送 rpc AllStream (stream StreamReqData) returns (stream StreamResData){}



```go

service Greeter {
    /*
    以下 分别是 服务端 推送流， 客户端 推送流 ，双向流。
    */
   rpc GetStream (StreamReqData) returns (stream StreamResData){}
   rpc PutStream (stream StreamReqData) returns (StreamResData){}
   rpc AllStream (stream StreamReqData) returns (stream StreamResData){}
 }

// GreeterClient is the client API for Greeter service.
//
// For semantics around ctx use and closing/ending streaming RPCs, please refer to https://godoc.org/google.golang.org/gRPC#ClientConn.NewStream.
type GreeterClient interface {
	//
	//以下 分别是 服务端 推送流， 客户端 推送流 ，双向流。
	GetStream(ctx context.Context, in *StreamReqData, opts ...gRPC.CallOption) (Greeter_GetStreamClient, error)
	PutStream(ctx context.Context, opts ...gRPC.CallOption) (Greeter_PutStreamClient, error)
	AllStream(ctx context.Context, opts ...gRPC.CallOption) (Greeter_AllStreamClient, error)
}

func (c *greeterClient) PutStream(ctx context.Context, opts ...gRPC.CallOption) (Greeter_PutStreamClient, error) {
	stream, err := c.cc.NewStream(ctx, &_Greeter_serviceDesc.Streams[1], "/helloworld.Greeter/PutStream", opts...)
	if err != nil {
		return nil, err
	}
	x := &greeterPutStreamClient{stream}
	return x, nil
}

func (*UnimplementedGreeterServer) PutStream(srv Greeter_PutStreamServer) error {
	return status.Errorf(codes.Unimplemented, "method PutStream not implemented")
}
```

当return出函数，就说明此次 推送 或者 接收 结束了，client 会 对应的 收到消息


### 不需要入参结构体或返回结构体的情况

[不需要入参结构体或返回结构体的情况](https://www.cnblogs.com/mind-water/articles/10399995.html)

1. 方法1:  可以在.proto文件中定义空的消息(也就是空结构体,这样做的好处是以后可以给结构体添加字段, 接口函数签名不变).

```go
message MyRespone {
}

务必注意,在源代码编写的时候,不能直接直接返回空指针, 否则会报"rpc error: code = Internal desc = gRPC: error while marshaling: proto: Marshal called with nil".  意思大概就是不能序列化nil指针

if err != nil {
       return nil, err  //错误! 正确写法 return &pb.MyRespone{},err
}
```

2. 方法2: 使用protobuf的内置空类型.

.proto文件需要类似这样定义(以上面非流式的例子做修改成不需要返回结构体为例子)

```go
syntax = "proto3";
import "google/protobuf/empty.proto";  //这个就是protobuf内置的空类型
package nonstream;

service EchoIface {
    rpc Echo (Ping) returns (google.protobuf.Empty) {}
}

message Ping {
 string request = 1;
}
```

