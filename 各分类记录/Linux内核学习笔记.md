# Linux内核笔记

## 网络

[图解Linux网络包接收过程](https://mp.weixin.qq.com/s?__biz=MjM5Njg5NDgwNA==&mid=2247484058&idx=1&sn=a2621bc27c74b313528eefbc81ee8c0f&chksm=a6e303a191948ab7d06e574661a905ddb1fae4a5d9eb1d2be9f1c44491c19a82d95957a0ffb6&scene=178&cur_album_id=1532487451997454337#rd)

网络设备驱动对应的逻辑位于drivers/net/ethernet
    比如intel系列网卡的驱动：drivers/net/ethernet/intel
网络协议栈，模块代码位于kernel和net目录
    linux-3.10.89/kernel
    linux-3.10.89/net
