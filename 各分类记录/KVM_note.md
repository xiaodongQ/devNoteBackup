# KVM note

## 安装kvm

* 参考：[cento7安装kvm并通过kvm命令行安装centos7](https://blog.csdn.net/yulsh/article/details/91790804)
* KVM虚拟化平台构建
    - 查看硬件是否支持虚拟化
        + `egrep "vmx|svm" /proc/cpuinfo` 或者 `grep -Ei 'vmx|svm' /proc/cpuinfo`
        + 如果有过滤出`vmx`或`svm`关键字就代表支持虚拟化，`vmx`是Intel的CPU，`svm`是AMD的CPU
    - 安装KVM
        + 由于Linux内核已经将KVM收录了，在安装系统时已经加入了KVM，我们只需要在命令行模式下启用KVM即可
            * `lsmod |grep kvm` 查看模块 (已经加载了，若没加载则执行下面加载命令)
            * `modprobe kvm` 加载模块
    - KVM虚拟机创建和管理所依赖的组件介绍
        + KVM虚拟机的创建依赖`qemu-kvm`
            * 虽然kvm的技术已经相当成熟而且可以对很多东西进行隔离，但是在某些方面还是无法虚拟出真实的机器。比如对网卡的虚拟，那这个时候就需要另外的技术来做补充，而`qemu-kvm`则是这样一种技术。
            * 它补充了kvm技术的不足，而且在性能上对kvm进行了优化
        + `virt-manager`，`virt-viewer`用来管理虚拟机
        + 在创建和管理KVM虚拟机时还需要`libvirt`这个重要的组件
            * 它是一系列提供出来的库函数，用以其他技术调用，来管理机器上的虚拟机
            * 包括各种虚拟机技术，kvm、xen与lxc等，都可以调用`libvirt`提供的api对虚拟机进行管理
    - 安装KVM所需组件：
        + `yum install -y virt-* libvirt bridge-utils qemu-img qemu-kvm`
            * 由于不通外网，配置本地yum源后使用，配置方式搜索本笔记中的：`配置本地yum源步骤`
        + 安装完成后启动`libvirtd`服务：
            * `ifconfig`查看，有虚拟网卡：`virbr0`
            * 也可通过网桥管理命令查看：`brctl show`，有网桥：`virbr0`
            * `service libvirtd status`
                - 若未启动则 `service libvirtd start`
    - 创建**物理桥接**设备，
        + 可以使用`virsh`创建桥设备关联网卡到桥接设备上
            * virsh是kvm虚拟机常用的管理工具，以下是一些常用的命令(下面的章节安装虚拟机后列出来了)
                - 查看在运行的虚拟机 `virsh list`
                - 查看创建的所有虚拟机 `virsh list --all`
                - 启动虚拟机 `virsh start win10` (此处win10是虚拟机的domain名称)
                - 另外还有：挂起虚拟机、恢复被挂起的虚拟机、开机启动虚拟机、关闭虚拟机、删除虚拟机
        + 需要将`NetworkManager`服务关闭，开机启动也关闭
            * `chkconfig NetworkManager off`
            * `service NetworkManager stop`
        + 然后创建桥接设备及关联网卡到桥接设备上
            * `virsh iface-bridge enp2s0f1(可联通的局域网网卡，e.g. 10网段) br0`
                - 如：`virsh iface-bridge enp2s0f1 br0`
                - 修改后原网卡ip会变为br0的ip
            * `brctl show` 可以看到`br0`网桥
            * 若要删除
                - iface-bridge 创建网桥
                    + 查看配置文件，iface-bridge 是直接将配置文件也改写了，也就是说，通过 iface-bridge 创建的桥接，重启依然生效
                - iface-unbridge 删除网桥
                    + `virsh iface-unbridge br0`

### 创建虚拟机并安装CentOS7(命令行方式)

* 将服务成功启动后，就可以使用KVM安装虚拟机了
* 首先需要准备一个操作系统的镜像文件，这里用的是CentOS7的镜像文件
    - `/root/xd_virtual/osimage/CentOS-7-x86_64-DVD-1708.iso`
* 权限调整
    - `vim /etc/libvirt/qemu.conf`
    - 把`#user = "root`和`#group = "root"`的注释符号`#`去掉，否则下面执行`virt-install`会报`权限不够`的错
    - 修改后重启`libvirtd`服务
        + `systemctl daemon-reload` 重载配置
        + `systemctl restart libvirtd` 重启libvirtd服务
        + `systemctl status libvirtd` 查看libvirtd服务
* 使用命令行安装这个CentOS7镜像文件(交互式安装，需要手动配置语言、时区等。 自动安装方式见下面的`无人值守安装`)：

```sh
virt-install \
--name=xdguest1 \
--memory=512,maxmemory=1024 \
--vcpus=1,maxvcpus=2 \
--os-type=linux \
--os-variant=rhel7 \
--location=/root/xd_virtual/osimage/CentOS-7-x86_64-DVD-1708.iso \
--disk path=/root/xd_virtual/osimage/kvm_data/xdguest1.img,size=10 \
--bridge=br0 \
--graphics=none \
--console=pty,target_type=serial \
--extra-args="console=tty0 console=ttyS0"
```

* 创建命令解释
    - 创建一个名为`xdguest1`的1个8核cpu，内存容量为512M的，硬盘容量为10GB
        + `--name` 指定虚拟机的名称
        + `--memory` 指定分配给虚拟机的内存资源大小
            * `maxmemory` 指定可调节的最大内存资源大小，因为KVM支持热调整虚拟机的资源
            * 原来的`--ram`弃用了
        + `--vcpus` 指定分配给虚拟机的CPU核心数量
            * `maxvcpus` 指定可调节的最大CPU核心数量
            * 此外，还可以用`sockets`、`cores`和`threads`指定CPU拓扑。如果省略了这些值，那么剩下的部分将自动填充，更倾向于使用`sockets`而不是`cores`和`threads`
        + `--os-type` 指定虚拟机安装的操作系统类型
            * man中已经找不到了，有的博客说该选项已经弃用
        + `--os-variant` 指定系统的发行版本
            * 指定`--os-variant=centos7`报错了，可改成`centos7.0`(列表里没有7.7)
            * `man virt-install`查看`os-variant`选项，可用`osinfo-query os|less`命令来查看支持的发行版
        + `--location`或者`-l` 指定ISO镜像文件所在的路径，支持使用网络资源路径，也就是说可以使用URL
        + `--disk path` 指定虚拟硬盘所存放的路径及名称，`size` 则是指定该硬盘的可用大小，单位是`G`
            * 报警告：`WARNING  /root/xd_virtual/osimage/kvm_data/xdguest1.img 可能不能被管理程序访问。您将需要授予 'qemu' 用户搜索以下目录的权限： ['/root']`
            * 报错：ERROR Cannot access storage file '/root/xd_virtual/osimage/kvm_data/xdguest1.img' (as uid:107, gid:107): 权限不够
            * 通过上面的`权限调整`操作即可解决
        + `--bridge`或者`-b` 指定使用哪一个桥接网卡，也就是说使用桥接的网络模式
        + `--graphics` 指定是否开启图形，此处设置`none`表示不支持
            * 显示类型可为`vnc`(参考man手册)，一些链接中的修改`vnc`端口就是用于图形界面，此处可不用
        + `--console` 定义终端的属性，`target_type` 则是定义终端的类型
        + `--extra-args` 或者`-x` 定义终端额外的参数

* 然后会出来选择配置界面，配置语言、时区、安装源、网络、密码等信息，就和我们在VMware里安装CentOS虚拟机是一样的，只不过这个是命令行形式，而VMware里是图形界面罢了。
    - **界面说明**：
        + 没配置过选项的会显示`[!]`，配置过的会显示`[x]`
    - 首先设置语言，`输入数字1`，回车进入选择界面
        + 例如我要选Chinese就按数字68并回车即可，回车之后会让你选择是中文简体还是繁体，也是按下相应的数字并回车即可
    - 配置完成之后又会再次回到配置界面，设置时区(`输入2`)
        + 回车后会出来一个选择界面，然后选 `1) Set timezone` (2是NTP配置，不需要)
        + 一步步选择从洲定位到城市，`2) Asia` -> `64) Shanghai`
    - 安装软件类型(`4`)
        + 可选择最小化安装(默认选中了) 或者 其他安装类型
    - 配置完成之后又会再次回到配置界面，设置安装位置(`输入5`)
        + 界面会显示`--disk path`选项中`size`设置的`10G`，默认已经选中了该项，输入`c`表示继续
        + 然后会出来自动分区选项(Autopartitioning Options)，默认选中了 2) 使用所有空间，输入`c`继续
        + 然后出来分区方案选择(Partition Scheme Options)，选择 1) 标准分区，输入`1`回车
        + 然后会显示选中页面，输入`c`回车继续
    - 配置完成之后又会再次回到配置界面，设置root密码(`输入8`)
        + 输入两次密码
    - 配置完成之后又会再次回到配置界面，开始安装(`输入b`)
    - 等待安装完成
        + 自己安装耗时大概估计：18:23:25~18:27:10，6分钟左右
    - 安装完成后，回车等待重启即可
        + 登录进入可`df -h`和`free -h`查看磁盘和内存信息，和安装虚拟机指定的参数是一致的
