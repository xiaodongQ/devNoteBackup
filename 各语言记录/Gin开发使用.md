## Gin

* Gin官网文档：[Documentation](https://gin-gonic.com/docs/)
* [Quickstart](https://gin-gonic.com/docs/quickstart/)
    - 下载安装Gin
        + `go get -u github.com/gin-gonic/gin`
        + 或手动下载https://github.com/gin-gonic/gin包，放到路径 src/github.com/gin-gonic/gin
            * github.com/ugorji/go/codec v1.1.7 依赖包缺少，拷贝过来
    - 使用
        + `import "github.com/gin-gonic/gin"`
        + 如果使用常量`http.StatusOK`，还要`import "net/http"`
        + 示例参考官网，有各种形式的请求和应答参数，如：[Examples Post url编码模式](https://gin-gonic.com/docs/examples/query-and-post-form/)

很简单的一个示例(注册了一个GET方法 http://localhost:8080/ping，应答为 {"message": "pong"})：

```golang
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    r.Run() // listen and serve on 0.0.0.0:8080
}
```

### restful路由

gin的路由来自`httprouter`库。因此httprouter具有的功能，gin也具有

#### :路由参数

冒号:加上一个参数名组成路由参数，可以使用`c.Params`的方法读取其值
(除了:，gin还提供了`*`号处理参数，`*`号能匹配的规则就更多)

```golang
 router.GET("/user/:name", func(c *gin.Context) {
        name := c.Param("name")
        c.String(http.StatusOK, "Hello %s", name)
    })
}

c.String返回content-type是plain或者text
```


```golang
 router.GET("/welcome", func(c *gin.Context) {
        firstname := c.DefaultQuery("firstname", "Guest")
        lastname := c.Query("lastname")

        c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
    })
```

#### query string 参数

客户端向服务器发送请求，除了路由参数，其他的参数无非两种，查询字符串`query string`和报文体`body参数`

* 所谓`query string`，即路由用`?`以后连接的`key1=value2&key2=value2`的形式的参数

这个key-value是经过`urlencode编码`

* 对于`body参数`的处理，经常会出现参数不存在的情况，可指定默认值`DefaultQuery`

```golang
router.GET("/welcome", func(c *gin.Context) {
        firstname := c.DefaultQuery("firstname", "Guest")
        lastname := c.Query("lastname")

        c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
    })
```

**注意，当firstname为空字串的时候，并不会使用默认的Guest值，空值也是值，DefaultQuery只作用于key`不存在`的时候，提供默认值。**

e.g.

```sh
☁  ~  curl http://127.0.0.1:8000/welcome
Hello Guest %

☁  ~  curl http://127.0.0.1:8000/welcome\?firstname\=%E4%B8%AD%E5%9B%BD
Hello 中国 %
```

#### body 参数

* 常见的格式就有四种。
    - 例如`application/json`，`application/x-www-form-urlencoded`, `application/xml` 和 `multipart/form-data`(此格式主要用于图片上传)

* 默认情况下，c.PostFORM解析的是`x-www-form-urlencoded`或`form-data`的参数。

http请求：

```golang
POST /form_post HTTP/1.1
Content-Type: application/x-www-form-urlencoded

message=this_is_great&nick=123
```

```golang
router.POST("/form_post", func(c *gin.Context) {
        message := c.PostForm("message")
        nick := c.DefaultPostForm("nick", "anonymous")

        c.JSON(http.StatusOK, gin.H{
            "status":  gin.H{
                "status_code": http.StatusOK,
                "status":      "ok",
            },
            "message": message,
            "nick":    nick,
        })
    })

调用c.JSON则返回json数据，其中`gin.H`封装了生成json的方式，是一个强大的工具。

使用golang可以像动态语言一样写字面量的json，对于嵌套json的实现，嵌套gin.H即可。
```

* 如果post请求是json格式 `application/json`
    - 需要做json和结构体的绑定
        + `type TestInfo struct{}`定义结构体(注意成员名首字母大写)，使用`json:"id"`形式的标签进行结构体和json绑定
        + 调用`BindJSON`方法，对请求json和定义的结构体绑定
    - 参考：[gin框架post路由的使用](https://juejin.im/post/5bd40c5d51882528366375c1#heading-5)

### 上传单个文件
[Golang 微框架 Gin 简介](https://www.jianshu.com/p/a31e4ee25305)

