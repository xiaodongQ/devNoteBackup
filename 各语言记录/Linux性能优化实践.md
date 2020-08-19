[TOC]

# 性能优化

* 本笔记主要包含极客时间两个性能优化相关的专栏的学习和实践笔记
    - [Linux性能优化实战](https://time.geekbang.org/column/intro/140)
    - [系统性能调优必知必会](https://time.geekbang.org/column/intro/308)

# [Linux性能优化实战](https://time.geekbang.org/column/intro/140)

## CPU性能篇

### 平均负载

[02 到底应该怎么理解“平均负载”？](https://time.geekbang.org/column/article/69618)

* 简单来说，**平均负载**是指单位时间内，系统处于*可运行状态*和*不可中断状态*的平均进程数，也就是平均活跃进程数，它和 CPU 使用率并没有直接关系。
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

* 前置准备
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
        + `stress -i 1 --timeout 600`, 用`-d`起写文件进程对虚拟机压力比较明显
        + `watch -d uptime`
        + `mpstat -P ALL 5 1`
        + `pidstat -u 5 1`
    - 大量进程场景
        + 当系统中运行进程超出 CPU 运行能力时，就会出现等待 CPU 的进程
        + `stress -c 8`，超出系统CPU核数，CPU处于严重过载状态

### CPU 上下文切换

[03 经常说的 CPU 上下文切换是什么意思？](https://time.geekbang.org/column/article/69859)

* CPU 上下文
    - 在每个任务运行前，CPU 都需要知道任务从哪里加载、又从哪里开始运行，也就是说，需要系统事先帮它设置好 CPU 寄存器和程序计数器（Program Counter，PC）。
    - CPU 寄存器，是 CPU 内置的容量小、但速度极快的内存。而程序计数器，则是用来存储 CPU 正在执行的指令位置、或者即将执行的下一条指令位置。它们都是 CPU 在运行任何任务前，必须的依赖环境，因此也被叫做 **CPU 上下文**。

* CPU 的上下文切换
    - **CPU 上下文切换**，就是先把前一个任务的 CPU 上下文（也就是 CPU 寄存器和程序计数器）保存起来，然后加载新任务的上下文到这些寄存器和程序计数器，最后再跳转到程序计数器所指的新位置，运行新任务。
    - 根据任务的不同，CPU 的上下文切换就可以分为几个不同的场景，也就是进程上下文切换、线程上下文切换以及中断上下文切换。
    - 进程上下文切换
        + Linux 按照特权等级，把进程的运行空间分为内核空间(对应CPU特权等级Ring 0)和用户空间(对应CPU特权等级Ring 3)， 进程在用户空间运行时，被称为进程的用户态，而陷入内核空间的时候，被称为进程的内核态。 从用户态到内核态的转变，需要通过系统调用来完成
        + 系统调用结束后，CPU 寄存器需要恢复原来保存的用户态，然后再切换到用户空间，继续运行进程
        + 所以一次系统调用的过程，其实是发生了两次 CPU 上下文切换(需要注意的是，系统调用过程中，并不会涉及到虚拟内存等进程用户态的资源，也不会切换进程。这跟我们通常所说的进程上下文切换是不一样的)，系统调用过程通常称为特权模式切换，而不是上下文切换
        + **进程的上下文切换**就比系统调用时多了一步：在保存当前进程的内核状态和 CPU 寄存器之前，需要先把该进程的*虚拟内存、栈等*保存下来；而加载了下一进程的内核态后，还需要刷新进程的虚拟内存和用户栈。
        + 根据[Tsuna](https://blog.tsunanet.net/2010/11/how-long-does-it-take-to-make-context.html)的测试报告，每次上下文切换都需要*几十纳秒到数微秒*的 CPU 时间。在进程上下文切换次数较多的情况下，很容易导致 CPU 将大量时间耗费在*寄存器、内核栈以及虚拟内存等资源的保存和恢复*上，进而大大缩短了真正运行进程的时间。这也正是导致平均负载升高的一个重要因素。
        + Linux 通过 TLB（Translation Lookaside Buffer）来管理虚拟内存到物理内存的映射关系
            * 转译后备缓冲器，也被翻译为页表缓存、转址旁路缓存，为CPU的一种缓存，由存储器管理单元用于改进虚拟地址到物理地址的转译速度。
            * 用处：其搜索关键字为虚拟内存地址，其搜索结果为物理地址。如果请求的虚拟地址在TLB中存在，CAM 将给出一个非常快速的匹配结果，之后就可以使用得到的物理地址访问存储器。如果请求的虚拟地址不在 TLB 中，就会使用标签页表进行虚实地址转换，而标签页表的访问速度比TLB慢很多。
            * 过程：当cpu要访问一个虚拟地址/线性地址时，CPU会首先根据虚拟地址的高20位（20是x86特定的，不同架构有不同的值）在TLB中查找。如果是表中没有相应的表项，称为TLB miss(不命中)，需要通过访问慢速RAM中的页表*计算*出相应的物理地址。同时，物理地址被*存放*在一个TLB表项中，*以后*对同一线性地址的访问，直接从TLB表项中获取物理地址即可，称为TLB hit(命中)。
            * [转译后备缓冲区](https://baike.baidu.com/item/%E8%BD%AC%E8%AF%91%E5%90%8E%E5%A4%87%E7%BC%93%E5%86%B2%E5%8C%BA/22685572?fromtitle=TLB&fromid=2339981)
            * [TLB的作用及工作过程](https://www.cnblogs.com/alantu2018/p/9000777.html)
        + 进程上下文切换潜在的性能问题: 当虚拟内存更新后，TLB 也需要刷新，内存的访问也会随之变慢。特别是在多处理器系统上，缓存是被多个处理器共享的，刷新缓存不仅会影响当前处理器的进程，还会影响共享缓存的其他处理器的进程。
    - 线程上下文切换
        + 线程与进程最大的区别在于，线程是调度的基本单位，而进程则是资源拥有的基本单位
        + 线程的上下文切换其实就可以分为两种情况：
            * 第一种， 前后两个线程属于不同进程。此时，因为资源不共享，所以切换过程就跟进程上下文切换是一样。
            * 第二种，前后两个线程属于同一个进程。此时，因为*虚拟内存是共享的*，所以在切换时，虚拟内存这些资源就保持不动，*只需要切换线程的私有数据、寄存器等*不共享的数据。
    - 中断上下文切换
        + 为了快速响应硬件的事件，中断处理会打断进程的正常调度和执行，转而调用中断处理程序，响应设备事件。
        + 跟进程上下文不同，中断上下文切换并不涉及到进程的用户态
        + 中断上下文，其实只包括内核态中断服务程序执行所必需的状态，包括 CPU 寄存器、内核堆栈、硬件中断参数等。
        + **对同一个 CPU 来说，中断处理比进程拥有更高的优先级**，所以*中断上下文切换并不会与进程上下文切换同时发生*。

* 进程调度的场景
    - 为了保证所有进程可以得到公平调度，CPU时间被划分为一段段的时间片，这些时间片再被轮流分配给各个进程。当某个进程的时间片耗尽了，就会被系统挂起，切换到其它正在等待 CPU 的进程运行。
    - 进程在系统资源不足（比如内存不足）时，要等到资源满足后才可以运行，这个时候进程也会被挂起，并由系统调度其他进程运行。
    - 当进程通过睡眠函数 *sleep* 这样的方法将自己*主动挂起*时，自然也会重新调度。
    - 当有优先级更高的进程运行时，为了保证高优先级进程的运行，当前进程会被挂起，由高优先级进程来运行
    - 发生硬件中断时，CPU 上的进程会被中断挂起，转而执行内核中的中断服务程序。

* 前置准备
    - [CPU 上下文切换（下）](https://time.geekbang.org/column/article/70077)
    - 使用 vmstat 这个工具，来查询系统的上下文切换情况。
    - vmstat 是一个常用的系统性能分析工具，主要用来分析系统的内存使用情况，报告虚拟内存的统计信息，也常用来分析 CPU 上下文切换和中断的次数。
    - vmstat  对系统的进程情况、内存使用情况、交换页和I/O块使用情况、中断以及CPU使用情况进行统计并报告相应的信息。
    - 需要特别关注的四列内容：
        + cs（context switch）是每秒上下文切换的次数
        + in（interrupt）则是每秒中断的次数
        + r（Running or Runnable）是就绪队列的长度，也就是正在运行和等待 CPU 的进程数。
        + b（Blocked）则是处于不可中断睡眠状态的进程数。
    - vmstat 只给出了系统总体的上下文切换情况，要想查看每个进程的详细情况，就需要使用 `pidstat` 了
        + 加上 -w 选项 `pidstat -w 2`
        + 两列内容是我们的重点关注对象
            * 一个是 cswch ，表示每秒自愿上下文切换（voluntary context switches）的次数
                - **自愿上下文切换**，是指进程无法获取所需资源，导致的上下文切换。
                - 比如说， I/O、内存等系统资源不足时，就会发生自愿上下文切换。
            * 另一个则是 nvcswch ，表示每秒非自愿上下文切换（non voluntary context switches）的次数。
                - **非自愿上下文切换**，则是指进程由于时间片已到等原因，被系统强制调度，进而发生的上下文切换。
                - 比如说，大量进程都在争抢 CPU 时，就容易发生非自愿上下文切换。

* `vmstat`结果示例:

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 9  0      0 672340   2116 561728    0    0     0     0 2742  266 99  1  0  0  0
 8  0      0 672204   2116 561736    0    0     0     0 2754  254 99  1  0  0  0
```

* `pidstat -w 2`结果示例：

```
15时28分58秒   UID       PID   cswch/s nvcswch/s  Command
15时29分00秒     0         9      4.48      0.00  rcu_sched
15时29分00秒     0        14      0.50      0.00  ksoftirqd/1
```

* 使用 sysbench 来模拟系统多线程调度切换的情况
    - sysbench 是一个**多线程**的基准测试工具(stress是多进程)，一般用来评估不同系统参数下的数据库负载情况。
    - centos root下安装: `yum install sysbench`
    - 模拟系统多线程调度的瓶颈
        + `sysbench --threads=10 --max-time=300 threads run` 以10个线程运行5分钟的基准测试，模拟多线程切换的问题
        + 对比下面的结果：
            * cs 列的上下文切换次数从之前的 37 骤然上升到了 200 万
            * r 列：就绪队列的长度已经到了 8，远远超过了系统 CPU 的个数 2，所以肯定会有大量的 CPU 竞争。
            * us（user）和 sy（system）列：这两列的 CPU 使用率加起来上升到了 100%，其中系统 CPU 使用率，也就是 sy 列高达 80%，说明 CPU 主要是被内核占用了。
            * in 列：中断次数也上升到了 5 万左右，说明*中断处理*也是个潜在的问题
        + 结论：系统的就绪队列过长，也就是正在运行和等待 CPU 的进程数过多，导致了大量的上下文切换，而上下文切换又导致了系统 CPU 的占用率升高。
        + 到底是什么进程导致了这些问题呢？使用`pidstat`检查： `pidstat -w -u 1` (查看上下文切换和CPU使用情况)
            * 可以看到是sysbench进程导致CPU使用率升高(%usr:39.30, %system:158.81，两者相加已经要200%了，共2个逻辑CPU)
        + 疑问：pidstat 输出的上下文切换次数，加起来也就几百，比 vmstat 的 139 万明显小了太多。这是怎么回事呢？
            * pidstat 默认显示进程的指标数据，加上 -t 参数后，才会输出线程的指标。(**不加-t则不统计线程，注意**)
            * `pidstat -t -u -w 1` 执行，可以看到 10个sysbench线程(__开头)的自愿和非自愿上下文切换都很高(都接近上十万)
    - 前面在观察系统指标时，除了上下文切换频率骤然升高，还有一个指标也有很大的变化：**中断次数**
        + 中断只发生在内核态
        + 而 pidstat 只是一个进程的性能分析工具，并不提供任何关于中断的详细信息，怎样才能知道中断发生的类型呢？
            * 从 /proc/interrupts 这个只读文件中读取。定义了各种中断
            * /proc 实际上是 Linux 的一个虚拟文件系统，用于内核空间与用户空间之间的通信。
            * /proc/interrupts 就是这种通信机制的一部分，提供了一个只读的中断使用情况。
            * `watch -d cat /proc/interrupts` 观察一段时间，你可以发现，
                - 变化速度比较快的是重调度中断（RES Rescheduling interrupts），这个中断类型表示，唤醒空闲状态的 CPU 来调度新的任务运行。 和 LOC, Local timer interrupts
        + 但当上下文切换次数超过一万次，或者切换次数出现数量级的增长时，就很可能已经出现了性能问题。这时，你还需要根据上下文切换的类型，再做具体分析。比方说：
            * 自愿上下文切换变多了，说明进程都在等待资源，有可能发生了 I/O 等其他问题；
            * 非自愿上下文切换变多了，说明进程都在被强制调度，也就是都在争抢 CPU，说明 CPU 的确成了瓶颈；
            * 中断次数变多了，说明 CPU 被中断处理程序占用，还需要通过查看 /proc/interrupts 文件来分析具体的中断类型。

* 空闲状态`vmstat 1`结果：

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 427988   2116 831828    0    0    14    16  807  820 21  2 77  0  0
 0  0      0 427988   2116 831828    0    0     0     0   34   37  0  0 100  0  0
```

* sysbench后的`vmstat 1`结果：

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 9  0      0 424868   2116 831880    0    0     0     0 53718 2002245 19 81  0  0  0
 8  0      0 424868   2116 831880    0    0     0     0 49898 2096424 20 79  1  0  0
 9  0      0 424868   2116 831880    0    0     0     0 50355 2080560 20 79  1  0  0
```

### CPU使用率

[05 基础篇：某个应用的CPU使用率居然达到100%，我该怎么办？](https://time.geekbang.org/column/article/70476)

* CPU使用率
    - 作为最常用也是最熟悉的 CPU 指标，你能说出 CPU 使用率到底是**怎么算出来的**吗？
    - 为了维护 CPU 时间，Linux 通过事先定义的节拍率（内核中表示为 HZ），触发时间中断，并使用全局变量 *Jiffies* 记录了开机以来的节拍数。每发生一次时间中断，Jiffies 的值就加 1。
    - 节拍率 HZ 是*内核的可配选项*，可以设置为 100、250、1000 等。不同的系统可能设置不同数值，你可以通过查询 /boot/config 内核选项来查看它的配置值。 (`CONFIG_HZ` 配置项，看自己的CentOS环境里：CONFIG_HZ=1000)
    - 正因为节拍率 HZ 是内核选项，所以用户空间程序并不能直接访问。为了方便用户空间程序，内核还提供了一个用户空间节拍率 USER_HZ，它总是固定为 100，也就是 1/100 秒
    - **Linux 通过 /proc 虚拟文件系统，向用户空间提供了系统内部状态的信息**，而 `/proc/stat` 提供的就是系统的 CPU 和任务统计信息。
        + `man 5 proc` 查看proc具体目录文件结构和说明
    - 我们通常所说的 CPU 使用率，就是除了空闲时间外的其他时间占总 CPU 时间的百分比，
        + `CPU使用率 = 1 - (空闲时间/总CPU时间)`
        + 根据这个公式，我们就可以从 /proc/stat 中的数据，很容易地计算出每个场景的 CPU 使用率
        + 这是开机以来的节拍数累加值，所以直接算出来的，是*开机以来*的平均 CPU 使用率，*一般没啥参考价值*。
    - 事实上，为了计算 CPU 使用率，性能工具一般都会取间隔一段时间（比如 3 秒）的两次值，作差后，再计算出这段时间内的平均 CPU 使用率
        + `平均CPU使用率 = 1 - ((空闲时间new-空闲时间old)/(总CPU时间new-总CPU时间old))`
        + 这个公式，就是我们用各种性能工具所看到的 CPU 使用率的*实际*计算方法。
    - 性能分析工具给出的都是间隔一段时间的平均 CPU 使用率，所以要注意间隔时间的设置，特别是用多个工具对比分析时，你一定要保证它们用的是相同的间隔时间。

* 分析进程的 CPU 问题
    - 通过 top、ps、pidstat 等工具，你能够轻松找到 CPU 使用率较高（比如 100% ）的进程。接下来，你可能又想知道，占用 CPU 的到底是代码里的哪个函数呢？
    - gdb?
        + 我猜你第一个想到的，应该是 GDB（The GNU Project Debugger）， 这个功能强大的程序调试利器。的确，GDB 在调试程序错误方面很强大。但是，我又要来“挑刺”了。请你记住，GDB 并不适合在性能分析的早期应用。
        + 因为 GDB 调试程序的过程会中断程序运行，这在线上环境往往是不允许的
        + pstack/gstack 是一个shell脚本，里面使用gdb bt和sed过滤统计一些程序调用栈，pstack是指向gstack的软连接
    - `perf`
        + 那么哪种工具适合在第一时间分析进程的 CPU 问题呢？我的推荐是 perf。perf 是 Linux 2.6.31 以后内置的性能分析工具
        + 使用 perf 分析 CPU 性能问题，两种最常见的用法(root用户下使用)
            * 第一种常见用法是 `perf top`
                - 类似于 top，它能够实时显示占用 CPU 时钟最多的函数或者指令，因此可以用来查找热点函数
                - 第一行包含三个数据，分别是采样数（Samples, CPU 时钟事件）、事件类型（event）和事件总数量（Event count） 采样数需要我们特别注意。如果采样数过少（比如只有十几个），那下面的排序和百分比就没什么实际参考价值了。
                - 下面表格式样的数据每一行包含四列
                    + 第一列 Overhead ，是该符号的性能事件在所有采样中的比例，用百分比来表示。
                    + 第二列 Shared ，是该函数或指令所在的动态共享对象（Dynamic Shared Object），如内核、进程名、动态链接库名、内核模块名等。
                    + 第三列 Object ，是动态共享对象的类型。比如 [.] 表示用户空间的可执行程序、或者动态链接库，而 [k] 则表示内核空间。
                    + 最后一列 Symbol 是符号名，也就是函数名。当函数名未知时，用十六进制的地址来表示。
            * 第二种常见用法 `perf record` 和 `perf report`
                - `perf top` 虽然实时展示了系统的性能信息，但它的缺点是并不保存数据，也就无法用于离线或者后续的分析。
                - 而 `perf record` 则提供了保存数据的功能，保存后的数据，需要你用 `perf report` 解析展示
                - `perf record [-g]`，一段时间后按Ctrl+C终止采样； 再执行`perf report`，展示类似于perf top的报告
                    + 在实际使用中，我们还经常为其加上-g参数，开启调用关系的采样，方便我们根据调用链来分析性能问题。
        + 其他案例遇到的问题：`perf report` 的输出中，只有 swapper 显示了调用栈，其他所有符号都不能查看堆栈情况
            * 执行 `man perf-report` 命令，找到 -g 参数的说明
            * 通过这个说明可以看到，-g 选项等同于 --call-graph，它的参数是后面那些被逗号隔开的选项，意思分别是输出类型、最小阈值、输出限制、排序方法、排序关键词、分支以及值的类型。
            * 可以看到：默认的参数是 graph,0.5,caller,function,percent
            * 堆栈显示不全，相关的参数当然就是最小阈值 `threshold`
                - threshold 的默认值为 0.5%，也就是说，事件比例超过 0.5% 时，调用栈才能被显示。再观察案例应用 app 的事件比例，只有 0.34%，低于 0.5%，所以看不到 app 的调用栈就很正常了。
            * 这种情况下，你只需要给 perf report 设置一个小于 0.34% 的阈值，就可以显示我们想看到的调用图了。
                - `perf report -g graph,0.3`
        + 其他案例：swapper 高达 99% 的比例，分析案例时，我们直接忽略了前面这个 99% 的符号，转而分析后面只有 0.3% 的 app。
            * 实际上， swapper 跟 SWAP分区 没有任何关系，它只在系统初始化时创建 init 进程，之后，它就成了一个最低优先级的空闲任务。也就是说，当 CPU 上没有其他任务运行时，就会执行 swapper 。所以，可以称它为“空闲任务”。
            * 因为在多任务系统中，次数多的事件，不一定就是性能瓶颈。所以，只观察到一个大数值，并不能说明什么问题。具体有没有瓶颈，还需要观测多个方面的多个指标，来交叉验证


* perf top执行示例1 (注意Shared 和 Object 是不同的两列, [.]是Object列的内容)：

```
Samples: 3M of event 'cpu-clock', Event count (approx.): 11007468281
Overhead  Shared Object                       Symbol
  23.49%  testServer                      [.] std::_List_iterator<testStruct>::operator++
  10.80%  testServer                      [.] std::_List_iterator<testStruct>::operator!=
```

* perf top执行示例2：

```
Samples: 817K of event 'cpu-clock', Event count (approx.): 5959404592
Overhead  Shared Object                 Symbol
  10.96%  [unknown]                 [.] 0x000000000059f1fa
   7.63%  [kernel]                  [k] __do_softirq
   3.96%  [unknown]                 [.] 0x000000000059f1f7
   2.38%  [unknown]                 [.] 0x000000000059ccf7
```

* 以 Nginx + PHP 的 Web 服务为例
    - 前置准备： 预先安装 docker、sysstat、perf、ab 等工具
        + docker安装(CentOS)
            * 查看是否已经安装过docker `yum list installed | grep docker`
            * 若安装过要重新安装：`yum remove –y docker.xxx` (xxx含docker.x86_64、docker-client.x86_64、docker-common.x86_64)
            * 安装：`yum install docker` (可加-y选项，安装过程提示都默认选yes)
            * `docker -v`, (自测CentOS环境)执行得到 Docker version 1.13.1, build 7f2769b/1.13.1
            * 启动docker：`service docker start`
                - make build构建docker镜像需要先启动docker
                - 否则报错："Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"
        + ab
            * ab（apache bench）是一个常用的 HTTP 服务性能测试工具，这里用来模拟 Ngnix 的客户端。
            * CentOS下安装：`yum -y install httpd-tools`
                - 除了`ab`外，http压测工具还可以选择`wrk`/`locust`/`Jmeter`，`wrk`比`ab`功能更强大(使用了一些操作系统特定的高性能I/O机制, 比如 select, epoll, kqueue 等。其实它是复用了 redis 的 `ae 异步事件驱动框架`)
                    + `wrk`可参考：[性能测试工具 wrk 安装与使用](https://www.cnblogs.com/savorboard/p/wrk.html)
        + Nginx 和 PHP环境
            * 配置比较麻烦，使用参考链接中的docker镜像：[linux-perf-examples](https://github.com/feiskyer/linux-perf-examples/tree/master/nginx-high-cpu)
    - 操作和分析
        + 打开两个终端，分别 SSH 登录到两台机器上
        + 首先，在第一个终端a执行下面的命令来运行 Nginx 和 PHP 应用：
            * `docker run --name nginx -p 10000:80 -itd feisky/nginx`
            * `docker run --name phpfpm -itd --network container:nginx feisky/php-fpm`
        + 然后，在第二个终端b使用 curl 访问 http://[VM1 的 IP]:10000，确认 Nginx 已正常启动。
            * `curl http://192.168.50.118:10000` 结果响应：It works!
        + 接着，测试一下这个 Nginx 服务的性能。在第二个终端运行下面的 ab 命令：
            * `ab -c 10 -n 100 http://192.168.50.118:10000/` 结果如下
        + 分析
            * 在b执行新的 `ab -c 10 -n 10000 http://192.168.50.118:10000/`
            * 在a找到 php-fpm 进程号(有多个，找一个分析)
                - `perf top -g -p 19318(其中一个php-fpm的进程号)`
            * 在容器外perf top只能看到十六进制地址，所以先在外面保存perf信息，再放到容器中分析
                - 参考：[CPU使用率达到100% 精选留言](https://time.geekbang.org/column/article/70476)
                - `perf record -g -p 19402` 运行一段时间后打断，生成 perf.data 文件，拷贝到容器中，在容器中进行分析
                - `docker cp perf.data phpfpm:/tmp` 拷贝到容器phpfpm
                - `docker exec -i -t phpfpm bash`   进入容器bash终端
                - `cd /tmp/`, 安装perf工具：`apt-get update && apt-get install -y linux-perf linux-tools procps`
                - 分析perf.data, `perf_4.9 report`，可以看到最终到了 `sqrt` 和 `add_function`
                    + 注意：最后运行的工具名字是容器内部安装的版本 perf_4.9，而不是 perf 命令，这是因为 perf 会去跟内核的版本进行匹配，但镜像里面安装的perf版本有可能跟虚拟机的内核版本不一致。
                - 拷贝出 Nginx 应用的源码
                    + `docker cp phpfmp:/app .`
                    + 查找源码中的函数调用 `grep sqrt -r app/` ，找到sqrt调用发现是测试代码未删除而一直在循环调用所以比较慢
                - 停止原来的应用`docker rm -f nginx phpfpm`，运行优化后的应用
                    + `docker run --name nginx -p 10000:80 -itd feisky/nginx:cpu-fix`
                    + `docker run --name phpfpm -itd --network container:nginx feisky/php-fpm:cpu-fix`
        + 在案例结束时，不要忘了清理环境，执行下面的 Docker 命令，停止案例中用到的 Nginx 进程：
            * `docker rm -f nginx phpfpm` (不停止则下次`docker run`会报错，"The container name "/nginx" is already in use by container")

* `ab -c 10 -n 10000 http://192.168.50.118:10000/`运行结果(中途打断了，执行2703个请求)：

```
[root@localhost build]# ab -c 10 -n 100 http://192.168.50.118:10000/
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.50.118 (be patient)...
..done


Server Software:        nginx/1.15.4
Server Hostname:        192.168.50.118
Server Port:            10000

Document Path:          /
Document Length:        9 bytes

Concurrency Level:      10
Time taken for tests:   172.423 seconds
Complete requests:      2703
Failed requests:        0
Write errors:           0
Total transferred:      464916 bytes
HTML transferred:       24327 bytes
Requests per second:    15.68 [#/sec] (mean)
Time per request:       637.895 [ms] (mean)
Time per request:       63.789 [ms] (mean, across all concurrent requests)
Transfer rate:          2.63 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   1.4      0      17
Processing:   126  635  82.9    639     910
Waiting:      125  635  82.9    639     910
Total:        126  636  82.9    639     910

Percentage of the requests served within a certain time (ms)
  50%    639
  66%    676
  75%    697
  80%    708
  90%    737
  95%    762
  98%    792
  99%    817
 100%    910 (longest request)
```

### 案例篇：CPU 使用率很高却找不到高CPU的应用

* [06 系统的 CPU 使用率很高，但为啥却找不到高 CPU 的应用？](https://time.geekbang.org/column/article/70822)
    - 回顾前面的内容，系统的 CPU 使用率，不仅包括进程用户态和内核态的运行，还包括`中断处理`、`等待 I/O` 以及`内核线程`等。所以，当你发现系统的 CPU 使用率很高的时候，`不一定`能找到相对应的高 CPU 使用率的`进程`。
    - 使用案例镜像
        + `docker run --name nginx -p 10000:80 -itd feisky/nginx:sp`
        + `docker run --name phpfpm -itd --network container:nginx feisky/php-fpm:sp`
    - 检查Nginx服务是否正常启动 `curl http://192.168.50.118:10000/`
    - `ab -c 5 -t 600 http://192.168.50.118:10000/` 并发5，时间600s
    - `top` 可以看到`R`状态的进程并不是`php-fpm`(此时是`S`状态)，而是`stress` (此时pidstat看并没有CPU很高的单个进程)
    - 最后问题是：进程的 PID 在变，这些进程都是`短时进程`或`崩溃重启`
* 排查此类问题
    - 进程的 PID 一直在变，一般两个原因：
        + 第一个原因，进程在不停地崩溃重启，比如因为段错误、配置错误等等，这时，进程在退出后可能又被监控系统自动重启了。
        + 第二个原因，这些进程都是短时进程，也就是在其他应用内部通过 `exec` 调用的外面命令。很难用 `top`这种间隔时间比较长的工具发现(上面是碰巧发现)
    - 要想继续分析下去，还得找到它们的父进程
        + `pstree` 可以用树状形式显示所有进程之间的关系
    - `execsnoop`
        + 这个案例中，我们使用了 top、pidstat、pstree 等工具分析了系统 CPU 使用率高的问题，并发现 CPU 升高是短时进程 stress 导致的，但是整个分析过程还是比较复杂的。对于这类问题，有没有更好的方法监控呢？
        + `execsnoop` 就是一个专为短时进程设计的工具。它通过 ftrace 实时监控进程的 exec() 行为，并输出短时进程的基本信息，包括进程 PID、父进程 PID、命令行参数以及执行的结果。
        + execsnoop 所用的 `ftrace` 是一种常用的动态追踪技术，一般用于分析 Linux 内核的运行时行为
        + 安装：
        + 包含在项目(GitHub)：[perf-tools](https://github.com/brendangregg/perf-tools)中
            * 其中的工具都是一系列脚本，`git clone` 或 `wget` 下来后执行即可(没有权限则chmod +x)
                - 其中`execsnoop`位置在[execsnoop](https://github.com/brendangregg/perf-tools/blob/master/execsnoop)
            * 下载所有脚本：`git clone --depth 1 https://github.com/brendangregg/perf-tools`
            * 下载单个脚本：`wget https://raw.githubusercontent.com/brendangregg/perf-tools/master/execsnoop`
        + `./execsnoop`或添加环境变量直接执行即可，执行结果中的列也很简单：`PID   PPID ARGS`
        + 关于`perf-tools`的说明
            * 作者是 `Brendan Gregg`，讲火焰图的时候还讲到他
                - [开发相关笔记记录](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%BC%80%E5%8F%91%E7%9B%B8%E5%85%B3%E7%AC%94%E8%AE%B0%E8%AE%B0%E5%BD%95.md)中搜`火焰图`，关于"Brendan Gregg"：[Brendan Gregg: 一个实战派大神](https://book.douban.com/review/7894012/)
            * `perf-tools`是一些用于Linux ftrace和perf_events的正在开发和不受支持的性能分析工具的杂项集合
                - 这些集合使用起来很简单：做一件事并把它做好

### 案例篇：系统中出现大量不可中断进程和僵尸进程怎么办

* [案例篇：系统中出现大量不可中断进程和僵尸进程怎么办](https://time.geekbang.org/column/article/71064)

* 进程状态回顾
    - 可以查看之前的记录(搜` ps`)：[开发相关笔记记录](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%BC%80%E5%8F%91%E7%9B%B8%E5%85%B3%E7%AC%94%E8%AE%B0%E8%AE%B0%E5%BD%95.md)
    - S(PROCESS STATE CODES): 进程状态
        + R    running or runnable (on run queue)，运行或者可运行状态
        + D    (Disk Sleep)uninterruptible sleep (usually IO)，不可中断睡眠状态(一般在跟硬件交互，最常见的是等待硬件设备的 I/O 响应)
        + S    interruptible sleep (waiting for an event to complete)，可中断的的sleep
        + Z    defunct ("zombie") process, terminated but not reaped by its parent，僵尸进程
        + T    stopped by job control signal，任务控制信号中止
        + t    stopped by debugger during the tracing
        + X    dead (should never be seen)，进程消亡，不会看到进程的该状态
        + W    paging (not valid since the 2.6.xx kernel)，2.6内核版本之后无效
* 操作分析
    - `docker run --privileged --name=app -itd feisky/app:iowait` 运行案例
    - `ps aux | grep /app`
        + root 4009 0.0 0.0 4376 1008 pts/0 `Ss+` 05:51 0:00 /app
        + root 4287 0.6 0.4 37280 33660 pts/0 `D+` 05:54 0:00 /app
        + `S` 表示可中断睡眠状态，`D` 表示不可中断睡眠状态
        + 而`s`表示这个进程是一个会话的领导进程；`+`表示前台进程组
            * `进程组`表示一组相互关联的进程，比如每个子进程都是父进程所在组的成员；
            * `会话`是指共享同一个控制终端的一个或多个进程组。
                - 终端和会话，可以查看之前的记录(搜` 进程组`)，其中涉及`SIGHUP`信号导致关闭终端会杀死进程的问题：[开发相关笔记记录](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%BC%80%E5%8F%91%E7%9B%B8%E5%85%B3%E7%AC%94%E8%AE%B0%E8%AE%B0%E5%BD%95.md)
    - `top` 分析
        + 过去 1 分钟、5 分钟和 15 分钟内的平均负载在依次减小，说明平均负载正在升高
        + 而 1 分钟内的平均负载已经达到系统的 CPU 个数，说明系统很可能已经有了性能瓶颈
        + 僵尸进程比较多，说明有子进程在退出时没被清理
        + iowait(`wa`) 分别是 60.5% 和 94.6%，有点不正常
            * 对与io比较高的问题，考虑是否有利用`零拷贝`原理的场景来减少拷贝次数和上下文切换，参考之前的记录(搜`零拷贝`)：[C++笔记](https://github.com/xiaodongQ/devNoteBackup/blob/master/各语言记录/C%2B%2B.md)
    - perf record跟踪调用堆栈，发现`sys_read()`直接读，绕过了系统缓存，这解释了`iowait`高的问题
        + `open(disk, O_RDONLY|O_DIRECT|O_LARGEFILE, 0755)`，去掉O_DIRECT
        + 示例中给出了源码(C程序)链接：[app.c](https://github.com/feiskyer/linux-perf-examples/blob/master/high-iowait-process/app.c)
    - 对于僵尸进程，用`pstree -aps 某个僵尸进程` 找到其父进程进行分析

top结果，参考链接中的示例(本地虚拟机环境跑得卡死，敲不了命令)：：

```sh
# 按下数字 1 切换到所有 CPU 的使用情况，观察一会儿按 Ctrl+C 结束
$ top
top - 05:56:23 up 17 days, 16:45,  2 users,  load average: 2.00, 1.68, 1.39
Tasks: 247 total,   1 running,  79 sleeping,   0 stopped, 115 zombie
%Cpu0  :  0.0 us,  0.7 sy,  0.0 ni, 38.9 id, 60.5 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.7 sy,  0.0 ni,  4.7 id, 94.6 wa,  0.0 hi,  0.0 si,  0.0 st
...

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 4340 root      20   0   44676   4048   3432 R   0.3  0.0   0:00.05 top
 4345 root      20   0   37280  33624    860 D   0.3  0.0   0:00.01 app
 4344 root      20   0   37280  33624    860 D   0.3  0.4   0:00.01 app
    1 root      20   0  160072   9416   6752 S   0.0  0.1   0:38.59 systemd
...
```

* 原链接中的留言摘录
    - 发现留言中很多有意义的思考和实践，也有一些踩到的坑和经验讨论
    - 生产系统出问题，“逻辑cpu是120的实体机，跑oracle数据库，业务刚接进来就卡死了，一分钟平均负载200多，运行队列也两百多，cpu的usr部分四五十，sys部分四十左右，iowait基本是零，idle个位数，top和pid stat都无法输出，perf top也无法，只能dstat”
        + 参考思路：
            * “这种情况应该要适当中断一下应用，让这些基本的操作可以正常执行后重新运行。” (先部分恢复基本操作环境，再使用工具分析)
            * “推荐把这些系统和进程的指标监控起来，避免应用已启动又没法操作了。” (防患于未然，监控起来后续触发时有排查环境，类似的有C/C++程序段错误时通过`backtrace`保存用户态的调用堆栈)
            * “还有就是预留几个CPU，专门用作运维、监控使用。这需要修改内核选项” (待查，技术债)
        + 思考：最近使用`zabbix`监控进程和日志报警，正好能用于这些监控；
    - 留言中一个排查I/O问题的实例(iostat查看到io使用率太高而CPU使用率并不多)：[服务器运维」如何解决服务器I/O过高的问题](https://www.jianshu.com/p/d7edbff7dda6)
        + top查看到`wa`磁盘iowait有点高(38.1%)，然后用`iostat`查看发现磁盘sda使用率99.24%明显有瓶颈，`iostat`、`dstat`查找具体占用IO高的进程，发现是`jdb2/sda1-8`(这个进程以前也碰到过，pg数据库写得比较频繁时，该进程占用很高的io)
            * jbd2的全称是journaling block driver，这个进程实现的是文件系统的日志功能，磁盘使用日志功能来保证数据的完整性。 需要评估一下安全和性能哪个更重要。jdb2的特点就是牺牲了性能保证了数据完整性
        + 实例中的问题和解决：“服务器上似乎运行了许多IO操作很频繁的程序，jdb2的特点就是牺牲了性能保证了数据完整性，也就是说是我运行的程序太多让jdb2忙不过来了。”，“因此我的最终解决方案就是，用kill把所有当前运行的高IO程序都干掉。最后解决了问题。”
        + 思考：对于业务中就是需要频繁IO的程序逻辑(数据库，先考虑减少数据库操作频率，批量操作)，不能直接kill，那就要评估性能和完整性的取舍了。

### 软中断

* [09 | 基础篇：怎么理解Linux软中断？](https://time.geekbang.org/column/article/71868)
* [系统的软中断CPU使用率升高，我该怎么办？](https://time.geekbang.org/column/article/72147)

* 软中断
    - 背景引言
        + 进程的不可中断状态是系统的一种保护机制，可以保证硬件的交互过程不被意外打断。所以，短时间的不可中断状态是很正常的。但是，当进程长时间都处于不可中断状态时，你就得当心了。
        + 其实除了 iowait，`软中断（softirq）`CPU 使用率升高也是最常见的一种性能问题
        + 中断其实是一种异步的事件处理机制，可以提高系统的并发处理能力。
    - 为了解决中断处理程序执行过长和中断丢失的问题，Linux 将中断处理过程分成了两个阶段，也就是上半部和下半部：
        + 上半部用来快速处理中断，它在中断禁止模式下运行，主要处理跟硬件紧密相关的或时间敏感的工作。
        + 下半部用来延迟处理上半部未完成的工作，通常以`内核线程`的方式运行。
    - 这两个阶段你也可以这样理解：
        + 上半部直接处理`硬件请求`，也就是我们常说的`硬中断`，特点是`快速`执行；
        + 而下半部则是`由内核触发`，也就是我们常说的`软中断`，特点是`延迟`执行。
            * `每个CPU`(逻辑CPU)都对应一个软中断内核线程，名字为 **“ksoftirqd/CPU 编号”**，比如说， 0 号 CPU 对应的软中断内核线程的名字就是 ksoftirqd/0。
            * 可`ps aux|grep ksoftirqd`查看
                - 注意：这些线程的名字外面都有`中括号`(`[]`)，这说明 ps `无法获取它们的命令行参数（cmline）`。*一般来说，ps 的输出中，名字括在中括号里的，一般都是内核线程。*
    - 查看软中断和内核线程
        + `/proc/softirqs` 提供了软中断的运行情况；特别注意以下这两点：
            * 第一，要注意软中断的类型，也就是这个界面中第一列的内容。(看文件中有10个软中断类型)
                - `TASKLET`类型的软中断 在不同 CPU 上的分布并不均匀。TASKLET 是最常用的软中断实现机制，每个 TASKLET 只运行一次就会结束 ，并且只在调用它的函数所在的 CPU 上运行。
            * 第二，要注意同一种软中断在不同 CPU 上的分布情况
        + `/proc/interrupts` 提供了硬中断的运行情况。
    - 当软中断事件的频率过高时，内核线程也会因为CPU使用率过高而导致软中断处理不及时，进而引发网络收发延迟、调度缓慢等性能问题。
    - 案例(sar、 hping3 和 tcpdump)
        + `docker run -itd --name=nginx -p 80:80 nginx`
            * `curl http://192.168.50.118` 检查Nginx服务是否正常启动
        + 使用 `hping3` 来模拟 Nginx 的客户端请求
            * `hping3`说明
                - hping是用于生成和解析TCP/IP协议数据包的开源工具。创作者是Salvatore Sanfilippo。目前最新版是hping3。
                - 安装(CentOS)：`yum install -y hping3`，安装后会有`hping`、`hping2`、`hping3`
                - 使用可参考：[hping3命令](https://man.linuxde.net/hping3)
            * 发送请求：`hping3 -S -p 80 -i u100 192.168.50.118` (这是一个 `SYN FLOOD` 攻击)
                - -S参数表示设置TCP协议的SYN（同步序列号）
                - -p表示目的端口为80
                - -i u100表示每隔100微秒发送一个网络帧
                - *如果在实践过程中现象不明显，可以尝试把100调小，比如调成10甚至1*，个人虚拟机调整为`5微秒`，服务端有卡顿
                - [什么是SYN Flood攻击?](https://www.cnblogs.com/popduke/p/5823801.html)
                    + SYN Flood (SYN洪水) 是种典型的DoS (Denial of Service，拒绝服务) 攻击。效果就是服务器TCP连接资源耗尽，停止响应正常的TCP连接请求。
            * `top` 中可以看到 `ksoftirqd/0` 进程占用的CPU较高(实验的虚拟机里到了100%)
                - 说明软中断比较有问题
            * 查看软中断变化`/proc/softirqs`
                - `watch -n1 -d "cat /proc/softirqs"`
                - 其中数据是系统运行以来的累积中断次数
                - 需要关注中断次数的变化速率(watch)，其中，`NET_RX`，也就是网络数据包接收软中断的变化速率最快
            * 继续分析。既然是网络接收的软中断，第一步应该就是观察系统的网络接收情况。 使用`sar`
                - `sar`不仅可以观察网络收发的吞吐量（BPS，每秒收发的字节数），还可以观察网络收发的 PPS，即每秒收发的网络帧数。
                - `sar -n DEV 1`
                    + `rxpck/s` 和 `txpck/s`：表示每秒接收、发送的网络帧数，也就是 PPS
                    + `rxkB/s` 和 `txkB/s`：表示每秒接收、发送的千字节数，也就是 BPS
                - 简单计算(enp0s3网卡)每个包的大小(收到的字节数/包的数量)： `4567.47 * 1000 / 77949.70 = 58.59` 字节(Byte)
                    + 这显然是很小的网络帧，也就是我们通常所说的小包问题
            * 排查包是从哪里发来的，使用 `tcpdump` 抓取 对应网卡(本环境是enp0s3) 上的包就可以了
                - `tcpdump -i enp0s3 -nn tcp port 80`
                    + `-nn` 不解析协议名和主机名、以及端口 (链接中-n只是不解析主机名，并没有显示端口80，而是"http")
                    + man手册中`-n 不要把 地址 转换成 名字 (指的是 主机地址, 端口号等)`，可能是版本问题，-n并没有覆盖端口
                    + [如何使tcpdump显示ip和端口号但不显示主机名和协议](如何使tcpdump显示ip和端口号但不显示主机名和协议)
                        * -n 只适用于主机名(不解析主机名)，但不适用于端口号
                - 可以看到是192.168.50.231向本机(.118)的80端口发送SYN包
    - 卡顿问题分析
        + 终端卡顿的问题，这个是由于网络延迟增大（甚至是丢包）导致的。
        + hping3大量发包，导致其他网络连接延迟，ssh通过网络连接，使ssh客户端感觉卡顿现象。

* `sar -n DEV 1`执行：

```
平均时间:     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
平均时间:    enp0s3  77949.70  52858.48   4567.47   3104.13      0.00      0.00      0.40      3.74
平均时间:        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
平均时间: virbr0-nic      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

* `tcpdump -i enp0s3 -n tcp port 80`结果：

```
15:00:23.774628 IP 192.168.50.231.4806 > 192.168.50.118.80: Flags [S], seq 912643938, win 512, length 0
15:00:23.774636 IP 192.168.50.231.4805 > 192.168.50.118.80: Flags [S], seq 1533383952, win 512, length 0
15:00:23.774642 IP 192.168.50.231.4804 > 192.168.50.118.80: Flags [R], seq 641221179, win 0, length 0
15:00:23.774683 IP 192.168.50.118.80 > 192.168.50.231.4806: Flags [S.], seq 414633220, ack 912643939, win 29200, options [mss 1460], length 0
```

### 如何迅速分析出系统CPU的瓶颈在哪里？

* [11 | 套路篇：如何迅速分析出系统CPU的瓶颈在哪里？](https://time.geekbang.org/column/article/72685)
* [12 | 套路篇：CPU 性能优化的几个思路](https://time.geekbang.org/column/article/73151)

* CPU 性能指标
    - 最容易想到的应该是 `CPU 使用率`
        + 用户 CPU 使用率
        + 系统 CPU 使用率(不包括中断)
        + 等待 I/O 的 CPU 使用率，通常也称为 `iowait`，表示等待 I/O 的时间百分比。
            * 注意：top 中的`iowait` (top中显示为：`wa`)，和pidstat中的`%wait`是不同的两个指标
                - pidstat 中， %wait 表示进程等待 CPU 的时间百分比。
                    + "Percentage of CPU spent by the task while waiting to run"
                - top 中 ，iowait% 则表示等待 I/O 的 CPU 时间百分比。
                    + "wa, IO-wait : time waiting for I/O completion"
                    + iowait高，通常说明系统与硬件设备的 I/O 交互时间比较长。
        + 软中断和硬中断的 CPU 使用率
        + 虚拟化环境中的 窃取 CPU 使用率（`steal`）和 客户 CPU 使用率（`guest`）
    - 第二个比较容易想到的，应该是`平均负载（Load Average）`
        + 理想情况下，平均负载等于逻辑 CPU 个数，这表示每个 CPU 都恰好被充分利用。如果平均负载大于逻辑 CPU 个数，就表示负载比较重了。
    - 第三个，也是在专栏学习前你估计不太会注意到的，`进程上下文切换`
        + 无法获取资源而导致的自愿上下文切换；
        + 被系统强制调度导致的非自愿上下文切换。
        + 上下文切换，本身是保证 Linux 正常运行的一项核心功能。但过多的上下文切换，会将原本运行进程的 CPU 时间，消耗在寄存器、内核栈以及虚拟内存等数据的保存和恢复上，缩短进程真正运行的时间，成为性能瓶颈。
    - 还有一个指标，`CPU 缓存的命中率`
        + 由于 CPU 发展的速度远快于内存的发展，CPU 的处理速度就比内存的访问速度快得多。
        + CPU 缓存的速度介于 CPU 和内存之间，缓存的是热点的内存数据
        + 这些缓存按照大小不同分为 L1、L2、L3 等三级缓存
            * 从 L1 到 L3，三级缓存的大小依次增大，相应的，性能依次降低（当然比内存还是好得多）。
            * 而它们的命中率，衡量的是 CPU 缓存的复用情况，命中率越高，则表示性能越好
* 性能工具
    - 说到性能工具，就不得不提性能领域的大师布伦丹·格雷格（Brendan Gregg）。他不仅是动态追踪工具 DTrace 的作者，还开发了许许多多的性能工具。在各种 Linux 性能优化的文章中，基本都能看到他的那张性能工具图谱。
        + [Brendan D. Gregg个人网站](http://www.brendangregg.com/)
        + 特别是网站中Linux Performance页面：[Linux Performance](http://www.brendangregg.com/linuxperf.html)
    - `uptime`查看系统的平均负载
    - `mpstat` 和 `pidstat` ，分别观察每个 CPU 和每个进程 CPU 的使用情况
    - `vmstat` 查看了系统的上下文切换次数和中断次数；然后通过 `pidstat`，观察进程的自愿上下文切换和非自愿上下文切换情况
    - `top` 查看系统和进程的CPU使用情况，再用`perf top` 观察进程的调用链
    - 通过 `perf record` 和 `perf report`，发现短时进程问题，`execsnoop`可以实时监控进程调用的外部命令。
    - 不可中断进程和僵尸进程的案例中，`top`观察到iowait升高的问题，并发现了大量的不可中断进程和僵尸进程。`dstat`发现是这是由磁盘读导致的，`pidstat`找到相关进程，再用`perf`分析调用链
    - 软中断的案例中，`top`观察到系统的软中断CPU使用率升高，接着查`/proc/softirqs`中变化快的软中断，然后通过`sar`发现是网络小包问题，最后再用 `tcpdump`抓包找出网络帧类型和来源
* 如何迅速分析 CPU 的性能瓶颈
    - 缩小排查范围，先运行几个支持指标较多的工具，如 `top`、`vmstat` 和 `pidstat`
        + 这三个命令，几乎包含了所有重要的 CPU 性能指标
    - 而找出导致性能问题的进程后，就要用进程分析工具来分析进程的行为
        + 比如使用 `strace` 分析系统调用情况
        + 以及使用 `perf` 分析调用链中各级函数的执行情况
        + 平均负载升高，可以跟 `vmstat` 输出的运行状态和不可中断状态的进程数做对比
        + 如果是不可中断进程数增多了(此时iowait会较高)，那么就需要做 I/O 的分析，也就是用 `dstat` 或 `sar` 等工具，进一步分析 I/O 的情况

* CPU 性能优化的几个思路
    - 动手之前，你可以先看看下面这三个问题。
        + a 首先，既然要做性能优化，那要`怎么判断它是不是有效`呢？特别是优化后，`到底能提升多少性能`呢？
        + b 第二，性能问题通常不是独立的，如果有`多个`性能问题同时发生，你`应该先优化哪一个`呢？
        + c 第三，提升性能的方法`并不是唯一`的，当有多种方法可以选择时，你会`选用哪一种`呢？是不是总选那个最大程度提升性能的方法就行了呢？
    - a 评估性能优化的效果，不要局限在单一维度的指标上
        + 至少要从`应用程序`和`系统资源`这两个维度，分别选择不同的指标
            * 好的应用程序是性能优化的最终目的和结果，系统优化总是为应用程序服务的
            * 系统资源的使用情况是影响应用程序性能的根源。所以，需要用系统资源的指标，来观察和分析瓶颈的来源。
        + 在进行性能测试时，有两个特别重要的地方你需要注意下。
            * 第一，要避免性能测试工具干扰应用程序的性能。
                - 通常，对 Web 应用来说，性能测试工具跟目标应用程序要在不同的机器上运行。
            * 第二，避免外部环境的变化影响性能指标的评估。
                - 这要求优化前、后的应用程序，都运行在相同配置的机器上，并且它们的外部依赖也要完全一致。
    - b 并不是所有的性能问题都值得优化
        + 在性能测试的领域，流传很广的一个说法是“二八原则”，也就是说 80% 的问题都是由 20% 的代码导致的。只要找出这 20% 的位置，你就可以优化 80% 的性能
        + 两个可以简化这个过程的方法。
            * 第一，如果发现是系统资源达到了瓶颈，比如 CPU 使用率达到了 100%，那么首先优化的一定是系统资源使用问题
            * 第二，针对不同类型的指标，首先去优化那些由瓶颈导致的，性能指标变化幅度最大的问题。
                - 比如产生瓶颈后，用户 CPU 使用率升高了 10%，而系统 CPU 使用率却升高了 50%，这个时候就应该首先优化系统 CPU 的使用。
    - c 性能优化并非没有成本
        + 不要想着“一步登天”，试图一次性解决所有问题；也不要只会“拿来主义”，把其他应用的优化方法原封不动拿来用，却不经过任何思考和分析。
* CPU优化
    - 应用程序优化
        + 从应用程序的角度来说，降低CPU使用率的最好方法当然是，排除所有不必要的工作，只保留最核心的逻辑。比如减少循环的层次、减少递归、`减少动态内存分配`等等。
        + 除此之外，应用程序的性能优化也包括很多种方法，这里列出最常见的几种
            * 编译器优化
                - 很多编译器都会提供优化选项，适当开启它们，在编译阶段你就可以获得编译器的帮助，来提升性能。比如， gcc 就提供了优化选项 -O2，开启后会自动对应用程序的代码进行优化。
            * 算法优化：使用复杂度更低的算法
            * 异步处理：使用异步处理，可以避免程序因为等待某个资源而一直阻塞，从而提升程序的并发处理能力。比如，把轮询替换为事件通知，就可以避免轮询耗费 CPU 的问题。
            * 多线程代替多进程：相对于进程的上下文切换，线程的上下文切换并不切换进程地址空间，因此可以降低上下文切换的成本
            * 善用缓存：经常访问的数据或者计算过程中的步骤，可以放到内存中缓存起来
    - 系统优化
        + 一方面要充分利用 CPU 缓存的本地性
        + 另一方面，就是要控制进程的 CPU 使用情况，减少进程间的相互影响。
        + 列举最常见的一些方法
            * `CPU 绑定`：把进程绑定到一个或者多个 CPU 上，可以提高 CPU 缓存的命中率，减少跨 CPU 调度带来的上下文切换问题。
            * `CPU 独占`：跟 CPU绑定类似，进一步将CPU分组，并通过CPU亲和性机制为其分配进程。
                - 这些CPU就由指定的进程独占，不允许其他进程再来使用这些 CPU。
            * 优先级调整：使用 nice 调整进程的优先级，正值调低优先级，负值调高优先级。
                - 适当降低非核心应用的优先级，增高核心应用的优先级，可以确保核心应用得到优先处理。
            * 为进程设置资源限制：使用 Linux cgroups 来设置进程的 CPU 使用上限
            * NUMA（Non-Uniform Memory Access）优化：NUMA优化其实就是让 CPU 尽可能只访问本地内存。
            * `中断负载均衡`：无论是软中断还是硬中断，它们的中断处理程序都可能会耗费大量的 CPU
                - 开启 irqbalance 服务或者配置 smp_affinity，就可以把中断处理过程自动负载均衡到多个 CPU 上。
    - `千万避免过早优化`

## 内存性能篇

* Linux管理内存
    - 我们通常所说的内存容量是指`物理内存`(也称主存)。大多主存都是DRAM(动态随机访问内存)。
    - 只有内核能直接访问物理内存。
    - Linux内核给每个进程都提供了一个独立的虚拟地址空间，并且这个地址空间是连续的。
        + 虚拟地址空间内部分为`内核空间`和`用户空间`两部分。
        + 32位系统的内核空间占用1G，用户空间3G；64位系统内核空间和用户空间都是128T
    - 进程在用户态时，只能访问用户空间内存；只有进入内核态后才可以访问内核空间内存。

---

# [系统性能调优必知必会](https://time.geekbang.org/column/intro/308)

## 基础设施优化

* [01 | CPU缓存：怎样写代码能够让CPU执行得更快？](https://time.geekbang.org/column/article/230194)
    - 缓存的原理使用，之前的笔记中也做过说明(搜 `利用CPU cache特性优化Go程序`)：[Go开发相关笔记.md](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%90%84%E8%AF%AD%E8%A8%80%E8%AE%B0%E5%BD%95/Go%E5%BC%80%E5%8F%91%E7%9B%B8%E5%85%B3%E7%AC%94%E8%AE%B0.md)
    - CPU 的多级缓存
        + CPU 缓存的材质 SRAM 比内存使用的 DRAM 贵许多，所以不同于内存动辄以`GB`计算，它的大小是以`MB`来计算的
            * `lscpu`查看自己的Linux机器，L1 32KB，L2 256KB，L3 9216KB
            * 有2个一级缓存(自己机器上显示为L1d 缓存和L1i 缓存，都是32KB)，这是因为，CPU 会区别对待指令与数据。比如，“1+1=2”这个运算，“+”就是指令，会放在`一级指令缓存`中，而“1”这个输入数字，则放在`一级数据缓存`中
        + 当下的 CPU 都是多核心的，每个核心都有自己的一、二级缓存，但三级缓存却是一颗 CPU 上所有核心共享的
        + 程序执行时，会先将内存中的数据载入到共享的三级缓存中，再进入每颗核心独有的二级缓存，最后进入最快的一级缓存，之后才会被 CPU 使用
        + CPU 访问一次内存通常需要 100 个时钟周期以上，而访问一级缓存只需要 4~5 个时钟周期，二级缓存大约 12 个时钟周期，三级缓存大约 30 个时钟周期
            * 2.4GHz频率则需要 1/2.4 = 0.42ns
        + 如果 CPU所要操作的数据在缓存中，则直接读取，这称为缓存命中。命中缓存会带来很大的性能提升，因此，我们的代码优化目标是提升 CPU 缓存的命中率
    - 提升数据缓存的命中率
        + 课程代码可clone下来：[russelltao/geektime_distrib_perf](https://github.com/russelltao/geektime_distrib_perf)
            * fork到了自己的github，便于自己的一些注释学习和实践：[xiaodongQ/geektime_distrib_perf](https://github.com/xiaodongQ/geektime_distrib_perf)
        + 示例：[traverse_1d_array.cpp](https://github.com/xiaodongQ/geektime_distrib_perf/tree/master/1-cpu_cache/traverse_1d_array)
            * `./traverse_1d_array -s 1`    耗时50,access count:8388608
            * `./traverse_1d_array -s 128`  耗时390,access count:8388608
            * `./traverse_1d_array -s 1024` Out of memory。。。
            * 调整new出的数组大小为2GB（上面是new了8GB）
                - -s 1，   20,access count:2097152
                - -s 128， 80,access count:2097152
                - -s 1024，600,access count:2097152
            * 可以看到访问同样总数据量的数据，以不同间隔步长访问，相同次数，第一个是最快的
            * 使用perf验证缓存命中率(指定关注的事件)
                - 事件说明
                    + 缓存未命中 `cache-misses` 事件
                    + 读取缓存次数 `cache-references` 事件
                        * 两者相除(未命中/读取次数)就是缓存的未命中率(`#`后面已经算好了)，用 1 相减就是命中率
                    + `L1-dcache-load-misses`和`L1-dcache-loads` 可计算L1缓存的命中率(`1-未命中率`)
                        * 相除可以得到L1缓存的未命中率(`#`后面已经算好了)
                    + perf stat 还可以通过指令执行速度反映出两种访问方式的优劣
                        * `instructions` 事件指明了进程执行的总指令数，而`cycles`事件指明了运行的时钟周期，二者相除就可以得到每时钟周期所执行的指令数，缩写为`IPC`(`#`后面算好了)
                        * 如果缓存未命中，则 CPU 要等待内存的慢速读取，因此 `IPC` 就会很低
                - `perf stat -e cache-references,cache-misses,instructions,cycles,L1-dcache-load-misses,L1-dcache-loads ./traverse_1d_array -s 1`
                    + 216,681      cache-references
                    + 75,368       cache-misses            #  34.783 % of all cache refs *缓存未命中率很高？*
                    + 29,937,457   instructions            #  1.53  insn per cycle *每时钟周期执行指令数更多*
                    + 19,613,832   cycles
                    + 107,690      L1-dcache-load-misses   #  0.70% of all L1-dcache hits
                    + 15,367,502   L1-dcache-loads
                - `perf stat -e cache-references,cache-misses,instructions,cycles,L1-dcache-load-misses,L1-dcache-loads ./traverse_1d_array -s 128`
                    + 11,963,620      cache-references
                    + 5,064,344       cache-misses          #  42.331 % of all cache refs
                    + 36,122,526      instructions          #  0.33  insn per cycle
                    + 108,431,911     cycles
                    + 6,519,388       L1-dcache-load-misses #  38.82% of all L1-dcache hits *L1未命中率很高*
                    + 16,792,408      L1-dcache-loads
            * 可以看到步长1遍历时，时钟周期(cycles)最少，缓存和L1缓存命中率反而小？
        + 示例：[traverse_2d_array.cpp](https://github.com/xiaodongQ/geektime_distrib_perf/tree/master/1-cpu_cache/traverse_2d_array)
            * 时间
                - `./traverse_2d_array -f`，`arr[i][j]`方式访问，耗时10ms (有时0)
                - `./traverse_2d_array -s`，`arr[j][i]`方式访问，耗时40ms (和链接中的80ms略不同)
            * 使用perf验证缓存命中率(指定关注的事件)
                - `perf stat -e cache-references,cache-misses,instructions,cycles,L1-dcache-load-misses,L1-dcache-loads ./traverse_2d_array -f`
                    + 274,098      cache-references
                    + 124,717      cache-misses           #  45.501 % of all cache refs *缓存未命中率很高？*
                    + 54,549,109   instructions           #  1.45  insn per cycle
                    + 37,682,643   cycles
                    + 150,201      L1-dcache-load-misses  #  0.84% of all L1-dcache hits
                    + 17,894,603   L1-dcache-loads
                - `perf stat -e cache-references,cache-misses,instructions,cycles,L1-dcache-load-misses,L1-dcache-loads ./traverse_2d_array -s`
                    + 22,999,477   cache-references
                    + 317,376      cache-misses           #   1.380 % of all cache refs
                    + 54,673,902   instructions           #   0.38  insn per cycle
                    + 144,144,986  cycles
                    + 6,583,124    L1-dcache-load-misses  #   36.71% of all L1-dcache hits *L1未命中率很高*
                    + 17,934,431   L1-dcache-loads
            * 连续访问时更快，理论上相差8倍，与 `CPU Cache Line` 相关，它定义了缓存一次载入数据的大小，Linux 上可以通过 `coherency_line_size` 配置查看它，通常是 64 字节
                - 当载入 `array[0][0]`时，若它们占用的内存不足 64 字节，CPU 就会顺序地补足后续元素。顺序访问的 `array[i][j]`因为利用了这一特点，所以就会比 `array[j][i]`要快。
                - 也正因为这样，当元素类型是4个字节的整数时，性能就会比8个字节的高精度浮点数时速度更快，因为缓存一次载入的元素会更多
                - 为什么执行时间相差 8 倍
                    + 在二维数组中，其实第一维元素存放的是地址，第二维存放的才是目标元素。由于 64 位操作系统的地址占用 8 个字节（32 位操作系统是 4 个字节），因此，每批 Cache Line 最多也就能载入不到 8 个二维数组元素，所以性能差距大约接近 8 倍。
            * 因此，遇到这种遍历访问数组的情况时，按照内存布局顺序访问将会带来很大的性能提升。
        + 提升指令缓存的方法
            * 利用CPU的**分支预测器**
                - 当代码中出现 if、switch 等语句时，意味着此时至少可以选择跳转到两段不同的指令去执行。如果分支预测器可以预测接下来要在哪段代码执行（比如 if 还是 else 中的指令），就可以提前把这些指令放在缓存中，CPU 执行时就会很快
                - 当数组中的元素完全随机时，分支预测器无法有效工作，而当 array 数组有序时，分支预测器会动态地根据历史命中数据对未来进行预测，命中率就会非常高
                - 示例：[branch_predict.cpp](https://github.com/xiaodongQ/geektime_distrib_perf/tree/master/1-cpu_cache/branch_predict)
                    + `./branch_predict -s` 遍历随机数组，耗时820ms
                    + `./branch_predict -f` 遍历有序数组，耗时280ms
                    + 使用perf验证缓存命中率
                        * `perf stat -e instructions,cycles,L1-icache-load-misses,branch-load-misses,branch-loads ./branch_predict -s`
                            - 1,503,492,588   instructions   #  0.45  insn per cycle
                            - 67,148,018      branch-load-misses
                            - 273,725,452     branch-loads
                        * `perf stat -e instructions,cycles,L1-icache-load-misses,branch-load-misses,branch-loads ./branch_predict -f`
                            - 1,501,361,425   instructions   #  1.22  insn per cycle *每时钟周期执行指令数多*
                            - 30,483          branch-load-misses
                            - 273,342,843     branch-loads
                - C/C++ 语言中编译器还给应用程序员提供了显式预测分支概率的工具，如果 if 中的条件表达式判断为“真”的概率非常高，我们可以用`likely`宏把它括在里面，反之则可以用`unlikely`宏
                    + 当然，CPU 自身的条件预测已经非常准了，仅当我们确信CPU条件预测不会准，且我们能够知晓实际概率时，才需要加入这两个宏。
                    + `#define likely(x) __builtin_expect(!!(x), 1)`
                    + `#define unlikely(x) __builtin_expect(!!(x), 0)`
                    + `if (likely(a == 1)){xxx}`
            * 多核 CPU 下的缓存命中率
                - 当多线程同时执行密集计算，且 CPU 缓存命中率很高时，如果将每个线程分别绑定在不同的 CPU 核心上，性能便会获得非常可观的提升
                - 操作系统提供了将进程或者线程绑定到某一颗 CPU 上运行的能力
                    + Linux: `sched_setaffinity` (设置CPU亲和性)
                        * 之前笔记也记录了`CPU亲和性 (绑定CPU)`：[CPU亲和性 (绑定CPU)](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%90%84%E8%AF%AD%E8%A8%80%E8%AE%B0%E5%BD%95/C%2B%2B.md)
                    + API
                        * `#include <sched.h>`
                        * `int sched_setaffinity(pid_t pid, size_t cpusetsize, cpu_set_t *mask);`
                        * `int sched_getaffinity(pid_t pid, size_t cpusetsize, cpu_set_t *mask);`
                    + 创建的子进程和线程继承CPU亲和性
                - `perf`提供了`cpu-migrations`事件，显示进程从不同的 CPU 核心上迁移的次数
                - 示例：[cpu_migrate.cpp](https://github.com/xiaodongQ/geektime_distrib_perf/tree/master/1-cpu_cache/cpu_migrate)
                    + 使用3个（共6个CPU核心）并发线程测试，不绑定CPU
                        * `./cpu_migrate -t 3 -s`，耗时(ms) costsum: 1641, avg: 547
                    + 使用3个（共6个CPU核心）并发线程测试，绑定CPU
                        * `./cpu_migrate -t 3 -f`，耗时 costsum: 1626, avg: 542
                        * 差别不大的样子，有时还更久，加大循环的次数(循环再`*20`)，差别明显一些
                            - 不绑定：costsum: 48344, avg: 16114
                            - 绑定：  costsum: 48330, avg: 16110
                    + 使用perf验证缓存命中率
                        * `perf stat -e cpu-migrations,cache-references,cache-misses,instructions,cycles,L1-dcache-load-misses,L1-dcache-loads,L1-icache-load-misses,branch-load-misses,branch-loads ./cpu_migrate -t 3 -s`
                            - 148              cpu-migrations *CPU迁移次数多一些*
                            - 2,702,230        cache-misses              #   20.835 % of all cache refs
                            - 290,246,973,176  instructions              #    1.56  insn per cycle
                            - 7,268,628        L1-dcache-load-misses     #    0.00% of all L1-dcache hits
                        * `perf stat -e cpu-migrations,cache-references,cache-misses,instructions,cycles,L1-dcache-load-misses,L1-dcache-loads,L1-icache-load-misses,branch-load-misses,branch-loads ./cpu_migrate -t 3 -f`
                            - 122              cpu-migrations
                            - 2,321,395        cache-misses              #   18.190 % of all cache refs
                            - 290,240,522,792  instructions              #    1.56  insn per cycle
                            - 6,520,711        L1-dcache-load-misses     #    0.00% of all L1-dcache hits
        + 思考：多线程并行访问不同的变量，这些变量在内存布局是相邻的（比如类中的多个变量），此时CPU缓存就会失效，为什么？又该如何解决呢？
            * 一片连续的内存被加载到不同cpu核心中（就是同一个cache line在不同的cpu核心），其中一个cpu核心中修改cache line,其它核心都失效(伪共享)，加锁也是加在cache line上，其它核心线程也被锁住，降低了性能。解决办法是填充无用字节数，使得我们真正需要高频并发读写的不同变量，不出现在一个cache line中(尽量在不同核缓存)
                - Go里面的处理示例：(搜 `利用CPU cache特性优化Go程序`)：[Go开发相关笔记.md](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%90%84%E8%AF%AD%E8%A8%80%E8%AE%B0%E5%BD%95/Go%E5%BC%80%E5%8F%91%E7%9B%B8%E5%85%B3%E7%AC%94%E8%AE%B0.md)
* [02 | 内存池：如何提升内存分配的效率？](https://time.geekbang.org/column/article/230221)
    - [内存优化总结:ptmalloc、tcmalloc和jemalloc](http://www.cnhalo.net/2016/06/13/memory-optimize/)

* [09 | 如何提升TCP三次握手的性能？](https://time.geekbang.org/column/article/237612)
    - 三次握手在一个 HTTP 请求中的平均时间占比在 10% 以上，在网络状况不佳、高并发或者遭遇 SYN 泛洪攻击等场景中，如果不能正确地调整三次握手中的参数，就会对性能有很大的影响
    - TCP 协议是由操作系统实现的，调整 TCP 必须通过操作系统提供的接口和工具，这就需要理解 Linux 是怎样把三次握手中的状态暴露给我们，以及通过哪些工具可以找到优化依据，并通过哪些接口修改参数
    - 客户端的优化
        + 客户端和服务器都可以针对三次握手优化性能。相对而言，主动发起连接的客户端优化相对简单一些，而服务器需要在监听端口上被动等待连接，并保存许多握手的中间状态，优化方法更为复杂一些
        + 三次握手建立连接的首要目的是同步序列号
            * 只有同步了序列号才有可靠的传输，TCP协议的许多特性都是依赖序列号实现的，比如流量控制、消息丢失后的重发等等，这也是三次握手中的报文被称为 SYN 的原因，因为 SYN 的全称就叫做 `Synchronize Sequence Numbers`
        + 客户端发送`SYN`开启了三次握手
            * 此时在客户端上用`netstat`命令可以看到连接的状态是 `SYN_SENT`（顾名思义，就是把刚 SYN 发送出去）
        + 客户端在等待服务器回复的 `ACK` 报文。正常情况下，服务器会在*几毫秒*内返回 ACK，但如果客户端迟迟没有收到 ACK 会怎么样呢？客户端会重发 SYN，重试的次数由 `tcp_syn_retries` 参数控制，默认是 6 次
            * `sysctl -a|grep tcp_syn_retries`，*默认*就是：`net.ipv4.tcp_syn_retries = 6`
            * 第 1 次重试发生在 1 秒钟后，接着会以翻倍的方式在第 2、4、8、16、32 秒共做 6 次重试，最后一次重试会等待 64 秒(即发送最后一次SYN后等待64秒)，如果仍然没有返回 ACK，才会终止三次握手。所以，总耗时是 1+2+4+8+16+32+64=127 秒，超过 2 分钟
        + 如果这是一台有明确任务的服务器，你可以根据网络的稳定性和目标服务器的繁忙程度修改重试次数，调整客户端的三次握手时间上限
            * 比如内网中通讯时，就可以适当调低重试次数，尽快把错误暴露给应用程序
    - 服务器端的优化
        + 当服务器收到 SYN 报文后，服务器会立刻回复`SYN+ACK`报文，既确认了客户端的序列号，也把自己的序列号发给了对方
            * 此时，服务器端出现了新连接，状态是 `SYN_RCV`（RCV 是 received 的缩写）
            * 这个状态下，服务器必须建立一个*SYN半连接队列*来维护未完成的握手信息，当这个队列溢出后，服务器将无法再建立新连接
        + 新连接建立失败的原因有很多，`netstat -s` 命令给出的统计结果中可以得到由于队列已满而引发的失败次数
            * `netstat -s | grep "SYNs to LISTEN"` 给出的是队列溢出导致 SYN 被丢弃的个数
            * 注意这是一个累计值，如果数值在持续增加，则应该调大 SYN 半连接队列
        + 修改队列大小的方法，是设置 Linux 的 `tcp_max_syn_backlog` 参数：
            * `net.ipv4.tcp_max_syn_backlog = 1024`
            * `sysctl -a|grep tcp_max_syn_backlog`查看*默认*为：`net.ipv4.tcp_max_syn_backlog = 256`
        + 如果 *SYN 半连接队列*已满，只能丢弃连接吗？
            * 并不是这样，开启 `syncookies` 功能就可以在不使用 SYN 队列的情况下成功建立连接
            * syncookies 是这么做的：服务器根据当前状态计算出一个值，放在己方发出的`SYN+ACK`报文中发出，当客户端返回 ACK 报文时，取出该值验证，如果合法，就认为连接建立成功
            * Linux 下如何开启 `syncookies` 功能
                - 修改 `tcp_syncookies` 参数即可，其中值为`0`时表示关闭该功能，`2`表示无条件开启功能，而`1`则表示仅当 SYN 半连接队列放不下时，再启用它
            * 由于 syncookie 仅用于应对 SYN 泛洪攻击（攻击者恶意构造大量的 SYN 报文发送给服务器，造成 SYN 半连接队列溢出，导致正常客户端的连接无法建立），这种方式建立的连接，许多 TCP 特性都无法使用
                - 所以，应当把 `tcp_syncookies` 设置为 1，仅在队列满时再启用。
                - `sysctl -a`查看，*默认*就是：`net.ipv4.tcp_syncookies = 1`
        + 当*客户端*接收到服务器发来的 `SYN+ACK` 报文后，就会回复 ACK 去通知服务器，同时己方连接状态从 `SYN_SENT` 转换为 `ESTABLISHED`，表示连接建立成功
        + *服务器端*连接成功建立的时间还要再往后，到它收到 ACK(需要等收到客户端应答的ACK) 后状态才变为 `ESTABLISHED`
            * 如果服务器没有收到 ACK，就会一直重发 `SYN+ACK` 报文
            * 当网络繁忙、不稳定时，报文丢失就会变严重，此时应该调大重发次数。反之则可以调小重发次数
            * 修改重发次数(注意和客户端重发SYN的`tcp_syn_retries`区分)的方法是，调整 `tcp_synack_retries` 参数：
                - `net.ipv4.tcp_synack_retries = 5`
                - `sysctl -a|grep tcp_synack_retries`查看，*默认*就是`net.ipv4.tcp_synack_retries = 5`
                - `tcp_synack_retries` 的默认重试次数是 5 次，与客户端重发 SYN 类似，它的重试会经历 1、2、4、8、16 秒，最后一次重试后等待 32 秒，若仍然没有收到 ACK，才会关闭连接，故共需要等待 63 秒
            * 服务器收到 ACK 后连接建立成功，此时，内核会把连接从 *SYN 半连接队列*中移出，再移入 *accept 队列*，等待进程调用 `accept` 函数时把连接取出来
            * 如果进程不能及时地调用 `accept` 函数，就会造成 *accept 队列* 溢出，最终导致建立好的 TCP 连接被*丢弃*
                - 实际上，丢弃连接只是 Linux 的默认行为，我们还可以选择向客户端发送 `RST`复位报文，告诉客户端连接已经建立失败
                - 打开这一功能需要将 `tcp_abort_on_overflow` 参数设置为 `1`
                    + `sysctl`查看，默认为：`net.ipv4.tcp_abort_on_overflow = 0`
                - 通常情况下，应当把 `tcp_abort_on_overflow` 设置为 0，因为这样更有利于应对突发流量
                    + 举个例子，当 *accept 队列*满导致服务器丢掉了 ACK，与此同时，客户端的连接状态却是 ESTABLISHED，进程就在建立好的连接上发送请求。只要服务器没有为请求回复 ACK，请求就会被多次重发。如果服务器上的进程只是`短暂的繁忙`造成 accept 队列满，那么当 accept 队列有空位时，再次接收到的请求报文由于含有 ACK，仍然会触发服务器端成功建立连接
                - 所以，`tcp_abort_on_overflow` 设为 `0` 可以提高连接建立的成功率，只有你非常肯定 accept 队列会长期溢出时，才能设置为 `1` 以尽快通知客户端
            * 那么，怎样调整 *accept 队列*的长度呢？
                - `listen` 函数的 `backlog`参数就可以设置 *accept 队列*的大小
                    + `int listen(int s, int backlog);`
                        * 如果一个连接请求到达时未完成连接 队列已满,那么客户端将接收到错误 `ECONNREFUSED`
                        * 如果下层协议支持重发,那么这个连接请求将被忽略,这样客户端 在重试的时候就有成功的机会
                        * 在TCP套接字中  backlog  的含义在Linux  2.2中已经改变.它指定了*已经完成连接正等待应用程序接收的套接字队列*的长度,而`不是`*未完成连接*的数目.未完成连接套接字队列(SYN半连接队列)的最大长度可以使用`tcp_max_syn_backlog`，sysctl设置，当打开`syncookies`时不存在逻辑上的最大长度,此设置将被忽略
                        * 以上参考man手册：`man listen`
                    + 事实上，`backlog` 参数还受限于 Linux 系统级的队列长度上限，当然这个上限阈值也可以通过 `somaxconn` 参数修改
                        * `net.core.somaxconn = 128`
                        * `sysctl -a`可查看设置，自己的CentOS设置成了`net.core.somaxconn = 1024`(vi /etc/sysctl.conf)
                - 当下各监听端口上的 accept 队列长度可以通过 `ss -ltn` 命令查看
                - 但 accept 队列长度是否需要调整该怎么判断呢？
                    + 还是通过 `netstat -s` 命令给出的统计结果，可以看到究竟有多少个连接因为队列溢出而被丢弃
                        * `netstat -s | grep "listen queue"`
                    + 如果持续不断地有连接因为 accept 队列溢出被丢弃，就应该调大 `backlog` 以及 `somaxconn` 参数
    - TFO 技术如何绕过三次握手？
        + 三次握手建立连接造成的后果就是，HTTP 请求必须在一次 `RTT`（Round Trip Time，从客户端到服务器一个往返的时间）后才能发送
            * Google 对此做的统计显示，三次握手消耗的时间，在 HTTP 请求完成的时间占比在 10% 到 30% 之间
        + 因此，Google 提出了 `TCP fast open` 方案（简称TFO），客户端可以在*首个 SYN 报文*中就携带请求，这节省了 1 个 RTT 的时间
        + 为了让客户端在 SYN 报文中携带请求数据，必须解决服务器的信任问题
        + 它把通讯分为两个阶段
            * 第一阶段为首次建立连接，这时走正常的三次握手，但在客户端的 SYN 报文会明确地告诉服务器它想使用 TFO 功能，这样服务器会把客户端 IP 地址用只有自己知道的密钥加密（比如 AES 加密算法），作为 Cookie 携带在返回的 SYN+ACK 报文中，客户端收到后会将 Cookie 缓存在本地。
            * 之后，如果客户端再次向服务器建立连接，就可以在第一个 SYN 报文中携带请求数据，同时还要附带缓存的 Cookie
        + 这种通讯方式下不能再采用经典的“先 `connect` 再 `write` 请求”这种编程方法，而要改用 `sendto` 或者 `sendmsg` 函数才能实现
        + 服务器收到后，会用自己的密钥验证 Cookie 是否合法，验证通过后连接才算建立成功，再把请求交给进程处理，同时给客户端返回 `SYN+ACK`
            * 虽然客户端收到后还会返回 ACK，但服务器不等收到 ACK 就可以发送 HTTP 响应了，这就减少了握手带来的 1 个 RTT 的时间消耗
            * 为了防止 SYN 泛洪攻击，服务器的 TFO 实现必须能够自动化地定时更新密钥
        + Linux 下怎么打开 TFO 功能
            * 通过 `tcp_fastopen` 参数。由于只有客户端和服务器同时支持时，TFO 功能才能使用，所以 `tcp_fastopen` 参数是`按比特位控制`的
            * 其中，第 1 个比特位为 `1` 时，表示作为客户端时支持 TFO；第 `2` 个比特位为 1 时，表示作为服务器时支持 TFO，所以当 `tcp_fastopen` 的值为 `3` 时（比特为 `0x11`）就表示完全支持 TFO 功能。
                - `net.ipv4.tcp_fastopen = 3`，`sysctl`查看默认不开启，为：`net.ipv4.tcp_fastopen = 0`
* [10 | 如何提升TCP四次挥手的性能？](https://time.geekbang.org/column/article/238388)
    - 上节介绍了建立连接时的优化方法，本节介绍四次挥手关闭连接时，如何优化性能
    - `close` 和 `shutdown` 函数都可以关闭连接，但这两种方式关闭的连接，不只功能上有差异，控制它们的 Linux 参数也不相同
        + `close` 函数会让连接变为孤儿连接，`shutdown` 函数则允许在半关闭的连接上长时间传输数据
        + TCP 之所以具备这个功能，是因为它是全双工协议，但这也造成四次挥手非常复杂
    - 四次挥手中可以用 `netstat` 命令观察到 `6` 种状态
    - 优化连接关闭时，不能仅基于主机端的视角，还必须站在*整个网络的层次*上，才能给出正确的解决方案
    - 四次挥手的流程
        + 为什么建立连接是三次握手，而关闭连接需要四次挥手呢？
            * 这是因为 TCP 不允许连接处于半打开状态时就单向传输数据，所以在三次握手建立连接时，服务器会把 ACK 和 SYN *放在一起*发给客户端
            * 其中，ACK 用来打开客户端的发送通道，SYN 用来打开服务器的发送通道
            * 这样，原本的四次握手就降为三次握手了
        + 但是当连接处于半关闭状态时，TCP 是允许单向传输数据的
            * 当主动方(先关闭连接的一方叫主动方，后关闭连接的一方叫被动方)关闭连接时，被动方仍然可以在不调用 `close` 函数的状态下，长时间发送数据，此时连接处于半关闭状态
            * 这一特性是 TCP 的双向通道互相独立所致，却也使得关闭连接必须通过四次挥手才能做到
        + 互联网中往往服务器才是主动关闭连接的一方
            * 一方面，由于被动方有多种应对策略，从而增加了主动方的处理分支
            * 另一方面，服务器同时为成千上万个用户服务，任何错误都会被庞大的用户数放大
            * 所以对主动方的关闭连接参数调整时，需要格外小心
        + 四次挥手只涉及两种报文：`FIN` 和 `ACK`
            * FIN 就是 Finish 结束连接的意思，谁发出 FIN 报文，就表示它将不再发送任何数据，关闭这一方向的传输通道
            * ACK 是 Acknowledge 确认的意思，它用来通知对方：你方的发送通道已经关闭
        + 当*主动方*关闭连接时
            * 会发送 FIN 报文，此时主动方的连接状态由 `ESTABLISHED` 变为 `FIN_WAIT1`
            * 当*被动方*收到 FIN 报文后，内核自动回复 ACK 报文，连接状态由 `ESTABLISHED` 变为 `CLOSE_WAIT`
                - 顾名思义，它在等待进程调用 `close` 函数关闭连接
            * 当主动方接收到这个 ACK 报文后，连接状态由 `FIN_WAIT1` 变为 `FIN_WAIT2`，*主动方*的*发送通道就关闭了*
        + 再来看*被动方*的发送通道是如何关闭的
            * 当被动方进入 `CLOSE_WAIT` 状态时，进程的 `read` 函数会返回 0，这样开发人员就会有针对性地调用 `close` 函数，进而触发内核发送 FIN 报文，此时被动方连接的状态变为 `LAST_ACK`
            * 当*主动方*收到这个 FIN 报文时，内核会自动回复 ACK，同时连接的状态由 `FIN_WAIT2` 变为 `TIME_WAIT`(即上面的过程)，Linux 系统下大约 `1 分钟`后 `TIME_WAIT` 状态的连接才会彻底关闭
            * 而*被动方*收到 ACK 报文后，连接就会*关闭*
    - 主动方的优化
        + 关闭连接有多种方式，比如进程异常退出时，针对它打开的连接，内核就会发送 `RST` 报文来关闭
            * `RST` 的全称是 `Reset` 复位的意思，它可以不走四次挥手强行关闭连接，但当报文延迟或者重复传输时，这种方式会导致数据错乱，所以这是不得已而为之的关闭连接方案
        + `安全关闭连接`的方式必须通过四次挥手，它由进程调用 `close` 或者 `shutdown` 函数发起
            * 这二者都会向对方发送 `FIN` 报文（`shutdown` 参数须传入 `SHUT_WR` 或者 `SHUT_RDWR` 才会发送 `FIN`）
                - 函数签名为：`int shutdown(int socket, int how);`，头文件`#include <sys/socket.h>`
                - `how`指定关闭的类型，`SHUT_RD`关闭读、`SHUT_WR`关闭写、`SHUT_RDWR`关闭读写
            * 区别在于 `close` 调用后，哪怕对方在半关闭状态下发送的数据到达主动方，进程也无法接收
            * 此时，调用`close`后的这个连接叫做*孤儿连接*，如果你用 `netstat -p` 命令，会发现连接对应的进程名为空
                - 而 `shutdown` 函数调用后，即使连接进入了 `FIN_WAIT1` 或者 `FIN_WAIT2` 状态，它也不是孤儿连接，进程仍然可以继续接收数据
        + 主动方发送 FIN 报文后，连接就处于 `FIN_WAIT1` 状态下，该状态通常应在*数十毫秒内*转为 `FIN_WAIT2`
            * 只有迟迟收不到对方返回的 ACK 时，才能用 `netstat` 命令观察到 `FIN_WAIT1` 状态
            * 此时，内核会定时重发 FIN 报文，其中重发次数由 `tcp_orphan_retries` 参数控制
                - 默认值是 0，特指 8 次
                - `sysctl`查看本机默认为：`net.ipv4.tcp_orphan_retries = 0`
                - 注意，orphan 虽然是孤儿的意思，该参数却不只对孤儿连接有效，事实上，它对所有 `FIN_WAIT1` 状态下的连接都有效
            * 如果 `FIN_WAIT1` 状态连接有很多，你就需要考虑降低 `tcp_orphan_retries` 的值。当重试次数达到 `tcp_orphan_retries` 时，连接就会直接关闭掉
        + 对于正常情况来说，调低 `tcp_orphan_retries` 已经够用，但如果遇到恶意攻击，FIN 报文根本无法发送出去
            * 首先，TCP 必须保证报文是有序发送的，FIN 报文也不例外，当发送缓冲区还有数据没发送时，FIN 报文也不能提前发送
            * 其次，TCP 有流控功能，当接收方将接收窗口设为`0`时，发送方就不能再发送数据。
            * 所以，当攻击者下载大文件时，就可以通过将接收窗口设为 `0`，导致 FIN 报文无法发送，进而导致连接一直处于 `FIN_WAIT1` 状态(一直收不到ACK，变不了`FIN_WAIT2`)
            * 解决这种问题的方案是调整 `tcp_max_orphans` 参数
                - `net.ipv4.tcp_max_orphans = 16384`
                - 查看本机默认：`net.ipv4.tcp_max_orphans = 32768`
            * 顾名思义，`tcp_max_orphans` 定义了孤儿连接的最大数量。当进程调用 `close` 函数关闭连接后，无论该连接是在 `FIN_WAIT1` 状态，还是确实关闭了，这个连接都与该进程无关了，它变成了*孤儿连接*
                - Linux 系统为防止孤儿连接过多，导致系统资源长期被占用，就提供了 `tcp_max_orphans` 参数
                - 如果孤儿连接数量大于它，新增的孤儿连接将不再走四次挥手，而是直接发送 `RST` 复位报文*强制关闭*
        + 当连接收到 ACK 进入 `FIN_WAIT2` 状态后，就表示主动方的发送通道已经关闭，接下来将等待对方发送 FIN 报文，关闭对方的发送通道
            * 这时，如果连接是用 `shutdown` 函数关闭的，连接*可以一直*处于 `FIN_WAIT2` 状态
            * 但对于 `close` 函数关闭的孤儿连接，这个状态不可以持续太久，而 `tcp_fin_timeout` 控制了这个状态下连接的持续时长
                - 查看本机默认为：`net.ipv4.tcp_fin_timeout = 60`
                - 它的默认值是 `60 秒`。这意味着对于孤儿连接，如果 60 秒后还没有收到 FIN 报文，连接就会直接关闭
                - 这个 60 秒并不是拍脑袋决定的，它与接下来介绍的 `TIME_WAIT` 状态的持续时间是相同的
        + `TIME_WAIT` 是主动方四次挥手的最后一个状态
            * 当收到被动方发来的 FIN 报文时，主动方回复 ACK，表示确认对方的发送通道已经关闭，连接(主动方连接)随之进入 `TIME_WAIT` 状态，等待 `60 秒`后关闭。理由如下：
                - `TIME_WAIT` 状态的连接，在主动方看来确实已经关闭了(主动方一收到FIN然后发送个ACK就变为TIME_WAIT，已经是最后一个状态了)。然而，被动方没有收到 ACK 报文前，连接还处于 `LAST_ACK` 状态
                - 如果这个 ACK 报文没有到达被动方，被动方就会重发 FIN 报文。重发次数仍然由前面介绍过的 `tcp_orphan_retries` 参数控制
            * 如果主动方不保留 `TIME_WAIT` 状态，会发生什么呢？
                - 此时连接的端口恢复了自由身，可以复用于新连接了。然而，被动方的 FIN 报文可能再次到达，这既可能是网络中的路由器重复发送，也有可能是被动方没收到 ACK 时基于 `tcp_orphan_retries` 参数重发
                - 这样，正常通讯的新连接就可能被重复发送的 FIN 报文误关闭
                - 保留 `TIME_WAIT` 状态，就可以应付重发的 FIN 报文，当然，其他数据报文也有可能重发，所以 `TIME_WAIT` 状态还能避免数据错乱
            * 为什么 `TIME_WAIT` 状态要保持 `60 秒`呢？
                - 这与孤儿连接 `FIN_WAIT2` 状态默认保留 60 秒的原理是一样的
                - 因为这两个状态都需要保持 `2MSL` 时长
                    + `MSL` 全称是 `Maximum Segment Lifetime`，它定义了一个报文在网络中的最长生存时间（报文每经过一次路由器的转发，IP 头部的 TTL 字段就会减 1，减到 0 时报文就被丢弃，这就限制了报文的最长存活时间）
                - 为什么是 `2 MSL` 的时长呢？这其实是相当于*至少允许报文丢失一次*。
                    + 比如，若 ACK 在一个 MSL 内丢失，这样被动方重发的 FIN 会在第 2 个 MSL 内到达，TIME_WAIT 状态的连接可以应对
                    + 为什么不是 4 或者 8 MSL 的时长呢？你可以想象一个丢包率达到百分之一的糟糕网络，连续两次丢包的概率只有万分之一，这个概率实在是太小了，忽略它比解决它更具性价比
                - 因此，`TIME_WAIT` 和 `FIN_WAIT2` 状态的最大时长都是 2 MSL，由于在 Linux 系统中，`MSL` 的值固定为 `30 秒`，所以它们都是 60 秒
        + 虽然 `TIME_WAIT` 状态的存在是有必要的，但它毕竟在消耗系统资源，比如 `TIME_WAIT` 状态的端口就无法供新连接使用
            * Linux 提供了 `tcp_max_tw_buckets` 参数，当 `TIME_WAIT` 的连接数量超过该参数时，新关闭的连接就不再经历 `TIME_WAIT` 而*直接关闭*
                - `net.ipv4.tcp_max_tw_buckets = 5000`
                - 查看默认为：`net.ipv4.tcp_max_tw_buckets = 32768`
            * 当服务器的并发连接增多时，相应地，同时处于 `TIME_WAIT` 状态的连接数量也会变多，此时就应当调大 `tcp_max_tw_buckets` 参数，减少*不同连接间数据错乱*的概率
        + 当然，`tcp_max_tw_buckets` 也不是越大越好，毕竟内存和端口号都是有限的
            * 如果服务器会主动向上游服务器发起连接的话，就可以把 `tcp_tw_reuse` 参数设置为 `1`，它允许作为客户端的新连接，在*安全条件下*使用 TIME_WAIT 状态下的端口
                - `net.ipv4.tcp_tw_reuse = 1`
                - 查看默认为：`net.ipv4.tcp_tw_reuse = 0`
            * 要想使 `tcp_tw_reuse` 生效，还得把 `timestamps` 参数设置为 `1`，它满足安全复用的先决条件（*对方也要*打开 `tcp_timestamps` ）：
                - `net.ipv4.tcp_timestamps = 1`
                - 查看默认为打开状态：`net.ipv4.tcp_timestamps = 1`
            * 老版本的 Linux 还提供了 `tcp_tw_recycle` 参数，它并*不要求* TIME_WAIT 状态存在 60 秒，*很容易导致数据错乱*，*不建议*设置为 `1`
                - `net.ipv4.tcp_tw_recycle = 0`
                - 查看默认为就为关闭：`net.ipv4.tcp_tw_recycle = 0`
                - 所以在 Linux 4.12 版本后，直接取消了这一参数
    - 被动方的优化
        + 当被动方收到 FIN 报文时，就开启了被动方的四次挥手流程
            * 内核自动回复 ACK 报文后，连接就进入 CLOSE_WAIT 状态，顾名思义，它表示等待进程调用 close 函数关闭连接
            * 内核没有权力替代进程去关闭连接，因为若主动方是通过 shutdown 关闭连接，那么它就是想在半关闭连接上接收数据。因此，Linux 并没有限制 `CLOSE_WAIT` 状态的持续时间
        + 当然，大多数应用程序并不使用 `shutdown` 函数关闭连接
            * 所以，当你用 `netstat` 命令发现大量 `CLOSE_WAIT` 状态时，要么是程序出现了 Bug，read 函数返回 0 时忘记调用 `close` 函数关闭连接，要么就是程序负载太高，`close` 函数所在的回调函数被延迟执行了。此时，我们应当在*应用代码层面*解决问题
        + 需要注意的是，如果被动方迅速调用 close 函数，那么被动方的 `ACK 和 FIN` 有可能*在一个报文中发送*，这样看起来，*四次挥手会变成三次挥手*，这只是一种特殊情况，不用在意
        + 再来看一种特例，如果连接双方`同时关闭连接`，会怎么样？
            * 两方发送 FIN 报文时，都认为自己是主动方，所以都进入了 `FIN_WAIT1` 状态，FIN 报文的重发次数仍由 `tcp_orphan_retries` 参数控制
            * 接下来，双方在等待 ACK 报文的过程中，都等来了 FIN 报文。这是一种新情况，所以连接会进入一种叫做 `CLOSING` 的*新状态*，它替代了 `FIN_WAIT2` 状态
            * 此时，内核回复 ACK 确认对方发送通道的关闭，仅己方的 FIN 报文对应的 ACK 还没有收到
            * 所以，`CLOSING` 状态与 `LAST_ACK` 状态下的连接很相似，它会在适时重发 FIN 报文的情况下最终关闭
    - 关闭连接时的 `SO_LINGER` 选项
        + 它希望用四次挥手替代 `RST` 关闭连接的方式，防止浏览器没有接收到完整的 HTTP 响应
        + socket将会等待到未发出的数据发送完成或者达到超时后被强制关闭
            * 内核将socket关闭前的这段等待时间，称为`Linger Time`(延迟时间)
        + 配置`SO_LINGER`选项可以将`Linger Time`配置得更长或更短，甚至可以禁止该延时时间。 配置该选项需要送一个`struct linger`结构体，其中包含两个int字段：开关(`int l_onoff`)和延迟时间(`int l_linger` Linux上指秒数)
            * 开关为0(关闭)时，保持TCP默认操作：等到(send buffer中的)未发出数据发送结束或者达到默认延迟时间后超时；
            * 开关为非零，延迟时间设置为0时：则内核立即关闭socket连接，send buffer中未发出的数据被丢弃，并向对端发送`RST`，此时不会有`TIME_WAIT`状态
            * 开关为非零，延迟时间非零时：send buffer中未发出的数据按设置的延迟时间发送，达到延迟还未发完则关闭socket
            * 关于`SO_LINGER`的设置，可参考：[Linux网络编程socket选项之SO_LINGER,SO_REUSEADDR](https://blog.csdn.net/feiyinzilgd/article/details/5894300)
        + 完全禁止延迟时间是一个非常糟糕的主意。如果完全禁止则每次关闭socket都是强制关闭而不是优雅关闭