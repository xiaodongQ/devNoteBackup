## libevent

* GitHub：[libevent](https://github.com/libevent/libevent)

* 文档：[Fast portable non-blocking network programming with Libevent](http://www.wangafu.net/~nickm/libevent-book/)
    - 文档开始的示例演示了不同网络IO的问题：
        + (http://www.wangafu.net/~nickm/libevent-book/01_intro.html)
        + 阻塞IO，用`send`和`recv`向 www.google.com 发送http GET
            * 问题：没接收到数据前recv不会返回；send没有刷新它的输出到内核的写缓冲中也不会返回
            * 如果同时有多个连接要处理，更具体一点，比如想读取两个连接的数据，由于读取是阻塞的，此时就不知道应该先读取哪个了
        + 接收到socket请求后fork进程处理
            * 对于上述阻塞IO的问题，可通过多进程或多线程处理(相比于多个进程，更倾向于创建线程池)。
            * 但是问题是：
                - 进程创建(甚至是线程创建)在某些平台上是非常昂贵的(资源消耗);
                - 如果同时要处理成千或者上万连接，每个CPU处理上万线程并不比处理少数线程更高效(上下文切换)
        + 非阻塞IO
            * 如果线程不是处理上面多连接问题的答案，那么答案是什么?
            * 设置sockets为非阻塞，Unix下：`fcntl(fd, F_SETFL, O_NONBLOCK);`
            * 示例中演示了一个对所有sockets设置为非阻塞 busy-polling (轮询) 的例子
                - 将socket文件描述符设置为非阻塞后，再进行`recv`读取，读取返回<0时，判断`errno == EAGAIN`(此时内核中没有可供该句柄读取的数据)，则先循环处理其他句柄，下一次再来对该连接句柄recv
            * 但是这样的问题是性能很糟糕：
                - 第一，当所有连接都没有数据时，循环将会快速无限轮询下去，消耗大量CPU周期
                - 第二，当处理到来的一个或两个连接时，不论是否有数据都需要给每个连接进行一次系统调用
            * 因此我们需要的方式是：内核一直等待，直到某个socket准备好给我一些数据时，进行通知
                - 最早的解决方式是使用`select`
        + `select`
            * 见链接中的示例，`select`会等到有socket准备好时，修改描述符集(readset、writeset、exceptionset)只保留准备好的描述符
