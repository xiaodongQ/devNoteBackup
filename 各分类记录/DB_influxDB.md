## influxDB

* [官网](https://www.influxdata.com/)
* influxDB是一个开源的时序数据库
    - influxDB1.x 开源时序数据库，TICK(Telegraf, InfluxDB, Chronograf, Kapacitor)栈的一部分
        + [Telegraf+Influxdb+Chronograf+Kapacitor主机性能监控告警](https://yq.aliyun.com/articles/714084)
            * `Telegraf`
                - 职能是数据采集，用于主机性能数据，包括主机CPU、内存、IO、进程状态、服务状态等
                - telegraf.exe
                    + `telegraf.exe -sample-config -input-filter cpu:mem -output-filter influxdb > telegraf_2.conf`
                    + `telegraf --config telegraf_2.conf`
                - 配置文件里需要添加插件配置(要不展示不出来信息，各类信息展示可参考界面的链接)：
                    + 添加配置项：`[[inputs.influxdb]]`
                    + [Configuration](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/influxdb#configuration)
                    + 各类input插件：[plugins/inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs)
                        * (consul、docker等组件、cpu、net等系统资源等配置都有说明)
                        * 一般以 `[[inputs.mem]]` 形式就启用了插件
                - Go语言实现，插件驱动，包含四种不同的插件
                    + Input Plugins 从系统、服务或者第三方API收集指标
                    + Processor Plugins 转换、装饰和/或筛选指标
                    + Aggregator Plugins 创建聚合指标(例如平均值`mean`、最小值`min`、最大值`max`、分位数`quantiles`等)
                    + Output Plugins 向各种目的地写入指标(可多个)
                    + 各类插件支持和配置，参考：[telegraf plugins](https://github.com/influxdata/telegraf/tree/master/plugins)
            * `Influxdb`
                - 职能是时序数据库，用于存储Telegraf采集来的数据
                - influxd.exe
                - 8086
                - 客户端连接
                    + 输入 `influx`，默认连接 http://localhost:8086
                    + `show databases` 测试查询是否ok
            * `Chronograf`
                - 职能是数据可视化，用于将Influxdb数据库的性能数据时序展示
                - chronograf.exe
                - 8888端口
            * `Kapacitor`
                - 职能是规则告警，用于配置告警规则将Influxdb数据库查询触发规则的数据进行告警
                - kapacitord.exe
                - 9092端口
            * 其中，时序数据库(此处`Influxdb`)可使用刚开源的`TDEngine`，可视化(此处`Chronograf`)可以使用`Grafana`替代使用
    - 被设计用于处理高写入和查询负载，并且提供一个类SQL语言`InfluxQL`用于和数据交互
* 管理工具
    - [InfluxDB 管理工具](https://learnku.com/articles/37151)
    - 自带 web 管理
        + 官方自从 1.3 版本开始把 web 界面取消了
    - InfluxDB Studio
        + 第三方开源监控软件
        + 下载：[InfluxDBStudio releases](https://github.com/CymaticLabs/InfluxDBStudio/releases)
            * v0.2.0 Beta 1
        + 将下载的资源  解压，找到InfluxDBStudio.exe，打开即可
    - Chronograf (推荐)
        + 支持 influxdb 的基础监控、管理以及数据展示、警报管理及数据库管理
        + influxDB官网下载链接(包含TICK组件):[Chronograf Download](https://portal.influxdata.com/downloads/)
            * 可选择最新版本，若直接下载1.8.5，链接为(Windows)：`https://dl.influxdata.com/chronograf/releases/chronograf-1.8.5_windows_amd64.zip`
        + chronograf.exe执行启动后，到浏览器中连接：`http://localhost:8888/`
            * 左边栏的`Explore`页可以查看数据库数据和统计情况
                - 创建查询，选择对应操作即可
                - 可以可视化展示

* 尝试使用可参考：[Telegraf+Influxdb+Chronograf+Kapacitor主机性能监控告警](https://yq.aliyun.com/articles/714084)
    - 注意配置Telegraf的input插件，收集信息

* [InfluxDB中文文档](https://jasper-zhang1.gitbooks.io/influxdb/content/Introduction/installation.html)
* InfluxDB默认使用下面的网络端口
    - TCP端口`8086`用作InfluxDB的客户端和服务端的http api通信
    - TCP端口`8088`给备份和恢复数据的RPC服务使用
* `NTP`
    - InfluxDB使用服务器本地时间给数据加时间戳，而且是UTC时区的。
    - 并使用NTP来同步服务器之间的时间，如果服务器的时钟没有通过NTP同步，那么写入InfluxDB的数据的时间戳可能不准确
* `influx`
    - 这个工具包含在InfluxDB的安装包里，是一个操作数据库的轻量级工具
    - 它直接通过InfluxDB的HTTP接口(如果没有修改，默认是8086)来和InfluxDB通信
    - 也可以直接发送裸的HTTP请求来操作数据库，例如curl
    - `influx -precision rfc3339` 登录数据库
        + `-precision`参数表明了任何返回的时间戳的格式和精度
        + `rfc3339`是让InfluxDB返回RFC339格式(`YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ`)的时间戳
        + 这样这个命令行已经准备好接收influx的查询语句了(简称`InfluxQL`)，用`exit`可以退出命令行
* 创建数据库
    - 第一次安装好InfluxDB之后是没有数据库的(除了系统自带的`_internal`)，因此创建一个数据库是我们首先要做的事
        + `_internal`数据库是用来存储InfluxDB内部的实时监控数据的
    - `CREATE DATABASE mydb` (关键字大小写无关，数据库名大小写敏感)
        + 数据库的名字可以是被双引号引起来的任意Unicode字符
        + 如果名称只包含ASCII字母，数字或下划线，并且不以数字开头，那么也可以不用引起来
        + 在输入上面的语句之后，并没有看到任何信息，这在CLI里，表示语句被执行并且没有错误，如果有错误信息展示，那一定是哪里出问题了，这就是所谓的没有消息就是好消息
        + `SHOW DATABASES` 查看已存在的数据库
* 读写数据
    - 大部分InfluxQL需要作用在一个特定的数据库上，可以在每一个查询语句上带上你想查的数据库的名字
    - CLI提供了一个更为方便的方式`USE <db-name>`，e.g. `USE mydb`
        + 以下的操作都作用于mydb这个数据库之上
    - 数据存储的格式
        + InfluxDB里存储的数据被称为`时间序列数据`，其包含一个数值，就像CPU的load值或是温度值类似
        + 时序数据有零个或多个数据点，每一个都是一个指标值
        + 数据点包括
            * `time`(一个时间戳)，
            * `measurement`(例如cpu_load)，
            * 至少一个`k-v`格式的`field`(也即指标的数值，例如 “value=0.64”或者“temperature=21.2”)，
            * 零个或多个`tag`，其一般是对于这个指标值的元数据(如“host=server01”，“region=EMEA”)
        + 可以将measurement类比于SQL里面的table，其*主键索引总是时间戳*
            * 不同之处在于，在InfluxDB里，你可以有几百万的measurements，不用事先定义数据的scheme，并且null值不会被存储
    - 将数据点写入InfluxDB，只需要遵守如下的行协议：
        + `<measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]`
        + e.g. `cpu,host=serverA,region=us_west value=0.64`
            * cpu是`measurement`，host和region是`tag`，value是`field`
        + e.g. `temperature,machine=unit42,type=assembly external=25,internal=37 1434067467000000000`
            * temperature是`measurement`，machine和type是`tag`，external和internal是`field`，1434067467000000000是纳秒时间戳
    - `insert`写入格式
        + `INSERT cpu,host=serverA,region=us_west value=0.64` insert后面接数据点
        + 在写入的时候没有包含时间戳，当没有带时间戳的时候，InfluxDB会自动添加本地的当前时间作为它的时间戳
        + `insert xdm,xdt1=12,xdt2="hh" xdf1="fff",xdf2=87`
            * 注意`xdm,xdt1`之间不能有空格，测试得貌似所有`,`前后都不能有空格
            * `xdt1=12` 此处12会当做字符串，`tag`的key和value都要求作为字符串存储
    - `select`查询格式
        + `SELECT "host", "region", "value" FROM "cpu"`
        + 查询的时候想要返回所有的字段和tag，可以用`*`，e.g. `SELECT * FROM "temperature"`
        + InfluxQL还有很多特性和用法没有被提及，包括支持golang样式的正则
        + `SELECT * FROM /.*/ LIMIT 1`
* 概念
    - `time` 在InfluxDB中所有的数据都有这一列
    - `fields`
        + fields由`field key`和`field value`组成
            * `field key`(butterflies和honeybees)都是字符串，它们存储元数据
            * `field value`是对应数据，可以是字符串、浮点数、整数、布尔值，因为InfluxDB是时间序列数据库，所以field value总是和时间戳相关联
        + 每组field key和field value的集合组成了`field set`
        + `field`是没有索引的
    - `tag`
        + tag由`tag key`和`tag value`组成
            * **`tag key`和`tag value`都作为字符串**存储，并记录在元数据中
        + tag不是必需的字段，但是在你的数据中使用tag总是大有裨益，因为不同于field，*`tag`是索引起来的*
            * 这意味着对tag的查询更快，`tag`是*存储常用元数据的最佳选择*
        + `tag set`是不同的每组`tag key`和`tag value`的集合(m*n个成员)
    - `measurement`
        + 作为`tag`，`fields`和`time`列的容器，measurement的名字是存储在相关fields数据的描述
        + `measurement`在概念上类似于表
        + 单个measurement可以有不同的`retention policy`
    - `retention policy` (保存策略)
        + 描述了InfluxDB保存数据的时间（DURATION）以及这些存储在集群中数据的副本数量（REPLICATION）
        + 未指定则InfluxDB自动创建存储策略`autogen`; 它具有无限的持续时间和复制因子设置为1
        + `duration`
            * `retention policy`中的一个属性，决定InfluxDB中数据保留多长时间。
            * 在`duration`之前的数据会自动从database中删除掉
    - `series`
        + 在InfluxDB中，`series`是共同`retention policy`，`measurement`和`tag set`的集合
        + 理解`series`对于设计数据`schema`以及对于处理InfluxDB里面的数据都是很有必要的
    - `point`
        + 就是具有相同`timestamp`的相同`series`的`field`集合
* 和SQL比较
    - InfluxDB是一个时间序列数据库，SQL数据库可以提供时序的功能，但严格说时序不是其目的。
        + 简而言之，InfluxDB用于存储*大量*的时间序列数据，并对这些数据进行*快速的实时分析*
    - InfluxDB的`measurement`和SQL数据库里的`table`类似
    - InfluxDB的`tag`类似于SQL数据库里索引的列
    - InfluxDB中的`field`类似于SQL数据库里没有索引的列
    - InfluxDB里面的数据点(`point`)类似于SQL数据库的行
    - InfluxDB的`continuous query`和`retention policy`与SQL数据库中的存储过程类似。
        + 它们被指定一次，然后定期自动执行
    - SQL数据库和InfluxDB之间存在一些重大差异。
        + SQL中的`JOIN`不适用于InfluxDB中的`measurement`
    - 在InfluxDB中`InfluxQL`是一种类SQL的语言。
        + 对于来自其他SQL或类SQL环境的用户来说，它已经被精心设计，而且还提供特定于存储和分析时间序列数据的功能
        + `SELECT * FROM "foodships" WHERE "planet" = 'Saturn'`，planet为Saturn的数据
        + `SELECT * FROM "foodships" WHERE "planet" = 'Saturn' AND time > '2015-04-16 12:00:01'`
            * InfluxQL允许在`WHERE`子句中指定查询的时间范围
            * 可以使用包含单引号的日期时间字符串，格式为`YYYY-MM-DD HH：MM：SS.mmm`(mmm为毫秒，为可选项，也可指定微秒或纳秒)
        + 还可以使用相对时间与`now()`来指代服务器的当前时间戳
            * `SELECT * FROM "foodships" WHERE time > now() - 1h`
                - 查询时间戳比服务器当前时间减1小时大的数据(即最近1小时)
                - 时间范围的可选单位有：`u`/`ms`/`s`/`m`/`h`/`d`/`w`
    - InfluxDB是针对时间序列数据进行了优化的数据库。
        + 这些数据通常来自分布式传感器组，来自大型网站的点击数据或金融交易列表等
        + 随着时间的推移开始显现的趋势，是我们从这些数据里真正想要看到的
        + 另外，时间序列数据通常是一次写入，很少更新
        + 结果是，由于优先考虑`create`和`read`数据的性能而不是`update`和`delete`，InfluxDB不是一个完整的`CRUD`数据库，更像是一个`CR-ud`
* InfluxDB的设计见解和权衡
    - InfluxDB是一个时间序列数据库。针对这种用例进行优化需要进行一些权衡，主要是以牺牲功能为代价来提高性能
        + 对于时间序列用例，我们假设如果相同的数据被多次发送，那么认为客户端几次都是同一笔数据
            * 优势：通过简化的冲突解决增加了写入性能
            * 劣势：不能存储重复数据;可能会在极少数情况下覆盖数据
        + 限制删除操作，从而增加查询和写入性能
        + 限制更新操作，从而增加查询和写入性能
        + 绝大多数写入都是接近当前时间戳的数据，并且数据是按时间递增的顺序添加
            * 优势：按时间递增的顺序添加数据明显更高效些
            * 劣势：随机时间或时间不按升序写入点的性能要低得多
        + 数据库可以处理大量的读取和写入
        + 多个客户端可以在高负载的情况下完成查询和写入数据库操作
            * 如果数据库负载较重，查询返回结果可能不包括最近的点(强调可用性，不要求强一致性)
        + InfluxDB善于管理不连续数据
            * 例如一个新的主机，开机并监控数据被写入一段时间，然后被关闭
        + InfluxDB具有非常强大的工具来处理聚合数据和大数据集
* schema设计建议
    - 哪些情况下使用tag
        + 把你经常查询的字段作为`tag`
        + 如果你要对其使用`GROUP BY()`，也要放在`tag`中
        + 如果你要对其使用`InfluxQL`函数，则将其放到`field`中
        + 如果你需要存储的值不是字符串，则需要放到`field`中，因为`tag value`只能是字符串
    - 避免InfluxQL中关键字作为标识符名称
        + 如果查询中包含除[A-z，_]以外的字符，则还需要将它们用双引号括起来
    - 不要有太多的`series`(`series`是共同`retention policy`，`measurement`和`tag set`的集合)
        + `tags`包含高度可变的信息，如UUID，哈希值和随机字符串，这将导致数据库中的大量`measurement`
            * 通俗地说是高`series cardinality` 高系列基数
            * `series cardinality`高是许多数据库高内存使用的主要原因
        + 如果系统有内存限制，请考虑将高cardinality数据存储为`field`而不是`tag`
    - 不要把多条信息放到一个`tag`里面
        + 将具有多条信息的单个tag拆分为多个单独的tag将简化查询并减少对正则表达式的需求