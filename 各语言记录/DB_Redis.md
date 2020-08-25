## Redis

* 场景
    - MongoDB存储数据，每日数据量5.5GB左右保存在一个日期命名的集合(collection)，按时间条件查询指定key的记录，查询很慢
        + a. 单次测试(还未考虑并发查询的情况)：key文档(document)数4707，查询耗时24244ms，总集合文档数 17302503
        + b. 5个并发，条件对应各自不同key，情况如下：
            * 查1天，count:3552, cost:1m28.2531345s
            * 查1天，count:2963, cost:1m28.8245294s
            * 查1天，count:4374, cost:2m29.8772464s
            * 查1天，count:4375, cost:3m21.6757686s
            * 查3天(依次查询)，count:18409, cost:4m8.1853038s
        + MongoDB部署机器的磁盘性能也是一个问题，磁盘快满了，，内存(共8G)也快用完了，主要还是这个服务器配置太低了，MongoDB的缓存内存没进行限制
            * 从3.4版本开始，默认情况下，WiredTiger 内部缓存将使用下面2种中更大的一种：`50% of (RAM - 1 GB)` 和`256 MB`。
    - 现需要查询跨天的记录，直接查数据库响应时间太慢，因此查询资料找缓存方案
        + [Memcache,Redis,MongoDB（数据缓存系统）方案对比与分析](https://blog.csdn.net/suifeng3051/article/details/23739295)
        + [memcache和redis、Mongodb优缺点及应用场景](https://blog.csdn.net/weiyi_xingdong/article/details/79992032?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)
        + [redis、memcache、mongoDB 对比](https://blog.csdn.net/yu487/article/details/85714822?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)
    - 使用Redis作为缓存
        + 配置文件redis.conf进行缓存大小和策略配置(使用LRU，Least Recently Used最近最少使用)
            * `maxmemory 1.5gb` (1.5GB作缓存，单位可参考下面的比较说明，搜索`内存单位可选如下`)
            * `maxmemory-policy allkeys-lru`
    - 内存瓶颈

* [官网](https://redis.io/)
* [中文官网](http://www.redis.cn/)
    - Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件
    - 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询
    -  Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）
* 安装
    - docker方式
        + 安装:`docker pull redis`
        + 使用
            * 官网简单示例：`docker run --name some-redis -d redis` (示例中默认没有指定端口绑定等参数)
        + 启动时指定配置文件(包含docker启动参数)
            * [docker启动redis、并加载配置文件](https://blog.csdn.net/qq_37936542/article/details/94550330)
            * 新建配置文件路径，示例目录：`/home/data/redis/conf`, `/home/data/redis/data`
            * 从官网下载redis-xxx.tar.gz解压得到配置文件，redis.conf
            * 根据需要修改配置文件内容
                - 开启redis验证：`requirepass 123456`
                    + 开启验证后，执行命令需要`auto 123456`才能继续，不鉴权否则会提示授权
                    + 若没有设置密码，且放开了bind允许外部连接，且redis为保护模式(protected-mode yes)时，则外部连接时会连接不上并提示相关错误信息(会提示Redis在保护模式中，并提供解决方式)
                        * 1. 关闭protected-mode模式(通过命令行设置，或通过配置文件修改protected-mode no重启生效)
                        * 2. 设置密码(requirepass)或设置bind地址(安全策略允许的话可`bind 0.0.0.0`)
                - 注释`bind 127.0.0.1`允许外部连接：`#bind 127.0.0.1`
                - 日志文件：`logfile "access.log"`，日志默认打印在(容器中的)标准输出，如果设置了日志文件，则`docker logs`就不显示日志了
            * `docker run -d --restart=always -p 6379:6379 -v /home/data/redis/conf/redis.conf:/etc/redis/redis.conf -v /home/data/redis/data:/data --name myredis redis redis-server /etc/redis/redis.conf`
                - `-d` daemon后台运行容器
                - `--restart=always` 每次docker服务重启后容器也自动重启
                - `-p` 端口映射 宿主机:容器
                - `-v` 映射配置文件和数据路径
                - `--name myredis` 设置一个容器名称
                - `redis` pull的容器
                - `redis-server /etc/redis/redis.conf` 启动命令
        + 客户端访问
            * 直接访问
                - `docker exec -it myredis redis-cli`
                - 或 `docker run -it --link myredis:redis --rm redis redis-cli -h redis -p 6379`
                    + `--rm` 指定退出容器时自动清理容器和文件系统(默认情况是不清理的)
                    + `--link`(Docker网络相关参数) 连接到另一个容器
            * 进入容器中的shell `docker exec -it myredis bash`，`redis-cli`
* `redis-cli`
    - [redis-cli, the Redis command line interface](http://www.redis.cn/topics/rediscli.html)
    - `redis-cli`是Redis的命令行接口，允许发送命令给Redis，直接从终端读取服务端的应答
        + `redis-cli --stat` 查看状态
        + 发布订阅模式
            * `redis-cli psubscribe '*'` 订阅所有信息
            * 执行`redis-cli`进入交互终端，执行 `publish 111test_channel 111test_message`(第一个参数指定通道，第二个指定消息)，执行后上面的订阅就能看到发布的信息
        + `redis-cli monitor` 也能看到执行的命令消息(终端执行的所有命令)，类似发布订阅模式
        + `redis-cli --latency` 测试时延毫秒(循环发送ping给Redis实例 100次/s，统计应答real time)
            * `redis-cli --latency-dist` 可彩色显示
* Redis配置
    - [Redis配置](http://www.redis.cn/topics/config.html)
    - Redis可以在没有配置文件的情况下通过内置的配置来启动，但是这种启动方式只适用于开发和测试。`合理的`配置Redis的方式是提供一个Redis配置文件，这个文件通常叫做 `redis.conf`。
        + 配置文件中的格式为`keyword argument1 argument2 ... argumentN`，如`requirepass "hello world"`
            * 若参数中有空格，则用`"`括起来
        + 配置Redis成为一个缓存
            * 如果你想把Redis当做一个缓存来用，所有的key都有过期时间，那么你可以考虑 使用以下设置（假设最大内存使用量为2M）：
                - `maxmemory 2mb`
                - `maxmemory-policy allkeys-lru`
                - redis.conf配置文件中有这两个参数的说明，内存单位可选如下
                    + 1k => 1000 bytes
                    + 1kb => 1024 bytes
                    + 1m => 1000000 bytes
                    + 1mb => `1024*1024` bytes
                    + 1g => 1000000000 bytes
                    + 1gb => `1024*1024*1024` bytes
            * 以上设置并不需要我们的应用使用`EXPIRE`(或相似的命令)命令去设置每个key的过期时间，因为 只要内存使用量到达2M，Redis就会使用类LRU算法自动删除某些key
            * 相比使用额外内存空间存储多个键的过期时间，使用缓存设置是一种更加有效利用内存的方式
                - 而且相比每个键固定的 过期时间，*使用LRU也是一种更加推荐的方式*，因为这样能使应用的热数据(更频繁使用的键) 在内存中停留时间更久
                - 基本上这么配置下的Redis可以当成*memcached*使用
            * 当我们把Redis当成缓存来使用的时候，如果应用程序同时也需要把Redis当成存储系统来使用，那么强烈建议**使用两个Redis实例**
                - 一个是缓存，使用上述方法进行配置，
                - 另一个是存储，根据应用的持久化需求进行配置，并且 只存储那些不需要被缓存的数据
* 持久化方式
    - [Redis persistence demystified](http://oldblog.antirez.com/post/redis-persistence-demystified.html)
    - [Redis 持久化之RDB和AOF](https://www.cnblogs.com/itdragon/p/7906481.html)
    - `RDB`(Redis DataBase)方式(默认，生成一个快照shotsnap)
        + Redis在指定的时间间隔内，若执行了指定次数的写操作(多少个key被修改了)，则会将内存中的数据写入到磁盘中，即在指定目录下生成一个dump.rdb文件。
            * Redis 重启会通过加载dump.rdb文件恢复数据。
        + redis.conf配置文件中有相关说明(查询*SNAPSHOTTING*，配置文件中的说明很详细)
            * 格式如：`save <seconds> <changes>`
                - e.g. `save 900 1`
            * 默认配置为900秒内有1个更改，300秒内有10个更改以及60秒内有10000个更改，则将内存中的数据快照写入磁盘
            * 几个相关配置(保持默认即可)
                - `dbfilename dump.rdb` 指定shotsnap文件名称
                - `dir ./` 指定本地数据库存放目录
                - `rdbcompression yes` 默认开启数据压缩
        + 发生异常时，最新快照之后的最近数据会丢失，所以该方式只适用于丢失最近数据并不是十分重要的场景
        + 优点：
            * 适合大规模的数据恢复。
            * 如果业务对数据完整性和一致性要求不高，RDB是很好的选择
        + 缺点：
            * 数据的完整性和一致性不高，因为RDB可能在最后一次备份时宕机了。
            * 备份时占用内存，因为Redis 在备份时会独立创建一个子进程，将数据写入到一个临时文件（此时内存中的数据是原来的两倍哦），最后再将临时文件替换之前的备份文件。
    - `AOF`(Append Only File)方式
        + 为了弥补RDB的不足（数据的不一致性），采用日志的形式来记录每个写操作，并追加到文件中。Redis 重启的会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作
        + 配置文件相关配置，查找*APPEND ONLY MODE*
            * `appendonly yes` 默认是no，关闭的
            * `appendfilename "appendonly.aof"` 指定本地数据库文件名
            * 更新日志条件
                - appendfsync always 同步持久化，每次发生数据变化会立刻写入到磁盘中。性能较差当数据完整性比较好（慢，安全）
                - appendfsync everysec 出厂默认推荐，每秒异步记录一次（默认值）
                - appendfsync no 不同步
            * 配置重写触发机制(自动重写append only file)
                - `auto-aof-rewrite-percentage 100`，设置为0则会关闭自动重写
                - `auto-aof-rewrite-min-size 64mb`，配置一个最小值，防止AOF文件大小变化达到100%，但文件内容依然很小的情况
        + 优点：数据的完整性和一致性更高
        + 缺点：因为AOF记录的内容多，文件会越来越大，数据恢复也会越来越慢
* 教程
    - 完整的各种命令可以参考官网：[commands](https://redis.io/commands#hash)
        + `keys *` 查找所有满足模式的key，此处为`*`所有key
    - [Redis interactive tutorial](http://try.redis.io/)
    - `SET server:name "fido"`，存储值"fido"，key为"server:name"
        + 若key存在，则为修改，可以修改value的类型
    - `GET server:name`，获取到"fido"
        + 不存在的key则会返回(nil)
    - `EXISTS server:name`，测试key:"server:name"是否存在，存在返回1，不存在返回0
    - `INCR connections`，数值类型`原子加1`(若不用incr而读取值并+1再设置的话，存在并发安全问题)
        + 只能操作数值类型，否则报错：(error) ERR value is not an integer or out of range
        + 若key不存在，则会新增一个key，且值为1(即相对原始值0操作)
    - `DEL connections` 删除key
    - `INCRBY connections 100` 加上一个数值
        + 若key不存在，则新增一个key，且值为100
    - `DECR connections` 和INCR对应，减一
        + 若key不存在，则新增一个key，且值为-1
    - `DECRBY connections 10` 减一个数值
        + 若key不存，则新增一个key
    - 生命周期(EXPIRE/TTL/PEXPIRE/PTTL，pexpire和pttl只是返回单位为毫秒)
        + `EXPIRE resource:lock 120` 设置生命周期为120秒(注意单位为秒)
            * 操作不存在的key，会返回0
        + `TTL resource:lock` 查看key(此处为resource:lock)的生命周期(秒数)
            * 普通key-value返回为-1，表示永远不会过期 (e.g. `set xd 10`, `ttl xd`)
            * 通过`expire xd 20`设置生命周期为20秒，每次ttl时都会减少秒数，直到key被删除，不存在的key ttl返回-2
            * 若使用`set`设置一个已存在的key，则ttl会被重置，变为永远不过期(-1)
        + `PERSIST resource:lock` 取消ttl设置，并会设置key为永久(-1)
            * `persist`设置成功后返回1，若key不存在或已经是永久，则返回为0
        + 可以在新增key的同时设置ttl
            * `set key1 "valuexxx" ex 10`，表示新增或修改key1的值为"valuexxx"，且生命周期为10秒
    - 结构体
        + 对所有Redis数据结构，不需要先创建key再设置值，而可以直接添加新元素(副作用是key不存在则会创建)
        + *list(list表示一系列有序值)*
            * RPUSH, LPUSH, LLEN, LRANGE, LPOP, and RPOP
            * `RPUSH friends "Alice"`，将新元素添加到list结尾
                - 操作后会返回list的长度
                - `RPUSH friends 1 2 3` 支持同时添加多个
                - 若key不存在则会新建list
                - 遍历不能用`get friends`，需要用lrange遍历
            * `LPUSH friends "Sam"`，将新元素添加到list开头
            * `LRANGE friends 0 -1`，给出list的子集
                - 第一个参数表示想查找的第一个元素的索引，第二个参数表示想查找的最后元素的索引
                    + 索引从0开始，表示第一个
                    + 第二个索引为`-1`则表示检索到最后一个，`-2`表示倒数第二个，以此类推(`-3`倒数第三个)
            * `LPOP friends` 删除list第一个元素并返回这个元素
            * `RPOP friends` 删除list最后一个元素并返回这个元素
            * `LLEN friends` list大小
        + *set*
            * set和list类似，不过它没有顺序，且每个元素只出现一次
                - list访问元素接近访问头尾元素的速度，且保留了顺序
                - set检查元素是否存在速度很快，且每个元素只有一个副本
            * SADD, SREM, SISMEMBER, SMEMBERS and SUNION
            * `SADD superpowers "flight"` 添加给定元素到set
                - 可以同时添加多个，返回添加成功的个数，全部添加失败返回0个
                - 对于已存在的非set数据，sadd会报错WRONGTYPE Operation against a key holding the wrong kind of value
            * `SREM superpowers "reflexes"` 从set删除元素
                - 可同时多个，返回删除成功的个数
            * `SISMEMBER superpowers "flight"` 检查是否存在于set，返回1存在，0不存在
            * `SMEMBERS superpowers` 返回set中所有成员
            * `SUNION superpowers birdpowers` 联合两个set，并返回所有成员(去重)
            * `SPOP letters 2` 从set中删除并返回几个元素
                - `SRANDMEMBER letters 2` 从set中随机返回几个元素，并不删除，若数量为负数则可能返回重复的元素值
        + *sorted set* 有序集合
            * Redis 1.2引入，有序集合中每个元素都有一个关联的score，用于排序
            * `ZADD hackers 1940 "Alan Kay"` 添加成员
                - 向常规set中添加会报错 WRONGTYPE Operation against a key holding the wrong kind of value
                - `smembers`也不能用于查看有序集合
            * `ZRANGE hackers 2 4` 查看有序集合，两个参数是索引范围
    - *hashes*
        + 哈希是string域和string值的映射，是表示目标绝佳的类型
        + `HSET user:1000 name "John Smith"`，key为user:1000，域为name，域的值为"John Smith"
            * `HGET user:1000 name` 获取指定域
            * `HGETALL user:1000` 获取保存的数据，返回结果中域和域的值都会作为单独的成员返回
        + `HMSET user:1001 name "Mary Jones" password "hidden" email "mjones@example.com"` 同时添加多个域
            * 类似多次对同一个key数据进行`hset`添加域
        + `HINCRBY user:1000 visits 1` 对hash中的visit域进行加1(原子加)
            * hash域中的数值类型和简单字符串处理一样
        + `HDEL user:1000 visits` 删除hash中某个域

## Go使用Redis

* [package redis](https://pkg.go.dev/github.com/garyburd/redigo/redis?tab=doc)

## 使用问题

* OOM
    - CentOS 共8G，redis使用 6G，存数据时发现redis的docker重新启动了，查看系统日志，发生了OOM，自动kill了进程
    - 关于redis启动时access.log日志的警告：`WARNING overcommit_memory is set to 0`
        + 内核参数overcommit_memory
            * [有关linux下redis overcommit_memory的问题](https://blog.csdn.net/whycold/article/details/21388455)
        + 内存分配策略，可选值：0、1、2
            * 0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程
            * 1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何
            * 2， 表示内核允许分配超过所有物理内存和交换空间总和的内存
        + Overcommit
            * Linux对大部分申请内存的请求都回复"yes"，以便能跑更多更大的程序。因为申请内存后，并不会马上使用内存。这种技术叫做Overcommit
            * 当linux发现内存不足时，会发生OOM killer(OOM=out-of-memory)。它会选择杀死一些进程(用户态进程，不是内核线程)，以便释放内存
            * 当oom-killer发生时，linux会选择杀死哪些进程？选择进程的函数是`oom_badness`函数(在mm/oom_kill.c中)，该函数会计算每个进程的点数(0~1000)。点数越高，这个进程越有可能被杀死。每个进程的点数跟`oom_score_adj`有关，而且`oom_score_adj`可以被设置(-1000最低，1000最高)
        + 如要设置该值
            * `/etc/sysctl.conf` ，改`vm.overcommit_memory=1`，然后`sysctl -p` 使配置文件生效
    - [解决redis启动时的三个警告](https://www.jianshu.com/p/a86e0248af58)
        + 按日志提示操作即可
        + 关于 `WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128`
            * 将`net.core.somaxconn=1024`添加到`/etc/sysctl.conf`中，然后执行`sysctl -p`生效配置
            * 关于somaxconn参数:
                - 定义了系统中每一个端口最大的监听队列的长度,这是个全局的参数,默认值为128.限制了每个端口接收新tcp连接侦听队列的大小
                - 对于一个经常处理新连接的高负载 web服务环境来说，默认的 128 太小了。大多数环境这个值建议增加到 1024 或者更多
            * 对于容器部署，貌似需要`docker run`时添加选项 `--sysctl net.core.somaxconn=1024`，/etc/sysctl.conf中添加未生效
        + 关于`# WARNING you have Transparent Huge Pages (THP) support enabled in your kernel`
            * echo never > /sys/kernel/mm/transparent_hugepage/enabled添加到/etc/rc.local中，然后执行source /etc/rc.local生效配置

```go
// /var/log/messages
May 21 16:26:30 localhost kernel: Out of memory: Kill process 16850 (redis-server) score 530 or sacrifice child
May 21 16:26:30 localhost kernel: Killed process 16850 (redis-server) total-vm:9900792kB, anon-rss:4390988kB, file-rss:0kB, shmem-rss:0kB
```

```go
// redis/data/access.log
1:M 21 May 2020 08:26:36.233 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1868 1:M 21 May 2020 08:26:36.233 # Server initialized
1869 1:M 21 May 2020 08:26:36.233 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1870 1:M 21 May 2020 08:26:36.233 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1871 1:M 21 May 2020 08:26:46.283 * DB loaded from disk: 10.042 seconds
1872 1:M 21 May 2020 08:26:46.283 * Ready to accept connections
```

## Redis源码

* 5.0.5版本
* Redis类型介绍：[An introduction to Redis data types and abstractions](https://redis.io/topics/data-types-intro)
    - `Redis Key` (键)
        + Redis的key是二进制安全的，这意味着可以使用任何二进制序列：从简单的"foo"到JPEG文件的内容，空的string也是一个有效的key
        + 很长的key不是一个好主意，例如一个1024长度的Key不仅在内存方面不是一个好主意，而且还因为在数据集中查找键可能需要进行多次代价高昂的键比较。即使有大的键场景，哈希(如SHA1)也是一个更好的主意，特别是从内存和带宽的角度来看
        + 很短的key通常也不是一个好主意，`"u1000flw"`写成`"user:1000:followers"`更具有可读性，增加的空间相对键对象本身和值对象使用的空间相比也是很小的。当然更短的key会消耗更少的内存，你的工作是找到正确的平衡
        + 试着保持一个模式，如 `"object-type:id"`是个好主意，如`"user:1000"`，点或破折号通常用于多词字段，如`"comment:1234:reply"`或`"comment:1234:reply-to"`
        + key允许的最大大小为512MB
    - `Redis Strings`
        + Values可以是任何形式的strings(包括二进制数据)，不超过512MB
        + `set`命令有很多有趣的选项
            * `set mykey newval nx` 已存在则设置失败(即需要key不存在)
            * `set mykey newval xx` 不存在则设置失败(即需要key存在)
        + `incr`、`incrby` 是原子增加
        + 在单个命令中设置或检索多个键值的能力也有助于减少延迟，`mset` `mget`
    - `Redis Lists`
        + 基于链表实现 Redis lists are implemented via Linked Lists
            * 而不是数组
            * `lpush`添加记录时间复杂度为`O(1)`
            * 缺点是index访问很慢
            * 若想从集合中获取中间元素，可使用另外一个数据结构：`Sorted sets`
        + 使用示例
            * 想象一下，你的主页显示了在一个照片分享社交网络上发布的最新照片，你想加快访问速度
                - 每当用户发布一张新照片时，我们就使用`LPUSH`将其ID添加到一个列表中
                - 当用户访问主页时，我们使用`LRANGE 0 9`来获取最近发布的10个记录
        + 限制列表成员个数(Capped lists)
            * 使用`LTRIM`命令只记住最新的N项并丢弃所有最旧的项
                - 和`LRANGE`类似，但是它没有显示指定的元素范围，而是将这个范围设置为新的list值。
                - 所有超出给定范围的元素都被移除
            * e.g. `ltrim mylist 0 2`，只保留索引0-2的记录
            * 使用场景：允许一个非常简单但有用的模式，添加并限制最近的个数
                - `LPUSH mylist <some element>`
                - `LTRIM mylist 0 999` 只保留最新的1000个元素(不返回数据)
        + `LRANGE`时间复杂度`O(N)`，访问头、尾为`O(1)`
        + 阻塞列表，队列(Blocking operations on lists)
            * 列表的特性使其适合于实现队列
            * 简单的生产者-消费者模型：
                - `LPUSH`生产，`RPOP`消费
                - 问题：队列为空则`RPOP`返回NULL，客户端进行轮询，但是这样会强制Redis和客户端处理无用的命令；并且轮询的等待会增加处理的延迟(即在等待期间有新的生产记录时无法立即响应)
            * 针对上面的问题，Redis实现了 `BRPOP` 和 `BLPOP`
                - `RPOP`和`LPOP`的阻塞版本，当队列为空则进行阻塞，直到有新的记录或到了用户指定的超时时间
                - e.g. `brpop xdlist 5` 从xdlist里取尾部元素，若队列空则阻塞，超时时间5秒(若到超时时间还没数据则返回NULL)
                - 也可以指定超时时间为`0`，此时阻塞永远不会超时
                - 也可以等待多个队列，直到有一个队列收到数据
            * `BRPOP`注意事项
                - 客户端以有序的方式提供服务:当某个元素被其他客户端推入时，第一个阻塞等待列表的客户端首先提供服务，以此类推
                - 返回的数据和`RPOP`不一样，`BRPOP`返回一个双元素数组，包括key的名称和值(因为支持阻塞多个队列)
                - 若到了超时时间则返回NULL
            * `RPOPLPUSH`(提供可靠队列 和 循环队列)
                - 该命令还有一个阻塞变体，称为`BRPOPLPUSH`
                - 可以使用`RPOPLPUSH`构建更安全的队列或循环队列
                    + 使用`BRPOP`或者`RPOP`获得的队列不可靠，因为消息可能会丢失
                    + 如网络问题或者消费者在收到消息后崩溃，但仍需处理消息
                - `RPOPLPUSH`或者`BRPOPLPUSH`提供了一种方式避免上面的问题
                    + 消费者获取消息，同时将其推入处理列表。一旦消息被处理，它将使用`LREM`命令从处理列表中删除该消息
                    + 另一个客户端可以监视处理列表中停留太长时间的项，并在需要时将那些超时的项再次推入队列
                - 详情可见：[RPOPLPUSH source destination](https://redis.io/commands/rpoplpush)
        + 自动创建和删除key
            * 基本上，我们可以用三条规则来总结这种行为
                - 当我们向聚合数据类型添加元素时，如果目标键不存在，则在添加元素之前会自动创建一个空的聚合数据类型
                - 当我们从聚合数据类型中删除元素时，如果值仍然为空，则键被自动销毁。流数据类型(Stream)是这一规则的唯一例外。
                - 调用只读命令，比如`LLEN`(返回列表的长度)，或者使用删除元素的写命令`DEL`，操作一个空或者不存在的key时，和操作一个空的聚合类型一样总是会产生相同的结果
    - `Redis Hashes`
        + 使用可参见上面的`- *hashes*`(下面也列几个示例)
            * `hmset user:1000 username antirez birthyear 1977 verified 1`
                - 设置指定hash key的多个域
                - `user:1000`为key，`username`为域，`antirez`为该域的值；`birthyear`为域，`1977`为该域的值...
            * `hmget user:1000 username birthyear no-such-field`
                - `user:1000`为key，获取三个域:`username` `birthyear` `no-such-field`(不存在的域，会返回NULL)
            * 可单独操作某个域：`hincrby user:1000 birthyear 10`
    - `Redis Sets`
        + 集合有利于表示对象之间的关系。
            * 例如，我们可以很容易地使用集合来实现标记(tag)
            * 建模这个问题的一个简单方法是为每个我们想标记的对象设置一个集合。该集合包含与对象关联的标记的id。
            * 如给新闻文章加标签，id为1000的文章，有多个标签`1` `2` `5` `77` (此处用标签的id，标签名称可用另外的hash对应)
                - 可使用：`sadd news:1000:tags 1 2 5 77`
                    + 获取指定文章的标签：
                        * `smembers news:1000:tags`
                - 也可使用反向关系：
                    + `sadd tag:1:news 1000`
                    + `sadd tag:2:news 1000`
                    + `sadd tag:5:news 1000`
                    + `sadd tag:77:news 1000`
                    + 可以用`sinter`来获取set的交集，所以可以过滤出同时包含多个标签的文章
                        * `sinter tag:1:news tag:2:news tag:5:news tag:77:news`
        + 应用：实现一个基于web的扑克游戏
            * 整付牌(没包括Joker鬼牌/大小王)：
                - `sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3 H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6 S7 S8 S9 S10 SJ SQ SK`
                - (C)lubs 梅花, (D)iamonds 方块, (H)earts 红心, (S)pades 黑桃
            * 不直接使用set而使用一个set的拷贝，以免每次游戏后，原来的牌都要重新加到set
                - 使用 `SUNIONSTORE`，把一个set存储到一个新的set
                - 拷贝纸牌：`sunionstore game:1:deck deck`
            * 使用`SPOP`发牌(会从set删除，若不删除则可用`SRANDMEMBER`)
                - `spop game:1:deck` 给玩家1发一张牌
                - `SCARD`可查看set个数，e.g. `scard game:1:deck`
    - `Redis Sorted sets`
        + 有序set是通过一个包含跳表和哈希表的双端数据结构实现的，所以每次我们添加一个元素，Redis都会执行`O(log(N))`操作

* [Redis 使用](http://www.redis.cn/documentation.html)
    - [Pub/Sub](http://www.redis.cn/topics/pubsub.html)
        + `subscribe foo bar` 订阅两个频道
        + `public foo Hello` 向频道foo发布消息(可在另一个客户端操作)
        + `UNSUBSCRIBE` 取消订阅(不过在其他客户端执行没反应)
        + 模式匹配订阅：
            * Redis 的Pub/Sub实现支持模式匹配。客户端可以订阅全风格的模式以便接收所有来自能匹配到`给定模式`的频道的消息
            * `psubscribe news.*` 订阅接收所有发到news.art.figurative, news.music.jazz形式频道的消息
                - `publish news.1 123` 形式发布
                - 可以多个`psubscribe`和`subscribe`一起订阅，会同时收到发布给匹配频道的消息(区别是pmessage和message)
    - [将redis当做使用LRU算法的缓存来使用](http://www.redis.cn/topics/lru-cache.html)
        + `LRU`是Redis唯一支持的回收方法(4.0之前)，Redis 4.0引入了新的`LFU`(Least Frequently Used)回收策略
        + `maxmemory`配置指令用于配置Redis存储数据时指定限制的内存大小
            * `redis.conf` 配置文件中的`maxmemory`指定
        + 到达内存限制后，根据回收策略(Eviction policies)不同而表现不同
            * redis.conf中的`maxmemory-policy` 中配置
    - `info`命令查看信息
        + 查看服务统计信息，可以加指定段只查看指定信息
            * [INFO [section]](https://redis.io/commands/info)
        + 数据库db信息 (`info keyspace`查看)
            * 不同的应用程序数据存储在不同的数据库下
            * redis下，数据库是由一个整数索引标识，而不是由一个数据库名称。默认情况下，一个客户端连接到数据库0(`db0`)。redis配置文件中下面的参数来控制数据库总数：`/etc/redis/redis.conf`，文件中，有个配置项 `databases = 16`
                - `select 3`切换到db3
            * 每个数据库都有属于自己的空间，不必担心之间的key冲突
            * 可参考：[redis db0-db15](https://www.jianshu.com/p/03c1276941cc)
        + 内存(`info memory`)
            * 可看到内存池使用jemalloc：`mem_allocator:jemalloc-5.1.0`
            * 关于jemalloc实现：[jemalloc 源码分析](https://youjiali1995.github.io/allocator/jemalloc/)
            * 什么场景使用`Ptmalloc2` `tcmalloc` `jemalloc`等，可参考：[02 | 内存池：如何提升内存分配的效率？](https://time.geekbang.org/column/article/230221)
