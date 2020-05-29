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
* [Zookeeper vs Etcd](https://cloud.tencent.com/developer/article/1553467)
    - Zookeeper 和 Etcd 都是非常优秀的分布式协调系统，zookeeper 起源于 Hadoop 生态系统，etcd 的流行是因为它是 kubernetes 的后台支撑
    - zookeeper 起源于 Hadoop，后来进化为 Apache 的顶级项目。现在已经被广泛使用在 Apache 的项目中，例如 Hadoop，kafka，solr 等等
    - etcd 是用 go 开发的，出现的时间并不长，不像 zookeeper 那么悠久和有名，但是前景非常好