## libevent

* GitHub：[libevent](https://github.com/libevent/libevent)

* 文档：[Fast portable non-blocking network programming with Libevent](http://www.wangafu.net/~nickm/libevent-book/)
    - 文档开始的示例演示了不同网络IO的问题：
        + (http://www.wangafu.net/~nickm/libevent-book/01_intro.html)
        + 阻塞IO，用`send`和`recv`向 www.google.com 发送http GET
            * `问题`：没接收到数据前recv不会返回；send没有刷新它的输出到内核的写缓冲中也不会返回
            * 如果同时有多个连接要处理，更具体一点，比如想读取两个连接的数据，由于读取是阻塞的，此时就不知道应该先读取哪个了
            * 对于上述阻塞IO的问题，可通过多进程或多线程处理(相比于多个进程，更倾向于创建线程池)。
        + 接收到socket请求后fork进程处理
            * 但是`问题`是：
                - 进程创建(甚至是线程创建)在某些平台上是非常昂贵的(资源消耗);
                - 如果同时要处理成千或者上万连接，每个CPU处理上万线程并不比处理少数线程更高效(上下文切换)
            * 如果线程/进程不是处理上面多连接问题的答案，那么答案是什么?
                - 非阻塞IO
        + 非阻塞IO
            * 设置sockets为非阻塞，Unix下：`fcntl(fd, F_SETFL, O_NONBLOCK);`
            * 示例中演示了一个对所有sockets设置为非阻塞 `busy-polling` (轮询) 的例子
                - 将socket文件描述符设置为非阻塞后，再进行`recv`读取，读取返回<0时，判断`errno == EAGAIN`(此时内核中没有可供该句柄读取的数据)，则先循环处理其他句柄，下一次再来对该连接句柄recv
            * 但是这样的`问题`是性能很糟糕：
                - 第一，当所有连接都没有数据时，循环将会快速无限轮询下去，消耗大量CPU周期
                - 第二，当处理到来的一个或两个连接时，不论是否有数据都需要给每个连接进行一次系统调用
            * 因此我们需要的方式是：内核一直等待，直到某个socket准备好给我一些数据时，进行通知
                - 最早的解决方式是使用`select`
        + `select`
            * 见链接中的示例，`select`会等到有socket准备好时，修改描述符集(readset、writeset、exceptionset)只保留准备好的描述符
                - `FD_ZERO(set)` 宏 清理文件描述符集fdset
                - `FD_SET(fd,set)` 向文件描述符集fdset中新增文件描述符fd (listener描述符)
                - `FD_CLR(fd,set)` 将对应fd移除
                - `FD_ISSET(fd,set)`来检测(返回true) 文件描述符集fdset中是否存在文件描述符fd(`select`之后set中只保留准备好的fd)
                - `select`返回后会把以前加入的但并无事件发生的fd清空，所以循环每次开始`select`前都要重新从array取得fd逐一加入（通过`FD_SET`，用前先将描述符集`FD_ZERO`），扫描array的同时取得fd最大值maxfd，用于select的第一个参数(maxfd+1)
                - 通过`FD_ISSET(fd,set)`判断指定描述符fd存在于指定set中时，调用处理函数(fd收到时已设置为nonblock，处理函数中send/recv失败时判断errno为 EAGAIN 的情况)
                - 示例中给每个接收到的fd定义了一个数据结构，由于fd不能超过FD_SETSIZE，所以定义成了结构体指针数组，fd作为index
            * 但是还是没有解决`问题`：
                - 生成和读取`select`花费的时间和提供给它的最大fd(即指定maxfd+1)是成比例的，当socket很多时`select`调用花费会急剧增加(用户态时花费跟FD_SET添加进去的感兴趣fd数量成比例；而内核态中，跟整个程序所使用的所有fd的数量成比例，而不管有多少fd被添加到感兴趣描述符集)
            * 不同的系统提供了不同的替换`select`的替代函数，包含`poll()`(Linux), `epoll()`(Linux), `kqueue()`(BSDs，包含Darwin), `evports`(Solaris), 和 `/dev/poll`(Solaris)
                - 这些都比`select()`有更好的性能
                - 除了`poll()`，其他函数都提供`O(1)`的时间复杂度，对一个socket来 添加、删除、IO就绪通知
                - 但是这些高效的函数并没有成为广泛的标准，在不同的系统上有各自的函数(且不包含其他函数)，所以如果要开发一个`可移植`的`高效`的异步应用，需要对这些做一个抽象包装。
                - 这就是`Libevent API`的最低级别做的事情，它提供了对于这些`select()`的替代函数一致的接口，在不同的操作系统上使用最高效的替代版本
        + `libevent`
            * 链接示例中演示了使用`Libevent`的低层次ROT13服务(编译时`-levent`)
                - 使用`Libevent 2`来替换`select()`
                - 注意，`fd_sets`现在没有了，使用`event_base`结构体(可以用`select()`, `poll()`, `epoll()`, `kqueue()`来实现)来对`事件`进行关联和解除关联
    - 对于socket server端，`bind()`前一般都对要监听的socket设置一下 `SO_REUSEADDR` 选项
        + `int one = 1; setsockopt(listener, SOL_SOCKET, SO_REUSEADDR, &one, sizeof(one));`
        + 关于`SO_REUSEADDR`(和`SO_REUSEPORT`)的说明
            * 一个socket发送一段buffer时，`send()`返回成功但是不意味着实际已经发出去了，而表示数据已经添加到了内核中的`send buffer`。(对于UDP，数据一般很快发送出去了，但是对于TCP而言会有相对较长的时延)
            * 结果是当关闭一个socket时，可能buffer中还有没发出的数据。当主动关闭连接而socket还有数据要发送时，socket将会进入`TIME_WAIT`状态。 socket将会等待到未发出的数据发送完成或者达到超时后被强制关闭。
                - (当调用`close()`时，将会尝试发送send buffer中的所有数据。？ 此时是否会阻塞close()直到延迟时间超时或者发送完成？[close()、shutdown()函数](https://www.cnblogs.com/f-ck-need-u/p/7623252.html#2-6-close-shutdown-))
                    + `man 3 setsockopt`(注意若不设置man的page页为3(POSIX Linux)，该函数则是展示了第2页(Linux)中的内容，其中并没有说明选项)，查看`SO_LINGER`项可知，若设置了该选项则会阻塞；若没设置，则系统处理close()时会让进程尽可能快地继续下去
                - 内核将socket关闭前的这段等待时间，称为`Linger Time`(延迟时间)。大多数系统上的延迟时间是全局可配置的，默认情况下相当长(很多系统上配置2分钟)
                - 配置`SO_LINGER`选项可以将`Linger Time`配置得更长或更短，甚至可以禁止该延时时间。 配置该选项需要送一个`struct linger`结构体，其中包含两个int字段：开关(`int l_onoff`)和延迟时间(`int l_linger` Linux上指秒数)。
                    + 开关为0(关闭)时，保持TCP默认操作：等到(send buffer中的)未发出数据发送结束或者达到默认延迟时间后超时；
                    + 开关为非零，延迟时间设置为0时：则内核立即关闭socket连接，send buffer中未发出的数据被丢弃，并向对端发送`RST`，此时不会有`TIME_WAIT`状态
                    + 开关为非零，延迟时间非零时：send buffer中未发出的数据按设置的延迟时间发送，达到延迟还未发完则关闭socket
                    + 关于`SO_LINGER`的设置，可参考：[Linux网络编程socket选项之SO_LINGER,SO_REUSEADDR](https://blog.csdn.net/feiyinzilgd/article/details/5894300)
                - 完全禁止延迟时间是一个非常糟糕的主意。如果完全禁止则每次关闭socket都是强制关闭而不是优雅关闭。
            * 如果没有设置`SO_REUSEADDR`：`TIME_WAIT`状态的socket被认为还是绑定在源地址和端口上，并且任何尝试向相同地址和端口绑定(`bind`)新socket的操作都会是失败的，直到socket被真正关闭，这可能会花费`Linger Time`配置的延迟时间。所以关闭连接后立即重新绑定源地址(比如重启的情况)很多情况下是会失败的。
            * 而如果尝试绑定的socket配置了`SO_REUSEADDR`：绑定到相同源地址和端口上的`TIME_WAIT`状态的socket则会被忽略(这些绑定相同地址的“半死亡”状态的socket有可能有副作用，但是实践中相当罕见)
            * 参考：[How do SO_REUSEADDR and SO_REUSEPORT differ?](https://stackoverflow.com/questions/14388706/how-do-so-reuseaddr-and-so-reuseport-differ)
            * 另外参考链接中的`SO_REUSEPORT`
                - `SO_REUSEPORT`允许将任意数量的套接字绑定到完全相同的源地址和端口，只要`所有之前`绑定的套接字在绑定之前都设置了`SO_REUSEPORT`。
                - 如果第一个bind的socket没有设置`SO_REUSEPORT`，则其他socket就算设置了`SO_REUSEPORT`来绑定相同地址也会失败
                - `SO_REUSEPORT`并不意味着`SO_REUSEADDR`，如果已被绑定的*没有*设置`SO_REUSEPORT`的socket关闭后为`TIME_WAIT`状态时，新的设置为`SO_REUSEPORT`的绑定也会失效。
                    + 要重新绑定`TIME_WAIT`的地址，需要新的socket设置为`SO_REUSEADDR`
                    + 或者`TIME_WAIT`的socket和新绑定的socket，都需要设置为`SO_REUSEPORT`
                - 当然，`SO_REUSEPORT`和`SO_REUSEADDR`在一个socket上可以同时设置
