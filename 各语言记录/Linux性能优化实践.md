## 平均负载

[到底应该怎么理解“平均负载”？](https://time.geekbang.org/column/article/69618)

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

## CPU 上下文切换

[经常说的 CPU 上下文切换是什么意思？](https://time.geekbang.org/column/article/69859)

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

## CPU使用率

[基础篇：某个应用的CPU使用率居然达到100%，我该怎么办？](https://time.geekbang.org/column/article/70476)

* CPU使用率
    - 作为最常用也是最熟悉的 CPU 指标，你能说出 CPU 使用率到底是*怎么算出来的*吗？
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
            * 在b执行新的 `ab -c 10 -n 10000 http://10.240.0.5:10000/`
            * 在a找到 php-fpm 进程号(有多个，找一个分析)
                - `perf top -g -p 19318`
            * 在容器外perf top只能看到十六进制地址，所以先在外面保存perf信息，再放到容器中分析
                - 参考：[CPU使用率达到100% 精选留言](https://time.geekbang.org/column/article/70476)
                - `perf record -g -p 19402` 运行一段时间后打断，生成 perf.data 文件，拷贝到容器中，在容器中进行分析
                - `docker cp perf.data phpfpm:/tmp` 拷贝到容器phpfpm
                - `docker exec -i -t phpfpm bash`   进入容器bash终端
                - `cd /tmp/`, 安装perf工具：`apt-get update && apt-get install -y linux-perf linux-tools procps`
                - 分析perf.data, `perf_4.9 report`
                    + 注意：最后运行的工具名字是容器内部安装的版本 perf_4.9，而不是 perf 命令，这是因为 perf 会去跟内核的版本进行匹配，但镜像里面安装的perf版本有可能跟虚拟机的内核版本不一致。

* `ab -c 10 -n 100 http://192.168.50.118:10000/`运行结果(虚拟机在跑别的程序，消耗了大部分CPU)：

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
Time taken for tests:   78.361 seconds
Complete requests:      100
Failed requests:        0
Write errors:           0
Total transferred:      17200 bytes
HTML transferred:       900 bytes
Requests per second:    1.28 [#/sec] (mean)
Time per request:       7836.124 [ms] (mean)
Time per request:       783.612 [ms] (mean, across all concurrent requests)
Transfer rate:          0.21 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   2.3      0      14
Processing:   832 7556 1197.7   7666   10593
Waiting:      831 7553 1197.6   7641   10528
Total:        832 7557 1198.2   7666   10606

Percentage of the requests served within a certain time (ms)
  50%   7666
  66%   7800
  75%   8032
  80%   8208
  90%   8372
  95%   8477
  98%   8588
  99%  10606
 100%  10606 (longest request)
```

