# Kubernetes

* [Kubernetes中文文档](https://www.kubernetes.org.cn/docs)
* 概述
    - Kubernetes单词起源于希腊语, 是“舵手”或者“领航员”的意思，是“管理者”和“控制论”的根源。
    - `K8s`是把用8代替8个字符“ubernete”而成的缩写。
    - Kubernetes是一个开源的，用于管理云平台中多个主机上的容器化的应用
    - 在Kubenetes中，所有的容器均在`Pod`中运行,一个Pod可以承载一个或者多个相关的容器
* kubelet
* replication controller

# etcd

* [etcd.io](https://etcd.io/)
* [etcd 快速入门](https://zhuanlan.zhihu.com/p/96428375?from_voters_page=true)
    - etcd 是一个分布式、可靠 key-value 存储的分布式系统。当然，它不仅仅用于存储，还提供共享配置及服务发现。
    - etcd vs Zookeeper
        + 此处选取 Zookeeper 作为典型代表与 etcd 进行比较，而不考虑 Consul 项目作为比较对象(Consul 的可靠性和稳定性还需要时间来验证)
        + 在项目实现、一致性协议易理解性、运维、安全等多个维度上，etcd 相比 zookeeper 都占据优势
            * 一致性协议： etcd 使用 `Raft` 协议，Zookeeper 使用 `ZAB`（类`PAXOS`协议），前者容易理解，方便工程实现；
            * 运维方面：etcd 方便运维，Zookeeper 难以运维；
            * 数据存储：etcd 多版本并发控制（MVCC）数据模型 ， 支持查询先前版本的键值对
            * 项目活跃度：etcd 社区与开发活跃
            * API：etcd 提供 HTTP+JSON, gRPC 接口，跨平台跨语言，Zookeeper 需要使用其客户端
            * 访问安全方面：etcd 支持 HTTPS 访问，Zookeeper 在这方面缺失
    - etcd应用场景
        + etcd 比较多的应用场景是用于服务发现，服务发现 (Service Discovery) 要解决的是分布式系统中最常见的问题之一，即在同一个分布式集群中的进程或服务如何才能找到对方并建立连接。
        + 和 Zookeeper 类似，etcd 有很多使用场景，包括：
            * 配置管理
            * 服务注册发现
            * 选主
            * 应用调度
            * 分布式队列
            * 分布式锁
    - 工作原理
        + 如何保持一致性
            * etcd 使用 Raft 协议来维护集群内各个节点状态的一致性。简单说，etcd 集群是一个分布式系统，由多个节点相互通信构成整体对外服务，每个节点都存储了完整的数据，并且通过 Raft 协议保证每个节点维护的数据是一致的
            * 每个 etcd 节点都维护了一个状态机，并且，任意时刻至多存在一个有效的主节点。主节点处理所有来自客户端写操作，通过 Raft 协议保证写操作对状态机的改动会可靠的同步到其他节点
        + 数据模型
            * 从逻辑角度看，etcd 的存储是一个扁平的二进制键空间，键空间有一个针对键（字节字符串）的词典序索引，因此范围查询的成本较低
            * 键空间维护了多个修订版本（Revisions），每一个原子变动操作（一个事务可由多个子操作组成）都会产生一个新的修订版本
            * etcd 将数据存放在一个持久化的 B+ 树中，处于效率的考虑，每个修订版仅仅存储相对前一个修订版的数据状态变化（Delta）
    - 与 etcd 交互
        + etcd 提供了 `etcdctl` 命令行工具 和 HTTP API 两种交互方法。
        + `etcdctl`命令行工具用 go 语言编写，也是对 HTTP API 的封装，日常使用起来也更容易。
        + `put`
            * 应用程序通过 `put` 将 key 和 value 存储到 etcd 集群中。
            * 每个存储的密钥都通过 Raft 协议复制到所有 etcd 集群成员，以实现一致性和可靠性
            * e.g. `etcdctl put foo bar` (key:foo, value:bar)
        + `get`
            * 从一个 etcd 集群中读取 key 的值
            * e.g. `etcdctl get foo` 读取键foo，会同时返回 key 和 value
                - `etcdctl get foo --print-value-only` 只读取 key 对应的值
            * `etcdctl get --prefix --rev=4 foo` # 访问第4个版本的key
        + `del`
            * 从一个 etcd 集群中删除一个 key 或一系列 key
            * e.g. `etcdctl del foo`
            * `etcdctl del --prefix zoo` 删除具有前缀的键
            * `etcdctl del --from-key b` 删除大于或等于键的字节值的键
        + `watch`
            * 使用watch观察一个键或一系列键来监视任何更新
            * e.g. `etcdctl watch foo` 对 foo 进行操作(put/del)可观察到更新
        + `lock`
            * 分布式锁 通过指定的名字加锁
            * e.g. `etcdctl lock mutex1`
        + `txn`
            * 从标准输入中读取多个请求，将它们看做一个原子性的事务执行
            * e.g. `./etcdctl txn -i` 进入交互式界面输入 判断条件、成功时执行操作、失败时执行操作
        + `compact`
            * 压缩过去的修订版本。压缩后，etcd删除历史版本，释放资源供将来使用。在压缩版本之前所有被修改的数据都将不可用
            * e.g. `etcdctl compact 5`
                - 若此时 `etcdctl get --rev=4 foo`，则会报错(因在压缩版本之前) Error: etcdserver: mvcc: required revision has been compacted
        + lease 与 TTL
            * etcd 也能为 key 设置超时时间，但与 redis 不同，etcd 需要先创建 lease，然后 `put` 命令加上参数 `-lease=` 来设置
            * lease 由生存时间（TTL）管理，
                - 授予租约：`etcdctl lease grant 30`
                    + 结果显示： "lease 694d6ee9ac06945d granted with TTL(30s)"
                - `etcdctl put --lease=694d6ee9ac06945d foo bar` 租赁，key为foo的键值对的TTL为30s
                - `etcdctl lease revoke 694d6ee9ac06945d` 撤销同一租约
            * `etcdctl lease keep-alive 694d6ee9ac06945d` 可以通过刷新其TTL来保持租约活着
            * `etcdctl lease timetolive 694d6ee9ac06945d` 检查租赁是否仍然存在或已过期，共多少、剩余多少时间
            * `etcdctl lease timetolive --keys 694d6ee9ac06945d` 获取哪些 key 使用了租赁信息
* [Zookeeper vs Etcd](https://cloud.tencent.com/developer/article/1553467)
    - Zookeeper 和 Etcd 都是非常优秀的分布式协调系统，zookeeper 起源于 Hadoop 生态系统，etcd 的流行是因为它是 kubernetes 的后台支撑
    - zookeeper 起源于 Hadoop，后来进化为 Apache 的顶级项目。现在已经被广泛使用在 Apache 的项目中，例如 Hadoop，kafka，solr 等等
    - etcd 是用 go 开发的，出现的时间并不长，不像 zookeeper 那么悠久和有名，但是前景非常好

## Kubernetes源码学习

* [Kubernetes源码之旅：从kubectl到API Server](https://www.kubernetes.org.cn/2324.html)
* `kubectl`
    - Kubernetes里的命令行接口叫做kubectl。它用来控制Kubernetes集群。阅读这部分源码实现是一个好的开始。我们要追踪的命令是kubectl create -f——它会从文件创建K8s资源


