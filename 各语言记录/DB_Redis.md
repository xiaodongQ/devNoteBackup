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
        + MongoDB部署机器的磁盘性能也是一个问题
    - 现需要查询跨天的记录，直接查数据库响应时间太慢，因此查询资料找缓存方案
        + [Memcache,Redis,MongoDB（数据缓存系统）方案对比与分析](https://blog.csdn.net/suifeng3051/article/details/23739295)
        + [memcache和redis、Mongodb优缺点及应用场景](https://blog.csdn.net/weiyi_xingdong/article/details/79992032?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)
        + [redis、memcache、mongoDB 对比](https://blog.csdn.net/yu487/article/details/85714822?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)
    - 使用Redis作为缓存
        + 配置文件redis.conf进行缓存大小和策略配置(使用LRU，Least Recently Used最近最少使用)
            * `maxmemory 1.5gb` (1.5GB作缓存，单位可参考下面的比较说明，搜索`内存单位可选如下`)
            * `maxmemory-policy allkeys-lru`

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
        + sorted set 有序集合
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