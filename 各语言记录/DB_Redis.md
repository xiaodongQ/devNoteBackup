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
* [Memcache,Redis,MongoDB（数据缓存系统）方案对比与分析](https://blog.csdn.net/suifeng3051/article/details/23739295)