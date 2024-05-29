# Linux内核学习笔记

[TOC]

## 网络

[图解Linux网络包接收过程](https://mp.weixin.qq.com/s?__biz=MjM5Njg5NDgwNA==&mid=2247484058&idx=1&sn=a2621bc27c74b313528eefbc81ee8c0f&chksm=a6e303a191948ab7d06e574661a905ddb1fae4a5d9eb1d2be9f1c44491c19a82d95957a0ffb6&scene=178&cur_album_id=1532487451997454337#rd)

网络设备驱动对应的逻辑位于drivers/net/ethernet
    比如intel系列网卡的驱动：drivers/net/ethernet/intel
网络协议栈，模块代码位于kernel和net目录
    linux-3.10.89/kernel
    linux-3.10.89/net

### 网络协议初始化

Linux内核中的`fs_initcall`和`subsys_initcall`类似，也是初始化模块的入口。
    linux-5.10.176\net\ipv4\af_inet.c

`fs_initcall`调用**`inet_init`**后开始网络协议栈注册。通过`inet_init`，将这些函数注册到了`inet_protos`和`ptype_base`数据结构中了

下面的`inet_init`很重要，定义了各类网络协议的操作api

```c
// linux-5.10.176\net\ipv4\af_inet.c
static int __init inet_init(void)
{
    struct inet_protosw *q;
    struct list_head *r;
    int rc;

    sock_skb_cb_check_size(sizeof(struct inet_skb_parm));

    // 定义tcp协议的 .close/.connect/.accept等操作api
    rc = proto_register(&tcp_prot, 1);
    if (rc)
        goto out;

    rc = proto_register(&udp_prot, 1);
    if (rc)
        goto out_unregister_tcp_proto;

    rc = proto_register(&raw_prot, 1);
    if (rc)
        goto out_unregister_udp_proto;

    rc = proto_register(&ping_prot, 1);
    if (rc)
        goto out_unregister_raw_proto;

    // 注册网络协议族的.create = inet_create
    (void)sock_register(&inet_family_ops);
    ...
    if (inet_add_protocol(&icmp_protocol, IPPROTO_ICMP) < 0)
        pr_crit("%s: Cannot add ICMP protocol\n", __func__);
    if (inet_add_protocol(&udp_protocol, IPPROTO_UDP) < 0)
        pr_crit("%s: Cannot add UDP protocol\n", __func__);
    // 添加tcp协议，里面会注册 .handler	=	tcp_v4_rcv
    if (inet_add_protocol(&tcp_protocol, IPPROTO_TCP) < 0)
        pr_crit("%s: Cannot add TCP protocol\n", __func__);
    ...
    
}
```

上面注册tcp协议相关接口的函数指针，定义：`tcp_prot`

```cpp
// linux-5.10.176\net\ipv4\tcp_ipv4.c
struct proto tcp_prot = {
    .name			= "TCP",
    .owner			= THIS_MODULE,
    .close			= tcp_close,
    .pre_connect		= tcp_v4_pre_connect,
    .connect		= tcp_v4_connect,
    .disconnect		= tcp_disconnect,
    .accept			= inet_csk_accept,
    .ioctl			= tcp_ioctl,
    .init			= tcp_v4_init_sock,
    .destroy		= tcp_v4_destroy_sock,
    .shutdown		= tcp_shutdown,
    .setsockopt		= tcp_setsockopt,
    .getsockopt		= tcp_getsockopt,
    .keepalive		= tcp_set_keepalive,
    .recvmsg		= tcp_recvmsg,
    .sendmsg		= tcp_sendmsg,
    ...
}
```

```cpp
// linux-5.10.10/net/ipv4/af_inet.c
static struct net_protocol tcp_protocol = {
    .early_demux	=	tcp_v4_early_demux,
    .early_demux_handler =  tcp_v4_early_demux,
    .handler	=	tcp_v4_rcv,
    .err_handler	=	tcp_v4_err,
    .no_policy	=	1,
    .netns_ok	=	1,
    .icmp_strict_tag_validation = 1,
};

// 注册网络协议族的 create 接口
static const struct net_proto_family inet_family_ops = {
    .family = PF_INET,
    .create = inet_create,
    .owner	= THIS_MODULE,
};
```

### socket 结构

`struct socket` 定义在 include\linux\net.h

```cpp
// include\linux\net.h
struct socket {
    socket_state		state;
    short			type;
    unsigned long		flags;
    struct file		*file;
    struct sock		*sk;
    const struct proto_ops	*ops;
    struct socket_wq	wq;
};
```

其中`struct proto_ops  *ops` 的初始化，根据`type`(`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW`)有不同的成员

proto_ops的定义：

```c
// linux-5.10.176\include\linux\net.h
// 定义各类socket的相关接口
struct proto_ops {
    int		family;
    unsigned int	flags;
    struct module	*owner;
    int		(*release)   (struct socket *sock);
    int		(*bind)	     (struct socket *sock,
                      struct sockaddr *myaddr,
                      int sockaddr_len);
    ...
}

// 上面的type也在该文件定义，注意不同架构枚举值可能不同(比如linux-5.10.176\arch\mips\include\asm\socket.h里，值是不同的)
enum sock_type {
    SOCK_STREAM = 1,
    SOCK_DGRAM  = 2,
    ...
}
```

上面ops的初始化位置：

```c
// linux-5.10.176\net\ipv4\af_inet.c
static struct inet_protosw inetsw_array[] =
{
    {
        .type =       SOCK_STREAM,
        .protocol =   IPPROTO_TCP,
        .prot =       &tcp_prot,
        .ops =        &inet_stream_ops,
        .flags =      INET_PROTOSW_PERMANENT |
                  INET_PROTOSW_ICSK,
    },

    {
        .type =       SOCK_DGRAM,
        .protocol =   IPPROTO_UDP,
        .prot =       &udp_prot,
        .ops =        &inet_dgram_ops,
        .flags =      INET_PROTOSW_PERMANENT,
       },
    ...
}

// 比如上面 SOCK_STREAM、TCP协议对应的ops如下(同在af_inet.c中定义)
const struct proto_ops inet_stream_ops = {
    .family		   = PF_INET,
    .flags		   = PROTO_CMSG_DATA_ONLY,
    .owner		   = THIS_MODULE,
    .release	   = inet_release,
    .bind		   = inet_bind,
    .connect	   = inet_stream_connect,
    .socketpair	   = sock_no_socketpair,
    .accept		   = inet_accept,
    .getname	   = inet_getname,
    .poll		   = tcp_poll,
    .ioctl		   = inet_ioctl,
    .gettstamp	   = sock_gettstamp,
    .listen		   = inet_listen,
    .shutdown	   = inet_shutdown,
    .setsockopt	   = sock_common_setsockopt,
    .getsockopt	   = sock_common_getsockopt,
    .sendmsg	   = inet_sendmsg,
    .recvmsg	   = inet_recvmsg,
    ...
}
```

### struct sock 结构

上述 `struct socket`里有`struct sock *sk;`成员，里面根据类型定义了操作(对后面跟踪`connect`流程很重要)

要看其初始化成了哪些io操作，需要看`socket`创建流程

```c
// linux-5.10.176\include\net\sock.h
struct sock {
    struct sock_common  __sk_common;
    socket_lock_t       sk_lock;
    atomic_t        sk_drops;
    int         sk_rcvlowat;
    struct sk_buff_head sk_error_queue;
    struct sk_buff      *sk_rx_skb_cache;
    struct sk_buff_head sk_receive_queue;
    ...
    struct proto		*skc_prot;
    ...
}

// linux-5.10.176\include\net\sock.h
struct sock_common {
    ...
    unsigned short		skc_family;
    volatile unsigned char	skc_state;
    ...
    // 里面定义一些sock的操作io，这部分信息是在创建socket时根据类型初始化的(sock_create)
    struct proto		*skc_prot;
    possible_net_t		skc_net;
    ...
```

上面skc_prot初始化为具体哪些信息，见下面的创建socket

### `socket()` 创建socket

调用流程：
__sys_socket -> sock_create -> __sock_create -> (net_proto_family结构)pf->create，实际调用到`inet_create`

```cpp
// linux-5.10.10/net/socket.c
int __sys_socket(int family, int type, int protocol)
{
    ...
    retval = sock_create(family, type, protocol, &sock);
    if (retval < 0)
        return retval;
    ...
}

int sock_create(int family, int type, int protocol, struct socket **res)
{
    return __sock_create(current->nsproxy->net_ns, family, type, protocol, res, 0);
}

int __sock_create(struct net *net, int family, int type, int protocol,
             struct socket **res, int kern)
{
    int err;
    struct socket *sock;
    // 
    const struct net_proto_family *pf;
    ...
    sock = sock_alloc();
    if (!sock) {
        net_warn_ratelimited("socket: no more sockets\n");
        return -ENFILE;	/* Not exactly a match, but its the
                   closest posix thing */
    }

    sock->type = type;
    ...
    // 根据协议族找到对应的 net_proto_family 结构
    // 这个在网络初始化时设置了(inet_init里调用`sock_register(&inet_family_ops);`)
    pf = rcu_dereference(net_families[family]);
    err = -EAFNOSUPPORT;
    if (!pf)
        goto out_release;

    /*
     * We will call the ->create function, that possibly is in a loadable
     * module, so we have to bump that loadable module refcnt first.
     */
    if (!try_module_get(pf->owner))
        goto out_release;

    /* Now protected by module ref count */
    rcu_read_unlock();

    // 对应协议族的create是 inet_create
    err = pf->create(net, sock, protocol, kern);
    if (err < 0)
        goto out_module_put;
    ...
    *res = sock;
    ...
}
```

```cpp
// linux-5.10.10/net/ipv4/af_inet.c
static int inet_create(struct net *net, struct socket *sock, int protocol,
               int kern)
{
    struct sock *sk;
    struct inet_protosw *answer;
    struct inet_sock *inet;
    struct proto *answer_prot;
    ...

    /* Look for the requested type/protocol pair. */
lookup_protocol:
    err = -ESOCKTNOSUPPORT;
    rcu_read_lock();
    // 根据socket类型找到对应操作信息，从全局 inetsw 中找，即找 inetsw_array
    // 对于tcp，为 struct proto tcp_prot
    list_for_each_entry_rcu(answer, &inetsw[sock->type], list) {

        err = 0;
        /* Check the non-wild match. */
        if (protocol == answer->protocol) {
            if (protocol != IPPROTO_IP)
                break;
        } else {
            /* Check for the two wild cases. */
            if (IPPROTO_IP == protocol) {
                protocol = answer->protocol;
                break;
            }
            if (IPPROTO_IP == answer->protocol)
                break;
        }
        err = -EPROTONOSUPPORT;
    }
    ...
    // struct sock 结构申请， answer_prot 是对应操作
    sk = sk_alloc(net, PF_INET, GFP_KERNEL, answer_prot, kern);
    if (!sk)
        goto out;

    err = 0;
    if (INET_PROTOSW_REUSE & answer_flags)
        sk->sk_reuse = SK_CAN_REUSE;

    inet = inet_sk(sk);
    ...

    sock_init_data(sock, sk);

    sk->sk_destruct	   = inet_sock_destruct;
    sk->sk_protocol	   = protocol;
    sk->sk_backlog_rcv = sk->sk_prot->backlog_rcv;
    ...
}
```

```cpp
// linux-5.10.10/net/ipv4/tcp_ipv4.c
struct proto tcp_prot = {
    .name			= "TCP",
    .owner			= THIS_MODULE,
    .close			= tcp_close,
    .pre_connect		= tcp_v4_pre_connect,
    .connect		= tcp_v4_connect,
    .disconnect		= tcp_disconnect,
    .accept			= inet_csk_accept,
    .ioctl			= tcp_ioctl,
    .init			= tcp_v4_init_sock,
    .destroy		= tcp_v4_destroy_sock,
    .shutdown		= tcp_shutdown,
    .setsockopt		= tcp_setsockopt,
    .getsockopt		= tcp_getsockopt,
    .keepalive		= tcp_set_keepalive,
    .recvmsg		= tcp_recvmsg,
    .sendmsg		= tcp_sendmsg,
    .sendpage		= tcp_sendpage,
    ...
}
```

### listen

代码位置：`__sys_listen`，\linux-5.10.176\net\socket.c
（Linux的系统调用在内核中的入口函数都是 `sys_xxx` ，但是如果我们拿着内核源码去搜索的话，就会发现根本找不到 `sys_xxx` 的函数定义，这是因为Linux的系统调用对应的函数全部都是由 `SYSCALL_DEFINE` 相关的宏来定义的。）

```c
// linux-5.10.176\net\socket.c
int __sys_listen(int fd, int backlog)
{
    // socket 定义在 include\linux\net.h
    struct socket *sock;
    int err, fput_needed;
    int somaxconn;

    sock = sockfd_lookup_light(fd, &err, &fput_needed);
    if (sock) {
        // 获取sysctl配置的 net.core.somaxconn 参数
        somaxconn = READ_ONCE(sock_net(sock->sk)->core.sysctl_somaxconn);
        // 取min(传入的backlog, 系统net.core.somaxconn)
        if ((unsigned int)backlog > somaxconn)
            backlog = somaxconn;

        err = security_socket_listen(sock, backlog);
        if (!err)
            // ops里是一系列socket操作的函数指针(如bind/accept)，inet_init(void)网络协议初始化时会设置
            // 其中，tcp协议的结构是 inet_stream_ops，里面的listen函数指针赋值为：inet_listen
            err = sock->ops->listen(sock, backlog);

        fput_light(sock->file, fput_needed);
    }
    return err;
}
```

* `sock = sockfd_lookup_light(fd, &err, &fput_needed)`相关说明

根据fd从fdtable里找到对应struct fd(file.h中定义)，并返回其中的socket相关数据成员(file结构的void* private_data成员)，此处即struct socket结构

```c
// linux-5.10.176\include\linux\file.h
struct fd {
    struct file *file;
    unsigned int flags;
};
```

```c
// linux-5.10.176\include\linux\fs.h
struct file {
    union {
        struct llist_node   fu_llist;
        struct rcu_head     fu_rcuhead;
    } f_u;
    struct path     f_path;
    struct inode        *f_inode;   /* cached value */
    ...
    void            *private_data;
    ...
}
```

* sock->ops->listen(sock, backlog)调用到的接口

```c
// linux-5.10.176\net\ipv4\af_inet.c
static int __init inet_init(void)
{
    ...
    // 各协议的重要结构和接口，定义在 inetsw_array(见上面的结构说明)，这里会按协议注册
    for (q = inetsw_array; q < &inetsw_array[INETSW_ARRAY_LEN]; ++q)
        inet_register_protosw(q);
    ...
}
```

```c
// linux-5.10.176\net\ipv4\af_inet.c
int inet_listen(struct socket *sock, int backlog)
{
    struct sock *sk = sock->sk;
    unsigned char old_state;
    int err, tcp_fastopen;

    lock_sock(sk);

    err = -EINVAL;
    if (sock->state != SS_UNCONNECTED || sock->type != SOCK_STREAM)
        goto out;

    old_state = sk->sk_state;
    if (!((1 << old_state) & (TCPF_CLOSE | TCPF_LISTEN)))
        goto out;

    // __sys_listen(linux-5.10.176\net\socket.c)调用时，传进来的的backlog值是min(调__sys_listen传入的backlog, 系统net.core.somaxconn)
    // 此处设置到struct socket中struct sock相应成员中： sk_max_ack_backlog
    WRITE_ONCE(sk->sk_max_ack_backlog, backlog);
    ...
}
```

### connect

```cpp
// linux-5.10.10/net/socket.c
int __sys_connect(int fd, struct sockaddr __user *uservaddr, int addrlen)
{
    int ret = -EBADF;
    struct fd f;

    f = fdget(fd);
    if (f.file) {
        struct sockaddr_storage address;

        // 用户态结构：sockaddr，拷贝到内核空间结构：sockaddr_storage
        ret = move_addr_to_kernel(uservaddr, addrlen, &address);
        if (!ret)
            // 里面调用对应协议注册的 connect op操作，对于tcp是 inet_stream_connect
            ret = __sys_connect_file(f.file, &address, addrlen, 0);
        fdput(f);
    }

    return ret;
}
```

```cpp
// linux-5.10.10/net/ipv4/af_inet.c
int inet_stream_connect(struct socket *sock, struct sockaddr *uaddr,
            int addr_len, int flags)
{
    int err;

    lock_sock(sock->sk);
    err = __inet_stream_connect(sock, uaddr, addr_len, flags, 0);
    release_sock(sock->sk);
    return err;
}
```

```cpp
// linux-5.10.10/net/ipv4/af_inet.c
int __inet_stream_connect(struct socket *sock, struct sockaddr *uaddr,
              int addr_len, int flags, int is_sendmsg)
{
    struct sock *sk = sock->sk;
    int err;
    long timeo;
    ...
    if (uaddr) {
        if (addr_len < sizeof(uaddr->sa_family))
            return -EINVAL;

        if (uaddr->sa_family == AF_UNSPEC) {
            // sk_prot的定义：#define sk_prot	__sk_common.skc_prot
            err = sk->sk_prot->disconnect(sk, flags);
            sock->state = err ? SS_DISCONNECTING : SS_UNCONNECTED;
            goto out;
        }
    }

    switch (sock->state) {
    ...
    case SS_UNCONNECTED:
        ...
        // sk_prot的定义：#define sk_prot	__sk_common.skc_prot
        // sk_prot的初始化，在socket()创建流程中，参考上面流程可知：tcp为 tcp_v4_connect
        err = sk->sk_prot->connect(sk, uaddr, addr_len);
        if (err < 0)
            goto out;

        sock->state = SS_CONNECTING;

        if (!err && inet_sk(sk)->defer_connect)
            goto out;

        /* Just entered SS_CONNECTING state; the only
         * difference is that return value in non-blocking
         * case is EINPROGRESS, rather than EALREADY.
         */
        err = -EINPROGRESS;
        break;
    ...
    }
    ...
}
```

### 接收

在服务器接收了SYN之后，会调用`tcp_conn_request`来处理连接请求，其中调用`inet_reqsk_alloc`来创建请求控制块，可见请求控制块的ireq_state被初始化为TCP_NEW_SYN_RECV；

(参考：[TCP 之 TCP_NEW_SYN_RECV状态](https://www.cnblogs.com/wanpengcoder/p/11751740.html))

```cpp

```
