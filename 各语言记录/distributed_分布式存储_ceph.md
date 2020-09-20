# 分布式存储

* [18 | 分布式存储：你知道对象存储是如何保存图片文件的吗？](https://time.geekbang.org/column/article/220609)
    - 保存像图片、音视频这类大文件，最佳的选择就是对象存储
        + 对象存储不仅有很好的大文件读写性能，还可以通过水平扩展实现近乎无限的容量，并且可以兼顾服务高可用、数据高可靠这些特性
    - 对象存储之所以能做到这么“全能”，最主要的原因是，对象存储是`原生的分布式存储系统`
        + 这里讲的“`原生分布式存储系统`”，是相对于 MySQL、Redis 这类单机存储系统来说的
        + 虽然这些非原生的存储系统，也具备一定的集群能力，但用它们构建大规模分布式集群的时候，其实是非常不容易的
    - 随着云计算的普及，很多新生代的存储系统，都是原生的分布式系统，它们一开始设计的目标之一就是分布式存储集群，比如说`Elasticsearch`、`Ceph`和国内很多大厂推出的新一代数据库，大多可以做到：
        + 近乎无限的存储容量
        + 超高的读写性能
        + 数据高可靠：节点磁盘损毁不会丢数据
        + 实现服务高可用：节点宕机不会影响集群对外提供服务
    - 对象存储它的查询服务和数据结构都非常简单，是最简单的原生分布式存储系统
        + 本节通过对象存储来认识一下分布式存储系统的一些共性
    - 对象存储数据是如何保存大文件的？
        + 对象存储对外提供的服务，其实就是一个近乎无限容量的大文件KV存储，所以对象存储和分布式文件系统之间，没有那么明确的界限
        + 对象存储的内部，有很多的存储节点，用于保存这些大文件，这个就是**数据节点的集群**
        + 另外，为了管理这些数据节点和节点中的文件，还需要一个存储系统保存集群的节点信息、文件信息和它们的映射关系。
            * 这些为了管理集群而存储的数据，叫做`元数据 (Metadata)`
            * 元数据对于一个存储集群来说是非常重要的，所以保存元数据的存储系统必须也是一个集群(**元数据集群**)
            * 元数据集群存储的数据量比较少，数据的变动不是很频繁，加之客户端或者网关都会缓存一部分元数据，所以元数据集群对并发要求也不高
            * 一般使用类似`ZooKeeper`或者`etcd`这类分布式存储就可以满足要求
        + 另外，存储集群为了对外提供访问服务，还需要一个**网关集群**，对外接收外部请求，对内访问元数据和数据节点
            * 网关集群中的每个节点不需要保存任何数据，都是`无状态`的节点
            * 有些对象存储没有网关，取而代之的是客户端，它们的功能和作用都是一样的
        + 那么，对象存储是如何来处理对象读写请求的呢？
            * 处理读和写请求的流程是一样的
            * 网关收到对象读写请求后，首先拿着请求中的Key，去元数据集群查找这个Key在哪个数据节点上，然后再去访问对应的数据节点读写数据，最后把结果返回给客户端
            * 以上是一个比较粗略的大致流程，相关名词改一改完全可以套用到绝大多数分布式文件系统和数据库上去，比如说 HDFS
    - 对象是如何拆分和保存的？
        + 一般来说，对象存储中保存的文件都是图片、视频这类大文件。
            * 在对象存储中，每一个大文件都会被拆成多个大小相等的`块（Block）`，拆分的方法很简单，就是把文件从头到尾按照固定的块大小，切成一块儿一块儿，最后一块的长度有可能不足一个块的大小，也按一块来处理。
            * 块的大小一般配置为`几十 KB` 到`几个 MB` 左右
        + 把大对象文件拆分成块的目的有两个：
            * 第一是为了提升读写性能，这些块儿可以分散到不同的数据节点上，这样就可以并行读写
            * 第二是把文件分成大小相等块，便于维护管理
        + 一般都会再把块聚合一下，放到块的容器里面
            * 这里的“容器”就是存放一组块的逻辑单元。容器这个名词，没有统一的叫法，比如在`ceph`中称为 `Data Placement`
            * 这个容器的概念，就比较类似于 MySQL 和 Redis 的“`分片`”的概念，都是复制、迁移数据的基本单位
            * 每个容器都会有 N 个副本，这些副本的数据都是一样的。其中有一个主副本，其他是从副本，主副本负责数据读写，从副本去到主副本上去复制数据，保证主从数据一致
        + 对象存储一般都不记录类似MySQL的Binlog这样的日志。主从复制的时候，复制的不是日志，而是整块儿的数据，这么做有两个原因：
            * 第一个原因是基于性能的考虑
                - 操作日志里面，实际上就包含着数据。在更新数据的时候，先记录操作日志，再更新存储引擎中的数据，相当于在磁盘上串行写了 2 次数据
                - 对于像数据库这种，每次更新的数据都很少的存储系统，这个开销是可以接受的
                - 对于对象存储来说，它每次写入的块儿很大，两次磁盘 IO 的开销就有些不太值得了
            * 第二个原因是它的存储结构简单，即使没有日志，只要按照顺序，整块儿的复制数据，仍然可以保证主从副本的数据一致性
        + 怎么把块儿映射到容器中
            * 可以参考：[《15 | MySQL 存储海量数据的最后一招：分库分表》](https://time.geekbang.org/column/article/217568)这节课中讲到的几种分片算法
            * 不同的系统选择实现的方式也不一样，有用哈希分片的，也有用查表法把对应关系保存在元数据中的
            * 找到容器之后，再去元数据中查找容器的N个副本都分布在哪些数据节点上。然后，网关直接访问对应的数据节点读写数据就可以了
    - 小结
        + 对象存储是最简单的分布式存储系统，主要由`数据节点集群`、`元数据集群`和`网关集群（或者客户端）`三部分构成
            * 数据节点集群负责保存对象数据，
            * 元数据集群负责保存集群的元数据，
            * 网关集群和客户端对外提供简单的访问 API，对内访问元数据和数据节点读写数据
        + 对象存储虽然简单，但是它具备一个分布式存储系统的全部特征。所有分布式存储系统共通的一些特性，对象存储也都具备，比如说数据如何分片，如何通过多副本保证数据可靠性，如何在多个副本间复制数据，确保数据一致性等等
* [15 | MySQL存储海量数据的最后一招：分库分表](https://time.geekbang.org/column/article/217568)
    - 解决海量数据的问题，必须要用到分布式的存储集群，因为 MySQL 本质上是一个单机数据库，所以很多场景下不是太适合存 TB 级别以上的数据
        + 但是，绝大部分的电商大厂，它的在线交易这部分的业务，比如说，订单、支付相关的系统，还是舍弃不了 MySQL，原因是，只有 MySQL 这类关系型数据库，才能提供`金融级的事务保证`
        + 那些新的分布式数据库提供的所谓的分布式事务，目前还达不到这些交易类系统对数据一致性的要求
    - 既然 MySQL 支持不了这么大的数据量，这么高的并发，还必须要用它，就需要进行`分片`
        + `分片`，也就是拆分数据。1TB 的数据，一个库撑不住，我把它拆成 100 个库，每个库就只有 10GB 的数据了，这不就可以了么？这种拆分就是所谓的 `MySQL 分库分表`

# ceph

* github：[ceph/ceph](https://github.com/ceph/ceph)
    - 源码编译
        + github下载源码(看git clone下来有`650M`)
        + 编译脚本
            * 默认是debug版本，可以如下添加编译选项，编译非debug版本(README中说会快5倍)
            * `ARGS="-DCMAKE_BUILD_TYPE=RelWithDebInfo" ./do_cmake.sh` 执行完后大小比较大，查看大小有1.7G
            * 会创建`build`目录，若编译或者编译检查失败，下一次执行需要手动删除`build`目录(再次执行上面脚本时会提示)
        + 问题
            * 脚本最后报错：Can't find sphinx-build
                - 解决：`yum install python-sphinx`
            * 安装上面的python包后，再次执行脚本，报错：GCC 7+ required due to C++17 requirements
                - 需要`gcc7`以上版本(支持C++17)，本机版本为`gcc 版本 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)`
                - gcc对C++各版本的支持，之前笔记也有记录(`#### C++11 编译器支持：`)：[devNoteBackup/各语言记录/C++.md](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%90%84%E8%AF%AD%E8%A8%80%E8%AE%B0%E5%BD%95/C%2B%2B.md)
                - 可以临时使用gcc高版本
                    + `yum -y install centos-release-scl`
                    + `yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils`
                    + `scl enable devtoolset-7 bash`
                    + 重启后会恢复原系统gcc版本，每次临时使用可以`scl enable devtoolset-7 bash`，会进入一个新的bash
                    + 如果要长期使用gcc 7.3的话
                        * `echo "source /opt/rh/devtoolset-7/enable" >>/etc/profile`
                    + 其他gcc版本可参考：[为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本](https://www.vpser.net/manage/centos-6-upgrade-gcc.html)
            * gcc版本升级后，再次执行报错：`Could NOT find verbs`
                - `yum install rdma-core-devel -y`
            * 报错：`Could NOT find udev`
                - `yum install systemd-devel -y`
            * 上面的问题，这里面都碰到了，直接继续下面的安装：[CentOS 7.6 源码编译安装Ceph](https://blog.csdn.net/helloanthea/article/details/103728684)
                - `yum install libblkid-devel -y`
                - `yum install keyutils-libs-devel -y`
                - `yum install openldap-devel -y`
                - `yum install leveldb-devel -y`
                - `yum install snappy-devel -y`
                - `yum install lz4-devel -y`
                - `yum install curl-devel -y`
                - 还有这些：`yum install libaio-devel openssl-devel expat-devel liboath-devel lttng-ust-devel libbabeltrace-devel python36-Cython fuse-devel libnl3-devel librabbitmq-devel libcap-ng-devel gperf librabbitmq-devel librdkafka-devel -y`
            * 继续报错：`Could NOT find zbc`


* 中文文档：[Ceph Documentation ](http://docs.ceph.org.cn/start/intro/)
    - 跟英文文档有些出入，貌似一些内容已经out了，建议通过中文文档大致了解后，后续深入再读英文文档(主要是一些概念)
* 英文文档：[Ceph Documentation](https://ceph.readthedocs.io/en/latest/start/intro/)
* Ceph简介
    - 所有 Ceph 存储集群的部署都始于部署一个个 `Ceph 节点`、`网络`和 `Ceph 存储集群`
        + `Ceph 存储集群`至少需要一个 `Ceph Monitor`、`Ceph Manager` 和 `Ceph OSD`(`Object Storage Daemon`守护进程)
            + 运行 Ceph 文件系统客户端时，还必须要有`元数据服务器`（`Ceph Metadata Server`）
        + `Monitors`
            * Ceph Monitor(`ceph-mon`)维护着*集群状态*的各种图表(map)，包括监视器图、管理器图、OSD图、MDS图、和 CRUSH 图，这些图(map)是Ceph守护程序相互协调所必需的关键集群状态
            * Monitor还负责守护进程和客户端的认证
            * 通常至少需要**三个**Monitor才能实现冗余和高可用
        + `Managers`
            * Ceph Manager守护进程(`ceph-mgr`)负责跟踪运行时指标和当前Ceph集群的状态，包括存储利用率、当前性能指标和系统负载
            * Manager守护进程还管理着一个基于Python的模块，用来管理和展示Ceph集群信息，包括一个Ceph仪表盘和REST API
            * 通常至少需要**两个**Manager才能实现高可用
        + `Ceph OSDs`
            * Ceph OSD(object storage daemon, `ceph-osd`)用来*存储数据*，处理数据的复制、恢复、再平衡，并通过检查其他OSD守护进程的心跳来向 `Ceph Monitors`和`Managers` 提供一些监控信息
            * 通常至少需要**三个**Ceph OSDs才能实现冗余和高可用
        + `MDSs`
            * Ceph Metadata Server（MDS, `ceph-mds`）为 Ceph 文件系统存储*元数据*(metadata)（也就是说，Ceph 块设备和 Ceph 对象存储不使用MDS ）
            * Ceph Metadata Server 允许 POSIX 文件系统的用户们，可以在不对Ceph存储集群造成负担的前提下，执行诸如`ls`、`find` 等基本命令
    - Ceph 把数据作为`对象`存储在逻辑存储池中。通过`CRUSH`算法，Ceph 可以计算出哪个归置组（`placement group`，PG）应该包含指定的对象(Object)，然后进一步计算出哪个Ceph OSD守护进程应该存储这个placement group。
        + CRUSH 算法使得 Ceph 存储集群能够动态地伸缩、再均衡和修复
* 硬件推荐
    - [硬件推荐](http://docs.ceph.org.cn/start/hardware-recommendations/)
    - 里面涉及CPU、RAM内存、硬盘、固态硬盘(SSD)、网络等的要求和注意事项
* 安装
    - [INSTALLING CEPH](https://ceph.readthedocs.io/en/latest/install/#installing-ceph)
    - 有很多种不同的方式来安装ceph，选择最适合自己的方法
        + 推荐方式：
            * `Cephadm`
                - `Cephadm`使用容器和`systemd`安装并管理一个Ceph集群，将`CLI`命令行接口(`command-line interface`)和GUI仪表盘紧密集成在一起
                    + `cephadm`只支持`Octopus`(版本代号，对应版本`v15.2.0` Octopus released)和更高的版本
                    + `cephadm`与新的业务流程API完全集成，并完全支持新的CLI和仪表板功能来管理集群部署
                    + `cephadm`需要容器支持(`podman`或者`docker`)和`Python3`
            * `Rook`
                - 部署和管理在Kubernetes中运行的Ceph群集，同时还支持通过Kubernetes API进行存储资源管理和配置
                - 推荐使用`Rook`的方式来在Kubernetes中运行`Ceph`，或者用来将一个已存在的Ceph存储集群连接到Kubernetes
                    + `Rook`只支持`Nautilus`(对应版本14.2.0)和更新的版本
        + 其他方式
            * `ceph-ansible` 使用`Ansible`自动化运维工具来部署和管理Ceph集群
                - `ceph-ansible`不集成新的编排API(orchestrator APIs，Nautlius和Octopus版本引入)，意味着新的管理特性和仪表盘集成不可用
            * `ceph-deploy` 一个用来快速部署集群的工具
                - **注意**：`ceph-deploy`已经不再维护了，在比`Nautilus`更新的版本上没有测试过(中文文档还是这种方式)
                - 不支持`RHEL8`, `CentOS 8`或者更新的操作系统
            * 还可以手动安装
                - [BUILD CEPH](https://ceph.readthedocs.io/en/latest/install/build-ceph/)
                - 链接中说debug版本差不多需要40GB。。。
                - 编译源码前，执行`./install-deps.sh`来安装一些依赖包(不过自己执行报错了，参考上面记录的章节：`- 源码编译`)
    - 自己使用`cephadm`方式来安装
        + [CEPHADM](https://ceph.readthedocs.io/en/latest/cephadm/#cephadm)
        + Cephadm通过管理器守护程序，通过SSH连接到主机来部署和管理Ceph集群，以添加，删除或更新Ceph守护程序容器。它不依赖于诸如`Ansible`，`Rook`或`Salt`等外部配置或编排工具
        + 需要(任何现代Linux发行版都是足够的)：
            * `Systemd`
            * `Podman`或者`Docker`来运行容器
            * 时间同步(如`chrony`或者`NTP`)
            * LVM2用来提供存储设备
        + `cephadm`方式是先在一个节点上部署一个Ceph cluster，然后把其他节点加进来，再部署各种所需的服务
            * [CentOS8使用cephadm部署和配置Ceph Octopus](https://blog.csdn.net/get_set/article/details/108092248)
        + 安装`cephadm`
            * 用curl直接获取单机脚本
                - `curl --silent --remote-name --location https://github.com/ceph/ceph/raw/octopus/src/cephadm/cephadm`
                    + 可去掉静默执行，省得失败了不知道原因，去掉`--silent`
                    + 执行报错了：Failed to connect to raw.githubusercontent.com port 443: Connection refused
                    + 在[ipaddress](https://www.ipaddress.com/)中查询`raw.githubuercontent.com`的真实IP(本机查得`199.232.68.133`)，修改hosts，`vi /etc/hosts`，添加一行绑定host：`199.232.68.133 raw.githubusercontent.com`，再次执行`curl`即可
                    + 参考：[报错Failed to connect to raw.githubusercontent.com port](https://www.cnblogs.com/Dylansuns/p/12309847.html)
                - `chmod +x cephadm`
                - 使用方式：`./cephadm <arguments...>`
                - MacOS上运行(把docker启动起来，另外`cephadm`需要以root运行)：`sudo ./cephadm version`
                    + `ceph version 15.2.4 (7447c15c6ff58d7fce91843b705a268a1917325c) octopus (stable)`
                    + 当前最新稳定为15.2.4(日期20200903)，版本代号octopus
                    + 貌似自动pull了ceph镜像，`docker images`查看，可看到`ceph/ceph`镜像，大小为`979MB`
            * 把`cephadm`命令直接安装在服务器上更方便
                - 若要安装`Octopus`版本的`cephadm`：
                    + `./cephadm add-repo --release octopus`，
                    + `./cephadm install`
                - 确认是否安装成功：`which cephadm`
                    + 还是安装一下比较方便，本机(CentOS7)查看已安装：`/usr/sbin/cephadm`
            * 一些商业Linux发行版(如RHEL, SLE)可能已经包括了最新的Ceph包，这样可以直接安装`cephadm`
                - `dnf install -y cephadm` 或者 `zypper install -y cephadm`
        + 创建一个新的集群
            * (CentOS7执行) `cephadm bootstrap --mon-ip 192.168.50.207(主机ip)`
                - 首先部署的这个节点是一个MON，需要提供IP地址
                - 报错：hostname is a fully qualified domain name (localhost.localdomain)，提示需要不带`.`的简单形式或者传选项`--allow-fqdn-hostname`
                - `hostname`查看本机主机名称为localhost.localdomain
                - `hostname localhost` 修改主机名为更简单形式(不带`.`)
                    + 临时生效，若要一直生效则需修改`/etc/hosts`文件
            * 可参考别人博客的操作：[CentOS8使用cephadm部署和配置Ceph Octopus](https://blog.csdn.net/get_set/article/details/108092248)
            * 这个命令会：
                - 创建一个有MON和MGR的新cluster
                - 为Ceph cluster生成一个SSH key，并添加到`/root/.ssh/authorized_keys`
                - 生成一个最小化的`/etc/ceph/ceph.conf`配置文件
                - 为用户client.admin生成`/etc/ceph/ceph.client.admin.keyring`
        + 使用Ceph CLI(command-line interface)
            * cephadm不需要任何ceph包安装在服务器上，不过还是推荐使能一个简单访问的`ceph`命令，有几个方法使能：
            * `cephadm shell`
                - 会使用主机上的容器镜像包运行一个bash终端，其中包含所有ceph包
                    + `docker ps`查看可以看到一个名为`recursing_ishizaka`的ceph容器(使用镜像 ceph/ceph:v15)
                    + 退出容器就自动kill掉了
            * `cephadm shell -- ceph -s` 可以不进入容器执行命令(`ceph -s`为要执行的命令)
            * 也可以安装`ceph-common`包，其中包括`ceph`, `rbd`, `mount.ceph`等命令
                - `cephadm add-repo --release octopus`
                - `cephadm install ceph-common`
                - 本机安装后，`ceph -v`查看，安装成功
                    + ceph version 15.2.4 (7447c15c6ff58d7fce91843b705a268a1917325c) octopus (stable)
                    + 若当前终端输入`ceph`按tab没有找到位置，则重新加载一下`.bashrc`或者`.zshrc`

## Ceph源码

* 《Ceph源码分析》
* Ceph整体架构
    - 2012年，Ceph发布了第一个稳定版本
    - Ceph的设计目标是采用商用硬件（Commodity Hardware）来构建大规模的、具有高可用性、高可扩展性、高性能的分布式存储系统
        + 系统的高可用性指的是系统某个部件失效后，系统依然可以提供正常服务的能力
            * Ceph通过数据多副本、纠删码来提供数据的冗余
        + 高可扩展性是指系统可以灵活地应对集群的伸缩
            * 一方面指集群的容量可以伸缩，集群可以任意地添加和删除存储节点和存储设备；
            * 另一方面指系统的性能随集群的增加而线性增加
    - Ceph的整体架构大致如下：
        + 最底层基于`RADOS`（reliable, autonomous, distributed object store），它是一个可靠的、自组织的、可自动修复、自我管理的分布式对象存储系统
            * 其内部包括`ceph-osd`后台服务进程和`ceph-mon`监控进程
        + 中间层`librados`库用于本地或者远程通过网络访问`RADOS`对象存储系统
            * 它支持多种语言，目前支持C/C++语言、Java、Python、Ruby和PHP语言的接口
        + 最上层面向应用提供3种不同的存储接口：
            * 块存储接口，通过`librbd`库提供了块存储访问接口。它可以为虚拟机提供虚拟磁盘，或者通过内核映射为物理主机提供磁盘空间
            * 对象存储接口，目前提供了两种类型的API，一种是和AWS的`S3`接口兼容的API，另一种是和OpenStack的`Swift对象`接口兼容的API
            * 文件系统接口，目前提供两种接口，一种是标准的`posix`接口，另一种通过`libcephfs`库提供文件系统访问接口
    - Ceph客户端接口，包含三种形式的存储：*块存储*、*对象存储*、*文件系统*
        + (块存储) `RBD`（rados block device）是通过`librbd`库对应用提供*块存储*，主要面向云平台的虚拟机提供虚拟磁盘
            * 传统`SAN`就是块存储，通过SCSI或者FC接口给应用提供一个独立的LUN或者卷。
            * `RBD`类似于传统的`SAN`存储，都提供数据块级别的访问
            * 目前RBD提供了两个接口，一种是直接在用户态实现，通过QEMU Driver供KVM虚拟机使用。另一种是在操作系统内核态实现了一个内核模块
            * 块存储用作虚拟机的硬盘，其对I/O的要求和传统的物理硬盘类似。块存储既需要有较好的随机I/O，又要求有较好的顺序I/O，而且对延迟有比较严格的要求
        + (对象存储) `RadosGW`基于`librados`提供了和Amazon `S3`接口以及OpenStack `Swift`接口兼容的*对象存储*接口
            * 相比于NAS存储，对象存储放弃了目录树结构，采用了扁平化组织形式（一般为三级组织结构），这有利于实现近乎无限的容量扩展
            * 由于Amazon在云存储领域的影响力，Amazon的S3接口已经成为事实上的对象存储的标准接口
                - 其接口分三级存储：Account/Bucket/Object（账户/桶/对象）
            * 在云计算领域，OpenStack已经成为广泛采用的云计算管理系统，OpenStack的对象存储接口Swift也成为广泛采用的接口
                - 其也采用分三级存储：Account/Container/Object（账户/容器/对象），每层节点数均没有限制
        + (文件系统) `CephFS`通过在RADOS基础之上增加了MDS（MetadataServer）来提供*文件存储*
            * 它提供了`libcephfs`库和标准的`POSIX`文件接口。
            * CephFS类似于传统的`NAS`存储，通过NFS或者CIFS协议提供文件系统或者文件目录服务
    - RADOS
        + `RADOS`是Ceph存储系统的基石，是一个可扩展的、稳定的、自我管理的、自我修复的对象存储系统，是Ceph存储系统的核心
        + 它完成了一个存储系统的核心功能，包括：
            * `Monitor`模块为整个存储集群提供全局的配置和系统信息；
            * 通过`CRUSH`算法实现对象的寻址过程；
            * 完成对象的读写以及其他数据功能；
            * 提供了数据均衡功能；
            * 通过`Peering`过程完成一个`PG`内存达成数据一致性的过程；
            * 提供数据自动恢复的功能；
            * 提供克隆和快照功能；
            * 实现了对象分层存储的功能；
            * 实现了数据一致性检查工具`Scrub`
        + `Monitor`
            * Monitor是一个独立部署的daemon进程。通过组成Monitor集群来保证自己的高可用。Monitor集群通过`Paxos`算法实现了自己数据的一致性
            * Cluster Map保存了系统的全局信息，主要包括：
                - Monitor Map：包括集群的fsid、所有Monitor的地址和端口、current epoch
                - OSD Map：所有OSD的列表，和OSD的状态等
                - MDS Map：所有的MDS的列表和状态
        + 对象存储
            * 这里所说的对象是指`RADOS`对象，要和RadosGW的S3或者Swift接口的对象存储区分开来
            * 对象是数据存储的基本单元，一般默认`4MB`大小
            * 一个对象由三个部分组成：
                - 对象标志（ID），唯一标识一个对象
                - 对象的数据，其在本地文件系统中对应一个文件，对象的数据就保存在文件中
                - 对象的元数据，以Key-Value（键值对）的形式，可以保存在文件对应的扩展属性中
        + `pool`和`PG`的概念
            * pool是一个抽象的存储池。它规定了*数据冗余*的类型以及对应的*副本*分布策略
            * 一个pool由多个PG构成
            * `PG（placement group）`从名字可理解为一个放置策略组，它是对象的集合，该集合里的所有对象都具有相同的放置策略：对象的副本都分布在相同的OSD列表上
        + 对象寻址过程
            * 对象寻址过程指的是查找对象在集群中分布的位置信息，过程分为两步：
                - 1. 对象到PG的映射。这个过程是静态hash映射（加入pg split后实际变成了动态hash映射方式），通过对object_id，计算出hash值，用该pool的PG的总数量pg_num对hash值取模，就可以获得该对象所在的PG的id号
                - 2. PG到OSD列表映射。这是指PG上对象的副本如何分布在OSD上。它使用Ceph自己创新的`CRUSH`算法来实现，本质上是一个伪随机分布算法
        + 数据读写过程
            * 写过程
                - Client向该PG所在的主OSD发送写请求
                - 主OSD接收到写请求后，同时向两个从OSD发送写副本的请求，并同时写入主OSD的本地存储中
                - 主OSD接收到两个从OSD发送写成功的ACK应答，同时确认自己写成功，就向客户端返回写成功的ACK应答
            * 在写操作的过程中，主OSD必须等待所有的从OSD返回正确应答，才能向客户端返回写操作成功的应答
        + 数据均衡
            * 当在集群中新添加一个OSD存储设备时，整个集群会发生数据的迁移，使得数据分布达到均衡
            * Ceph数据迁移的*基本单位*是`PG`，即数据迁移是将PG中的所有对象作为一个整体来迁移
            * 迁移触发的流程为：
                - 当新加入一个OSD时，会改变系统的CRUSH Map，
                - 从而引起对象寻址过程中的第二步(PG到OSD的哈希变动了)，PG到OSD列表的映射发生了变化，从而引发数据的迁移。
        + `Peering`
            * 当OSD启动，或者某个OSD失效时，该OSD上的主PG会发起一个Peering的过程
            * Ceph的Peering过程是指一个PG内的所有副本通过PG日志来*达成数据一致*的过程
        + `Recovery`和`Backfill`
            * Ceph的Recovery过程是根据在Peering的过程中产生的、依据PG日志推算出的不一致对象列表来*修复*其他副本上的数据
            * 当某个OSD长时间失效后重新加入集群，它已经无法根据PG日志来修复，就需要执行Backfill（回填）过程。Backfill过程是通过逐一对比两个PG的对象列表来修复
        + 纠删码
            * `纠删码（Erasure Code）`的概念早在20世纪60年代就提出来了，最近几年被广泛应用在存储领域
            * 原理：将写入的数据分成N份原始数据块，通过这N份原始数据块计算出M份效验数据块，N+M份数据块可以分别保存在不同的设备或者节点中。可以允许最多M个数据块失效，通过N+M份中的任意N份数据，就还原出其他数据块
        + 快照和克隆
            * `快照（snapshot）`就是一个存储设备在某一时刻的全部*只读*镜像
            * `克隆（clone）`是在某一时刻的全部*可写*镜像
            * 快照和克隆的区别在于快照只能读，而克隆可写
            * RBD的克隆实现是在基于RBD的快照基础上，在客户端librbd上实现了`Copy-on-Write（cow）`(写时复制)克隆机制
        + `Cache Tier`
            * (Tier /tɪə/ 层；行、列)
            * RADOS实现了以pool为基础的*自动分层存储*机制。
                - 它在第一层可以设置cache pool，其为高速存储设备（例如SSD设备）。
                - 第二层为data pool，使用大容量低速存储设备（如HDD设备）可以使用EC模式来降低存储空间
            * 通过`Cache Tier`，可以提高关键数据或者热点数据的性能，同时降低存储开销
                - Cache Tier层为高速I/O层，保存热点数据，或称为活跃的数据
        + `Scrub`
            * Scrub机制用于系统检查数据的一致性。
            * 它通过在后台定期（默认每天一次）扫描，比较一个PG内的对象分别在其他OSD上的各个副本的元数据和数据来检查是否一致
    - 本章介绍了Ceph的系统架构，通过本章，可以对Ceph的基本架构和各个模块的组件有了整体的了解，并对一些基本概念及读写的原理、各个数据功能模块有了大致了解
* Ceph通用模块
    - 介绍Ceph源代码*通用库*中的一些比较关键而又比较复杂的*数据结构*
        + `Object`和`Buffer`相关的数据结构是普遍使用的
        + 线程池`ThreadPool`可以提高消息处理的并发能力
        + `Finisher`提供了异步操作时来执行回调函数
        + `Throttle`在系统的各个模块各个环节都可以看到，它用来限制系统的请求，避免瞬时大量突发请求对系统的冲击
        + `SafteTimer`提供了定时器，为超时和定时任务等提供了相应的机制
    - `Object`
        + 对象Object是默认为4MB大小的数据块。一个对象就对应本地文件系统中的一个文件
        + 在代码实现中，有`object`、`sobject`、`hobject`、`ghobject`等不同的类
            * 结构`object_t`对应本地文件系统的一个文件
                - `src/include/object.h`中定义(代码中的缩进风格都用的2个空格，Redis里是4个)
            * `sobject_t`在`object_t`之上增加了`snapshot`信息，用于标识是否是快照对象
                - 也在`src/include/object.h`中定义
            * `hobject_t`是hash object的缩写，其在`sobject_t`基础上增加了一些字段
                - `src/common/hobject.h`中定义
            * `ghobject_t`在对象`hobject_t`的基础上，添加了`generation`字段和`shard_id`字段，这个用于`ErasureCode`(纠删码)模式下的`PG`
                - 也在`src/common/hobject.h`中定义
    - `Buffer`
        + Buffer就是一个命名空间，在这个命名空间下定义了Buffer相关的数据结构，这些数据结构在Ceph的源代码中广泛使用
            * `buffer::raw`类是基础类，其子类完成了Buffer数据空间的分配
            * `buffer::ptr`类实现了Buffer内部的一段数据
            * `buffer::list`封装了多个数据段
        + `buffer::raw`
            * `src/include/buffer_raw.h`中定义
            * 类`buffer::raw`是一个原始的数据Buffer，在其基础之上添加了长度、引用计数和额外的crc校验信息
            * 下列类都*继承*了`buffer::raw`，实现了data对应内存空间的申请：
                - 类`raw_malloc`实现了用`malloc`函数分配内存空间的功能
                    + `src/common/buffer.cc`中定义
                - 类`class buffer::raw_mmap_pages`实现了通过mmap来把内存匿名映射到进程的地址空间
                    + 貌似不再使用了，nautilus.rst版本更新记录中：drop the unused buffer::raw_mmap_pages
                - 。。。还有其他几个继承raw的类
                - 类`class buffer::raw_char`使用了C++的new操作符来申请内存空间
                    + `src/common/buffer.cc`中定义
        + `buffer::ptr`
            * `src/include/buffer.h`中定义
            * 类buffer::ptr就是对于buffer::raw的一个部分数据段
        + `buffer::list`
            * `src/include/buffer.h`中定义
            * 类buffer::list是一个使用广泛的类，它是多个buffer::ptr的列表，也就是多个内存数据段的列表
    - 线程池(`ThreadPool`)
        + 线程池（ThreadPool）在分布式存储系统的实现中是必不可少的，在Ceph的代码中广泛用到
        + `src/common/WorkQueue.h`中定义(共享提交到多个工作队列的工作的线程池)
            * 注意：`src/crimson/os/alienstore/thread_pool.h`中也有一个`ThreadPool`定义，在命名空间`crimson::os`中：`crimson::os:ThreadPool`(用来调度来自seastar fibers的non-seastar任务，seastar是啥？)
