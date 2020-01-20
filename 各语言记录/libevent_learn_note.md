## libevent

* GitHub：[libevent](https://github.com/libevent/libevent)
* 个人fork学习路径：[libevent学习](https://github.com/xiaodongQ/libevent)

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
        + man手册：[poll](https://linux.die.net/man/2/poll)
            * `poll`和`select`类似，等待文件描述符集中的fd直到其准备好用于IO操作，测试示例参考libevent源码学习链接中的test_xd目录
            * `int poll(struct pollfd *fds, nfds_t nfds, int timeout);`
                - `fds`，需要监控的文件描述符集合，结构体包含：
                    + `int fd;`，打开的文件描述符。若为负，则忽略事件，且revents字段返回0
                    + `short events;`入参，指定感兴趣的事件，位掩码(bit mask)。若指定为0则所有事件忽略，且revents字段返回0
                    + `short revents;`出参，内核将实际发生的事件填入其中
                - `nfds`，指定fds的数量
                - `timeout`，超时时间 毫秒数，达到该超时之前`poll`会阻塞。设置为`负数`则一直阻塞，设置为0则立即返回(尽管无准备好的fd)
            * 优点
                - poll() 不要求开发者计算最大文件描述符加一的大小。
                - poll() 在应付大数目的文件描述符的时候速度更快，相比于select
                - 它没有最大连接数的限制，原因是它是基于链表来存储的
                - 在调用函数时，只需要对参数进行一次设置就好了
            * 缺点
                - 大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义（epoll可以解决此问题）
                - 与select一样，poll返回后，需要轮询pollfd来获取就绪的描述符，这样会使性能下降
                - 同时连接的大量客户端在一时刻可能只有很少的就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降
            * 优缺点参考：[poll 的使用方法及代码](https://blog.csdn.net/weixin_43825537/article/details/90211331)
            * 关于select、poll、epoll三者比较，可参考：[IO多路复用的三种机制Select，Poll，Epoll](https://www.jianshu.com/p/397449cadc9a)
        + man手册：[epoll](https://linux.die.net/man/4/epoll)
            * `epoll`是`poll`的一个变体，既可用作Edge Triggered ( ET )边缘触发，也可用于Level Triggered ( LT )水平触发，可以很好地扩展到监测大量fd
            * 三个系统调用用于控制使用`epoll`：
                - 参考：[epoll使用详解](https://blog.csdn.net/ljx0305/article/details/4065058)
                - `int epoll_create(int size);`
                    + 创建一个epoll实例的文件描述符句柄，该句柄用于后续所有的epoll相关接口。不再使用时需要`close`关闭句柄
                    + 从Linux2.6.8起，`size`参数就被忽略了，但需要`>0`
                - `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);`
                    + epoll句柄的控制接口，epoll的事件注册函数
                        * `epfd`，传的是`epoll_create`创建的句柄
                        * `op`，指定动作，支持以下动作：`EPOLL_CTL_ADD`注册新的fd到epfd中、`EPOLL_CTL_MOD`修改已经注册的fd的监听事件、`EPOLL_CTL_DEL`从epfd中删除一个fd
                        * `fd`，第三个参数是需要`监听`的fd
                        * `struct epoll_event *event`，告诉内核需要监听什么事件(可组合，位掩码)
                            - 包含两个成员：`__uint32_t events;`指定epoll的事件，EPOLLIN/EPOLLOUT/EPOLLPRI/EPOLLERR/EPOLLHUP/EPOLLET等等
                            - 和`epoll_data_t data;`用户数据变量
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
        + 比较
            * `./rot13_server_poll`       10线程10000请求，cost:530.000000 ms
            * `./rot13_server_select`     cost:520.000000 ms
            * `./rot13_server_accept_for` cost:860.000000 ms
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
