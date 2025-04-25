# 资源下载

下载：kernel-debuginfo 和 kernel-debuginfo-common
搜索centos-debuginfo，此处选一个阿里云的：https://developer.aliyun.com/mirror/centos-debuginfo/
到阿里云的镜像站，下载比较快：
`uname -r 为：4.18.0-348.7.1.el8_5.x86_64`
则下载：
`kernel-debuginfo-4.18.0-348.7.1.el8_5.x86_64.rpm`
`kernel-debuginfo-common-x86_64-4.18.0-348.7.1.el8_5.x86_64.rpm`

systemtap、crash都需要

## kdump 和 crash

### 1、安装kdump：

`yum install kexec-tools`

配置文件中定义了保存位置：

```sh
# vi /etc/kdump.conf
# 保存位置
path /var/crash
# 生成coredump的行为
# default <reboot | halt | poweroff | shell | dump_to_rootfs> 
```

grub里定义了发生崩溃时，分配的内存：`crashkernel`，auto是自动分配，也可限制大小`crashkernel=512M`。

如果修改grub则需要更新grub配置，并使能生效

```sh
[CentOS-root@xdlinux ➜ download ]$ cat /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/cl_desktop--mme7h3a-swap rd.lvm.lv=cl_desktop-mme7h3a/root rd.lvm.lv=cl_desktop-mme7h3a/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true
```

### 2、启动kdump

```sh
[CentOS-root@xdlinux ➜ ~ ]$ service kdump status
Redirecting to /bin/systemctl status kdump.service
● kdump.service - Crash recovery kernel arming
   Loaded: loaded (/usr/lib/systemd/system/kdump.service; enabled; vendor preset: enabled)
   Active: active (exited) since Sun 2025-03-23 09:04:16 CST; 1 weeks 0 days ago
  Process: 1612 ExecStart=/usr/bin/kdumpctl start (code=exited, status=0/SUCCESS)
 Main PID: 1612 (code=exited, status=0/SUCCESS)
    Tasks: 0 (limit: 200021)
   Memory: 0B
   CGroup: /system.slice/kdump.service
```

### 3、手动触发crash

触发系统panic：
`echo c > /proc/sysrq-trigger`

### 4、检查内核转储文件

```sh
[CentOS-root@xdlinux ➜ ~ ]$ ll /var/crash 
total 0
drwxr-xr-x 2 root root 67 Mar 30 10:29 127.0.0.1-2025-03-30-10:29:58

[CentOS-root@xdlinux ➜ ~ ]$ ll /var/crash/127.0.0.1-2025-03-30-10:29:58 
total 295M
# 上次内核的dmesg信息
-rw------- 1 root root  98K Mar 30 10:29 kexec-dmesg.log
-rw------- 1 root root 295M Mar 30 10:29 vmcore
# 崩溃时的dmesg信息
-rw------- 1 root root  80K Mar 30 10:29 vmcore-dmesg.txt
```

### 5、安装crash，用于分析coredump文件

`yum install crash`

分析dump文件需要内核vmlinux，安装对应内核的dbgsym包（没有则手动下载rmp安装：http://debuginfo.centos.org）

```sh
# 1. 安装基础工具包
sudo yum install -y kexec-tools crash

# 2. 安装内核调试符号包（关键依赖）
# 手动下载rpm安装：http://debuginfo.centos.org/8/x86_64/Packages/
sudo yum install -y kernel-debuginfo kernel-debuginfo-common

# 3. 确认安装
rpm -qa | grep -E "kexec-tools|crash|kernel-debuginfo"

```

到阿里云的镜像站，下载比较快：
    `uname -r 为：4.18.0-348.7.1.el8_5.x86_64`
则下载：
    `kernel-debuginfo-4.18.0-348.7.1.el8_5.x86_64.rpm`
    `kernel-debuginfo-common-x86_64-4.18.0-348.7.1.el8_5.x86_64.rpm`
`rpm -ivh`手动安装，会安装到：`/usr/lib/debug/lib/modules`

```sh
[CentOS-root@xdlinux ➜ download ]$ ll /usr/lib/debug/lib/modules 
total 0
drwxr-xr-x 5 root root 63 Mar 30 12:04 4.18.0-348.7.1.el8_5.x86_64
[CentOS-root@xdlinux ➜ download ]$ ll /usr/lib/debug/lib/modules/4.18.0-348.7.1.el8_5.x86_64 
total 847M
drwxr-xr-x  8 root root   80 Mar 30 12:04 internal
drwxr-xr-x 13 root root  141 Mar 30 12:04 kernel
drwxr-xr-x  2 root root   52 Mar 30 12:04 vdso
-rwxr-xr-x  1 root root 847M Dec 22  2021 vmlinux
```

