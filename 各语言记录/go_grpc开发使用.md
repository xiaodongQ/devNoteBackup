### gRPC环境和protoc.exe、protoc-gen-go.exe

**gRPC调用时数据错位，可能是客户端pb协议和服务端不一致**

[gRPC-go protoc](https://www.jianshu.com/p/ec3e75e5aad1)
作者：Feng_Sir
链接：https://www.jianshu.com/p/ec3e75e5aad1
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

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

