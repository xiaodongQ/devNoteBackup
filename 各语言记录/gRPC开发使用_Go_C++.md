## grpc

### grpc介绍

* [Go使用grpc+http打造高性能微服务](https://blog.csdn.net/RA681t58CJxsgCkJ31/article/details/78601747)

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

* HTTP/1的主要问题
    - Head-of-line blocking，新请求的发起必须等待服务器对前一个请求的回应，无法同时发起多个请求，导致很难充分利用TCP连接。
    - 头部冗余
        + HTTP头部包含大量重复数据，比如cookies，多个请求的cookie可能完全一样

### gRPC环境和protoc.exe、protoc-gen-go.exe (Go环境)

(C++环境查看下面的"C++示例流程和源码结构"章节)

**gRPC调用时数据错位，可能是客户端pb协议和服务端不一致**

* 参考：[gRPC-go protoc](https://www.jianshu.com/p/ec3e75e5aad1)
    - 由于之前安装比较早，找的参考来源都是各种博客，建议参考官网 [gRPC Go Quick Start](https://grpc.io/docs/quickstart/go/)
    - 不过该参考链接中步骤与官网一致。(现在网络上各种技术博客复制粘贴同质化太严重，且不少有错误的也原样抄袭，建议从官网获取信息)
* 步骤

1、下载protobuf的编译器protoc
下载：`https://github.com/google/protobuf/releases`
window：
  下载: 本人protoc-3.9.0-win64.zip
  解压，把bin目录下的protoc.exe复制到GOPATH/bin下，GOPATH/bin加入环境变量。
当然也可放在其他目录，需加入环境变量，能让系统找到protoc.exe (本人放到了C:\Go\bin)

linux：
    下载：protoc-3.3.0-linux-x86_64.zip
解压，把bin目录下的protoc复制到GOPATH/bin下，GOPATH/bin加入环境变量。
如果喜欢编译安装的，也可下载源码自行安装，最后将可执行文件加入环境变量。

2、获取protobuf的编译器插件protoc-gen-go
  进入GOPATH目录
  运行
> `go get -u github.com/golang/protobuf/protoc-gen-go`
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
* 启动服务
    - `net.Listen`
            * 一般来说`protoc`会查找PATH目录来获取插件，但是也可以使用这个标志来指定额外的可执行插件。
            * 此外，`EXECUTABLE`可能是`NAME=PATH`的形式， 在这种情况下，给定的插件名被映射到给定的可执行文件，即使可执行文件本身的名称不同
            * e.g. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin`
        + `--cpp_out=OUT_DIR` 生成C++头文件和源文件，`OUT_DIR`指定生成文件放置的目录
        + `--java_out/--js_out/--python_out=OUT_DIR` 生成Java/JavaScript/Python等

## Go示例流程和源码结构

* Go `protoc --go_out=plugins=gRPC:./ *.proto`
    - [gRPC Basics - Go](https://grpc.io/docs/tutorials/basic/go/)
        + [routeguide示例](https://github.com/grpc/grpc-go/tree/master/examples/route_guide/routeguide)
    - 生成服务端和客户端代码
        + `protoc -I routeguide/ routeguide/route_guide.proto --go_out=plugins=grpc:routeguide`
            * `--go_out=plugins=grpc:路径` 指定grpc插件，并指定生成文件的存放路径
            * 会生成 route_guide.pb.go 文件，其中包含：
                - 用于填充、序列化和检索 请求和响应 消息类型的所有protocol buffer代码
                - 一个interface类型(或存根stub)(`type RouteGuideClient interface`)，供 客户端 使用RouteGuide服务中定义的方法进行调用
                - 服务器要实现的接口类型(`type RouteGuideServer interface`)，包含RouteGuide服务中定义的方法
    - 创建服务端
        + 可以看到生成的route_guide.pb.go文件，定义了一个interface：`type RouteGuideServer interface {...}`，其中包含了需要实现的方法
        + 实现：
            * 定义一个struct，要实现的方法绑定到该struct上，`type routeGuideServer struct {...}`
            * 而后依次实现proto中定义的方法：`func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {...}`
                - routeGuideServer类型实现了所有服务方法，其中
                - 简单rpc
                    + `rpc GetFeature(Point) returns (Feature) {}`
                        * 生成的代码接口参数中有一个context上下文：`GetFeature(context.Context, *Point) (*Feature, error)`
                - 服务端流
                    + `rpc ListFeatures(Rectangle) returns (stream Feature) {}`
                        * `ListFeatures(*Rectangle, RouteGuide_ListFeaturesServer) error`
                        * RouteGuide_ListFeaturesServer是一个interface，其中成员有：
                            - `Send(*Feature) error`
                                + 服务端通过`Send`把结果按流的方式发送给客户端(类似先注册一个函数)
                            - `grpc.ServerStream`(也是一个interface)
                - 客户端流
                    + `rpc RecordRoute(stream Point) returns (RouteSummary) {}`
                        * `RecordRoute(RouteGuide_RecordRouteServer) error`
                        * RouteGuide_RecordRouteServer是一个interface，其中成员：
                            - `SendAndClose(*RouteSummary) error` 发送完结果即结束
                            - `Recv() (*Point, error)` 接收客户端流，需要检查错误返回，接收报错`err == io.EOF`则表示接收结束，可SendAndClose发送结果；接收错误为nil则表示正常且未接收完
                            - `grpc.ServerStream`
                - 双向流
                    + `rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}`
                        * `RouteChat(RouteGuide_RouteChatServer) error`
                        * RouteGuide_RouteChatServer interface成员：
                            - `Send(*RouteNote) error` 向客户端发送信息
                            - `Recv() (*RouteNote, error)` 从客户端接收信息
                            - `grpc.ServerStream`
                        * 发送和读取可以是任何顺序
        + 启动服务
            * 指定要监听的端口：`lis, err := net.Listen("tcp", fmt.Sprintf(":%d", *port))`
            * 创建一个 gRPC server：`grpcServer := grpc.NewServer()`
            * 把服务的实现注册到gRPC server上
                - `pb.RegisterRouteGuideServer(grpcServer, &routeGuideServer{})`
            * 调用gRPC server的`Serve()`函数，阻塞等待，直到服务被kill或者调用`Stop()`
                - 调用之后，客户端就可以进行请求了
    - 创建客户端
        + 要调用服务端接口，首先要创建一个gRPC通道
            * 通过`grpc.Dial`来创建通信通道： `conn, err := grpc.Dial(*serverAddr)`，最后conn需要释放 `defer conn.Close()`
                - 签名为：`func Dial(target string, opts ...DialOption) (*ClientConn, error)`
                - 创建的conn是`ClientConn`类型的struct，里面定义了一系列信息，上下文、认证、选项、锁 等等信息
            * 也可通过`DialOptions`来设置认证选项
        + gRPC通道创建好后，需要一个客户端存根`stub`
            * `client := pb.NewRouteGuideClient(conn)`，
                - 传入的是上面创建的conn结构
                - 返回一个结构体指针，结构体中包含成员：`cc *grpc.ClientConn`，相当于将conn包装了一层，包装成各自proto对应的client
        + 然后就可以用上面创建的存根/客户端 来调用服务端接口了
            * `feature, err := client.GetFeature(context.Background(), &pb.Point{409146138, -746188906})`
            * 传入一个上下文`context.Context`，可用来改变RPC的行为，像超时等行为
    - 设置超时
        + 客户端
            * `ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(1*time.Minute))`
                - 或者使用WithTimeout(其封装了WithDeadline): `ctx, cancel := context.WithTimeout(context.Background(), 1*time.Minute)`
                - 对于err的解析，参考:[带入gRPC：gRPC Deadlines](https://segmentfault.com/a/1190000016601876)
            * `defer cancel()` 注意不要忘记对应返回的cancel函数的调用
            * 然后将`ctx`作为调用grpc结构的第一个参数(上下文)，若不用超时则第一个参数传`context.Background()`即可
            * (*解析err场景*：对返回stream做`Recv()`)达到超时后，`Recv()`会返回错误，
                - `pong, err := stream.Recv()`，对err解析：
                - "google.golang.org/grpc/codes" 导入grpc codes包
                - "google.golang.org/grpc/status" 导入grpc status包
                - `statusErr, ok := status.FromError(err)` 判断是否为grpc的错误，若是则ok为true，并返回`*Status`类型
                - `if statusErr.Code() == codes.DeadlineExceeded {}`, `codes.DeadlineExceeded`即为grpc超时的错误码
                    + 并不能直接用 `if err == context.DeadlineExceeded`做判断(结果为false)
                - **问题**，对于stream，由于连接一直在的，设置的deadline超时会将正常持续的stream置为超时?
                    + 找到这个:"A deadline would always terminate the stream when it is exceeded. Since the stream is kept open indefinitely/forever a deadline on ClientContext would almost always terminate the stream too early."，参考[How to detect (physical) disconnect when using a bidirectional stream in Grpc](https://stackoverflow.com/questions/49276298/how-to-detect-physical-disconnect-when-using-a-bidirectional-stream-in-grpc)
                    + 所以结论还是使用`一直连接`接收的stream时，不应设置deadline
                    + 对于多个服务的grpc调用，若设置了超时时间则会解析传递，实现原理相关源码解析，可参考：[gRPC 系列——grpc超时传递原理](https://xiaomi-info.github.io/2019/12/30/grpc-deadline/)
        + 服务端
            * `if ctx.Err() == context.Canceled {}`
        + 可参考(里面包含了C++和Go的服务端和客户端设置示例)：[grpc deadlines](https://www.cnblogs.com/029zz010buct/p/9487568.html)
            * 其中例子来源于官网文档：[gRPC and Deadlines](https://grpc.io/blog/deadlines/)

```golang
// 设置超时的客户端，由于第二个参数需要 time.Time 类型，所以使用time.Now().Add()获取一个Time类型数据
// ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(15*time.Second))
ctx, cancel := context.WithTimeout(context.Background(), 15*time.Second) // WithTimeout实际是对WithDeadline的封装
stream, _ := testClient.GetSecurityQuotes10(ctx, req) //返回stream
for {
    pong, err := stream.Recv()
    if err == io.EOF {
        log.Printf("c.Echo: io.EOF %v ", err)
        break
    }
    if err != nil {
        statusErr, ok := status.FromError(err)
        if ok && statusErr.Code() == codes.DeadlineExceeded {
            log.Printf("recv timeout, trigger deadline errinfo:[%v]", err)
            break
        } else {
            log.Printf("stream.Recv error: %v", err)
        }
    }
    log.Println("data:", pong)
}

// 若为Go服务端，则检查是否超时取消
// if ctx.Err() == context.Canceled {}
```

## C++示例流程和源码结构

* CentOS下安装gRPC
    - 参考：[gRPC C++ - Building from source](https://github.com/grpc/grpc/blob/master/BUILDING.md#grpc-c---building-from-source)
    - 编译依赖的先决条件`pkg-config` `libtool` `cmake`等包和工具
    - 先克隆项目(包括子模块)
        + `git clone https://github.com/grpc/grpc`，也可加上-b选项指定tag
        + `cd grpc`
        + `git submodule update --init`
            * 子模块也可通过克隆时用`--recursive`递归方式来克隆带子模块的版本库
    - 使用cmake编译
        + `mkdir -p cmake/build`
        + `cd cmake/build`
        + `cmake -DBUILD_SHARED_LIBS=ON ../..` (原链接中为=ON，设置不生效，grep查找看用到的地方都是ifeq)
            * 若不`-DBUILD_SHARED_LIBS=ON`开启动态库编译，则只编译出.a静态库
            * 执行完后会在 cmake/build目录生成编译需要的相关文件，检查是否生成，若没有则有问题
        + `make`(应该可以不用这步)
        + 安装gRPC(可以通过`CMAKE_INSTALL_PREFIX`配置安装位置，默认`/usr/local`):
            * 配置表示需要生成protoc需要的各语言的plugin等包
            * cmake ../.. -DgRPC_INSTALL=ON                \
                  -DCMAKE_BUILD_TYPE=Release       \
                  -DgRPC_ABSL_PROVIDER=package     \
                  -DgRPC_CARES_PROVIDER=package    \
                  -DgRPC_PROTOBUF_PROVIDER=package \
                  -DgRPC_SSL_PROVIDER=package      \
                  -DgRPC_ZLIB_PROVIDER=package
        + `make`(若无protobuf，会一并安装protobuf，3.x版本，2.x不支持)
            * `protoc`生成proto2语法的协议时会报错，xxx.proto:35:3: Expected "required", "optional", or "repeated"
            * 若有两个不同版本的protobuf，一般是不兼容的，要同时用的话其中一个可使用静态库的方式
            * grpc用的是protocbuf 3.x版本
            * 自动安装出问题的话，也可以手动到`cd grpc/third_party/protobuf`，然后执行`make install`(`make`是编译grpc时才执行的)
        + `make install`
            * 遇到的问题：
                - 安装后找不到grpc相关的pkgconfig，`gpr.pc` `grpc.pc` `grpc_unsecure.pc` `grpc++.pc` `grpc++_unsecure.pc`
                - `/usr/local/lib/pkgconfig`目录
                - 重试过好多次，调整选项还是没有。。。从另外一台机器拷贝过来了

* 实际使用遇到的问题
    - 使用`.set_`方法时，int 和 string类型混用，导致段错误，报错 `CHECK failed: value != nullptr:`
        + 注意字段格式！ 转换体里面最好也是加一个 `try...catch`
    - int 和 string类型，经常性的会用到`stoi/stod`，最好加一个`try...catch`捕获异常，常会有数据来源有问题的情况，这时不捕获一下很难定位到哪个位置，或可能怀疑到别的问题上去了

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
            - 客户端超时选项：使用 `context.set_deadline(timespec);`
                + `std::chrono::system_clock::time_point deadline = std::chrono::system_clock::now() + std::chrono::seconds(client_connection_timeout);` 用的是C++11的std::chrono库
                + `context.set_deadline(deadline);` (每个grpc客户端的ClientContext context;)
                + *解析err场景*: 循环读取stream，**若一直连接则不应该使用超时**(参考go章节的超时说明，搜索："对于stream，由于连接一直在的，设置的deadline超时会将正常持续的stream置为超时?")
                    * `while (reader->Read(&res))`
                    * Read退出while后，检查是否正常结束，`Status status = reader->Finish();`
                    * 若非ok，`if (status.ok())`，`status.error_code()`，`status.error_message()`
            - 若为服务端
                + 使用`if (context->IsCancelled()) {}` 进行检查客户端是否超时或取消
            - 可参考(里面包含了C++和Go的服务端和客户端设置示例)：[grpc deadlines](https://www.cnblogs.com/029zz010buct/p/9487568.html)
                + 其中例子来源于官网文档：[gRPC and Deadlines](https://grpc.io/blog/deadlines/)
        * 类似本地方法一样调用方法：`Status status = stub_->GetFeature(&context, point, feature);`
        * service定义：`rpc GetFeature(Point) returns (Feature) {}`
        * 调用：`Status status = stub_->GetFeature(&context, point, feature);`
        * 注意：服务端实现时，生成的代码入参为指针，而客户端调用时，送的是引用
        * 注意：`grpc::Status` 是一个类，而不是一个枚举值
            - `bool ok() const { return code_ == StatusCode::OK; }`
                + 判断是否调用正常
            - `StatusCode error_code() const { return code_; }`
                + 调用返回的错误码
            - `grpc::string error_message() const { return error_message_; }`
                + 调用返回的错误信息
            - `grpc::string error_details() const { return binary_error_details_; }`
                + 返回的详细信息(通常是`google.rpc.Status`序列化后的信息)
            - [Status定义](https://grpc.github.io/grpc/cpp/grpcpp_2impl_2codegen_2status_8h_source.html)
            - `StatusCode::OK`才是枚举，[StatusCode枚举值](https://grpc.github.io/grpc/cpp/namespacegrpc.html#aff1730578c90160528f6a8d67ef5c43baf6f3078af147d683afc70e09695c7a65)
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

## 负载均衡

* [Load Balancing in gRPC](https://github.com/grpc/grpc/blob/master/doc/load-balancing.md)
* [gRPC负载均衡（客户端负载均衡）](https://www.cnblogs.com/FireworksEasyCool/p/12912839.html)

## 客户端调用过程分析

* `conn, err := grpc.Dial(serverAddr, xxx...)`
    - `Dial`创建一个客户端连接，默认情况下，该函数是非阻塞的(不会等到连接创建完成才返回，而是后台进行连接)
        + 可指定 `grpc.WithBlock()`来使其变成阻塞，阻塞直到创建连接成功，若一直不成功则阻塞到超时(go test超时为10min)
        + e.g. `conn, err := grpc.Dial(serverAddr, grpc.WithBlock())`
    - 可指定多个选项
* 函数调用
    - 原型：`func Dial(target string, opts ...DialOption) (*ClientConn, error)`
        + 函数体只有：`return DialContext(context.Background(), target, opts...)`
    - `func DialContext(ctx context.Context, target string, opts ...DialOption) (conn *ClientConn, err error)`
        + 在非阻塞情况下，ctx不会对连接进行操作
* gRPC client会解析域名，然后会维护一个lb负载均衡
    - [简析gRPC client 连接管理](https://blog.csdn.net/weixin_33806300/article/details/88803908)
    - gRPC 管理连接的方式，默认情况下，大于10s没有数据发送，gRPC 就会认为是个idle 连接。server 端会给client 端发送一个GOAWAY 的包。client 收到这个包之后就会主动关闭连接。下次需要发包的时候，就会重新建立连接

* [GRPC连接池的设计与实现](https://blog.didiyun.com/index.php/2019/12/30/4629/)