### 6、crash分析

方法：

```sh
# 进入crash分析界面（指定内核符号和vmcore路径）
crash /usr/lib/debug/lib/modules/$(uname -r)/vmlinux /var/crash/*/vmcore

# 常用命令：
  - bt       # 查看崩溃时的调用栈
  - log      # 查看内核日志
  - ps       # 查看崩溃时的进程状态
  - exit     # 退出
```

#### 实操

1、加载：

```sh
[CentOS-root@xdlinux ➜ download ]$ crash /var/crash/127.0.0.1-2025-03-30-10\:29\:58/vmcore /usr/lib/debug/lib/modules/`uname -r`/vmlinux

crash 7.3.0-2.el8
Copyright (C) 2002-2021  Red Hat, Inc.
Copyright (C) 2004, 2005, 2006, 2010  IBM Corporation
Copyright (C) 1999-2006  Hewlett-Packard Co
Copyright (C) 2005, 2006, 2011, 2012  Fujitsu Limited
Copyright (C) 2006, 2007  VA Linux Systems Japan K.K.
Copyright (C) 2005, 2011, 2020-2021  NEC Corporation
Copyright (C) 1999, 2002, 2007  Silicon Graphics, Inc.
Copyright (C) 1999, 2000, 2001, 2002  Mission Critical Linux, Inc.
This program is free software, covered by the GNU General Public License,
and you are welcome to change it and/or distribute copies of it under
certain conditions.  Enter "help copying" to see the conditions.
This program has absolutely no warranty.  Enter "help warranty" for details.
 
GNU gdb (GDB) 7.6
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-unknown-linux-gnu"...

WARNING: kernel relocated [324MB]: patching 103007 gdb minimal_symbol values

      KERNEL: /usr/lib/debug/lib/modules/4.18.0-348.7.1.el8_5.x86_64/vmlinux
    DUMPFILE: /var/crash/127.0.0.1-2025-03-30-10:29:58/vmcore  [PARTIAL DUMP]
        CPUS: 16
        DATE: Sun Mar 30 10:29:39 CST 2025
      UPTIME: 7 days, 01:25:30
LOAD AVERAGE: 0.00, 0.00, 0.00
       TASKS: 340
    NODENAME: xdlinux
     RELEASE: 4.18.0-348.7.1.el8_5.x86_64
     VERSION: #1 SMP Wed Dec 22 13:25:12 UTC 2021
     MACHINE: x86_64  (3792 Mhz)
      MEMORY: 31.4 GB
       PANIC: "sysrq: SysRq : Trigger a crash"
         PID: 35261
     COMMAND: "zsh"
        TASK: ffff9a25a9511800  [THREAD_INFO: ffff9a25a9511800]
         CPU: 2
       STATE: TASK_RUNNING (SYSRQ)

crash> 
```

2、ps、bt、log

