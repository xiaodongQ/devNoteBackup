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
    - [Redis分布式锁](http://www.redis.cn/topics/distlock.html)
        + 分布式锁在很多场景中是非常有用的原语， 不同的进程必须以独占资源的方式实现资源共享就是一个典型的例子
        + Redis提出了一种算法，叫`Redlock`,认为这种实现比普通的单实例实现更安全
            * 链接中列出了各种语言对RedLock的实现版本
        + 按Redlock的思路和设计方案，算法只需具备3个特性就可以实现一个最低保障的分布式锁
            * 安全属性（Safety property）: 独享（相互排斥）。在任意一个时刻，只有一个客户端持有锁
            * 活性A(Liveness property A): 无死锁。即便持有锁的客户端崩溃（crashed)或者网络被分裂（gets partitioned)，锁仍然可以被获取
            * 活性B(Liveness property B): 容错。 只要大部分Redis节点都活着，客户端就可以获取和释放锁.
        + 实现Redis分布式锁的最简单的方法就是在Redis中创建一个key，这个key有一个失效时间（TTL)，以保证锁最终会被自动释放掉（这个对应特性2）。当客户端释放资源(解锁）的时候，会删除掉这个key
            * 从表面上看，似乎效果还不错，但是这里有一个问题：这个架构中存在一个严重的`单点`失败问题
            * 通过增加一个slave节点通常是行不通的，这样做，不能实现资源的独享,因为Redis的主从同步通常是异步的
                - 客户端A从master获取到锁，在master把锁同步到slave之前master宕机，slave变为master，客户端B从新master获取锁，这样就导致A和B都获取到了锁
                - 如果可以接受这种小概率错误，那用这个基于复制的方案就完全没有问题。否则的话，建议实现下面描述的解决方案
        + 先看 单Redis实例实现分布式锁的正确方法
            * 尽管存在竞态，结果仍然是可接受的，另外，这里讨论的单实例加锁方法也是分布式加锁算法的基础
            * 获取锁使用命令：`SET resource_name my_random_value NX PX 30000`
                - 这个命令仅在不存在key的时候才能被执行成功（`NX`选项），并且这个key有一个30秒的自动失效时间（`PX`属性）
            * 这个key的值是“`my_random_value`”(一个随机值），这个值在所有的客户端必须是唯一的，所有同一key的获取者（竞争者）这个值都不能一样
            * value的值必须是*随机数*主要是为了更安全的释放锁，释放锁的时候使用*脚本*告诉Redis:只有key存在并且存储的值和我指定的值一样才能告诉我删除成功
                - 可以通过Lua脚本实现
                - 使用这种方式释放锁可以避免删除别的客户端获取成功的锁
                - 举个例子：客户端A取得资源锁，但是紧接着被一个其他操作阻塞了，当客户端A运行完毕其他操作后要释放锁时，原来的锁早已超时并且被Redis自动释放，并且在这期间资源锁又被客户端B再次获取到。如果仅使用DEL命令将key删除，那么这种情况就会把客户端B的锁给删除掉
                - 使用Lua脚本就不会存在这种情况，因为脚本仅会删除value等于客户端A的value的key（value相当于客户端的一个签名）
            * 这个随机字符串应该怎么设置？可从`/dev/urandom`产生的一个20字节随机数，只要这个数在你的任务中是唯一的就行
                - 例如一种安全可行的方法是使用`/dev/urandom`作为`RC4`的种子和源产生一个伪随机流;
                - 一种更简单的方法是把以毫秒为单位的unix时间和客户端ID拼接起来，理论上不是完全安全，但是在多数情况下可以满足需求
            * key的失效时间，被称作“`锁定有效期`”。
                - 它不仅是key自动失效时间，而且还是一个客户端持有锁多长时间后可以被另外一个客户端重新获得
            * 基于Redis单实例，假设这个单实例总是可用，这种方法已经足够安全。现在让我们扩展一下，假设Redis*没有总是可用的保障*
        + `Redlock`算法
            * 在Redis的分布式环境中，我们假设有N个Redis master。这些节点完全互相独立，不存在主从复制或者其他集群协调机制
            * 为了取到锁，客户端应该执行以下操作:
                - 获取当前Unix时间，以毫秒为单位。
                - 依次尝试从N个实例，使用相同的key和随机值获取锁
                    + 在步骤2，当向Redis设置锁时,客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。
                    + 这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果
                    + 如果服务器端没有在规定时间内响应，客户端应该尽快尝试另外一个Redis实例
                - 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。
                    + 当且仅当从大多数（这里是3个节点）的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功
                - 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）
                    + 如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功）
            * 失败时重试
                - 当客户端无法取到锁时，应该在一个随机延迟后重试,防止多个客户端在同时抢夺同一资源的锁（这样会导致脑裂，没有人会取到锁）
                - 同样，客户端取得大部分Redis实例锁所花费的时间越短，脑裂出现的概率就会越低（必要的重试），所以，理想情况一下，客户端应该同时（并发地）向所有Redis发送SET命令
                - 当客户端从大多数Redis实例获取锁失败时，应该尽快地释放（部分）已经成功取到的锁，这样其他的客户端就不必非得等到锁过完“有效时间”才能取到
                    + 然而，如果已经存在网络分裂，客户端已经无法和Redis实例通信，此时就只能等待key的自动释放了
            * 释放锁
                - 释放锁比较简单，向所有的Redis实例发送释放锁命令即可，不用关心之前有没有从Redis实例成功获取到锁.
            * 性能，崩溃恢复和Redis同步
                - 当一个redis节点重启后，只要它不参与到任意当前活动的锁，没有被当做“当前存活”节点被客户端重新获取到,算法的安全性仍然是有保障的
                    + 为了达到这种效果，我们只需要将新的redis实例，在一个TTL时间内，对客户端不可用即可，在这个时间内，所有客户端锁将被失效或者自动释放
                - 使用延迟重启可以在不采用持久化策略的情况下达到同样的安全，然而这样做有时会让系统转化为彻底不可用。比如大部分的redis实例都崩溃了，系统在TTL时间内任何锁都将无法加锁成功
            * 如果你的工作可以拆分为许多小步骤，可以将有效时间设置的小一些，使用锁的一些扩展机制。
                - 在工作进行的过程中，当发现锁剩下的有效时间很短时，可以再次向redis的所有实例发送一个Lua脚本，让key的有效时间延长一点（前提还是key存在并且value是之前设置的value)
    - [SETNX key value](http://www.redis.cn/commands/setnx.html)
        + 将key设置值为value，如果key不存在，这种情况下等同SET命令
        + 当key存在时，什么也不做。SETNX是”`SET if Not eXists`”的简写
        + 返回值
            * 如果key被设置了，返回`1`
            * 如果key没有被设置，返回`0`
        + e.g. `SETNX mykey "Hello"`，`GET mykey`
        + 设计模式：使用`SETNX`加锁
            * `SETNX lock.foo <current Unix time + lock timeout + 1>`
            * (类似set时nx的含义，`set mykey newval nx`，不存在才设置成功)
            * redis 官方不推荐 setnx 来实现分布式锁(https://redis.io/commands/setnx) 官方给出了 the Redlock algorithm 各语言的实现版本是来实现分布式锁的机制(即上面的各语言版本的`Redlock`)
        + 如果客户端出现故障，崩溃或者其他情况无法释放该锁会发生什么情况？
            * 原来锁被C3持有，但是C3宕机，然后C1和C2同时读取锁的时间戳，然后去DEL后SETNX获取锁，由于竞态条件导致 C1 和 C2 都获取到了锁(C1和C2同时SETNX获取时，不会保证并行安全？)
        + 使用以下算法避免这种情况
            * C4 发送`SETNX lock.foo`，已经崩溃的客户端 C3 仍然持有该锁，所以Redis将会返回0给 C4
            * C4 发送GET lock.foo检查该锁是否已经过期。如果没有过期，C4 客户端sleep一段时间，再从最开始进行重试操作
            * 如果锁已经过期，则C4尝试执行`GETSET lock.foo <current Unix timestamp + lock timeout + 1>`
                - 由于GETSET 的语意，C4会检查已经过期的旧值是否仍然存储在lock.foo中。如果是的话，C4 会获得锁
                - 如果另一个客户端，假如为 C5 ，比 C4 更快的通过GETSET操作获取到锁，那么 C4 执行GETSET操作会返回一个没有过期的时间戳。C4 将会从第一个步骤重新开始
        + 为了使这种加锁算法更加的健壮，持有锁的客户端应该一直检查锁是否过期，保证使用DEL释放锁之前不会过期，因为客户端故障的情况可能很复杂
            * 不止是崩溃，还可能由于一些操作阻塞一段时间，并且在阻塞比较久并恢复后尝试执行DEL（而此时该LOCK可能已经被其他客户端所持有）


## Redis集群

* [Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)
    - 每个节点都需要开放两个端口，普通端口和集群总线端口(在普通端口基础上固定+10000)
    - 一个Redis至少要6个节点，3主3备
    - Redis集群数据分片并不使用一致性哈希，而是用 哈希槽(hash slot)，Redis集群中有`16384`个hash slot
        + 要计算给定key的hash slot，通过对key的`CRC16`(循环冗余校验)对16384取模
        + 每个集群节点都负责一部分hash slot
        + 移动hash slot到另一个节点，并不需要停止服务
        + every hash slot has from 1 (the master itself) to N replicas (N-1 additional slaves nodes).
    - 通过`redis-cli`创建集群
        + 创建6个实例目录，`mkdir 7000 7001 7002 7003 7004 7005` (`mkdir cluster-test`，`cd cluster-test`)
        + 每个目录创建一个`redis.conf`配置文件，其中指定端口和启用集群
        + 编译`redis-server`(下载源码后，gcc版本有要求，4.8会报很多字段找不到，临时升级到了9.x)
        + 启动一个实例，`cd 7000`，`../redis-server ./redis.conf`
            * 依次后台启动这6个实例 `nohup ../redis-server ./redis.conf &`
        + `redis-cli`创建集群(5.0版本及之后)(Redis4和3版本，用`redis-trib.rb`)
            * 完整命令：`redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1`
                - `--cluster-replicas 1` 表示每创建一个master都需要一个slave
        + 创建成功显示：[OK] All 16384 slots covered.
    - 也可以通过`create-cluster`脚本创建集群
        + 脚本中做的操作和上面操作差不多
    - 测试集群
        + `redis-cli -c -p 7000` 连接集群
        + 然后就可以操作了：`set foo bar`
            * 会显示写入到哪个节点了：`-> Redirected to slot [12182] located at 127.0.0.1:7002`
            * `get foo`
                - `-> Redirected to slot [12182] located at 127.0.0.1:7002-> Redirected to slot [12182] located at 127.0.0.1:7002`
* [Redis Cluster Specification](https://redis.io/topics/cluster-spec)
    - Redis集群规范

## Redis核心技术与实战

* [02 | 数据结构：快速的Redis有哪些慢操作？](https://time.geekbang.org/column/article/268253)
    - Redis接收到一个键值对操作后，能以`微秒级别`的速度找到数据，并快速完成操作
        + 为啥 Redis 能有这么突出的表现呢？
        + 一方面，这是因为它是内存数据库，所有操作都在内存上完成，内存的访问速度本身就很快。
        + 另一方面，这要归功于它的数据结构。高效的数据结构是 Redis 快速处理数据的基础
    - 底层数据结构一共有 6 种，分别是`简单动态字符串`、`双向链表`、`压缩列表`、`哈希表`、`跳表`和`整数数组`
        + 数据的保存形式(键值对中值的数据类型)：`String`（字符串）、`List`（列表）、`Hash`（哈希）、`Set`（集合）和 `Sorted Set`，这些类型由底层数据结构实现
        + `String` 类型的底层实现只有一种数据结构，也就是简单动态字符串
        + 而 `List`、`Hash`、`Set` 和 `Sorted Set` 这四种数据类型，都有两种底层实现结构(参考链接图示)，通常情况下，我们会把这四种类型称为`集合类型`，它们的特点是一个键对应了一个集合的数据
    - 为了实现从键到值的快速访问，Redis 使用了一个哈希表来保存所有键值对
        + 哈希桶中的元素保存的并不是值本身，而是指向具体值的*指针*
        + 哈希桶中的 `entry` 元素中保存了`*key`和`*value`指针，分别指向了实际的键和值
        + 因为这个哈希表保存了所有的键值对，所以，我也把它称为`全局哈希表`
    - 当你往Redis中写入大量数据后，就可能发现操作有时候会突然变慢了。
        + 这其实是因为你忽略了一个潜在的风险点，那就是哈希表的*冲突问题*和 *rehash* 可能带来的操作阻塞
    - Redis 解决`哈希冲突`的方式，就是*链式哈希*
        + 哈希冲突链上的元素只能通过指针逐一查找再操作。
        + 如果哈希表里写入的数据越来越多，哈希冲突可能也会越来越多，这就会导致某些哈希冲突链过长，进而导致这个链上的元素查找耗时长，效率降低
    - 所以，Redis 会对哈希表做 `rehash` 操作
        + rehash也就是增加现有的哈希桶数量，让逐渐增多的entry元素能在更多的桶之间分散保存，减少单个桶中的元素数量，从而减少单个桶中的冲突
        + 为了使 rehash 操作更高效，Redis 默认使用了*两个全局哈希表*：哈希表 1 和哈希表 2
        + 刚插入数据时，默认使用哈希表 1，此时的哈希表 2 并没有被分配空间。随着数据逐步增多，Redis 开始执行 rehash
        + 这个过程分为三步：
            * 给哈希表 2 分配更大的空间，例如是当前哈希表 1 大小的两倍
            * 把哈希表 1 中的数据重新映射并拷贝到哈希表 2 中
            * 释放哈希表 1 的空间
        + 第二步涉及大量的数据拷贝，如果一次性把哈希表 1 中的数据都迁移完，为了避免造成 Redis 线程阻塞
            * Redis 采用了`渐进式 rehash`
            * 简单来说就是在第二步拷贝数据时，Redis 仍然正常处理客户端请求，每处理一个请求时，从哈希表 1 中的第一个索引位置开始，顺带着将*这个索引位置*上的*所有 entries* 拷贝到哈希表 2 中
            * 等处理下一个请求时，再顺带拷贝哈希表 1 中的下一个索引位置的 entries
            * 这样就巧妙地把一次性大量拷贝的开销，分摊到了多次处理请求的过程中，避免了耗时操作，保证了数据的快速访问
    - 集合数据操作效率
        + 对于 String 类型来说，找到哈希桶就能直接增删改查了，所以，哈希表的 O(1) 操作复杂度也就是它的复杂度
        + 对于集合类型来说，即使找到哈希桶了，还要在集合中再进一步操作
        + `整数数组`和`双向链表`比较常见，它们的操作特征都是顺序读写，也就是通过数组下标或者链表的指针逐个元素访问，操作复杂度基本是`O(N)`，操作效率比较低
        + `压缩列表`实际上类似于一个数组，数组中的每一个元素都对应保存一个数据。
            * 和数组不同的是，压缩列表在表头有三个字段 `zlbytes`、`zltail` 和 `zllen`，分别表示列表长度、列表尾的偏移量和列表中的 entry 个数；压缩列表在表尾还有一个 `zlend`，表示列表结束
            * 在压缩列表中，如果我们要查找定位第一个元素和最后一个元素，可以通过表头三个字段的长度直接定位，复杂度是 `O(1)`。
            * 而查找其他元素时，就没有这么高效了，只能逐个查找，此时的复杂度就是 `O(N)` 了
        + `跳表`在链表的基础上，增加了多级索引，通过索引位置的几个跳转，实现数据的快速定位
            * 当数据量很大时，跳表的查找复杂度就是 `O(logN)`
    - 集合常见操作的复杂度
        + 单元素操作，是指每一种集合类型对单个数据实现的增删改查操作。
            * 例如，Hash 类型的 `HGET`、`HSET` 和 `HDEL`，Set 类型的 `SADD`、`SREM`、`SRANDMEMBER` 等
            * 这些操作的复杂度由集合采用的数据结构决定，例如`HGET`、`HSET`和`HDEL`是对哈希表做操作，所以它们的复杂度都是`O(1)`
            * Set 类型用哈希表作为底层数据结构时，它的 `SADD`、`SREM`、`SRANDMEMBER` 复杂度也是 `O(1)`
            * 另外，集合类型支持同时对多个元素进行增删改查，例如，`HMSET` 增加 M 个元素时，复杂度就从 `O(1)` 变成 `O(M)` 了
        + 范围操作，是指集合类型中的遍历操作，可以返回集合中的所有数据
            * 这类操作的复杂度一般是 `O(N)`，比较耗时，我们应该尽量避免
            * 不过，Redis提供了 `SCAN`系列操作（包括 `HSCAN`，`SSCAN` 和 `ZSCAN`），这类操作实现了*渐进式遍历*，每次只返回有限数量的数据，避免了一次性返回所有元素而导致的 Redis 阻塞
        + 统计操作，是指集合类型对集合中所有元素个数的记录
            * 例如 `LLEN` 和 `SCARD`。这类操作复杂度只有 `O(1)`，这些结构专门记录了元素的个数统计
        + 例外情况，是指某些数据结构的特殊记录
            * 例如压缩列表和双向链表都会记录表头和表尾的偏移量，因此对于 List 类型的 `LPOP`、`RPOP`、`LPUSH`、`RPUSH` 这四个操作来说，它们是在列表的头尾增删元素，这就可以通过偏移量直接定位，所以它们的复杂度也只有 `O(1)`
* [04 | AOF日志：宕机了，Redis如何避免数据丢失？](https://time.geekbang.org/column/article/271754)
    - 一旦Redis所在服务器宕机，内存中的数据将全部丢失
    - `AOF`(Append Only File)，AOF 日志和一些常见数据库的`写前日志`（Write Ahead Log, `WAL`）正好相反，它是`写后日志`，“写后”的意思是 Redis 是先执行命令，把数据写入内存，然后才记录日志
        + 传统数据库的日志，例如 redo log（重做日志），记录的是修改后的*数据*，而 AOF 里记录的是 Redis 收到的每一条*命令*，这些命令是以文本形式保存的
            * 链接示例：“`*3`”表示当前命令有三个部分，每部分都是由“`$+数字`”开头，后面紧跟着具体的命令、键或值。这里，“`数字`”表示这部分中的命令、键或值一共有多少*字节*
        + 为了避免额外的检查开销，Redis 在向 AOF 里面记录日志的时候，并不会先去对这些命令进行语法检查。
            * 所以，如果先记日志再执行命令的话，日志中就有可能记录了错误的命令，Redis 在使用日志恢复数据时，就可能会出错
            * Redis 使用写后日志这一方式的一大好处是，可以避免出现记录错误命令的情况
        + 除此之外，AOF 还有一个好处：它是在命令执行后才记录日志，所以不会阻塞当前的写操作
    - 不过，AOF 也有两个潜在的风险
        + 首先，如果刚执行完一个命令，还没有来得及记日志就宕机了，那么这个命令和相应的数据就有丢失的风险
        + 其次，AOF 虽然避免了对当前命令的阻塞，但可能会给下一个操作带来阻塞风险，如果在把日志文件写入磁盘时，磁盘写压力大，就会导致写盘很慢，进而导致后续的操作也无法执行了。
    - 这两个风险都是和 AOF 写回磁盘的时机相关的，如果我们能够控制一个写命令执行完后 AOF 日志`写回磁盘的时机`，这两个风险就解除了
    - 三种写回策略(AOF 配置项 `appendfsync` 的三个可选值)
        + `Always`，同步写回：每个写命令执行完，*立马*同步地将日志写回磁盘
            * “同步写回”可以做到基本不丢数据，但是它在每一个写命令后都有一个慢速的落盘操作，不可避免地会影响主线程性能
        + `Everysec`，每秒写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，*每隔一秒*把缓冲区中的内容写入磁盘
            * 如果发生宕机，上一秒内未落盘的命令操作仍然会丢失
            * 这能算是，在避免影响主线程性能和避免数据丢失两者间取了个折中
        + `No`，操作系统控制的写回：每个写命令执行完，只是先把日志写到AOF文件的内存缓冲区，由*操作系统决定*何时将缓冲区内容写回磁盘
            * 该方式落盘的时机已经不在 Redis 手中了，只要 AOF 记录没有写回磁盘，一旦宕机对应的数据就丢失了
        + 总结一下就是：想要获得高性能，就选择 `No` 策略；如果想要得到高可靠性保证，就选择 `Always` 策略；如果允许数据有一点丢失，又希望性能别受太大影响的话，那么就选择 `Everysec` 策略
        + 三种写回策略体现了系统设计中的一个重要原则 ，即 `trade-off`，或者称为“`取舍`”，指的就是在性能和可靠性保证之间做取舍
    - 随着接收的写命令越来越多，AOF 文件会越来越大，要注意AOF 文件过大带来的性能问题
        + 一是，文件系统本身对文件大小有限制，无法保存过大的文件
        + 二是，如果文件太大，之后再往里面追加命令记录的话，效率也会变低
        + 三是，如果发生宕机，AOF中记录的命令要一个个被重新执行，用于故障恢复，如果日志文件太大，整个恢复过程就会非常缓慢，这就会影响到 Redis 的正常使用
    - 日志文件太大了怎么办？
        + `AOF 重写机制`
            * AOF 重写机制就是在重写时，Redis根据`数据库的现状`创建一个新的AOF文件
            * 也就是说，读取数据库中的所有键值对，然后对每一个键值对用一条命令记录它的写入
            * 例如当读取了键值对“testkey”: “testvalue”之后，重写机制会记录 `set testkey testvalue` 这条命令。这样，当需要恢复时，可以重新执行该命令，实现“testkey”: “testvalue”的写入
        + 为什么重写机制可以把日志文件变小呢?
            * 旧日志文件中的多条命令，在重写后的新日志中变成了一条命令
            * 当一个键值对被多条写命令反复修改时，AOF文件会记录相应的多条命令。但是，在重写的时候，是根据这个键值对当前的最新状态，为它生成对应的写入命令(只要一条命令即可)
    - AOF 重写会阻塞吗?
        + 和 AOF 日志由主线程写回不同，`重写过程`是由后台线程`bgrewriteaof`来完成的，这也是为了避免阻塞主线程，导致数据库性能下降。
        + “一个拷贝，两处日志”
            * “一个拷贝”就是指，每次执行重写时，主线程 `fork` 出后台的 bgrewriteaof 子进程。此时，fork 会把主线程的内存拷贝一份给 bgrewriteaof 子进程，这里面就包含了数据库的最新数据
            * 然后，bgrewriteaof 子进程就可以在不影响主线程的情况下，逐一把拷贝的数据写成操作，记入重写日志
            * “两处日志”：因为主线程未阻塞，仍然可以处理新来的操作。此时，如果有写操作，第一处日志就是指正在使用的 AOF 日志，Redis 会把这个操作写到它的缓冲区；第二处日志，就是指新的 AOF 重写日志。这个操作也会被写到重写日志的缓冲区
        + 总结来说，每次AOF重写时，Redis会先执行一个内存拷贝，用于重写；
            * 然后，使用两个日志保证在重写过程中，新写入的数据不会丢失。而且，因为Redis采用额外的线程进行数据重写，所以，这个过程并不会阻塞主线程