## libevent

* GitHub：[libevent](https://github.com/libevent/libevent)
* 个人fork学习路径：[libevent学习](https://github.com/xiaodongQ/libevent)
* 网络相关编程学习和实践一并在该笔记记录

* 文档：[Fast portable non-blocking network programming with Libevent](http://www.wangafu.net/~nickm/libevent-book/)
* 文档开始的示例演示了不同网络IO的问题：
    - (http://www.wangafu.net/~nickm/libevent-book/01_intro.html)
    - 阻塞IO，用`send`和`recv`向 www.google.com 发送http GET
        + `问题`：没接收到数据前recv不会返回；send没有刷新它的输出到内核的写缓冲中也不会返回
        + 如果同时有多个连接要处理，更具体一点，比如想读取两个连接的数据，由于读取是阻塞的，此时就不知道应该先读取哪个了
        + 对于上述阻塞IO的问题，可通过多进程或多线程处理(相比于多个进程，更倾向于创建线程池)。
    - 接收到socket请求后fork进程处理
        + 但是`问题`是：
            * 进程创建(甚至是线程创建)在某些平台上是非常昂贵的(资源消耗);
            * 如果同时要处理成千或者上万连接，每个CPU处理上万线程并不比处理少数线程更高效(上下文切换)
        + 如果线程/进程不是处理上面多连接问题的答案，那么答案是什么?
            * 非阻塞IO
    - 非阻塞IO
        + 设置sockets为非阻塞，Unix下：`fcntl(fd, F_SETFL, O_NONBLOCK);`
        + 示例中演示了一个对所有sockets设置为非阻塞 `busy-polling` (轮询) 的例子
            * 将socket文件描述符设置为非阻塞后，再进行`recv`读取，读取返回<0时，判断`errno == EAGAIN`(此时内核中没有可供该句柄读取的数据)，则先循环处理其他句柄，下一次再来对该连接句柄recv
        + 但是这样的`问题`是性能很糟糕：
            * 第一，当所有连接都没有数据时，循环将会快速无限轮询下去，消耗大量CPU周期
            * 第二，当处理到来的一个或两个连接时，不论是否有数据都需要给每个连接进行一次系统调用
        + 因此我们需要的方式是：内核一直等待，直到某个socket准备好给我一些数据时，进行通知
            * 最早的解决方式是使用`select`
    - `select`
        + 见链接中的示例，`select`会等到有socket准备好时，修改描述符集(readset、writeset、exceptionset)只保留准备好的描述符
            * `FD_ZERO(set)` 宏 清理文件描述符集fdset (描述符集的数据结构为：`fd_set`)
            * `FD_SET(fd,set)` 向文件描述符集fdset中新增文件描述符fd (listener描述符)
            * `FD_CLR(fd,set)` 将对应fd移除
            * `FD_ISSET(fd,set)`来检测(返回true) 文件描述符集fdset中是否存在文件描述符fd(`select`之后set中只保留准备好的fd)
            * `select`返回后会把以前加入的但并无事件发生的fd清空(Linux用每位的0/1开关，MinGW的实现是用结构体数组标识每个fd)，所以循环每次开始`select`前都要重新从array取得fd逐一加入（通过`FD_SET`，用前先将描述符集`FD_ZERO`），扫描array的同时取得fd最大值maxfd，用于select的第一个参数(maxfd+1)
            * 通过`FD_ISSET(fd,set)`判断指定描述符fd存在于指定set中时，调用处理函数(fd收到时已设置为nonblock，处理函数中send/recv失败时判断errno为 EAGAIN 的情况)
            * 示例中给每个接收到的fd定义了一个数据结构，由于fd不能超过FD_SETSIZE，所以定义成了结构体指针数组，fd作为index
        + 但是还是没有解决`问题`：
            * 生成和读取`select`花费的时间和提供给它的最大fd(即指定maxfd+1)是成比例的，当socket很多时`select`调用花费会急剧增加(用户态时花费跟FD_SET添加进去的感兴趣fd数量成比例；而内核态中，跟整个程序所使用的所有fd的数量成比例，而不管有多少fd被添加到感兴趣描述符集)
        + 不同的系统提供了不同的替换`select`的替代函数，包含`poll()`(Linux), `epoll()`(Linux), `kqueue()`(BSDs，包含Darwin), `evports`(Solaris), 和 `/dev/poll`(Solaris)
            * 这些都比`select()`有更好的性能
            * 除了`poll()`，其他函数都提供`O(1)`的时间复杂度，对一个socket来 添加、删除、IO就绪通知
            * 但是这些高效的函数并没有成为广泛的标准，在不同的系统上有各自的函数(且不包含其他函数)，所以如果要开发一个`可移植`的`高效`的异步应用，需要对这些做一个抽象包装。
            * 这就是`Libevent API`的最低级别做的事情，它提供了对于这些`select()`的替代函数一致的接口，在不同的操作系统上使用最高效的替代版本
    - `libevent`
        + 链接示例中演示了使用`Libevent`的低层次ROT13服务(编译时`-levent`)
            * 使用`Libevent 2`来替换`select()`
            * 注意，`fd_sets`现在没有了，使用`event_base`结构体(可以用`select()`, `poll()`, `epoll()`, `kqueue()`来实现)来对`事件`进行关联和解除关联
    - 另外补充`poll`和`epoll`
        + `poll`，[poll](https://linux.die.net/man/2/poll)
            * `poll`和`select`类似，等待文件描述符集中的fd直到其准备好用于IO操作，测试示例参考libevent源码学习链接中的test_xd目录
            * `int poll(struct pollfd *fds, nfds_t nfds, int timeout);`
                - `fds`，需要监控的文件描述符集合，结构体包含：
                    + `int fd;`，打开的文件描述符。若为负，则忽略事件，且revents字段返回0
                    + `short events;`入参，指定感兴趣的事件，位掩码(bit mask)。若指定为0则所有事件忽略，且revents字段返回0
                    + `short revents;`出参，内核将实际发生的事件填入其中
                - `nfds`，指定fds的数量
                - `timeout`，超时时间 毫秒数，达到该超时之前`poll`会阻塞。设置为`负数`则一直阻塞，设置为0则立即返回(尽管无准备好的fd)
            * `POLLIN`可读/`POLLPRI`紧急数据可读/`POLLOUT`可写/`POLLRDHUP`连接关闭/`POLLERR`错误条件/`POLLHUP`挂起/`POLLNVAL`无效请求(fd未打开)
            * 优点
                - poll() 不要求开发者计算最大文件描述符加一的大小。
                - poll() 在应付大数目的文件描述符的时候速度更快，相比于select，它没有最大连接数的限制，原因是它是基于`链表`来存储的
                - 在调用函数时，只需要对参数进行一次设置就好了
            * 缺点
                - 大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义（epoll可以解决此问题）
                - 与select一样，poll返回后，需要轮询pollfd来获取就绪的描述符，这样会使性能下降
                - 同时连接的大量客户端在一时刻可能只有很少的就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降
            * 优缺点参考：[poll 的使用方法及代码](https://blog.csdn.net/weixin_43825537/article/details/90211331)
            * 关于select、poll、epoll三者比较，可参考：[IO多路复用的三种机制Select，Poll，Epoll](https://www.jianshu.com/p/397449cadc9a)
        + `epoll`，[epoll](https://linux.die.net/man/4/epoll)
            * `epoll`是`poll`的一个变体，既可用作Edge Triggered ( ET )边缘触发，也可用于Level Triggered ( LT )水平触发，可以很好地扩展到监测大量fd
            * 三个系统调用用于控制使用`epoll`：
                - 参考：[epoll使用详解](https://www.jianshu.com/p/ee381d365a29)
                - `int epoll_create(int size);` `int epoll_create1(int flags);`(flags设置为0时和epoll_create忽略size一样)
                    + 创建一个epoll实例的文件描述符句柄，该句柄用于后续所有的epoll相关接口。不再使用时需要`close`关闭句柄
                    + 从Linux2.6.8起，`size`参数就被忽略了，但需要`>0`
                - `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);`
                    + epoll句柄的控制接口，epoll的事件注册函数
                        * `epfd`，传的是`epoll_create`创建的句柄
                        * `op`，指定动作，支持以下动作：
                            - `EPOLL_CTL_ADD`注册新的fd到epfd中、
                            - `EPOLL_CTL_MOD`修改已经注册的fd的监听事件、
                            - `EPOLL_CTL_DEL`从epfd中删除一个fd
                                + `EPOLL_CTL_DEL`时最后一个参数会忽略，可送NULL，不过kernel2.6.9之前需要一个非NULL(参考man epoll_ctl,BUGS)
                        * `fd`，第三个参数是需要`监听`的fd
                        * `struct epoll_event *event`，告诉内核需要监听什么事件(可组合，位掩码)
                            - 包含两个成员：
                                + `__uint32_t events;` 指定epoll的事件，包含：`EPOLLIN`可读/`EPOLLOUT`可写/`EPOLLRDHUP`连接关闭/`EPOLLPRI`紧急数据可读/`EPOLLERR`错误条件(不需单独add设置)/`EPOLLHUP`挂起(不需add)/`EPOLLET`边缘触发(默认是水平触发)/`EPOLLONESHOT`一次性的行为
                                + `epoll_data_t data;` 用户数据变量，联合体：其中一个成员是`int fd;`
                - `int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);`
                    + 等待epoll描述符上的I/O事件产生，类似于select()调用。
                        * `epfd`，传的是`epoll_create`创建的句柄
                        * `struct epoll_event *events`，从内核得到的可用事件集合
                        * `maxevents`，`epoll_wait`返回最大为`maxevents`，`maxevents`必须大于0
                        * `timeout`，超时时间 毫秒数，达到该超时之前`epoll_wait`会阻塞。设置为`-1`则一直阻塞，设置为0则立即返回(尽管无准备好的事件)
            * 关于`ET`(Edge Triggered边缘触发)、`LT`(Level Triggered水平触发)两种工作模式
                - `LT`(缺省方式)，水平触发只要有数据没有处理就会一直通知下去
                    + 同时支持block和no-block socket
                    + 如果不作任何操作，内核还是会继续通知，所以，这种模式编程出错误可能性要小一点。传统的select/poll都是这种模型的代表．
                    + 优点：当进行socket通信的时候，保证了数据的完整输出，进行IO操作的时候，如果还有数据，就会一直的通知你。
                    + 缺点：由于只要还有数据，内核就会不停的从内核空间转到用户空间，所有占用了大量内核资源，试想一下当有大量数据到来的时候，每次读取一个字节，这样就会不停的进行切换。内核资源的浪费严重。效率来讲也是很低的。
                - `ET`，边缘触发仅当状态发生变化的时候才获得通知，即只有数据到来时才触发，不管缓存区中是否还有数据
                    + 高速工作方式
                    + 只支持no-block socket
                    + 在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll通知。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知。
                    + 注意，如果一直不对这个fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once).
                    + 优点：每次内核只会通知一次，大大减少了内核资源的浪费，提高效率。
                    + 缺点：不能保证数据的完整。不能及时的取出所有的数据。
            * epoll的优点
                - 监视的描述符数量不受限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左 右，具体数目可以`cat /proc/sys/fs/file-max`察看,一般来说这个数目和系统内存关系很大。
                    + select的最大缺点就是进程打开的fd是有数量限制的。这对于连接数量比较大的服务器来说根本不能满足。虽然也可以选择多进程的解决方案( Apache就是这样实现的)，不过虽然linux上面创建进程的代价比较小，但仍旧是不可忽视的，加上进程间数据同步远比不上线程间同步的高效，所以也不是一种完美的方案。
                - IO的效率不会随着监视fd的数量的增长而下降。epoll不同于select和poll轮询的方式，而是通过每个fd定义的回调函数来实现的。只有就绪的fd才会执行回调函数。
                - 如果没有`大量的idle-connection或者dead-connection`，epoll的效率并不会比select/poll高很多，但是当遇到大量的idle-connection，就会发现epoll的效率大大高于select/poll。
            * 参考(可查看三者比较，总结得比较好)：[I/O模型之二：Linux IO模式及 select、poll、epoll详解](https://www.jianshu.com/p/fe54ca4affe8)
        + 运行比较(客户端10线程跑10000个请求，脚本中跑5次并求平均时间)
            * `./rot13_server_accept_fork` avg clock() cost: 642 ms, gettimeofday():2569 ms
            * `./rot13_server_select`      avg clock() cost: 496 ms, gettimeofday():604 ms
            * `./rot13_server_poll`        avg clock() cost: 486 ms, gettimeofday():686 ms
            * `./rot13_server_epoll`       avg clock() cost: 486 ms, gettimeofday():602 ms
            * `./rot13_server_libevent`    (accept: Too many open files)
* 对于socket server端，`bind()`前一般都对要监听的socket设置一下 `SO_REUSEADDR` 选项
    - `int one = 1; setsockopt(listener, SOL_SOCKET, SO_REUSEADDR, &one, sizeof(one));`
    - 关于`SO_REUSEADDR`(和`SO_REUSEPORT`)的说明
        + 一个socket发送一段buffer时，`send()`返回成功但是不意味着实际已经发出去了，而表示数据已经添加到了内核中的`send buffer`。(对于UDP，数据一般很快发送出去了，但是对于TCP而言会有相对较长的时延)
        + 结果是当关闭一个socket时，可能buffer中还有没发出的数据。当主动关闭连接而socket还有数据要发送时，socket将会进入`TIME_WAIT`状态。 socket将会等待到未发出的数据发送完成或者达到超时后被强制关闭。
            * (当调用`close()`时，将会尝试发送send buffer中的所有数据。？ 此时是否会阻塞close()直到延迟时间超时或者发送完成？[close()、shutdown()函数](https://www.cnblogs.com/f-ck-need-u/p/7623252.html#2-6-close-shutdown-))
                - `man 3 setsockopt`(注意若不设置man的page页为3(POSIX Linux)，该函数则是展示了第2页(Linux)中的内容，其中并没有说明选项)，查看`SO_LINGER`项可知，若设置了该选项则会阻塞；若没设置，则系统处理close()时会让进程尽可能快地继续下去
            * 内核将socket关闭前的这段等待时间，称为`Linger Time`(延迟时间)。大多数系统上的延迟时间是全局可配置的，默认情况下相当长(很多系统上配置2分钟)
            * 配置`SO_LINGER`选项可以将`Linger Time`配置得更长或更短，甚至可以禁止该延时时间。 配置该选项需要送一个`struct linger`结构体，其中包含两个int字段：开关(`int l_onoff`)和延迟时间(`int l_linger` Linux上指秒数)。
                - 开关为0(关闭)时，保持TCP默认操作：等到(send buffer中的)未发出数据发送结束或者达到默认延迟时间后超时；
                - 开关为非零，延迟时间设置为0时：则内核立即关闭socket连接，send buffer中未发出的数据被丢弃，并向对端发送`RST`，此时不会有`TIME_WAIT`状态
                - 开关为非零，延迟时间非零时：send buffer中未发出的数据按设置的延迟时间发送，达到延迟还未发完则关闭socket
                - 关于`SO_LINGER`的设置，可参考：[Linux网络编程socket选项之SO_LINGER,SO_REUSEADDR](https://blog.csdn.net/feiyinzilgd/article/details/5894300)
            * 完全禁止延迟时间是一个非常糟糕的主意。如果完全禁止则每次关闭socket都是强制关闭而不是优雅关闭。
        + 如果没有设置`SO_REUSEADDR`：`TIME_WAIT`状态的socket被认为还是绑定在源地址和端口上，并且任何尝试向相同地址和端口绑定(`bind`)新socket的操作都会是失败的，直到socket被真正关闭，这可能会花费`Linger Time`配置的延迟时间。所以关闭连接后立即重新绑定源地址(比如重启的情况)很多情况下是会失败的。
        + 而如果尝试绑定的socket配置了`SO_REUSEADDR`：绑定到相同源地址和端口上的`TIME_WAIT`状态的socket则会被忽略(这些绑定相同地址的“半死亡”状态的socket有可能有副作用，但是实践中相当罕见)
        + 参考：[How do SO_REUSEADDR and SO_REUSEPORT differ?](https://stackoverflow.com/questions/14388706/how-do-so-reuseaddr-and-so-reuseport-differ)
        + 另外参考链接中的`SO_REUSEPORT`
            * `SO_REUSEPORT`允许将任意数量的套接字绑定到完全相同的源地址和端口，只要`所有之前`绑定的套接字在绑定之前都设置了`SO_REUSEPORT`。
            * 如果第一个bind的socket没有设置`SO_REUSEPORT`，则其他socket就算设置了`SO_REUSEPORT`来绑定相同地址也会失败
            * `SO_REUSEPORT`并不意味着`SO_REUSEADDR`，如果已被绑定的*没有*设置`SO_REUSEPORT`的socket关闭后为`TIME_WAIT`状态时，新的设置为`SO_REUSEPORT`的绑定也会失效。
                - 要重新绑定`TIME_WAIT`的地址，需要新的socket设置为`SO_REUSEADDR`
                - 或者`TIME_WAIT`的socket和新绑定的socket，都需要设置为`SO_REUSEPORT`
            * 当然，`SO_REUSEPORT`和`SO_REUSEADDR`在一个socket上可以同时设置
* 关于`listen`函数的第二个参数，`int listen(int socket, int backlog);`
    - [listen()函数中backlog参数分析](https://blog.csdn.net/ordeder/article/details/21551567)
    - 当socket函数创建一个套接字时，它被假设为一个主动套接口(客户端socket，用于调用connect去连接服务端)，listen函数把一个未连接的套接字转换成一个被动套接字(告诉内核接受指向该socket的连接请求)。根据TCP状态转换图，调用listen导致套接字从`CLOSED`状态转换到`LISTEN`状态。
    - 第二个参数`backlog`规定了内核应该为相应socket(套接字)排队的最大连接个数。一般为 `未完成三次握手队列` 和 `已经完成三次握手队列` 的大小之和
        + 未完成连接队列（incomplete connection queue）
            * `SYN`已由某个客户发出并到达服务器，而服务器正在等待完成相应的TCP三次握手过程。这些套接字处于`SYN_RECV`状态。
        + 已完成连接队列（completed connection queue）
            * 每个已完成TCP三次握手过程的客户对应其中一项。这些套接字处于`ESTABLISHED`状态。
    - 当来自客户的`SYN`到达时，TCP在未完成连接队列中创建一个新项，然后服务器给该项组装`SYN`响应，其中稍带对客户SYN的`ACK`（即SYN+ACK）(三次握手的第二个分节)，该项一直保留在未完成连接队列中，直到三次握手的第三个分节或者达到该项的超时时间。如果三次握手正常完成，该项就从未完成连接队列移到已完成连接队列的队尾。当进程调用`accept`时，已完成连接队列中的队头项将返回给进程(从已完成连接队列中取出一个“连接”)，如果队列为空，则进程进入睡眠，直到TCP在该队列中放入一项才唤醒它(监听套接字的已完成连接队列中的元素个数大于0，那么该套接字是可读的)。

## 预备知识(官网文档)

* 官网链接：[The Libevent Reference Manual: Preliminaries](http://www.wangafu.net/~nickm/libevent-book/Ref0_meta.html)
* 设计目标
    - 可移植性(Portability)。跨平台，即使某个环境没有非阻塞IO，libevent也应该支持类似方式
    - 速度(Speed)。使用每个平台最快的非阻塞IO来实现，并且不引入太多的开销
    - 可扩展性(Scalability)。即使同时有1万个活跃的socket(C10K)
    - 便利性(Convenience)。用Libevent编写程序最自然的方式应该是稳定的、可移植的方式
* Libevent分成以下组件
    - `evutil`
        + 抽象出不同平台的网络实现之间的差异的通用功能。
    - `event` and `event_base`
        + 这是Libevent的核心。它为各种特定于平台、基于事件的非阻塞IO后端提供了一个抽象API
    - `bufferevent`
        + 这些函数为Libevent的基于事件的核心提供了更方便的包装。它们让应用程序的请求缓冲读和写，让人知道IO什么时候已经发生，而不是通知什么时候套接字准备好了。
    - `evbuffer`
        + 此模块实现bufferevents底层的缓冲区，并提供高效和方便的访问功能
    - `evhttp`
        + 一个简单的HTTP 客户端/服务器实现
    - `evdns`
        + 一个简单的DNS 客户端/服务器实现
    - `evrpc`
        + 一个简单的RPC实现
* 库 Libevent编译后，默认安装以下库
    - `libevent_core`
        + 所有核心事件和缓冲区功能。这个库包含所有的event_base、evbuffer、bufferevent和utility实用程序函数。
    - `libevent_extra`
        + 定义了特定协议的功能。你的应用程序可能需要也可能不需要这些功能，包括HTTP、DNS和RPC
    - `libevent`
        + 这个库存在是有历史原因的，同时包含`libevent_core` 和 `libevent_extra`，**不应该**使用这个库
    - 下面的库安装在某些系统上(Linux下安装了)
        + `libevent_pthreads`
            * 这个库添加了基于`pthreads`可移植线程库的线程和锁的实现。它和`libevent_core`库是分开的，因此不需要链接到`pthreads`来使用Libevent，除非真的要以多线程方式使用Libevent
        + `libevent_openssl`
            * 这个库用于支持使用`bufferevents`和`OpenSSL`库来进行加密通信。它和`libevent_core`库是分开的，因此不需要链接到这个库来使用Libevent，除非真的要使用加密连接
* 头文件 所有Libevent公共头文件都安装在`event2`目录下(自己的CentOS环境：/usr/local/include/event2)
    - Libevent 2.0版本修改了它的api，使其更加合理，更少出错。尽可能使用Libevent 2.0版本的APIs ()
    - 老版本和2.0版本，头文件有所区分，老版本的头文件不会放到`event2`目录，新老头文件的替换参照，参考链接
    - 1.4版本前，只有一个`libevent`库(目前分成了`libevent_core` 和 `libevent_extra`库)
    - 2.0版本前，不支持锁，需要通过 不在两个线程中同时使用同一个结构体 来保证线程安全。

## 设置Libevent库

* [Setting up the Libevent library](http://www.wangafu.net/~nickm/libevent-book/Ref1_libsetup.html)
* 消息记录(Log messages in Libevent)
    - `event.h`中定义了`#define EVENT_LOG_DEBUG 0`等宏定义，用于定义消息的等级
        + 包含老版本的`#define _EVENT_LOG_DEBUG EVENT_LOG_DEBUG`等，已弃用
    - 可以自定义日志记录函数
        + 日志记录的函数指针原型：`typedef void (*event_log_cb)(int severity, const char *msg);`(定义在`event.h`中)
        + 通过`void event_set_log_callback(event_log_cb cb);`将自定义的函数指针设置给全局函数
            * 日志记录的全局函数定义为：`static event_log_cb log_fn = NULL;`，定义在`log.c`文件中
            * 若该指针为NULL，则记录时默认输出到stderr：`(void)fprintf(stderr, "[%s] %s\n", severity_str, msg);`，内容可查看`log.c`中的`event_log()`函数
        + 参考链接中演示了两个自定义日志记录的示例(丢弃和记录文件)
        + 注意：在自定义的日志记录函数中调用Libevent的函数是不安全的
    - 可以自定义错误处理函数，其中也不应该调用Libevent接口。且调用后不应该返回而将控制权给Libevent(可能出现未定义行为)
    - `log.c` 实际记录日志的操作定义在该文件中，底层为`event_log()`，`event_logv_()`对其进行封装，再由各个等级的日志记录调用该封装接口
    - 通常debug日志没有开启，debug日志比较繁琐，在大多数情况下不一定有用
        + 可通过调用`void event_enable_debug_logging(ev_uint32_t which);`来设置是否开启，送`EVENT_DBG_NONE`时是默认不开启，送`EVENT_DBG_ALL`则开启所有debug日志的支持
* 内存管理(Memory management)
    - 默认情况下，Libevent使用C库的内存管理函数来分配堆中的内存。
    - 使用`event_set_mem_functions()`函数来设置自定义的内存管理函数(malloc/realloc/free)，定义在`event.c`中(头文件为：`<event2/event.h>`)。参考链接中自定义了几个内存管理相关的函数。
        + 可以通过判断 `EVENT_SET_MEM_FUNCTIONS_IMPLEMENTED` 宏定义是否存在来判断是否开启了 `event_set_mem_functions` 函数(即是否允许自定义内存管理函数)。编译时该宏定义和函数是同时定义的，存在于同一个`#if`预编译语句块
    - 对于自定义的malloc和realloc函数，需要返回和C库布局一样的内存空间。如果在多个线程中使用Libevent，自定义函数还需要保持`线程安全`
* 锁和线程(Locks and threading)
    - Libevent结构体通常有三种方式处理多线程，有些结构天生单线程、有些结构可选锁定、有些结构总是被锁定
    - 若要获得Libevent中的锁，必须告诉Libevent使用哪些锁函数。需要在调用Libevent的分配函数来申请用于多线程的结构之前这样做。如果使用的是`pthreads`库，或者本地`Windows`线程代码，那么就比较幸运了。预先定义的函数将会设置Libevent来使用正确的`Pthreads`或`Windows`函数
    - 锁和线程结构和函数相关的文件：`thread.h`、`evthread_pthread.c`(Pthreads)、`evthread.c`
        + (Linux使用)`int evthread_use_pthreads(void);`，用于设置Libevent来使用`Pthreads`锁和线程函数。其声明在`thread.h`，实现在`evthread_pthread.c`。其中会调用到`int evthread_use_pthreads_with_flags(int flags)`，该函数会注册设置一系列Libevent基于Pthreads实现的锁相关函数

---

## 网络相关编程

* UDP
    - TCP和UDP都使用相同的网络层（IP），TCP却向应用层提供与UDP完全不同的服务。
        + TCP提供一种面向连接的、可靠的字节流服务
        + UDP是无连接的，可能丢包、无序、重复(发"abc"收到"ab" "bc")

## TCP封包解包

* TCP封包解包
    - [最简单的TCP网络封包解包](https://developer.aliyun.com/article/228575)
        + 代码见(更完整一点)：[TCP/IP 网络数据封包和解包](https://blog.csdn.net/dai_jing/article/details/17914445)
    - TCP为什么需要进行封包解包？
        + TCP采用字节流的方式，即以字节为单位传输字节序列。那么，我们recv到的就是一串毫无规则的字节流
        + 如果要让这无规则的字节流有规则，那么，就需要我们去定义一个规则。那便是所谓的“封包规则”
    - 封包结构是怎么样的？
        + 网络封包由两部分组成：`包头`、`数据`
            - 包头域是定长的，数据域是不定长的
        + 包头必然包含两个信息：`操作码`、`包长度`
        + 伪代码如下(`* 封包伪代码`)
    - 收包中存在的一个问题（粘包，半包）
        + TCP`粘包`是指发送方发送的若干包数据到接收方接收时粘成一包，从接收缓冲区看，后一包数据的头紧接着前一包数据的尾
        + 由于丢包和TCP重传，接收者收到的数据流中可能不是完整的数据包，或者是一部分，或者粘着别的数据包，因此，接收者还需要对接收到的数据流的数据进行`分包`

* 封包结构代码

```cpp
// 数据结构题大小
#define NET_PACKET_DATA_SIZE 1024 
// 缓冲大小
#define NET_PACKET_SIZE (sizeof(NetPacketHeader) + NET_PACKET_DATA_SIZE) * 10

/// 包头
struct NetPacketHeader
{
    unsigned short      wDataSize;  ///< 数据包大小，包含封包头和封包数据大小
    unsigned short      wOpcode;    ///< 操作码
};

/// 网络数据包
struct NetPacket
{
    NetPacketHeader     Header;                         ///< 包头
    unsigned char       Data[NET_PACKET_DATA_SIZE];     ///< 数据
};

/// 测试1的网络数据包定义
// 即TCP传输的应用层数据结构，以字节流的方式传输，接收方判断收到完整的包后即为本示例结构，然后数据分包给应用层处理
struct NetPacket_Test1
{
    int     nIndex;
    char    name[20];  // 本示例是在32位系统的对齐补齐方式，对齐为4字节，所以此处20字节不用另外对齐
    char    sex[20];
    int     age;
    char    arrMessage[512];
};
```

* 服务端类定义

```cpp
class TCPServer
{
public:
    // 省略构造析构对句柄的创建和销毁

    /// 发送数据
    bool SendData(unsigned short nOpcode, const char* pDataBuffer, const unsigned int& nDataSize);
private:
    // 省略监听和accept到的socket句柄

    char m_cbSendBuf[NET_PACKET_SIZE];
};
```

* 服务端发送(封包)

```cpp
// 只发送一个sizeof测试结构的数据，按字节方式发送
bool TCPServer::SendData( unsigned short nOpcode, const char* pDataBuffer, const unsigned int& nDataSize )
{
    // 让指针指向m_cbSendBuf，并且定义的是包头结构体指针，便于操作前面 sizeof(包头) 字节内容
    // m_cbSendBuf 中除了要发送的数据外，会在前面多加一个封包头
    NetPacketHeader* pHead = (NetPacketHeader*) m_cbSendBuf;
    pHead->wOpcode = nOpcode;//操作码
 
    // 数据封包
    if ( (nDataSize > 0) && (pDataBuffer != 0) )
    {
        // 要发送的数据
        CopyMemory(pHead+1, pDataBuffer, nDataSize); 
    }
 
    // 发送消息，额外加了一个包头，并更新了包头字段的值
    const unsigned short nSendSize = nDataSize + sizeof(NetPacketHeader);//包的大小是发送数据的大小加上包头大小
    // 长度更新到包头(包头结构长度+原数据长度)
    pHead->wDataSize = nSendSize;//包大小
    int ret = ::send(mAcceptSocket, m_cbSendBuf, nSendSize, 0);
    return (ret > 0) ? true : false;
```

* 客户端类定义

```cpp
class TCPClient
{
public:
    /// 主循环，里面循环接收字节流，并进行解包分包判断
    void run();

    /// 处理网络消息，传入的已经是完整大小的应用层结构，里面转换一下指针即可直接按结构题使用
    // NetPacket_Test1* pMsg = (NetPacket_Test1*) pDataBuffer;
    bool OnNetMessage(const unsigned short& nOpcode, 
        const char* pDataBuffer, unsigned short nDataSize);
private:
    // 省略服务端句柄 mServerSocket

    char m_cbRecvBuf[NET_PACKET_SIZE]; // 接收buf，可能不满一个完整的应用层结构体
    char m_cbDataBuf[NET_PACKET_SIZE]; // 应用层数据
    int  m_nRecvSize;                  // 接收到buf的长度，对应的是 m_cbRecvBuf 的长度
}

```

* 客户端接收(解包和分包)

```cpp
void TCPClient::run()
{
    int nCount = 0;
    for (;;)
    {
        // 接收数据，往 m_cbRecvBuf 中继续追加，请求读取大小足够大，实际读取到 nRecvSize 大小
        int nRecvSize = ::recv(mServerSocket,
            m_cbRecvBuf+m_nRecvSize, 
            sizeof(m_cbRecvBuf)-m_nRecvSize, 0);
        if (nRecvSize <= 0)
        {
            // 服务端close断开的话，被动方变为 CLOSE_WAIT
            std::cout << "服务器主动断开连接!" << std::endl;
            break;
        }
 
        // 保存已经接收数据的大小
        m_nRecvSize += nRecvSize;
 
        // 接收到的数据够不够一个包头的长度
        // 不够一个包头大小(主要为了获取该包头里的数据长度)，则会进入for(;;)的下一次循环接收
        while (m_nRecvSize >= sizeof(NetPacketHeader))
        {
            // 收够5个包，主动与服务器断开
            if (nCount >= 5)
            {
                ::closesocket(mServerSocket);
                break;
            }
 
            // 读取包头
            NetPacketHeader* pHead = (NetPacketHeader*) (m_cbRecvBuf);
            const unsigned short nPacketSize = pHead->wDataSize;
 
            // 判断是否已接收到足够一个完整包的数据
            // 此处的大小是从包头读取的，有可能不只一个 包头+sizeof应用层结构
            if (m_nRecvSize < nPacketSize)
            {
                // 还不够拼凑出一个完整包，则退出while，进行for(;;)的循环读取
                break;
            }
 
            // 拷贝到数据缓存
            // 此处只拷贝包头指定的长度 nPacketSize ，剩余数据还是留在buf中
            // m_cbDataBuf 每次覆盖
            CopyMemory(m_cbDataBuf, m_cbRecvBuf, nPacketSize);
 
            // 从接收缓存移除
            // 把本次完整的包放到应用层数据中后，就从buf中移除对应数据(通过前移的方式)
            // "head-abc,hea"，此处不是 nPacketSize？ memmove(buf, buf+整包大小, buf大小)？不应该是buf-整包大小吗
                // 这样会导致移动后，原有的一些位置还保留了原值，memmove与memcopy类似，只是内存重叠更灵活
            // 注意此处move(copy)整个buf大小，保证偏移整包后，前移后原来最后的位置都清0了，把相应的'\0'也做了move
            MoveMemory(m_cbRecvBuf, m_cbRecvBuf+nPacketSize, m_nRecvSize);
            m_nRecvSize -= nPacketSize;
 
            // 解密数据，对 m_cbDataBuf 做处理，网络序(大端)转本地字节序
            // ...
 
            // 分派数据包，让应用层进行逻辑处理
            pHead = (NetPacketHeader*) (m_cbDataBuf);
            const unsigned short nDataSize = nPacketSize - (unsigned short)sizeof(NetPacketHeader);
            // 跳过封包头后的数据及其大小
            OnNetMessage(pHead->wOpcode, m_cbDataBuf+sizeof(NetPacketHeader), nDataSize);
 
            ++nCount;
        }
    }
 
    std::cout << "已经和服务器断开连接!" << std::endl;
}

```