* 安装完成后，这时我们是处于一个虚拟终端的，如果要退出来虚拟机，应该说是切出来，按`Ctrl + ]` 即可(`Ctrl+D`只会退出root登录)
    - 切出虚拟机后，可以看到`/root/xd_virtual/osimage/kvm_data/`目录下多了一个虚拟机的磁盘文件
        + `/root/xd_virtual/osimage/kvm_data/xdguest1.img`
        + `--disk path`配置的目录+文件名称
        + 安装过程中就会生成该文件
    - 在宿主机上可以
        + 查看kvm进程：`ps -fe|grep kvm`
        + 查看运行中的虚拟机：`virsh list`
        + 查看虚拟机配置文件：
            * `/etc/libvirt/qemu/`目录，`xdguest1`虚拟机的配置文件为：`xdguest1.xml`
            * `/etc/libvirt/qemu/networks/default.xml`
            * `/etc/libvirt/qemu/networks/autostart/default.xml` 为上面配置文件的软链接

#### 无人值守安装(命令行指定ks)

* 上面的安装需要交互式选择设置，不方便安装

* 无人值守安装(通过kickstart，ks文件)
    - [kvm虚拟化二： 字符界面管理及 无人值守安装](https://www.cnblogs.com/caya-yuan/p/10534606.html)
    - ks.cfg文件如下
        + `rootpw` 操作的密码，需要`grub-crypt`命令对原始密码加密
        + 由于安装源直接在`virt-install` 命令中指定，因此ks文件中没有安装源配置项
        + 由于安装的是虚拟机，而kvm中`qemu-img`创建的磁盘，在kvm中默认识别为vda、vdb...之类的磁盘，因此ks文件中，不能再像安装物理机一样指定为sda、sdb之类的磁盘
    - 开始安装，执行下面的安装执行命令

* 安装执行命令
    - 下面的磁盘可以先创建好
        + `/root/xd_virtual/osimage/kvm_data/xdks01.img`
        + e.g. `qemu-img create -f qcow2 /kvm/os/vm-01.qcow2 16G`
        + 磁盘镜像格式推荐 `qcow2` 格式

```sh
virt-install \
--name=xdks01 \
--disk path=/root/xd_virtual/osimage/kvm_data/xdks01.img,size=10,device=disk,bus=virtio,perms=rw,cache=writethrough \
--graphics none \
--vcpus=1,maxvcpus=2 \
--memory=2048,maxmemory=2048\
--location=/root/xd_virtual/osimage/CentOS-7-x86_64-DVD-1708.iso \
--network bridge=br0 \
--os-type=linux \
--os-variant=rhel7 \
--initrd-inject=/root/xd_virtual/osimage/ks.cfg \
--extra-args="ks=file:/ks.cfg console=pty,target_type=serial console=ttyS0,115200n8"
```

* ks.cfg文件

```sh
# ks.cfg， ks文件内容

# install
install
# root password 密码，此处需要用grub-crypt工具对需要的密码进行加密，获取的加密内容放到下面
rootpw  --iscrypted $6$xxxx$Nr.xxxxx/xxx/xxx/xxx.xxxx
authconfig --enableshadow --passalgo=sha512
firewall --service=ssh
text
firstboot --disable
keyboard us
lang en_US.UTF-8
selinux --disabled
timezone Aisa/Shanghai
# 配置网络
network --onboot yes --device eth0 --bootproto static --ip 192.168.0.64 --netmask 255.255.252.0 --gateway=192.168.31.254 --nameserver=1.1.1.1,8.8.8.8 --hostname testhost
# 如果局域网内有dhcp服务器，也可自动获取网络配置
#network  --bootproto=dhcp --device=eth0 --onboot=on --ipv6=auto

# 指定引导分区
zerombr
bootloader --location=mbr --driveorder=vda
# 清除硬盘分区
clearpart --drives=vda --all --initlabel
  part /boot --fstype="ext4" --ondisk=vda --size=500
  part / --fstype="ext4" --ondisk=vda --grow --size=1
  part swap --fstype="swap" --ondisk=vda --recommende

reboot

%packages --ignoremissing
@base
@core
@perl-runtime
java-1.8.0-openjdk
keepalived
expect
tree
kexec-tools
net-tools
iptables-services
perf
iperf3
valgrind
dstat
iotop
ntp
%end
```


#### 管理和操作虚拟机的一些常用命令

* 管理虚拟机的一些常用命令：
    - `virsh list`          # 查看正在运行的虚拟机
    - `virsh list --all`    # 查看创建的所有虚拟机(包括未启动的)
    - `virsh console xxx`   # 进入指定的虚拟机，进入的时候还需要按一下回车
        + 如果出现执行后卡住的情况，可以用可视化客户端进入虚拟机后进行如下操作：
            * 使用virsh console进入Linux虚拟机会使用一个tty叫ttyS0，默认情况下不允许使用ttyS0登录系统
            * `cat /etc/securetty` 检查是否有ttyS0，没有则用下面命令添加
            * `echo "ttyS0" >> /etc/securetty` 添加允许ttyS0登录系统
            * `grubby --update-kernel=ALL --args="console=ttyS0"`  更新内核参数
            * 然后重启虚拟机，即可进入
            * 参考：[VIRSH CONSOLE进入CENTOS 7虚拟机卡住不动](http://blog.ljmict.com/?p=89)
    - `virsh start xxx`     # 启动虚拟机
    - `virsh shutdown xxx`  # 关闭虚拟机
    - `virsh destroy xxx`   # 强制停止虚拟机
    - `virsh undefine xxx`  # 彻底销毁虚拟机，会删除虚拟机配置文件，但不会删除虚拟磁盘
        + `virsh define xml配置文件` # 通过新虚拟机的配置文件，定义新的虚拟机
    - `virsh autostart xxx` # 设置宿主机开机时该虚拟机也开机 **开机自启动**
        + 若设置自启动报错：Unable to find security driver for model selinux
            * 解决方式：删掉相关的虚拟机内xml文件的`seclable`标签(和selinux有关)
            * 修改虚拟机配置文件，如：`/etc/libvirt/qemu/xdguest1_copy.xml`(或该目录下其他虚拟机文件)，若没有加载则再通过`virsh edit xdguest1_copy`修改一下
            * [虚拟机在启动或者迁移(MIGRATE)时失败](http://www.selinuxplus.com/?p=621)
    - `virsh autostart --disable xxx`   # 解除开机启动
    - `virsh suspend xxx`               # 挂起虚拟机
        + 挂起后，`virsh console`连接的界面操作不了，但是不会退出(不同于shutdown时的直接退出)
        + 挂起时在ssh终端输入的指令，在恢复后还是会依次执行
    - `virsh resume xxx`                # 恢复挂起的虚拟机
    - `virsh dominfo xxx`               # 显示虚拟机的基本信息
    - `virsh dumpxml xxx`               # 显示虚拟机的当前配置文件
    - `virsh setmem xxx 51200`          # 给不活动虚拟机设置内存大小
    - `virsh setvcpus xxx 4`            # 给不活动虚拟机设置cpu个数
    - `virsh edit xxx`                  # 编辑配置文件
    + < img src="创建虚拟机截图/virsh命令示例.jpg" />

* 补充
    - [Linux下KVM虚拟机基本管理及常用命令](https://blog.csdn.net/demonson/article/details/84140143)
    - `virsh create /etc/libvirt/qemu/CentOS6.5.xml` 通过配置文件启动虚拟机
        + 和`virsh define xml配置文件`方式类似
        + [Virsh中创建虚拟机两种方式define和create的区别](https://www.cnblogs.com/kouryoushine/p/8950393.html)
            * 本质上两者一样的，都是从xml配置文件创建虚拟机
            * `define` 从xml配置文件创建主机但是不启动
            * `create` 同样是从xml配置文件创建主机，但是可以指定很多选项，比如是否启动，是否连接控制台
    - 虚拟机创建快照
        + 快照一定需要`qcow2`磁盘格式才行
        + 转换磁盘镜像文件格式为`qcow2`：`qemu-img convert -f raw CentOS6u5.raw -O qcow2 CentOS6u5.raw.qcow2`
        + 创建快照：`virsh snapshot-create CentOS6u5`
        + 查看快照：`virsh snapshot-list CentOS6u5`
        + 恢复快照：`virsh snapshot-revert CentOS6u5  1479043349`
        + 删除快照：`virsh snapshot-delete CentOS6u5 1479043349`
    - 显示虚拟机内存和cpu的使用情况
        + `virt-top`
    - 显示虚拟机分区信息
        + `virt-df xdguest1`
    - 给虚拟机添加硬盘
        + `virsh attach-disk` 添加硬盘（lvm卷）或者USB
        + `virsh detach-disk` 卸载
        + `lvcreate` 添加lvm卷
    - 改变虚拟机的参数
        + 内存
            * `virsh dominfo xdguest1 | grep memory` 查看内存
            * `virsh setmem xdguest1 524288` 动态设置内存为512MB，单位为`KB`
        + 更改CPU
            * 需要修改配置文件，因此需要停止虚拟机：`virsh shutdown kvm-1`
            * `virsh edit kvm-1`，然后修改`vcpu`标签中的CPU个数，然后通过配置文件启动虚拟机：`virsh create /etc/libvirt/qemu/kvm-1.xml`

* KVM的两种远程管理方式
* 参考：[开源虚拟化KVM（一）搭建部署与概述](https://www.cnblogs.com/hai-better/p/10588475.html)
* virt-manager(前面yum已经安装了，推荐)
    - 在xshell中的ssh中执行`virt-manager`命令，会出来操作界面(安装了Xmanager的前提下)
    - 终端上有报错提示(不过不影响工具连接)：
        + `libGL error: No matching fbConfigs or visuals found`
        + `libGL error: failed to load driver: swrast`
    - 第二次重新烧宿主机安装kvm对应组件后，执行`virt-manager`时，报下面的错，并打开不了客户端界面：
        + `(virt-manager:9052): Gtk-WARNING **: 03:25:42.683: cannot open display:`
        + 解决方式
            * 有说直接重启的，有说要安装下面的包的，先安装了包`virt-manager`试得还不行，然后重启一下就可以了
                - 所以目前不确定是 重启就能解决 还是需要 安装包之后重启就可以
                - 新环境可以先尝试重启，不行再安装包
                - (后续：新环境直接重启，可以正常调出可视化界面)
                    + 或者在~/.bashrc里添加：`export DISPLAY=xx.xx.xx.xx:0.0` (pc客户端的ip)
            * `yum -y install xorg-x11-font-utils`
            * [**打开virt-manager报错:** (virt-manager:6079): Gtk-WARNING **: 19:57:38.863: cannot open display:](https://blog.csdn.net/li1121567428/article/details/94041871?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.add_param_isCf&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.add_param_isCf)
* VNC图形化显示
    - VNC是一个优秀的远程管理软件，它有两部分组成`VNCServer`，`VNCViewer`
    - 安装
        + [Linux服务器上安装配置VNC Server](https://blog.csdn.net/aiynmimi/article/details/76850984?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)
        + 服务端
            * `yum install tigervnc tigervnc-server -y`
            * 修改配置(可不改？)：
                - `cp /lib/systemd/system/vncserver@.service /lib/systemd/system/vncserver@.service_bak`
                - `vi /lib/systemd/system/vncserver@.service`
                    + 里面的<USER>改成用户名，若用root则改成下面这样：
                    + `ExecStart=/sbin/runuser -l root -c "/usr/bin/vncserver %i"`
                    + `PIDFile=/root/.vnc/%H%i.pid`
            * 重启`systemd`
            * 设置密码，两次输入即可(可不设置？)
                - `vncpasswd`
            * 启动：`systemctl start vncserver@:1.service`
            * 自启动：`systemctl enable vncserver@:1.service`
    - 客户端登录
        + `192.168.30.9:1`
        + 防火墙关闭`iptables -F`