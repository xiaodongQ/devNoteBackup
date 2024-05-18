# Linux内核学习笔记

## 网络

[图解Linux网络包接收过程](https://mp.weixin.qq.com/s?__biz=MjM5Njg5NDgwNA==&mid=2247484058&idx=1&sn=a2621bc27c74b313528eefbc81ee8c0f&chksm=a6e303a191948ab7d06e574661a905ddb1fae4a5d9eb1d2be9f1c44491c19a82d95957a0ffb6&scene=178&cur_album_id=1532487451997454337#rd)

网络设备驱动对应的逻辑位于drivers/net/ethernet
    比如intel系列网卡的驱动：drivers/net/ethernet/intel
网络协议栈，模块代码位于kernel和net目录
    linux-3.10.89/kernel
    linux-3.10.89/net

### 初始化相关

Linux内核中的`fs_initcall`和`subsys_initcall`类似，也是初始化模块的入口。
    linux-5.10.176\net\ipv4\af_inet.c

`fs_initcall`调用**`inet_init`**后开始网络协议栈注册。通过`inet_init`，将这些函数注册到了`inet_protos`和`ptype_base`数据结构中了

```c
// linux-5.10.176\net\ipv4\af_inet.c
static int __init inet_init(void)
{
	struct inet_protosw *q;
	struct list_head *r;
	int rc;

	sock_skb_cb_check_size(sizeof(struct inet_skb_parm));

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
	...
    if (inet_add_protocol(&icmp_protocol, IPPROTO_ICMP) < 0)
		pr_crit("%s: Cannot add ICMP protocol\n", __func__);
	if (inet_add_protocol(&udp_protocol, IPPROTO_UDP) < 0)
		pr_crit("%s: Cannot add UDP protocol\n", __func__);
    ...
}
```

比如上面注册tcp协议相关接口的函数指针，定义在：`tcp_prot`

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
    ...
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
}
```

