## MongoDB

MongoDB并非芒果(mango)的意思，而是源于 Humongous（巨大）一词。

[MongoDB 教程](https://www.runoob.com/mongodb/mongodb-tutorial.html)

MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。
MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

* 官网文档：
    - 关于手册
        + [guide, manual, tutorial之间的区别](https://www.cnblogs.com/jiangleads/p/11238232.html)
        + `Tutorial`，教程(tutorial)是一系列课程，侧重于给没有经验的人一步一步进行详细的指导
        + `Guide`，指南(guide)是一个简单的“操作方法”，有足够的信息可以帮助您入门。
        + `Manual`，手册(manual)是一套完整，深入的说明
    - [Connect to MongoDB](https://docs.mongodb.com/guides/server/drivers/)
        + 更详细深入的文档，查看Manual:[The MongoDB 4.2 Manual](https://docs.mongodb.com/manual/#the-mongodb-version-manual)
        + 安装客户端shell并连接MongoDB
            * [Procedure](https://docs.mongodb.com/guides/server/drivers/#check-your-environment)
            * 过程
                - 下载安装：
                    + 按说明中的提示进行下载，此处选`Linux`点进链接后->点击`Server`->出来选择页面可以选择`Version`+`OS`(RHEL 7.0，针对自己的CentOS虚拟机)+`Package`(shell)。下载得到"mongodb-org-shell-4.2.2-1.el7.x86_64.rpm"，进行安装`rpm -ivh xxx.rpm`
                - 安装后：则会有`mongo`可执行文件(一般在/usr/bin下，可which mongo查看)，若路径没有在环境变量中则可添加
                - 获取MongoDB连接字符串
                    + 格式：`mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[database][?options]]`
                    + [Connection String URI Format](https://docs.mongodb.com/manual/reference/connection-string/#mongodb-uri)
                - 连接到MongoDB实例
                    + `mongo mongodb://$[hostlist]/$[database]?authSource=$[authSource] --username $[username]`
                    + e.g. `mongo "mongodb://192.168.50.118:27017/?authSource=xdtest --username admin"` 即可进入MongoDB操作shell终端
                    + e.g. 如果没有设置访问控制，则 `mongo "mongodb://192.168.50.118"`会直接连接(端口默认27017)
                    + 可见下面的`MongoDB - 连接`章节
                - 进行CRUD操作，shell终端里特别注意中英文字符不要打错了(肉眼看不明显)
                    + 查看下面的`插入文档` `查询文档` `创建集合` 等
        + 具体CRUD操作和其他MongoDB Shell的方法，参考`Manual`手册
            * [MongoDB CRUD Operations](https://docs.mongodb.com/manual/crud/)
    - C++ API
        + [MongoDB C++ Driver](http://mongocxx.org/api/current/)，可以在搜索框搜索api

## centos7 mongodb c++驱动安装

* [mongodb c++ 驱动](https://www.jianshu.com/p/c982a2960175)
    - 翻译来自官网链接：[Installing the mongocxx driver](http://mongocxx.org/mongocxx-v3/installation/)
    - 1. 安装c驱动，见下面的步骤(**步骤一**)
    - 2. 下载最新的 mongocxx driver
        + `git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1`
        + `cd mongo-cxx-driver/build`
    - 3. 配置驱动
        + `cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..` (CentOS)
    - 4. 编译和安装驱动
        + 若用默认的 MNMLSTC 的C++17 `make EP_mnmlstc_core` (实际安装未执行该步，不确定是否有影响，make正常安装成功，应该是之前已经安装了boost的原因，若无boost则需要进行该步骤)
            * 编译步骤中有：`Performing download step (git clone) for 'EP_mnmlstc_core'`，会clone `EP_mnmlstc_core`
            * 关于 `MNMLSTC Core`，MNMLSTC核心是一个c++ 11库，它添加了c++ 14及以后版本中包含的一些库特性，特性内容可以参考链接
                - mongocxx驱动用了C++17的`std::optional` `std::string_view`特性，需要选择用`MNMLSTC/core`还是`boost`，两种库都有该特性，非Windows平台默认用的是`MNMLSTC/core`(可以通过宏配置)，官网链接做了说明：[Step 2: Choose a C++17 polyfill](http://mongocxx.org/mongocxx-v3/installation/)进行了说明
                - 若要了解MNMLSTC Core可以参考：[MNMLSTC Core](http://mnmlstc.github.io/core/)
            * 在一台ubuntu上尝试部署时MNMLSTC这个模块报错了，指定了使用boost(需另外安装)，第3步的配置进行调整：`cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DBSONCXX_POLY_USE_MNMLSTC:Bool=OFF -DBSONCXX_POLY_USE_BOOST:Bool=ON -DBOOST_INCLUDEDIR=/usr/local/boost/include ..`
                - 但是有的应用程序并不需要使用到boost，该方式下也需要包含boost的头，所以还是放弃了这种方式
                - 尝试git clone最新版本的mongocxx driver(安装时提示要求mongodb C驱动的版本至少为1.15.0)，下面的make时会去下载：`[  1%] Performing download step (git clone) for 'EP_mnmlstc_core'`
        + `make && make install`

* C驱动安装(**步骤一**)
    - 参考: [mongodb c 驱动](https://www.jianshu.com/p/d77680254418) (第一步安装c驱动，翻译链接)
        + 翻译来自官网链接：[Installing the MongoDB C Driver (libmongoc) and BSON library (libbson)](http://mongoc.org/libmongoc/current/installing.html)

```sh
# 版本可能变化，比如我在一台新机器安装时mongocxx要求1.15.0(截止20200409最新版本1.16.2)
# 当前版本可查看：https://github.com/mongodb/mongo-c-driver/releases/
  $ wget https://github.com/mongodb/mongo-c-driver/releases/download/1.16.2/mongo-c-driver-1.16.2.tar.gz
  $ tar xzf mongo-c-driver-1.16.2.tar.gz
  $ cd mongo-c-driver-1.16.2
  $ mkdir cmake-build
  $ cd cmake-build
  $ cmake -DENABLE_AUTOMATIC_INIT_AND_CLEANUP=OFF ..

  $ make
  $ make install
```

## mongodb安装(服务端安装)

* 官网有一系列平台的安装指导
    - 参考：[Install MongoDB Community Edition on Red Hat or CentOS](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)
    - docker安装：
        + `docker pull mongo`
        + 启动：`docker run -p 27017:27017 --restart=always --name mymongodb -d mongo`
            * `-v <LocalDirectoryPath>:/data/db` 可选，指定数据存储位置
            * 指定路径：`docker run -p 27017:27017 --restart=always -v /home/data/mongo:/data/db --name mymongodb -d mongo`
            * 指定配置文件：`-v /home/data/mongo_conf:/data/configdb`


* 通过下载tar包安装(非官方文档)：[Linux平台安装MongoDB](https://www.runoob.com/mongodb/mongodb-linux-install.html)
    - `curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.12.tgz`   # 下载 (对应版本修改下)
    - `tar -zxvf mongodb-linux-x86_64-3.0.6.tgz`                                   # 解压
    - `mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb`                         # 将解压包拷贝到指定目录
    - MongoDB 的可执行文件位于 bin 目录下，所以可以将其添加到 PATH 路径中：
        + `export PATH=<mongodb-install-directory>/bin:$PATH`
        + <mongodb-install-directory> 为MongoDB的安装路径。如本文的 /usr/local/mongodb 。
*  创建数据库目录
    - MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。
    - `mkdir -p /data/db`
* 命令行中运行 MongoDB 服务
    - `$ ./mongod`
* 创建用户，并设置密码
    - `db.createUser({user:'admin',pwd:'123qwe',roles:[{role:'userAdminAnyDatabase',db:'admin'}]})`
* MongoDB后台管理 Shell
    - 如果你需要进入MongoDB后台管理，你需要先打开mongodb装目录的下的bin目录，然后执行mongo命令文件。
    - MongoDB Shell是MongoDB自带的交互式Javascript shell,用来对MongoDB进行操作和管理的交互式环境。
    - 当你进入mongoDB后台后，它默认会链接到 test 文档（数据库）：

```sh
$ cd /usr/local/mongodb/bin
$ ./mongo
MongoDB shell version: 3.0.6
connecting to: test
Welcome to the MongoDB shell.
```

* 连接池
    - [examples/mongocxx/pool.cpp](https://github.com/mongodb/mongo-cxx-driver/blob/master/examples/mongocxx/pool.cpp)
    - 使用 `mongocxx::pool`
    - url添加pool大小：`mongodb://192.168.50.118:27017/?authSource=xdtest&minPoolSize=5&maxPoolSize=10`
    - api
        + 官网说明：[Connection pools](http://mongocxx.org/mongocxx-v3/connection-pools/)
            * [examples](mongo-cxx-driver/examples/mongocxx/pool.cpp)
        + [pool.hpp](http://mongocxx.org/api/current/pool_8hpp_source.html)
        + [mongocxx::client](http://mongocxx.org/api/current/classmongocxx_1_1client.html)

```cpp
#include <mongocxx/pool.hpp>

    mongocxx::instance inst{};
    mongocxx::uri uri{"mongodb://localhost:27017/?minPoolSize=3&maxPoolSize=3"};
    mongocxx::pool pool{uri};

    // mongocxx::client，每个线程用一个client 不要多个复用
    auto client = pool.acquire();
    // (*client)["test"] 为 mongocxx::database
    // coll 为 mongocxx::collection
    auto coll = (*client)["test"][collection_names];

    // coll.insert_one，find等操作
    // mongocxx::cursor cur = coll.find(xxx);
```

* double精度问题
    - 货币使用 Decimal类型 (New in version 3.4.)
    - `NumberDecimal()`
        + `NumberDecimal("2.099")`
    - [Using the Decimal BSON Type](https://docs.mongodb.com/manual/tutorial/model-monetary-data/index.html#using-the-decimal-bson-type)

### NoSQL

关系数据库管理系统（Relational Database Management System：RDBMS）
    关系模型是非常适合于客户服务器编程，远远超出预期的利益，今天它是结构化数据存储在网络和商务应用的主导技术。
    关系型数据库遵循ACID规则
        1、A (Atomicity) 原子性
            事务里的所有操作要么全部做完，要么都不做
        2、C (Consistency) 一致性
            数据库要一直处于一致的状态，事务的运行不会改变数据库原本的一致性约束。
                例如现有完整性约束a+b=10，如果一个事务改变了a，那么必须得改变b，使得事务结束后依然满足a+b=10，否则事务失败。
        3、I (Isolation) 隔离性
            指并发的事务之间不会互相影响
        4、D (Durability) 持久性
            指一旦事务提交后，它所做的修改将会永久的保存在数据库上，即使出现宕机也不会丢失。

NoSQL 是一项全新的数据库革命性运动 提倡运用非关系型的数据存储
    NoSQL有时也称作`Not Only SQL`的缩写，是对不同于传统的关系型数据库的数据库管理系统的统称。
    NoSQL用于**超大规模数据**的存储。

RDBMS vs NoSQL
RDBMS
- 高度组织化结构化数据
- 结构化查询语言（SQL） (SQL)
- 数据和关系都存储在单独的表中。
- 数据操纵语言，数据定义语言
- 严格的一致性
- 基础事务

NoSQL
- 代表着不仅仅是SQL
- 没有声明性查询语言
- 没有预定义的模式
- 键-值对 存储，列存储，文档存储，图形数据库
- 最终一致性，而非ACID属性
- 非结构化和不可预知的数据
- CAP定理(CAP theorem)
    + 在计算机科学中, CAP定理（CAP theorem）, 又被称作 布鲁尔定理（Brewer's theorem）,它指出对于一个分布式计算系统来说，不可能同时满足以下三点:
        * 一致性(Consistency) (所有节点在同一时间具有相同的数据)
        * 可用性(Availability) (保证每个请求不管成功或者失败都有响应)
        * 分隔容忍(Partition tolerance) (系统中任意信息的丢失或失败不会影响系统的继续运作)
    + CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。
    + 因此，根据 CAP 原理将 NoSQL 数据库分成了满足`CA`原则、满足`CP`原则和满足`AP`原则三大类：
        * CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
        * CP - 满足一致性，分区容忍性的系统，通常性能不是特别高。
        * AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。
    + `BASE` 是NoSQL数据库通常对可用性及一致性的弱要求原则:
        * Basically Availble --基本可用
        * Soft-state --软状态/柔性事务。 "Soft state" 可以理解为"无连接"的, 而 "Hard state" 是"面向连接"的
        * Eventual Consistency -- 最终一致性， 也是是 ACID 的最终目的。
- 高性能，高可用性和可伸缩性

* NoSQL 数据库分类
    - 列存储
        + (按列存储数据的。最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。)
        + Hbase
        + Cassandra
        + Hypertable
    - 文档存储
        + (文档存储一般用类似json的格式存储，存储的内容是文档型的。这样也就有机会对某些字段建立索引，实现关系数据库的某些功能。)
        + MongoDB
        + CouchDB
    - key-value存储
        (通过key快速查询到其value。一般来说，存储不管value的格式，照单全收。)
        + Tokyo Cabinet / Tyrant
        + Berkeley DB
        + MemcacheDB
        + Redis
    - 图存储
        + (图形关系的最佳存储。使用传统关系数据库来解决的话性能低下，而且设计使用不方便。)
        + Neo4J
        + FlockDB
    - 对象存储
        + (通过类似面向对象语言的语法操作数据库，通过对象的方式存取数据。)
        + db4o
        + Versant
    - xml数据库
        + (高效的存储XML数据，并支持XML的内部查询语法，比如XQuery,Xpath。)
        + Berkeley DB XML
        + BaseX

### MongoDB

MongoDB 将数据存储为一个**文档**，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 `JSON` 对象。
字段值可以包含其他文档，数组及文档数组。

2007年10月，MongoDB由10gen团队所发展。2009年2月首度推出。

* MongoDB适用场景
    - [1.3.10 MongoDB适用场景](https://www.cnblogs.com/clsn/p/8214194.html#auto_id_19)
    - 网站数据、缓存等大尺寸、低价值的数据
    - 在高伸缩性的场景，用于对象及JSON数据的存储
* 慎用场景
    - `PB数据`持久存储、大数据分析、数据湖(数据湖最初由大数据厂商提出)

* 业务场景(股票十档快照和逐笔成交数据)为什么要选择mongodb
    - 数据
        + 十档数据，每天平均 1700 万条数据，内存中大小15.02 GB，对象平均大小935 bytes，存储大小6.09 GB，索引(2个索引，Code字段升序162.59MB、默认`_id`一个索引)总大小328.27 MB
        + 逐笔数据，平均每天 5000 万条数据，内存中大小6.48 GB，对象平均大小118 bytes，存储大小1.10 GB，索引(2个索引，Code升序330.06 MB、)总大小917.91 MB
    - 理由
        + 读多写少，不需要做多表关联，不需要事务(MongoDB支持事务，且支持仅限于本机的单文档事务)
        + 文档型记录，表字段版本不同可能修改(不过不频繁)，不需要迁移
        + 集群易扩展(暂时没用到)
    - [为什么使用MongoDB？](https://developer.aliyun.com/article/505850)
        + MongoDB的优势：大数据量高性能，易扩展，高可用性，轻松实现大数据量的存储
        + mongoDB(非结构化数据库)不仅可以处理结构化数据，而且更适合处理非结构化数据(文本、图像、超媒体等信息)。它突破了关系型数据库结构定义不易改变而且数据定长的限制，在处理连续信息和非结构化信息中有着关系型数据库无法比拟的优势
            * 关系型数据库不擅长：大量数据的写入；字段不固定，表结构变更；简单查询需要快速返回结果
        + 特性
            * 面向文档存储：BSON
            * 面向集合存储：易存储对象类型的数据
            * 强大的查询功能：支持丰富的查询表达式。使用json形式的标记，可以查询文档中内嵌的对象及数组
            * 完整的查询功能：全索引支持包括内嵌对象及数组。查询优化器生成高效查询计划
            * 模式自由：集合中的键不要求一致，键值不要求一致的数据类型
            * 复制（主从复制／副本集）及自动故障转移:用于故障恢复，`读扩展`;Mongo数据库支持服务器之间的数据复制，支持主从模式以及服务器之间的相互复制
            * 分片（自动分片）：用于负载均衡，`写扩展`。支持水平的数据库集群，动态添加额外的机器
            * GridsFS：避免文件系统对目录做文件数目的限制。不会产生磁盘碎片
            * 支持RUBY，PYTHON，JAVA，C++等多语言，多平台
            * 查询监视：监视工具用于分析数据库操作性能
            * 缓存服务：对关系性数据库数据进行缓存，减少数据压力
            * 自动处理碎片
        + 不同于MySQL索引用的B+树，MongoDB用的是`B树`(即`B-树`，此处`-`并不是减号，而是连接符)：[为什么 MongoDB 使用 B 树](https://draveness.me/whys-the-design-mongodb-b-tree/)
            * B-树的查询`最好`时间复杂度是 `O(1)`
            * 由于B+树所有的 data 域都在根节点，所以查询指定 key 的节点必须从根节点索引到叶节点，时间复杂度固定为 `O(log n)`
            * B+树叶节点两两相连可大大增加区间访问性，可使用在范围查询等，而B-树每个节点 key 和 data 在一起，则无法区间查找

#### 概念

在mongodb中基本的概念是文档、集合、数据库


SQL术语/概念    MongoDB术语/概念    解释/说明
database        database            数据库
table           collection          数据库表/集合
row             document            数据记录行/文档
column          field               数据字段/域
index           index               索引
table           joins               表连接,MongoDB不支持
primary key     primary key         主键,MongoDB自动将_id字段设置为主键

数据库服务和客户端(针对mysql/oracle/mongodb)
Mysqld/Oracle   mongod
mysql/sqlplus   mongo

##### 数据库

一个mongodb中可以建立多个数据库。
MongoDB的单个实例可以容纳多个独立的数据库，每一个都有自己的集合和权限，不同的数据库也放置在不同的文件中。
    "show dbs" 命令可以显示所有数据的列表。

```sql
        $ ./mongo   # 连接的是本地的服务，要连接其他设备上的服务，参考上面通过mongo uri进行连接
        MongoDB shell version: 3.0.6
        connecting to: test
        > show dbs
        local  0.078GB
        test   0.078GB
        >
```
    执行 "db" 命令可以显示当前数据库对象或集合。
    运行"use"命令，可以连接到一个指定的数据库。

数据库也通过名字来标识。数据库名可以是满足以下条件的任意UTF-8字符串。
    不能是空字符串（"")。
    不得含有' '（空格)、.、$、/、\和\0 (空字符)。
    应全部小写。
    最多64字节。

##### 文档(Document)

文档是一组键值(key-value)对(即 `BSON` )。 `Binary JSON`(二进制JSON)
MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这与关系型数据库有很大的区别，也是 MongoDB 非常突出的特点。

需要注意的是：
    文档中的键/值对是有序的。
    MongoDB区分类型和大小写。
    MongoDB的文档不能有重复的键。
    文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。

文档键命名规范：
    键不能含有\0 (空字符)。这个字符用来表示键的结尾。
    .和$有特别的意义，只有在特定环境下才能使用。
    以下划线"_"开头的键是保留的(不是严格要求的)。

##### 集合

集合就是 MongoDB 文档组，类似于 RDBMS （关系数据库管理系统：Relational Database Management System)中的表格。

集合存在于数据库中，集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。

capped collections
Capped collections 就是固定大小的collection。

db.createCollection("mycoll", {capped:true, size:100000})

##### 元数据

数据库的信息是存储在集合中。它们使用了系统的命名空间：
    dbname.system.*
在MongoDB数据库中名字空间 <dbname>.system.* 是包含多种系统信息的特殊集合(Collection)，如下:
    dbname.system.namespaces    列出所有名字空间。
    dbname.system.indexes   列出所有索引。
    dbname.system.profile   包含数据库概要(profile)信息。
    dbname.system.users 列出所有可访问数据库的用户。
    dbname.local.sources    包含复制对端（slave）的服务器信息和状态。

MongoDB 数据类型
    String
        在 MongoDB 中，UTF-8 编码的字符串才是合法的。
    Integer
    Boolean
    Double
    Min/Max keys
    Array
    Timestamp
        BSON 有一个特殊的时间戳类型用于 MongoDB 内部使用，与普通的 日期 类型不相关。
        时间戳值是一个 64 位的值。前32位是一个 time_t 值（与Unix新纪元相差的秒数）后32位是在某秒中操作的一个递增的序数
            BSON 时间戳类型主要用于 MongoDB 内部使用。在大多数情况下的应用开发中，你可以使用 BSON 日期类型。
    Object  用于内嵌文档。
    Null    用于创建空值
    Symbol  基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。
    Date
        表示当前距离 Unix新纪元（1970年1月1日）的毫秒数。日期类型是有符号的, 负数表示 1970 年之前的日期。
    Object ID     对象 ID。用于创建文档的 ID。
        ObjectId 类似唯一主键，可以很快的去生成和排序，包含 12 bytes，
            前 4 个字节表示创建 unix 时间戳,格林尼治时间 UTC 时间，比北京时间晚了 8 个小时
            接下来的 3 个字节是机器标识码
            紧接的两个字节由进程 id 组成 PID
            最后三个字节是随机数
    Binary Data   二进制数据。用于存储二进制数据。
    Code          代码类型。用于在文档中存储 JavaScript 代码。
    Regular expression 正则表达式类型。用于存储正则表达式。

#### MongoDB - 连接

  命令行中运行 MongoDB 服务
`$ ./mongod`

标准 URI 连接语法：

`mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]`
    mongodb:// 这是固定的格式，必须要指定。
    username:password@ 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登陆这个数据库
    host1 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制集，请指定多个主机地址。
    portX 可选的指定端口，如果不填，默认为27017
    /database 如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开 test 数据库。
    ?options 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开


使用默认端口来连接 MongoDB 的服务。
`mongodb://localhost`

使用用户 admin 使用密码 123456 连接到本地的 MongoDB 服务上。输出结果如下所示：
`mongodb://admin:123456@localhost/`


#### 创建数据库
`use DATABASE_NAME` 如果数据库不存在，则创建数据库，否则切换到指定数据库。

```sql
> use runoob
> db
> show dbs  可以看到，我们刚创建的数据库 runoob 并不在数据库的列表中， 要显示它，我们需要向 runoob 数据库插入一些数据。
> db.runoob.insert({"name":"菜鸟教程"})
> show dbs
```

MongoDB 中默认的数据库为 test，如果你没有创建新的数据库，集合将存放在 test 数据库中。
>注意: 在 MongoDB 中，集合只有在内容插入后才会创建! 就是说，创建集合(数据表)后要再插入一个文档(记录)，集合才会真正创建。

#### 删除数据库
`db.dropDatabase()`

```sql
接下来我们切换到数据库 runoob：
> use runoob

执行删除命令：
> db.dropDatabase()
```

#### MongoDB 创建集合
`db.createCollection(name, options)`
    name: 要创建的集合名称
    options: 可选参数, 指定有关内存大小及索引的选项
        capped 如果为 true，则创建固定集合。当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数。
        autoIndexId 如为 true，自动在 _id 字段创建索引。默认为 false。
        size 为固定集合指定一个最大值，以千字节计（KB）。
        max 指定固定集合中包含文档的最大数量
        在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。

```sql
在 test 数据库中创建 runoob 集合： #如上，MongoDB 中默认的数据库为 test，如果你没有创建新的数据库，集合将存放在 test 数据库中
> use test
switched to db test
> db.createCollection("runoob")
{ "ok" : 1 }
>

如果要查看已有集合，可以使用 show collections 或 show tables 命令：
> show collections
runoob
system.indexes
```

```sql
下面是带有几个关键参数的 createCollection() 的用法：
创建固定集合 mycol，整个集合空间大小 6142800 KB, 文档最大个数为 10000 个。
> db.createCollection("mycol", { capped: true, autoIndexId: true, size: 6142800, max: 10000 } )
{ "ok" : 1 }

在 MongoDB 中，你不需要创建集合。当你插入一些文档时，MongoDB 会自动创建集合。
> db.mycol2.insert({"name" : "菜鸟教程"})
> show collections
mycol2
```

#### MongoDB 删除集合
语法格式：
`db.collection.drop()`

删除集合 mycol2 :
`>db.mycol2.drop()`

清空集合 "mycol2" 的数据：
`db.mycol2.remove({})`

#### MongoDB 插入文档
MongoDB 使用 insert() 或 save() 方法向集合中插入文档，语法如下：
`db.COLLECTION_NAME.insert(document)`

以下文档可以存储在 MongoDB 的 runoob 数据库 的 col1 集合中：

```sql
>db.col1.insert({title: 'MongoDB 教程',
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
```

以上实例中 col1 是集合名，如果该集合不在该数据库中， MongoDB 会自动创建该集合并插入文档。

注意： json里面的成员可以是数组，像上面示例中的"tags"

查看已插入文档：

```sql
> db.col1.find()
{ "_id" : ObjectId("56064886ade2f21f36b03134"), "title" : "MongoDB 教程", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "mongodb", "database", "NoSQL" ], "likes" : 100 }
>
```

除了`insert`，插入文档还可以使用 `db.col1.save(document)` 命令

* 3.2 版本后还有以下几种语法可用于插入文档:
    - db.collection.insertOne():向指定集合中插入一条文档数据
        + `> var document = db.collection.insertOne({"a": 3})`
    - db.collection.insertMany():向指定集合中插入多条文档数据
        + `> var res = db.collection.insertMany([{"b": 3}, {'c': 4}])`

#### MongoDB 查询文档

* `db.collection.find(query, projection)`
    - query：可选，使用查询操作符指定查询条件
    - projection：可选，返回指定字段(键值)。查询时若返回文档中所有键值，只需省略该参数即可（默认省略）
    - e.g. `db.collection.find({}, {Name:1})`，
        + 查询所有记录(条件为空)，并只返回Name字段 和 `_id`字段 (`_id`字段默认返回)
        + 若要查询除某字段外所有字段，则`{Name:0}`形式即可
        + `db.collection.find({Class:5}, {Name:1, _id:0})` 只返回班级为5的所有记录中的Name字段(_id字段不返回)

* 如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：
    - `>db.col1.find().pretty()`
    - pretty() 方法以格式化的方式来显示所有文档。

* 除了 `find()` 方法之外，还有一个 `findOne()` 方法(注意：大小写敏感)，它只返回一个文档。

* [查询条件](https://docs.mongodb.com/manual/tutorial/query-documents/)
    - `db.inventory.find( { status: "A", qty: { $lt: 30 } } )`
* `$in`
    - `{ field: { $in: [<value1>, <value2>, ... <valueN> ] } }`
    - [$in](https://docs.mongodb.com/manual/reference/operator/query/in/)

##### MongoDB AND 条件
MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件。
`>db.col1.find({key1:value1, key2:value2}).pretty()`

e.g. 通过 "by" 和 "title" 键来查询 菜鸟教程 中 MongoDB 教程 的数据(by和title都是json中的字段键值，不是MongoDB的关键字)
`> db.col1.find({"by":"菜鸟教程", "title":"MongoDB 教程"}).pretty()`

* mongo中字段不存在或者为null：`{key:null}`

##### MongoDB OR 条件
MongoDB OR 条件语句使用了关键字 $or

```sql
>db.col1.find(
   {
      $or: [{key1: value1}, {key2:value2}]
   }
).pretty()
```

* e.g. `>db.col1.find({$or:[{"by":"菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()`

* e.g. 'where likes>50 AND (by = '菜鸟教程' OR title = 'MongoDB 教程')' ("likes"是json里面的键值，这个网站里的示例不好，老用sql的关键字作为键值)
    - `>db.col1.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()`
    - 注意，此处的or语法是作为一个整体当做and的后半部分，结构如 `条件1 and (条件2 or 条件3)`

注意

##### MongoDB 条件操作符
MongoDB 与 RDBMS Where 语句比较
如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

操作          格式                     范例                                        RDBMS中的类似语句
等于          {<key>:<value>}         db.col1.find({"by":"菜鸟教程"}).pretty()      where by = '菜鸟教程'
小于          {<key>:{$lt:<value>}}   db.col1.find({"likes":{$lt:50}}).pretty()    where likes < 50
小于或等于    {<key>:{$lte:<value>}}  db.col1.find({"likes":{$lte:50}}).pretty()    where likes <= 50
大于          {<key>:{$gt:<value>}}   db.col1.find({"likes":{$gt:50}}).pretty()    where likes > 50
大于或等于    {<key>:{$gte:<value>}}  db.col1.find({"likes":{$gte:50}}).pretty()    where likes >= 50
不等于        {<key>:{$ne:<value>}}   db.col1.find({"likes":{$ne:50}}).pretty()    where likes != 50

e.g. "likes" 大于 100 的数据
`db.col1.find({likes: {$gt:100}})`

##### $type操作符
是基于BSON(Bin­ary JSON)类型来检索集合中匹配的数据类型，并返回结果。

如果想获取 "col1" 集合中 title 为 String 的数据，你可以使用以下命令：

```sql
db.col1.find({"title" : {$type : 2}}) # 2为sting类型的编号
或
db.col1.find({"title" : {$type : 'string'}})
```

* 其他的类型对应关系，参考：[MongoDB $type 操作符](https://www.runoob.com/mongodb/mongodb-operators-type.html)
    - 注意MongoDB中默认的数字类型是 `double` (1)
    - Double    1
    - String    2
    - Object    3
    - Array 4
    - Binary data 5
    - Undefined   6   已废弃。
    - Object id   7
    - Boolean 8
    - Date    9
    - Null    10
    - Regular Expression  11
    - JavaScript  13
    - Symbol  14
    - JavaScript (with scope) 15
    - 32-bit integer  16
    - Timestamp   17
    - 64-bit integer  18
    - Min key 255 Query with -1.
    - Max key 127

##### limit
如果你需要在MongoDB中读取指定数量的数据记录，可以使用MongoDB的Limit方法
`>db.COLLECTION_NAME.find().limit(NUMBER)`

##### skip
我们除了可以使用limit()方法来读取指定数量的数据外，还可以使用skip()方法来跳过指定数量的数据
`>db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)`

* 实验结果：并不是先limit(m)获取到结果后，再skip(n)
    - 会先skip(n)，再limit(m)

#### MongoDB 排序

在 MongoDB 中使用 sort() 方法对数据进行排序，sort() 方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而 -1 是用于降序排列。
`>db.COLLECTION_NAME.find().sort({KEY:1})`

* 若为升序排列，则不存在该key或者key为null的文档排在前面

e.g. 以下实例演示了 col1 集合中的数据按字段 likes 的降序排列：

```sql
>db.col1.find({},{"title":1,_id:0}).sort({"likes":-1})
{ "title" : "PHP 教程" }
{ "title" : "Java 教程" }
{ "title" : "MongoDB 教程" }
>
```

#### MongoDB 索引
索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。

索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构

MongoDB使用 createIndex() 方法来创建索引。
`>db.collection.createIndex(keys, options)`
    语法中 Key 值为你要创建的索引字段，1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可。
    `>db.col1.createIndex({"title":1})`

1、查看集合索引

db.col1.getIndexes()
2、查看集合索引大小

db.col1.totalIndexSize()
3、删除集合所有索引

db.col1.dropIndexes()
4、删除集合指定索引

db.col1.dropIndex("索引名称")

#### MongoDB 更新文档

* 官网CRUD说明：[MongoDB CRUD Operations](https://docs.mongodb.com/manual/crud/)

* [MongoDB 更新文档](https://www.runoob.com/mongodb/mongodb-update.html)
* `update()`
    - `update()` 方法用于更新已存在的文档
    - `db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})`
        + 该命令只会修改第一条记录，若要修改多条满足条件的记录，设置`multi`参数为true
            * `db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})`
    - 语法：

```sql
db.collection.update(
   <query>,              #update的查询条件，类似sql update查询内where后面的
   <update>,             #update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
   {
     upsert: <boolean>,  #可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入
     multi: <boolean>,   #可选，mongodb默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新
     writeConcern: <document> #可选，抛出异常的级别
   }
)
```

* `save()`
    - `save()` 方法通过传入的文档来替换已有文档
    - 语法：

```sql
db.collection.save(
   <document>,                #文档数据
   {
     writeConcern: <document> #可选，抛出异常的级别
   }
)
```

* 示例
    - `db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );`
        + 只更新第一条记录
    - `db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );`
        + 全部更新
        + 相当于{upsert:false, multi:true}
    - `db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );`
        + 只添加第一条
        + 相当于{upsert:true, multi:false}
    - `db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );`
        + 全部添加进去
        + 相当于{upsert:true, multi:true}
    - `db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );`
        + 只更新第一条记录
        + 相当于{upsert:false, multi:false}

#### MongoDB 删除文档

* [Delete Documents](https://docs.mongodb.com/manual/tutorial/remove-documents/#delete-documents)
    - `db.collection名.deleteMany({xxx})` 删除所有
    - `db.collection名.deleteOne({xxx})` 删除单个
* [MongoDB 删除文档](https://www.runoob.com/mongodb/mongodb-remove.html)
    - `remove()`函数用来移除集合中的数据
        + 在执行remove()函数前先执行find()命令来判断执行的条件是否正确，这是一个比较好的习惯
        + 示例
            * `db.col.remove({'title':'MongoDB 教程'})`
                - 若只想删除第一条找到的记录可以设置 justOne 为 1
                - `db.col.remove({'title':'MongoDB 教程'}, 1)`
            * `db.col.remove({})` 删除所有记录
        + 语法：

```sql
db.collection.remove(
   <query>,    #（可选）删除的文档的条件
   <justOne>   #（可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档
)
```


### 使用mongocxx操作

为了使用 C++ 驱动创建一个文档，需要在两个可用的生成器接口（builder interfaces）中选择一个使用：
    流生成器（Stream builder）：bsoncxx::builder::stream 一个使用流操作的文档生成器，在构建文字文档时表现较好。
    基本生成器（Basic builder）：bsoncxx::builder::basic 生成器实例中更加方便的文档生成器，包含了一些调用方法。

```cpp
{
   "name" : "MongoDB",
   "type" : "database",
   "count" : 1,
   "versions": [ "v3.2", "v3.0", "v2.6" ],
   "info" : {
               "x" : 203,
               "y" : 102
            }
}
```

使用流生成器构建接口，你可以使用下面代码构建这个文档：

```cpp
auto builder = bsoncxx::builder::stream::document{};
bsoncxx::document::value doc_value = builder
  << "name" << "MongoDB"
  << "type" << "database"
  << "count" << 1
  << "versions" << bsoncxx::builder::stream::open_array
    << "v3.2" << "v3.0" << "v2.6"
  << close_array
  << "info" << bsoncxx::builder::stream::open_document
    << "x" << 203
    << "y" << 102
  << bsoncxx::builder::stream::close_document
  << bsoncxx::builder::stream::finalize;
```

### database

是否存在collection()
bool mongocxx::database::has_collection (   bsoncxx::string::view_or_value  name    )   const
(http://mongocxx.org/api/current/classmongocxx_1_1database.html#a3ba82f39975496491ebce87ce09029ed)

    Returns
    bool whether the collection exists in this database.
    Exceptions
    mongocxx::operation_exception   if the underlying 'listCollections' command fails.
    使用：xddb.has_colloection(xdcollection)



### collection

**计数问题** 数量

count() 这个方法已经被弃用，取而代之的是count_documents()和estimated_document_count()。
    对应终端js shell: db.collection.count(query, options)

count_documents() 数据量很大时很慢
    对应终端js shell：db.collection.countDocuments()

estimated_document_count 使用该接口快速返回近似值
    Returns
    The count of the documents that matched the filter.
    Exceptions
    mongocxx::query_exception   if the count operation fails.
    使用：xdcollection.estimated_document_count()
    对应终端js shell：db.collection.estimatedDocumentCount()
    New in version 4.0.3.



## 内存问题

* [MongoDB内存限制](https://blog.csdn.net/qq_32523587/article/details/82219170)
    - 从3.4版本开始，默认情况下，WiredTiger 内部缓存将使用下面2种中更大的一种：
        + `50% of (RAM - 1 GB)` 和`256 MB`。
    - 通过文件系统缓存，MongoDB的自动使用未被wiredtiger缓存或由其他进程使用所有可用内存。
    - 调整WiredTiger内部缓存的方法：`storage.wiredTiger.engineConfig.cacheSizeGB` 和 `--wiredTigerCacheSizeGB`
        + 容器部署时配置cacheSizeGB选项，YAML格式的配置文件，绑定配置文件查看章节：`指定配置文件`
    - [Memory Use](https://docs.mongodb.com/v3.4/core/wiredtiger/index.html#memory-use)

* 优化(未试过)
    - [MongoDB 生产环境笔记](https://www.cnblogs.com/operationhome/p/10734047.html#%E4%BA%8C%E3%80%81%E6%B7%BB%E5%8A%A0-swap-%E5%88%86%E5%8C%BA)
    - `swappiness`
        + [Set vm.swappiness to 1 or 0](https://docs.mongodb.com/manual/administration/production-notes/index.html#set-vm-swappiness-to-1-or-0)
        + 可以通过查看`/proc/sys/vm/swappiness`内容的值来确定系统对 SWAP 分区的使用原则
        + 当swappiness 内容的值为 0 时，表示最大限度地使用物理内存，物理内存使用完毕后，才会使用 SWAP 分区。当swappiness 内容的值为 100 时，表示积极地使用 SWAP 分区，并且把内存中的数据及时地置换到 SWAP 分区。
        + 如果我们运行的主机系统 RHEL / CentOS 的内核版本在 2.6.32-303 及以上，我们可以把该值设置为 `1`(设置成1防止OOM，0则内存用满后可能OOM)
        + 临时修改
            * `sysctl -w vm.swappiness=1`
        + 永久修改
            * `/etc/sysctl.conf`配置文件中，新增或修改`vm.swappiness = 1`
            * `sysctl -p` 生效
        + 查看`sysctl -a |grep vm.swappiness`
    - 建议使用 `XFS` 文件系统(单独对磁盘分区)
        + 使用`WiredTiger`存储引擎时，推荐使用`XFS`系统
        + > When running MongoDB in production on Linux, you should use Linux kernel version 2.6.36 or later, with either the XFS or EXT4 filesystem. If possible, use XFS as it generally performs better with MongoDB
        + > With the WiredTiger storage engine, using XFS is strongly recommended for data bearing nodes to avoid performance issues that may occur when using EXT4 with WiredTiger
        + 参考官网：[Kernel and File Systems](https://docs.mongodb.com/manual/administration/production-notes/#kernel-and-file-systems)
    - 禁用 Transparent Huge Pages (THP)
        + THP 不适用于在数据库上，因为数据库是不连续的内存访问模式，我们需要禁用THP以确保使用MongoDB获得最佳性能。
        + 查看 Transparent Huge Pages 状态
            * `cat /sys/kernel/mm/transparent_hugepage/enabled`
            * `cat /sys/kernel/mm/transparent_hugepage/defrag`
            * 如果输出结果为`[always]`表示 THP 启用了。`[never]`表示 THP 禁用、`[madvise]`表示（只在MADV_HUGEPAGE标志的VMA中使用THP
        + 配置关闭，需要创建systemd service文件(使用`systemd (systemctl)`的系统，如CentOS7)
            * 创建文件：`/etc/systemd/system/disable-transparent-huge-pages.service`，内容参考官网
                - 部分系统是使用`/sys/kernel/mm/redhat_transparent_hugepage/enabled`路径，CentOS7不是
            * 重新加载systemd unit文件，`sudo systemctl daemon-reload`
            * 启动服务：`sudo systemctl start disable-transparent-huge-pages`
            * `cat /sys/kernel/mm/transparent_hugepage/enabled` 查看状态是否已经改变
            * 配置开机启动该服务：`sudo systemctl enable disable-transparent-huge-pages`
            * 自定义`tuned` 或 `ktune`配置文件
                - CentOS7里用的是`tuned`(systemctl list-unit-files查看服务启动情况可看到)
                    + `tuned` 是一项守护程序，它会使用`udev`来监控联网装置，并且根据选择的配置文件对系统设置进行静态和动态的微调。(静态微调主要包括预定义的 sysctl 和 sysfs 设置和对几种配置工具的单次激活，例如ethtool；动态微调使得在任何给定系统的运行时间内，不同的系统组件能够以不同的方式被使用)
                    + [2.5. TUNED](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/power_management_guide/tuned)
                - CentOS7下的操作
                    + 创建新的`tuned`配置文件目录：`sudo mkdir /etc/tuned/virtual-guest-no-thp`
                    + 编辑配置文件：`/etc/tuned/virtual-guest-no-thp/tuned.conf`，内容查看下面的官网链接
                    + 使能配置：`sudo tuned-adm profile virtual-guest-no-thp`
                    + [Red Hat/CentOS 7 and 8](https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/index.html#red-hat-centos-7-and-8)
            * 可参考官网：[Disable Transparent Huge Pages (THP)](https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/index.html#disable-transparent-huge-pages-thp)
        + ulimit 设置
            * 官网推荐设置：[Recommended ulimit Settings](https://docs.mongodb.com/manual/reference/ulimit/#recommended-ulimit-settings)
            * 对比了一下自己的机器，还需设置(可在启动脚本ulimit设置对应选项)：
                - `-v (virtual memory): unlimited [1]`
                - `-n (open files): 64000`
                - `-u (processes/threads): 64000`

* MongoDB 存储引擎
    - 存储引擎（Storage Engine）是MongoDB的核心组件，负责管理数据如何存储在硬盘（Disk）和内存（Memory）上。
        + 从MongoDB 3.2 版本开始，MongoDB 支持多数据存储引擎（Storage Engine）
        + MongoDB支持的存储引擎有：`WiredTiger`，`MMAPv1`和`In-Memory`
    - 从MongoDB 3.2 版本开始，`WiredTiger`成为MongDB默认的Storage Engine
        + 用于将数据持久化存储到硬盘文件中，WiredTiger提供*文档级别（Document-Level）的并发控制*，*检查点（CheckPoint）*，*数据压缩和本地数据加密（ Native Encryption）*等功能
        + 文档级别的并发控制（Document-Level Concurrency Control）
            * MongoDB在执行写操作时，WiredTiger 在文档级别进行并发控制
                - 就是说，在同一时间，多个写操作能够修改同一个集合中的不同文档；
                - 当多个写操作修改同一个文档时，必须以序列化方式执行；
                - 这意味着，如果该文档正在被修改，其他写操作必须等待，直到在该文档上的写操作完成之后，其他写操作相互竞争，获胜的写操作在该文档上执行修改操作
            * 对于大多数读写操作，WiredTiger使用乐观并发控制（optimistic concurrency control），如果WiredTiger检测到两个操作发生冲突时，导致MongoDB将其中一个操作重新执行，这个过程是系统自动完成的。
        + 检查点（Checkpoint）
            * 在Checkpoint操作开始时，WiredTiger提供指定时间点（point-in-time）的数据库快照（Snapshot），该Snapshot呈现的是内存中数据的一致性视图。
            * Checkpoint担当的是还原点（Recovery Point），Checkpoint操作能够缩短MongoDB从Journal日志文件还原数据的时间
            * 在默认情况下，WiredTiger创建Checkpoint的时间间隔是60s，或产生2GB的Journal文件。在WiredTiger创建新的Checkpoint期间，上一个Checkpoint仍然是有效的，这意味着，即使MongoDB在创建新的Checkpoint期间遭遇到错误而异常终止运行，只要重启，MongoDB就能从上一个有效的Checkpoint开始还原数据
    - `In-Memory`，MongoDB不仅能将数据持久化存储到硬盘文件中，而且还能将数据只保存到内存中；
        + `In-Memory`存储引擎用于将数据只存储在内存中，只将少量的元数据和诊断日志（Diagnostic）存储到硬盘文件中，由于不需要Disk的IO操作，就能获取索取的数据，In-Memory存储引擎大幅度降低了数据查询的延迟（Latency）
    - [MongoDB 存储引擎：WiredTiger和In-Memory](https://www.cnblogs.com/ljhdo/p/4947357.html)

## 服务状态

* `db.serverStatus()` 返回服务状态的概览
    - [db.serverStatus()](https://docs.mongodb.com/manual/reference/method/db.serverStatus/index.html#db-serverstatus)

## 聚合

* `db.getCollection("PlateAndStock").aggregate([{$group:{_id:"$PlateCode", num:{$sum:1}}}, {$sort:{_id:1}}])`
    - 相当于`select PlateCode,count(1) from PlateAndStock group by PlateCode order by PlateCode`
    - 若还要进行过滤，则用`$match`，e.g. `aggregate([{$group:{}}, {$sort:{}}, {$match:{num:{$gt:10}}}])`
* [Aggregation](https://docs.mongodb.com/manual/aggregation/)
    - MongoDB提供三种方式执行聚合
        + 聚合管道(aggregation pipeline)
        + map-reduce函数
        + 单一目的聚集方法(single purpose aggregation methods)，提供下面几个方法
            * `db.collection.distinct("PlateCode")`
            * `db.collection.count()`
            * `db.collection.estimatedDocumentCount()`
    - 下例中演示的是聚合管道方式，$match过滤status字段为"A"的记录，

```sql
db.orders.aggregate([
   { $match: { status: "A" } },
   { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

