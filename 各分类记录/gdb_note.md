## gdb note

### 基本命令

* 运行
    - `gdb xxx`之后，`run`/`r`可以在gdb下运行命令；
    - 如果命令需要参数则跟在`run`之后，或者可以用`set args`设置
* 指定源码路径
    - 使用方式
        + `dir ./temp`
        + `dir /root/cpp_path/temp`
* 显示程序当前停住的代码行附近的代码
    - `list`/`l`，也可指定特定文件和函数附近`list [file:]function`
        + e.g. `list func`
        + e.g. `list main.cpp:11`
    - 可设置一次list出现的行数
        + `set listsize 20`
        + `show listsize`
    - `list 5,10` 5到10行
    - `l test.c:1` 列出指定文件源码
        + `l test.c:1,test.c:3`
* 显示栈帧
    - 如果遇到断点而暂停执行，或者coredump可以显示栈帧
    - 通过`backtrace`/`bt`可以显示栈帧，`bt full`可以显示局部变量
        + 不带参数：打印当前调用函数的栈帧信息，每个栈帧显示一行
        + `backtrace <n>` n 为正整数时，表示打印栈顶 n 层的栈信息；n 为负整数时，那么表示打印栈底 n 层的栈信息
    - 调用栈是被在内存中分割成一些连续的相邻的数据块，它们被称为“`栈帧`”，简称`“帧”`。
        + 每个帧都包含有调用一个函数的相关数据，包括被调用函数的局部变量和被调用函数的地址等等
        + `frame`/`f`是非常有用的命令，它可以用来显示当前帧的信息
        + 如果没有参数，就是当前行的信息
        + `info proc` 查看进程
* 断点
    - 添加
        + `break` / `b`
            * break line-number                    在执行给定行之前
            * break function-name                  在进入指定的函数之前
                - 如果是类成员函数，需要加上类名，如：`b CFilePro::recvFileCli`
                - 若定义时指定了命名空间，则打断点时也需指定，如：`b XDNS::CFilePro::recvFileCli`
            * break line-or-function if condition  如果condition（条件）是真，程序到达指定行或函数时停止
                - `break 行号 if 条件`
        + 可以在各个原文件中设置断点
            * break filename:line-number
            * break filename:function-name
        + 回车会在上一个位置再次设置一个断点
        + 如果需要断点在`main()`处，直接执行`start`就可以
    - 查看
        + `info break`
    - 删除
        + `delete`
            * delete 5 (指定编号)
            * delete 1-10(连续的断点号)
        + `clear`
            * clear list.c:12           //删除文件：行号的所有断点
            * clear 12                  //删除行号的所有断点
            * clear list.c:list_delet   //删除文件：函数的所有断点
            * clear 删除断点是基于行的，不是把所有的断点都删除
    - 临时断点
        + 在使用gdb调试时，如果想让断点只生效一次，可以使用`tbreak`命令（缩写为`tb`），和设置断点的过程一样
            * 临时断点13显示:*del*, 普通断点:*keep*
    - 条件断点
        + `break 行号 if 条件`，意思是只有在条件满足的时候，断点才会被触发
        + `b main.cpp:222 if i==100` (i为100时触发断点)
    - 忽略断点
        + 在设置了断点之后，可以使用命令`ignore 断点编号i cnt`来忽略断点，意思是接下来的cnt次编号为i的断点触发都不会让程序暂停，只有第cnt+1次断点触发才会让程序暂停

```sh
# 临时断点，显示为del，而普通断点显示为keep
Num     Type           Disp Enb Address            What
13      breakpoint     del  y   0x0000000000415383 in Thread_func(void*) at src/func.cpp:224
14      breakpoint     keep y   0x0000000000415353 in Thread_func(void*) at src/func.cpp:222
```

* 单步执行
    - 单步执行有两个命令`next`/`n`和`step`/`s`
    - 两者的区别是`next`遇到函数不会进入函数内部，`step`会执行到函数内部
        + 如果没有函数调用，s的作用与n的作用并无差别，仅仅是继续执行下一行
        + 后面也可以跟数字，表明要执行的次数
    - `s`进入函数单步调试时，执行`finish`继续完成该函数调用
        + 当遇到没有调试信息的函数，`s`命令是否跳过该函数，可以通过`step-mode`选项设置
        + `show step-mode` 查看
        + `set step-mode on` 不跳过
        + `set step-mode off` 跳过，默认情况下，它是会跳过的
    - `skip` 跳过执行
        + 使用方式：
            * `skip function add` 使用`step`将不会进入add函数
            * `skip file gdbStep.c` 这样gdbStep.c中的函数都不会进入
        + `info skip`         查看skip情况
        + `skip delete [num]` 删除skip
        + `skip enable [num]` 使能skip
        + `skip disable [num]` 不使能skip
