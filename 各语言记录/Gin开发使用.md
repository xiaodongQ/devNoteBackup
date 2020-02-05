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

### restful路由

gin的路由来自httprouter库。因此httprouter具有的功能，gin也具有

#### :路由参数

冒号:加上一个参数名组成路由参数，可以使用c.Params的方法读取其值
(除了:，gin还提供了*号处理参数，*号能匹配的规则就更多)

```go
 router.GET("/user/:name", func(c *gin.Context) {
        name := c.Param("name")
        c.String(http.StatusOK, "Hello %s", name)
    })
}

c.String返回content-type是plain或者text
```


```go
 router.GET("/welcome", func(c *gin.Context) {
        firstname := c.DefaultQuery("firstname", "Guest")
        lastname := c.Query("lastname")

        c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
    })
```

#### query string 参数

客户端向服务器发送请求，除了路由参数，其他的参数无非两种，查询字符串query string和报文体body参数

* 所谓query string，即路由用?以后连接的key1=value2&key2=value2的形式的参数

这个key-value是经过urlencode编码

* 对于参数的处理，经常会出现参数不存在的情况，可指定默认值DefaultQuery

```go
router.GET("/welcome", func(c *gin.Context) {
        firstname := c.DefaultQuery("firstname", "Guest")
        lastname := c.Query("lastname")

        c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
    })
```

**注意，当firstname为空字串的时候，并不会使用默认的Guest值，空值也是值，DefaultQuery只作用于key不存在的时候，提供默认值。**

e.g.

```sh
☁  ~  curl http://127.0.0.1:8000/welcome
Hello Guest %

☁  ~  curl http://127.0.0.1:8000/welcome\?firstname\=%E4%B8%AD%E5%9B%BD
Hello 中国 %
```

#### body 参数

常见的格式就有四种。
例如application/json，application/x-www-form-urlencoded, application/xml 和 multipart/form-data(此格式主要用于图片上传)

默认情况下，c.PostFORM解析的是x-www-form-urlencoded或form-data的参数。

```go
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

调用c.JSON则返回json数据，其中gin.H封装了生成json的方式，是一个强大的工具。

使用golang可以像动态语言一样写字面量的json，对于嵌套json的实现，嵌套gin.H即可。
```

### 上传单个文件
[Golang 微框架 Gin 简介](https://www.jianshu.com/p/a31e4ee25305)