```sh

crash> ps
   PID    PPID  CPU       TASK        ST  %MEM     VSZ    RSS  COMM
>     0      0   0  ffffffff96a18840  RU   0.0       0      0  [swapper/0]
>     0      0   1  ffff9a2403880000  RU   0.0       0      0  [swapper/1]
      0      0   2  ffff9a2403884800  RU   0.0       0      0  [swapper/2]
>     0      0   3  ffff9a24038ab000  RU   0.0       0      0  [swapper/3]
>     0      0   4  ffff9a24038a9800  RU   0.0       0      0  [swapper/4]
>     0      0   5  ffff9a24038ae000  RU   0.0       0      0  [swapper/5]
...

crash> bt
PID: 35261  TASK: ffff9a25a9511800  CPU: 2   COMMAND: "zsh"
 #0 [ffffb694057a3b98] machine_kexec at ffffffff954641ce
 #1 [ffffb694057a3bf0] __crash_kexec at ffffffff9559e67d
 #2 [ffffb694057a3cb8] crash_kexec at ffffffff9559f56d
 #3 [ffffb694057a3cd0] oops_end at ffffffff9542613d
 #4 [ffffb694057a3cf0] no_context at ffffffff9547562f
 #5 [ffffb694057a3d48] __bad_area_nosemaphore at ffffffff9547598c
 #6 [ffffb694057a3d90] do_page_fault at ffffffff95476267
 #7 [ffffb694057a3dc0] page_fault at ffffffff95e0111e
    [exception RIP: sysrq_handle_crash+18]
    RIP: ffffffff959affd2  RSP: ffffb694057a3e78  RFLAGS: 00010246
    RAX: ffffffff959affc0  RBX: 0000000000000063  RCX: 0000000000000000
    RDX: 0000000000000000  RSI: ffff9a2afe296858  RDI: 0000000000000063
    RBP: 0000000000000004   R8: 0000000000000456   R9: ffff9a2400057460
    R10: ffffffff959136f0  R11: ffffb694057a3d30  R12: 0000000000000000
    R13: 0000000000000000  R14: ffffffff962af240  R15: 0000000000000000
    ORIG_RAX: ffffffffffffffff  CS: 0010  SS: 0018

crash> log
[    0.000000] Linux version 4.18.0-348.7.1.el8_5.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 8.5.0 20210514 (Red Hat 8.5.0-4) (GCC)) #1 SMP Wed Dec 22 13:25:12 UTC 2021
[    0.000000] Command line: BOOT_IMAGE=(hd0,gpt6)/vmlinuz-4.18.0-348.7.1.el8_5.x86_64 root=/dev/mapper/cl_desktop--mme7h3a-root ro crashkernel=auto resume=/dev/mapper/cl_desktop--mme7h3a-swap rd.lvm.lv=cl_desktop-mme7h3a/root rd.lvm.lv=cl_desktop-mme7h3a/swap rhgb quiet
[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'

crash> kmem -i
                 PAGES        TOTAL      PERCENTAGE
    TOTAL MEM  8013423      30.6 GB         ----
         FREE  7215135      27.5 GB   90% of TOTAL MEM
         USED   798288         3 GB    9% of TOTAL MEM
       SHARED    32189     125.7 MB    0% of TOTAL MEM
      BUFFERS      915       3.6 MB    0% of TOTAL MEM
       CACHED   481884       1.8 GB    6% of TOTAL MEM
         SLAB    27734     108.3 MB    0% of TOTAL MEM

   TOTAL HUGE        0            0         ----
    HUGE FREE        0            0    0% of TOTAL HUGE

   TOTAL SWAP   262143      1024 MB         ----
    SWAP USED        0            0    0% of TOTAL SWAP
```

#### crash常用命令汇总

一、基础命令
命令	用途	示例/参数
bt	查看崩溃时的调用栈（Backtrace）	
    bt：当前任务的调用栈
    bt -l：所有CPU的调用栈
    bt <PID>：指定进程的调用栈
ps	查看崩溃时的进程状态	
    ps：所有进程列表
    ps -t：显示线程
    ps <PID>：查看特定进程的详细信息
log	查看内核日志（dmesg输出）	
    log：完整内核日志
    log -m：按时间排序日志
vm	查看内存使用情况	
    vm：系统内存统计
    vm -v：详细内存信息
sys	查看系统信息	
    sys：系统基本信息（启动时间、CPU等）
    sys config：内核编译配置
exit 或 q	退出crash工具	

二、高级调试命令
命令	用途	示例/参数
dis	反汇编指令	
    dis <函数名>：反汇编函数
    dis <地址>：反汇编指定地址的代码
struct	查看结构体定义	
    struct task_struct：查看任务结构体
    struct task_struct.comm：查看结构体成员定义
search	搜索内存中的值	
    search -u deadbeef：搜索十六进制值
    search -s "panic"：搜索字符串
irq	查看中断状态	
    irq -b：中断统计信息
mod	查看内核模块信息	
    mod：已加载模块列表
    mod -S <模块名>：查看模块的符号信息
kmem	分析内核内存分配	
    kmem -i：SLAB分配器统计
    kmem -s：内存泄漏检查
task	查看任务（进程）的详细信息	
    task <PID>：显示任务的内核栈、寄存器等
files	查看进程打开的文件描述符	
    files <PID>：显示进程的文件句柄
net	查看网络状态	
    net -s：网络设备统计
    net -S：套接字状态

三、实战示例

快速参考流程图：
启动crash → 2. 检查调用栈 (`bt`) → 3. 查看进程 (`ps`) 
   → 4. 分析内存 (`vm`/`kmem`) → 5. 反汇编关键函数 (`dis`) → 退出
