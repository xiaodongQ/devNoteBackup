# Kubernetes

* 部署
    - master、node节点**都需要**安装`kubelet` `kubeadm` `kubectl`
        + [Installing kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)
        + `kubeadm`并不会为你安装或管理`kubelet` 或 `kubectl`，需要你保证二者和`kubeadm`版本匹配
    - 步骤
        + [k8s集群搭建教程 (centos k8s 搭建)](https://juejin.im/post/5cb7dde9f265da034d2a0dba)
        + 采用国内阿里云镜像源(连不上官方链接的google源)，安装kubelet、kubeadm、kubectl:
            * 排版原因离得比较远，搜索：`kubernetes镜像源`
        + 配置下面列出的源后，`yum list | grep kubeadm`
            * 结果显示可用的包为：kubeadm.x86_64  1.18.3-0  @kubernetes
        + `yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`
        + `systemctl enable --now kubelet` 开机启动kubelet
            * `--now`参数会在设置的同时启动服务
        + centos7用户还需要设置路由：
            * `lsmod|grep br_netfilter`查看是否加载了模块
        + k8s要求关闭swap(不关闭则下面的`kubeadm init`初始化时会出现报错提示)
            * 使用 kubeadm 部署集群必须关闭 Swap分区，各节点均需要执行本操作
            * `swapoff -a && sysctl -w vm.swappiness=0` 关闭swap
            * 取消开机挂载swap：`/etc/fstab`配置文件，注释`swap`那行
                - 或通过sed替换 `sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab`
        + 创建Kubernetes集群的镜像准备(Master端，Node端有所不同)
            * `kubeadm config images list` 查看集群使用的容器镜像都有哪些
                - 结果如：k8s.gcr.io/kube-apiserver:v1.18.3
                - 但是有个WARNING: kubeadm cannot validate，由于网络连接原因验证不了
            * 如果镜像连接正常，则执行：`kubeadm config images pull` 即可
            * 由于国内网络环境，无法下载到 k8s.gcr.io 的镜像，通过拉取docker容器后改tag来绕过该问题
                - [Kubernetes：如何解决从k8s.gcr.io拉取镜像失败问题](https://blog.csdn.net/u010096900/article/details/82792617)
                    + 上面链接的镜像源貌似也连不上，用阿里云镜像仓库，参考：[国内拉取google kubernetes镜像](https://blog.csdn.net/networken/article/details/84571373)
                - docker.io仓库对google的容器做了镜像，可以通过下列命令下拉取相关镜像：
                    + (版本号设置成跟上面`kubeadm config images list`列出的一致)
                    + docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.18.3
                    + docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.18.3
                    + docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.18.3
                    + docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.3
                    + docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
                    + docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0
                    + docker pull coredns/coredns:1.6.7
                - 修改镜像tag(改成和上面`kubeadm config images list`列出的包名和版本一致)
                    + docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.18.3 k8s.gcr.io/kube-apiserver:v1.18.3
                    + docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.18.3 k8s.gcr.io/kube-controller-manager:v1.18.3
                    + docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.18.3 k8s.gcr.io/kube-scheduler:v1.18.3
                    + docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.3 k8s.gcr.io/kube-proxy:v1.18.3
                    + docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2 k8s.gcr.io/pause:3.2
                    + docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0
                    + docker tag coredns/coredns:1.6.7 k8s.gcr.io/coredns:1.6.7
                - 删除修改tag前的镜像(可以用`docker images`查看已有镜像，修改tag后会生成一个新的镜像)
                    + docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.18.3
                    + docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.18.3
                    + docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.18.3
                    + docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.3
                    + docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
                    + docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0
                    + docker rmi coredns/coredns:1.6.7
        + Node端镜像准备
            * 比Master端需要的包要少
            * 同上面一样配置阿里镜像源，拉取镜像
                - docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.3
                - docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
            * 修改镜像tag
                - docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.3 k8s.gcr.io/kube-proxy:v1.18.3
                - docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2 k8s.gcr.io/pause:3.2
            * 删除修改tag前的镜像
                - docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.3
                - docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
        + 初始化Master：
            * `kubeadm init --apiserver-advertise-address 192.168.50.207 --pod-network-cidr 192.168.50.0/24`
                - `--apiserver-advertise-address` 指定与其它节点通信的接口
                - `--pod-network-cidr` 指定pod网络子网，使用fannel网络必须使用这个CIDR
            * 其他选项可以查看help：`kubeadm init -h|less`
            * 执行结果报错:
                - [ERROR Swap]: running with swap on is not supported. Please disable swap
                    + 需要关闭swap，参考前面的操作，搜索：`k8s要求关闭swap`
                - [ERROR DirAvailable--var-lib-etcd]: /var/lib/etcd is not empty
                    + 手动安装过etcd，`yum remove etcd`卸载后，删除`/var/lib/etcd`
            * `/var/log/messages`中报错：
                - ubelet.go:2187] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message
    - etcd
        + `yum install etcd -y`
        + yum安装的etcd默认配置文件在/etc/etcd/etcd.conf

```sh
# kubernetes镜像源
# 上面链接中配置的源是google，采用国内阿里云镜像源(exclude=kube*)：

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```


* [Kubernetes中文文档](https://www.kubernetes.org.cn/docs)
* 概述
    - Kubernetes单词起源于希腊语, 是“舵手”或者“领航员”的意思，是“管理者”和“控制论”的根源。
    - `K8s`是把用8代替8个字符“ubernete”而成的缩写。
    - Kubernetes是一个开源的，用于管理云平台中多个主机上的容器化的应用
    - 在Kubenetes中，所有的容器均在`Pod`中运行,一个Pod可以承载一个或者多个相关的容器
* kubelet
* replication controller


* [官网教程](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/)
* 1. 创建集群
    * 一个 Kubernetes 集群包含两种类型的资源:
        + Master 调度整个集群
            * Master 负责管理整个集群。 Master 协调集群中的所有活动，例如调度应用、维护应用的所需状态、应用扩容以及推出新的更新。
        + Nodes 负责运行应用
            * Node 是一个虚拟机或者物理机，它在 Kubernetes 集群中充当工作机器的角色
            * 每个Node都有 Kubelet , 它管理 Node 而且是 Node 与 Master 通信的代理
            * Node 还应该具有用于​​处理容器操作的工具，例如 Docker 或 rkt
            * 处理生产级流量的 Kubernetes 集群至少应具有三个 Node
            * Node 使用 Master 暴露的 Kubernetes API 与 Master 通信
                - 终端用户也可以使用 Kubernetes API 与集群交互
    - Kubernetes 既可以部署在物理机上也可以部署在虚拟机上
        + 可以使用 Minikube 开始部署 Kubernetes 集群。 Minikube 是一种轻量级的 Kubernetes 实现
            * Minikube 是一种轻量级的 Kubernetes 实现，可在本地计算机上创建 VM 并部署仅包含一个节点的简单集群
            * Minikube 可用于 Linux ， macOS 和 Windows 系统
            * Minikube CLI 提供了用于引导群集工作的多种操作，包括启动、停止、查看状态和删除
            * 参考链接中，可以使用预装有 Minikube 的在线终端进行体验
        + `minikube version`
        + `minikube start` 执行后，Minikube会启动一个虚拟机，Kubernetes集群运行在虚拟机中
        + `kubectl version` 可以看到客户端和服务端的板块，可以检查是否配置好和集群通信
            * 客户端为`kubectl`的版本
            * 服务端为Kubernetes的版本
        + `kubectl cluster-info` 查看Kubernetes集群信息
        + `kubectl get nodes` 查看所有节点，可以看到只有一个(上面创建的)节点
            * NAME       STATUS   ROLES    AGE     VERSION
            * minikube   Ready    master   3m27s   v1.17.3
    * [使用 Minikube 创建集群](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)
* 2. 部署应用
    - [使用 kubectl 创建 Deployment](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/)
    - 一旦运行了 Kubernetes 集群，就可以在其上部署容器化应用程序。
    - 为此需要创建 Kubernetes Deployment 配置
        + Deployment 指挥 Kubernetes 如何创建和更新应用程序的实例
        + 创建 Deployment 后，Kubernetes master 将应用程序实例调度到集群中的各个节点上
        + 创建应用程序实例后，Kubernetes Deployment 控制器会持续监视这些实例。
        + 如果托管实例的节点关闭或被删除，则 Deployment 控制器会将该实例替换为群集中另一个节点上的实例
        + 这提供了一种自我修复机制来解决机器故障维护问题
    - 在没有 Kubernetes 这种编排系统之前，安装脚本通常用于启动应用程序，但它们不允许从机器故障中恢复。
        + 通过创建应用程序实例并使它们在节点之间运行， Kubernetes Deployments 提供了一种与众不同的应用程序管理方法。
    - 创建 Deployment 时，需要指定应用程序的容器映像以及要运行的副本数
        + 之后可通过更新 Deployment 来更改该信息;
    - 操作
        + `kubectl create deployment`命令，创建一个app
            * 需要提供 deployment的名称 和 app镜像的地址(包括外部Docker hub中完整存储镜像的仓库url)
            * e.g. `kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1`
                - 执行之后结果显示："deployment.apps/kubernetes-bootcamp created"
                - 这样就通过创建deployment创建好了一个应用
            * 创建时会做如下事情：
                - 搜索一个应用程序可运行的合适的节点
                - 调度应用运行在该节点上
                - 配置集群在有需要的情况下重调度实例到新节点上
        + `kubectl get deployment` 列出deployment
            * 执行结果如下：
                - NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
                - kubernetes-bootcamp   1/1     1            1           6m16s
        + Kubernetes中运行的Pods运行在私有、隔离的网络，默认情况下只对同集群的其他节点可见
* 3. 查看应用信息
    - 查看 pod 和工作节点
        + pod
            * 上面2中创建 Deployment 时，Kubernetes 添加了一个 Pod 来托管你的应用实例
            * Pod 是 Kubernetes 抽象出来的，表示一组一个或多个应用程序容器（如 Docker 或 rkt ），以及这些容器的一些共享资源
                - 共享存储，作为卷
                - 网络，作为唯一的集群 IP 地址
                - 有关每个容器如何运行的信息，例如容器映像版本或要使用的特定端口
            * 如果它们紧耦合并且需要共享磁盘等资源，这些容器应在一个 Pod 中编排。
            * Pod是 Kubernetes 平台上的原子单元。 当我们在 Kubernetes 上创建 Deployment 时，该 Deployment 会在其中创建包含容器的 Pod （而不是直接创建容器）
        + 工作节点
            * 一个 pod 总是运行在 工作节点
            * 工作节点是 Kubernetes 中的参与计算的机器，可以是虚拟机或物理计算机，具体取决于集群
            * 每个工作节点由主节点管理。工作节点可以有多个 pod ，Kubernetes 主节点会自动处理在集群中的工作节点上调度 pod
            * 每个 Kubernetes 工作节点至少运行:
                - Kubelet，负责 Kubernetes 主节点和工作节点之间通信的过程; 它管理 Pod 和机器上运行的容器
                - 容器运行时（如Docker，rkt）负责从仓库中提取容器镜像，解压缩容器及运行应用程序
    - 获取有关已部署的应用程序及其环境的信息。 最常见的操作可以使用以下 `kubectl` 命令完成：
        + `kubectl get` - 列出资源
        + `kubectl describe` - 显示有关资源的详细信息
        + `kubectl logs` - 打印 pod 和其中容器的日志
        + `kubectl exec` - 在 pod 中的容器上执行命令

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

