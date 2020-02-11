## sql

```go
Db, err = sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/chitchat?parseTime=true")
    if err != nil {
        log.Fatal(err)
    }
```

返回的Db对象只是一个数据库操作的对象，它并不是一个连接。

go封装了连接池，不会暴露给开发者。当Db对象开始数据库操作的时候，go的连接池才会惰性的建立连接，查询完毕之后又会释放连接，连接会返回到连接池之中。

CURD(增删改查): Create, Update, Read, Delete

* 增：

```go
rs, err := Db.Exec("INSERT INTO posts (content, author) Values (?, ?)", post.Content, post.Author)

    为了避免sql注入，mysql参数则使用?占位符。
```

>sql注入：所谓SQL注入，就是通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。

* 删：

```go
rs, err := Db.Exec("DELETE FROM posts WHERE id=?", post.Id)
```

* 改：

```go
rs, err := Db.Exec("UPDATE posts SET author=? WHERE id=?", post.Author, post.Id)
```

* 查：

分查询获取单条和多条。

获取单条记录比较简单，只需要定义一个结构，再查询结果后Scan其值就好

单条：

```go
func RetrievePost(id int) (post Post, err error){
    post = Post{}
    err = Db.QueryRow("SELECT id, content, author FROM posts WHERE id=?", id).Scan(
        &post.Id, &post.Content, &post.Author)
    return
}
```

多条：

```go
func RetrievePosts()(posts []Post, err error){

    rows, err := Db.Query("SELECT id, content, author FROM posts")
    for rows.Next(){
        post := Post{}
        err := rows.Scan(&post.Id, &post.Content, &post.Author)
        if err != nil{
            log.Println(err)
        }
        posts = append(posts, post)
    }
    rows.Close()
    return
}

此处注意 rows.Close()，迭代rows的过程中，如果因为循环内的代码执行问题导致循环退出，此时数据库连接池并不知道连接的情况，不会自动回收，因此需要手动指定rows.Close方法。
```


### mysql go

#### open连接

func Open(driverName, dataSourceName string) (*DB, error)

参数dataSourceName的格式：

`数据库用户名:数据库密码@[tcp(localhost:3306)]/数据库名`

e.g.
root:hikc45b6@tcp(127.0.0.1:3306)/xd_new_schema"

Open函数可能只是验证其参数，而不创建与数据库的连接。如果要检查数据源的名称是否合法，应调用返回值的Ping方法。

#### 查询列表Query
读取mysql的数据需要有一个绑定的过程，db.Query方法返回一个rows对象，这个数据库连接随即也转移到这个对象，因此我们**需要定义row.Close**操作。

```go
router.GET("/persons", func(c *gin.Context) {
  rows, err := db.Query("SELECT id, first_name, last_name FROM person")
  defer rows.Close()

  if err != nil {
   log.Fatalln(err)
  }

  persons := make([]Person, 0)

  for rows.Next() {
   var person Person
   rows.Scan(&person.Id, &person.FirstName, &person.LastName)
   persons = append(persons, person)
  }
  if err = rows.Err(); err != nil {
   log.Fatalln(err)
  }

  c.JSON(http.StatusOK, gin.H{
   "persons": persons,
  })

 })
```

>使用make，而不是直接使用var persons []Person的声明方式。还是有所差别的，使用make的方式，当数组切片没有元素的时候，Json会返回[]。如果直接声明，json会返回null。

#### 查询单条QueryRow

```go
err := db.QueryRow("SELECT id, first_name, last_name FROM person WHERE id=?", id).Scan(
   &person.Id, &person.FirstName, &person.LastName,
  )
```


## redis

>Remote Dictionary Server(Redis)是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API。
它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Map), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。


Redis本质上也是一种键值数据库的，但它在保持键值数据库简单快捷特点的同时，又吸收了部分关系数据库的优点。

通常局限点来说，Redis也以消息队列的形式存在，作为内嵌的List存在，满足实时的高并发需求。

