## 平均负载

[到底应该怎么理解“平均负载”？](https://time.geekbang.org/column/article/69618)

* 简单来说，平均负载是指单位时间内，系统处于*可运行状态*和*不可中断状态*的平均进程数，也就是平均活跃进程数，它和 CPU 使用率并没有直接关系。
    - 可运行状态的进程，是指正在使用 CPU 或者正在等待 CPU 的进程，也就是我们常用 ps 命令看到的，处于 R 状态（Running 或 Runnable）的进程。
    - 不可中断状态的进程则是正处于内核态关键流程中的进程，并且这些流程是不可打断的，比如最常见的是等待硬件设备的 I/O 响应，也就是我们在 ps 命令中看到的 D 状态（Uninterruptible Sleep，也称为 Disk Sleep）的进程。
        + 不可中断状态实际上是系统对进程和硬件设备的一种保护机制。

* `uptime`
    - `09:04:25 up 21 min,  4 users,  load average: 0.00, 0.03, 0.05`
    - 当前时间、系统运行时间以及正在登录用户数。 过去 1 分钟、5 分钟、15 分钟的平均负载（Load Average）。
    - 三个不同时间间隔的平均值，其实给我们提供了，分析系统负载**趋势**的数据来源，让我们能更全面、更立体地理解目前的负载状况
    - 一般当平均负载高于CPU数量*70%*的时候，你就应该分析排查负载高的问题了。一旦负载过高，就可能导致进程响应变慢，进而影响服务的正常功能。但70%这个数字并不是绝对的，最推荐的方法，还是把系统的平均负载监控起来，然后根据更多的历史数据，判断负载的变化趋势。

* 查看cpu个数(逻辑个数)
    - 查看逻辑CPU的个数(若该cpu支持超线程，则1个物理核对应2个逻辑核/线程。 若支持超线程则与物理核是两倍的关系)
        + `cat /proc/cpuinfo |grep "model name"`
        + 或 `cat /proc/cpuinfo |grep "processor"|wc -l`
    - 物理CPU个数
        + `cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l`
    - 查看CPU是几核心(物理核数，每个cpu的物理核数，若有多个物理cpu，核心数都一样就一条记录)
        + `cat /proc/cpuinfo |grep "cores"|uniq`

* 平均负载与 CPU 使用率
    - 由上面的定义：平均负载是指单位时间内，处于可运行状态和不可中断状态的进程数。 所以，平均负载不仅包括了正在使用CPU的进程，还包括**等待 CPU 和等待 I/O**的进程。
    - 而 **CPU 使用率**，是单位时间内 CPU 繁忙情况的统计，跟平均负载并**不一定完全对应**
        + *CPU 密集型进程*，使用大量 CPU 会导致平均负载升高，此时这两者是一致的；
        + *I/O 密集型进程*，等待 I/O 也会导致平均负载升高，但 CPU 使用率不一定很高；
        + 大量等待 CPU 的进程调度也会导致平均负载升高，此时的 CPU 使用率也会比较高

* 前置条件
    - 用`iostat`、`mpstat`、`pidstat`，找出平均负载升高的根源。
    - 自己的环境(与链接中不一样):
        + CentOS虚拟机 7.6.1810
        + 2个逻辑CPU、内存1.4G
    - 预先安装 stress 和 sysstat
        + sysstat(CentOS中自带了)包含了常用的 Linux 性能工具，用来监控和分析系统的性能。 本次用到其中的`mpstat` 和 `pidstat`
            * `sar`/`iostat`也包含其中，本案例暂时没用到
        + stress是一个Linux系统压力测试工具（本案例用作异常进程模拟平均负载升高的场景。）
            * 先`yum install -y epel-release`, 再 `yum install -y stress`
                - EPEL (Extra Packages for Enterprise Linux)是基于Fedora的一个项目，为“红帽系”的操作系统提供额外的软件包，适用于RHEL、CentOS和Scientific Linux.
                - 首先我们需要安装一个叫”epel-release”的软件包，这个软件包会自动配置yum的软件仓库。当然你也可以不安装这个包，自己配置软件仓库也是一样的。 软件仓库配置目录在：`/etc/yum.repos.d/`
            * 使用
                - `-t N` 或 `--timeout N`：运行秒数
                - `-c N` 或 `--cpu N`：产生多个处理sqrt()函数的CPU进程
                    + `stress -c 2 -t 10` 其两个进程测CPU，跑10s
                - `-i N` 或 `--io N`：产生多个处理sync()函数的磁盘I/O进程
                - `-m N` 或 `--vm N`：产生多个处理malloc()/free()内存分配函数的进程
                    + `--vm-bytes B`：指定内存的byte数为B，默认值是256MB
                    + `--vm-hang N`：指定malloc分配的内存多少秒后free()释放掉，默认不释放，0无效
                - `-d N` 或 `--hdd N`：产生多个处理write()/unlink()的进程
                    + `--hdd-bytes B`：指定每个hdd进程处理的byte字节数，默认1GB

* 案例
    - CPU密集型进程： CPU 使用率 100%
        + 在第一个终端运行 stress 命令，模拟一个 CPU 使用率 100% 的场景：`stress --cpu 1 --timeout 600`
        + 在第二个终端运行 uptime 查看平均负载的变化情况：`watch -n3 -d uptime`
        + 在第三个终端运行 mpstat 查看 CPU 使用率的变化情况：`mpstat -P ALL 5`
        + 用pidstat来查询是哪个进程导致了 CPU 使用率为 100%：`pidstat -u 5 1`
    - I/O密集型进程： 模拟 I/O 压力(不停执行sync)
        + `stress -i 1 --timeout 600`
        + `watch -d uptime`
        + `mpstat -P ALL 5 1`
        + `pidstat -u 5 1`
    - 大量进程场景
        + 当系统中运行进程超出 CPU 运行能力时，就会出现等待 CPU 的进程
        + 