* 继续执行
    - 调试时，使用`continue`/`c`命令继续执行程序。
        + 程序遇到断点后再次暂停执行；如果没有断点，就会一直执行到结束
    - `until`/`u`命令
        + 继续运行到指定位置
        + 如：`u main.cpp:29` 继续运行到29行
* attach到进程
    - `gdb attach pid号`
    - `bt full`查看当前栈帧。此时可使用`print`等查看信息
    - 还可以通过`info proc`查看进程信息
* 生成内核转储文件
    - 设置系统允许生成core文件，参考下面记录的`### core`章节
    - `gcore pid号`也可生成core，无需停止正在执行的程序以获得转储文件
    - `gdb -c core文件 进程名` 查看转储文件，`bt`查看转储时的调用栈
* [调试器GDB的基本使用方法](https://www.jianshu.com/p/adcf474f5561)
* [GDB调试入门指南](https://zhuanlan.zhihu.com/p/74897601)


* 多线程调试
    - [gdb 调试多线程](https://www.cnblogs.com/sssblog/p/10815184.html)
    - `info inferiors` 查看进程
    - `info threads` 查看所有线程
        + `*` 为当前线程
    - `thread n` 切换到第n个线程
        + 要操作指定线程，则使用编号，不是线程号
    - `bt` 查看线程栈结构
    - `break testThread.cpp:5 thread all`
        + 在所有线程中相应的行上设置断点
    - `thread apply ID1 ID2 command`
        + 让一个或者多个线程执行GDB命令command
    - `thread apply all command`
        + 让所有被调试线程执行GDB命令command
        + e.g. `thread apply all bt` 打印所有线程堆栈(类似pstack)
    - `set scheduler-locking on`
        + 只运行当前线程
        + 可设置线程是以什么方式来执行指定命令
            * `set scheduler-locking on 命令`
            * 如单步时都用当前线程：`set scheduler-locking on step`
    - `set scheduler-locking off`
        + 所有线程并发运行
* 查看类信息
    - `p this` 打印类地址
    - `p *this` 打印类信息
    - `set print pretty on`
        + 以树形打印对象的成员，可以清晰展示继承关系，设置为off时对象较大时会显示比较乱
    - `set print object on`
        + 这个选项可以看到派生对象的真实类名，虽然ptype也可以打印出对象
        + `ptype obj/class/struct`
            * 查看obj/class/struct的成员(只打印定义)，但是会把基类指针指向的派生类识别为基类(不多态显示)
    - 打印完整string信息
        + `show print elements` 查看限制打印的字符串长度
        + `set print elements 0` 不限制
    - 另外
        + `set print array on`
        + `set print union on`
        + `set print elements 0` 设置这个最大限制数 设置为没有限制
        + 用`bt full`命令显示各个函数的局部变量值
            * bt full n，意思是从内向外显示n个栈帧，及其局部变量

### gdb调试死锁

* gdb调试死锁
    - gdb -> 然后，attach 进程号
    - `thread apply all bt` 和`pstack`打印结果差不多，找关键字
    - 也可以
        + info threads
        + thread 某个线程，e.g. `thread 3`
        + 然后bt查看调用栈
        + 再thread 进入其他线程，e.g. `thread 2`，然后bt查看调用栈，若在等待同一个锁，则说明也有死锁
    - 可参考：[GDB：调试死锁](https://blog.csdn.net/guowenyan001/article/details/46238355)
* pstack
    - `pstack`/`gstack` 是一个shell脚本，里面使用gdb bt和sed过滤统计一些程序调用栈，pstack是指向gstack的软连接
        + `which pstack` 路径为 /usr/bin/pstack
        + `ll /usr/bin/pstack`，得到 `lrwxrwxrwx. 1 root root 6 8月  22 20:26 /usr/bin/pstack -> gstack`，可知pstack实际是指向gstack的软链接
    - `pstack 进程号`，找`lock_wait`关键字，多次执行若一直阻塞在某个锁上，很有可能是死锁(不过不排除是程序执行较快多次正常加锁解锁)

### 加载符号表

* [使用GDB调试将符号表与程序分离后的可执行文件](https://www.cnblogs.com/dongc/p/9690754.html)
* `a.out`应用程序为例
    - `g++ test.cpp -g` 生成`a.out`
* 生成符号文件：
    - `objcopy --only-keep-debug a.out a.out.symbol`
* 生成发布文件
    - 去掉执行文件中的符号：
        + a. `objcopy --strip-debug a.out a.out`
        + b. `strip a.out`
    - 关联
        + `objcopy --add-gnu-debuglink=a.out.symbol a.out`
* 确认调试信息文件链接
    - `objdump -s -j .gnu_debuglink a.out`
* gdb调试并加载符号文件(`-s`)：
    - `gdb -c core文件 -s 符号表文件 bin可执行文件`
    - 或者 把符号文件放在应用程序同级目录
* 找不到代码路径
    - `dir src_dir` 拷贝代码后，通过dir指定

* 加载符号表
    - `gdb -c /corefile/core-a.out-2112-1605365074 a.out`
    - gdb的默认搜索路径
        + `show debug-file-directory`
        + 看默认是："/usr/lib/debug"，可以把符号文件拷贝到该目录
    - 设置全局调试信息目录的名称(下面加载符号表的几种方法，不需要进行本处的设置)
        + `set debug-file-directory /root/workspace/cpp_path/cpp_test`
        + 若没有前面的 "添加调试信息文件链接"操作，然后符号文件移动到其他目录，设置路径后也不会成功加载符号表
        + 实验结果：添加了关联且符号文件移动到其他目录，设置了debuglink也没成功
    - 如下方式加载符号表
        + 方法1：到gdb中执行
            * `symbol-file /root/workspace/cpp_path/cpp_test/a.out.symbol`
        + 会读取符号文件`a.out.symbol`
    - 若gdb时未指定执行文件和core文件
        + 可用`file`命令载入主执行文件
        + 再用`core`命令载入coredump文件
        + `info sharedlibrary`就可以查看链接的库信息了
        + 参考：[GDB动态库搜索路径](https://blog.csdn.net/_xiao/article/details/23289971)
    - 也可以通过下列方式加载符号表
        + 方法2： `gdb -c /corefile/core-a.out-2112-1605365074 -e a.out -s a.out.symbol`
            * `-c` 指定core文件
            * `-e` 指定binary，用线上的binary即可
                - 不用`-e`则加载不到符号表
            * `-s` 指定符号表
            * 另外其他选项
                - `-d directory` 将目录加到源代码搜索路径中
        + 方法3：
            * `gdb -c /corefile/core-a.out-2112-1605365074 -se a.out.symbol`
            * 用`-se`指定符号文件的同时指定其为执行文件，此时不用加`a.out`了
    - `strace gdb -c corefile testbin >> strace_search_symbol.log 2>&1`
        + 可以看到加载符号表的过程

(gdb) bt
#0  0x00007f874f04a337 in raise () from /lib64/libc.so.6
#1  0x00007f874f04ba28 in abort () from /lib64/libc.so.6
#2  0x00007f874f043156 in __assert_fail_base () from /lib64/libc.so.6
#3  0x00007f874f043202 in __assert_fail () from /lib64/libc.so.6
#4  0x0000000000400883 in main ()


### core文件

* 允许生成 core文件
    - [Linux生成core文件、core文件路径设置](https://blog.csdn.net/u011417820/article/details/71435031)
    - `ulimit -c 设置大小`
        + 注意在终端敲命令执行ulimit时，只会影响本终端，其他终端起的进程并不会受影响，所以在启动服务的终端设置
        + 在系统配置中修改：`/etc/security/limits.conf`，(.bashrc中添加只影响终端session)
    - `/etc/sysctl.conf`
        + 添加配置项：`kernel.core_pattern=/corefile/core-%e-%p-%t`，
            * 然后`sysctl -p`生效，会改变`/proc/sys/kernel/core_pattern`中的值
            * **注意** `=`后面的内容不要用`"`，否则设置的路径不生效，不生成core
        + `/proc/sys/kernel/core_pattern` 临时设置core文件保存位置和文件名格式
            * 重启后会恢复，完全生效需修改sysctl.conf
        + "|/usr/libexec/abrt-hook-ccpp %s %c %p %u %g %t e %P %I %h" CentOS7默认内容
            * 注意最后的`/`后面是文件命名格式
            * `%p` 添加pid
            * `%u` 添加当前uid
            * `%g` 添加当前gid
            * `%s` 添加导致产生core的信号
            * `%t` 添加core文件生成时的unix时间
            * `%h` 添加主机名
            * `%e` 添加导致产生core的命令名(触发core生成的进程/线程名，不过名称可能不完整)(建议程序中单独设置线程名)
        + 可以查看`/proc/sys/kernel/core_pattern`中的内容，已改成了`/etc/sysctl.conf`设置的格式
            * 注意设置的目录需要存在(提前新建好)，否则不会生成core文件
        + 由于corefile都比较大，可监控生成core文件的个数，防止磁盘被占满
            * 可固定一个存储位置(如/corefile/)并进行脚本监控个数，写一个监控脚本，脚本执行可添加到crontab中
    - `kill`
        + `kill -l` 打印信号列表，(man中提示在`/usr/include/linux/signal.h`也可找到，看了下内容只#define了两个宏。。)
            * 找其中的包含文件，可看到定义在`/usr/include/asm-generic/signal.h`中
        + `kill -s` 指定发送的信号，信号可以以*信号名*或者数字发送