[Redis快速入门](https://www.yiibai.com/redis/redis_quick_guide.html)

* Redis 特点

一、Redis数据库完全在内存中，使用磁盘仅用于持久性。

二、相比许多键值数据存储，Redis拥有一套较为丰富的数据类型。

三、Redis可以将数据复制到任意数量的从服务器。

* Redis 优势

1、异常快速：Redis的速度非常快，每秒能执行约11万集合，每秒约81000+条记录。

2、支持丰富的数据类型：Redis支持Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。这使得它非常容易解决各种各样的问题，因为我们知道哪些问题是可以处理通过它的数据类型更好。

3、操作都是原子性：所有Redis操作是原子的，这保证了如果两个客户端同时访问的Redis服务器将获得更新后的值。

4、多功能实用工具：Redis是一个多实用的工具，可以在多个用例如缓存，消息，队列使用(Redis原生支持发布/订阅)，任何短暂的数据，应用程序，如Web应用程序会话，网页命中计数等。

## gorm

### ORM概念:
ORM:(Object/Relation Mapping): 对象/关系映射

ORM的实现思想：
将关系数据库中表中的记录映射成为对象，以对象的形式展现，程序员可以把对数据库的操作转化为对对象的操作。
因此ORM的目的是为了方便开发人员以面向对象的思想来实现对数据库的操作。

ORM需要解决的问题是，能否把对象的数据直接保存到数据库中，又能否直接从数据库中拿到一个对象？要想做到上面两点，则必须要有映射关系。

### ORM的优缺点

优点：
orm的技术特点，提高了开发效率。可以自动对实体Entity对象与数据库中的Table进行字段与属性的映射；不用直接SQL编码，能够像操作对象一样从数据库中获取数据

缺点：
orm会牺牲程序的执行效率和会固定思维模式，在从系统结构上来看，采用orm的系统多是多层系统的，系统的层次太多，效率就会降低，orm是一种完全面向对象的做法，所以面向对象的做法也会对性能产生一定的影响。

ORM只是一种帮助我们解决一些重复的、简单的劳动，我们不能一劳永逸的靠工具来解决问题，有些特殊问题还是需要进行特殊处理的。

### GORM

* [GORM 中文文档](http://gorm.book.jasperxu.com/)
* [官网文档](https://gorm.io/docs/index.html)
* `GORM`: The fantastic ORM library for Golang
    -  注意和Grails框架的GORM区分开，`Grails' object relational mapping (ORM)` 是 `Grails` 的 ORM 实现，基于 Hibernate 3。 Grails 是一套用于快速 Web 应用开发的开源框架，它基于 Groovy 编程语言，并构建于 Spring、Hibernate 和其它标准 Java 框架之上，从而为大家带来一套能实现超高生产力的一站式框架。
        + **注意，此处的for MongoDB，并不是go中的gorm**
        + 参考：[GORM for MongoDB 3.0​ 发布](https://www.oschina.net/news/51215/gorm-for-mongodb-3-0%E2%80%8B)
* 数据库驱动
    - 关于MongoDB
        + mongodb官方**没有**关于go的mongodb的驱动，因此只能使用第三方驱动
        + `mgo`就是使用最多的一种，mgo（音mango）是MongoDB的Go语言驱动，它用基于Go语法的简单API实现了丰富的特性，并经过良好测试。
        + [golang中使用mongodb的操作类以及如何封装](https://www.cnblogs.com/spnt/p/4686128.html)
        + [mgo官网文档](https://gopkg.in/mgo.v2)
            * `go get gopkg.in/mgo.v2`
            * `import "gopkg.in/mgo.v2"`
                - `Find()`方法来根据条件查询collect(对应sql中的表)
                - 条件在代码中的格式可参考：[Specify AND Conditions](https://docs.mongodb.com/manual/tutorial/query-documents/#specify-and-conditions)
                - 由于上面链接中的bson.A找不到，使用bson.M来实现or，[Golang MongoDB bson.M查询&修改](https://blog.csdn.net/LightUpHeaven/article/details/82663146)
                - [golang和mongodb中的ISODate时间交互问题](https://www.tuicool.com/articles/J7zqiuJ)

示例(mgo流程和bson序列化)：

```golang
import "gopkg.in/mgo.v2"

type DbResult struct {
    ID string `bson:"ID`
    Name string `bson:"Name"`
    Age int `bson:"age"`    //bson标签是在MongoDB中的字段名称，注意struct字段名和Mongo中一致时，也需要定义bson
}

// 数据库
session, err := mgo.Dial(mongoURL)
if err != nil {
    panic("failed to connect database")
}
defer session.Close()
// 获取collection
collect := session.DB("student").C(collectname)

dbresult := []*DbResult{}
query := bson.M{
    "Id": bson.M{"$gt": 5}, // Id>5 and (Name=name1 or Name=name2)
    "$or": []bson.M{
        bson.M{"Name": name1},
        bson.M{"Name": name2},
    }}
finderr := collect.Find(query).Limit(10).All(&dbresult)
//直接按字段读取dbresult的值即可
log.Println(dbresult[0].ID)
```

可以加上 db.SingularTable(true) 让gorm转义struct名字的时候不用加上s

* 记录操作:

```go
// 创建
  db.Create(&Product{Code: "L1212", Price: 1000})
// 读取
  db.First(&product, 1) // 查询id为1的product
  db.First(&product, "code = ?", "L1212") // 查询code为l1212的product
// 更新 - 更新product的price为2000
  db.Model(&product).Update("Price", 2000)
// 删除 - 删除product
  db.Delete(&product)


  // 获取所有记录
db.Find(&users)
// 获取最后一条记录，按主键排序
db.Last(&user)
```


* 连接:

```go
  func main() {
  db, err := gorm.Open("mysql", "user:password@/dbname?charset=utf8&parseTime=True&loc=Local") //为了处理time.Time，您需要包括parseTime作为参数。 loc 设置时区
  defer db.Close()
}
```

* 表操作：

```go
// 检查模型`User`表是否存在
db.HasTable(&User{})
// 检查表`users`是否存在
db.HasTable("users")

// 为模型`User`创建表
db.CreateTable(&User{})

// 删除模型`User`的表
db.DropTable(&User{})
// 删除表`users`
db.DropTable("users")

// 删除模型`User`的表和表`products`
db.DropTableIfExists(&User{}, "products")

// 修改模型`User`的description列的数据类型为`text`
db.Model(&User{}).ModifyColumn("description", "text")

// 删除模型`User`的description列
db.Model(&User{}).DropColumn("description")

// 添加外键
db.Model(&User{}).AddForeignKey("city_id", "cities(id)", "RESTRICT", "RESTRICT")

// 为`name`列添加索引`idx_user_name`
db.Model(&User{}).AddIndex("idx_user_name", "name")
// 添加唯一索引
db.Model(&User{}).AddUniqueIndex("idx_user_name", "name")
// 删除索引
db.Model(&User{}).RemoveIndex("idx_user_name")
```

* gorm.Model 结构体
包括字段ID，CreatedAt，UpdatedAt，DeletedAt

**删除具有DeletedAt字段的记录，它不会冲数据库中删除，但只将字段DeletedAt设置为当前时间，并在查询时无法找到记录**

软删除:
如果模型有DeletedAt字段，它将自动获得软删除功能！
那么在调用Delete时不会从数据库中永久删除，而是只将字段DeletedAt的值设置为当前时间。
 软删除的记录将在查询时被忽略

 // 使用Unscoped查找软删除的记录
db.Unscoped().Where("age = 20").Find(&users)
// 使用Unscoped永久删除记录
db.Unscoped().Delete(&order)

* 列名

列名是字段名的蛇形小写
type User struct {
  ID uint             // 列名为 `id`
  Name string         // 列名为 `name`
  Birthday time.Time  // 列名为 `birthday`
  CreatedAt time.Time // 列名为 `created_at`
}

* 重设列名

type Animal struct {
    AnimalId    int64     `gorm:"column:beast_id"`         // 设置列名为`beast_id`
    Birthday    time.Time `gorm:"column:day_of_the_beast"` // 设置列名为`day_of_the_beast`
    Age         int64     `gorm:"column:age_of_the_beast"` // 设置列名为`age_of_the_beast`
}


omitempty，tag里面加上omitempy，可以在序列化的时候忽略0值或者空值
