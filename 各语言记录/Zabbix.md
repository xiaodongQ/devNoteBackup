## Zabbix


* 网站/服务器 的可用性 和 监控内容
    - [Zabbix 3.0 从入门到精通(zabbix使用详解)](https://www.cnblogs.com/clsn/p/7885990.html)
    - 高可靠性(高可用)，HA(High Avaiable)
    - 一个衡量可靠性的标准——X个9，X代表数字3~5
        + X个9表示在软件系统1年时间的使用过程中，系统可以正常使用时间与总时间（1年）之比
            * 1个9：`(1-90%)*365=36.5天`，表示该软件系统在连续运行1年时间里最多可能的业务中断时间是36.5天
            * 2个9：`(1-99%)*365=3.65天`，表示该软件系统在连续运行1年时间里最多可能的业务中断时间是3.65天
            * 3个9：`(1-99.9%)*365*24=8.76小时`，表示该软件系统在连续运行1年时间里最多可能的业务中断时间是8.76小时
            * 4个9：`(1-99.99%)*365*24=0.876小时=52.6分钟`，表示该软件系统在连续运行1年时间里最多可能的业务中断时间是52.6分钟
            * 5个9：`(1-99.999%)*365*24*60=5.26分钟`，表示该软件系统在连续运行1年时间里最多可能的业务中断时间是5.26分钟
            * 6个9：`(1-99.9999%)*365*24*60*60=31秒`，示该软件系统在连续运行1年时间里最多可能的业务中断时间是31秒
    - 监控内容
        + 监控硬件，温度/风扇转速等：ipmitool
        + cpu相关：lscpu、uptime、top、htop vmstat mpstat
        + 内存：free
        + 磁盘：df、dd、iotop
        + 网络：iftop、netthogs
    - 监控工具总览
        + mrtg 流量监控出图
        + nagios 监控
        + cacti 流量监控出图
        + zabbix 监控+出图

### Zabbix介绍

- 介绍
    + `Zabbix` 是由 `Alexei Vladishev` 开发的一种网络监视、管理系统，基于 Server-Client架构。可用于监视各种网络服务、服务器和网络机器等状态。
        * `Alexei Vladishev` 是 `Zabbix LLC` 公司的CEO兼创始人
    + Server 端基于 C语言、Web 管理端 frontend 则是基于 PHP 所制作的
    + 在客户端如 UNIX, Windows 中安装 Zabbix Agent 之后，可监视 CPU Load、网络使用状况、硬盘容量等各种状态。而就算没有安装 Agent 在监视对象中，Zabbix 也可以经由 SNMP、TCP、ICMP、利用 IPMI、SSH、telnet 对目标进行监视。
    + 另外，Zabbix 包含 XMPP 等各种 Item 警示功能。
- Zabbix 主要由2部分构成 zabbix server和 zabbix agent
- 安装(Zabbix 3.0)
    + 安装zabbix源

### 官网文档

[Zabbix 产品手册](https://www.zabbix.com/documentation/4.0/zh/manual)

* 介绍
    - Zabbix 由 Alexei Vladishev 创建，目前由其成立的公司—— Zabbix SIA 积极的持续开发更新维护， 并为用户提供技术支持服务。
    - Zabbix 是一个企业级分布式开源监控解决方案。
    - Zabbix 软件能够监控众多网络参数和服务器的健康度、完整性。
    - Zabbix 使用灵活的告警机制，允许用户为几乎任何事件配置基于邮件的告警。
    - Zabbix 基于存储的数据提供出色的报表和数据可视化功能。
    - Zabbix 支持主动轮询（polling）和被动捕获（trapping）。Zabbix所有的报表、统计数据和配置参数都可以通过基于 Web 的前端页面进行访问。
    - Zabbix 是免费的。Zabbix 是根据 GPL 通用公共许可证的第二版编写和发布的。这意味着产品源代码是免费发布的，可供公共使用。
* Zabbix 概述
    - `Zabbix server` 是 Zabbix软件的核心组件，agent 向其报告可用性、系统完整性信息和统计信息。server也是存储所有配置信息、统计信息和操作信息的核心存储库。
    - `数据库` 所有配置信息以及 Zabbix 采集到的数据都被存储在数据库中。
    - `Web 界面` 为了从任何地方和任何平台轻松访问 Zabbix ，我们提供了基于 web 的界面。该界面是 Zabbix server 的一部分，通常（但不一定）和 Zabbix server 运行在同一台物理机器上。
    - `Zabbix proxy` 可以代替 Zabbix server采集性能和可用性数据。(Zabbix proxy在Zabbix的部署是可选部分，可分担单个server负载)
    - `Zabbix agents` 部署在被监控目标上，用于主动监控本地资源和应用程序，并将收集的数据发送给 Zabbix server。
    - `数据流` 如果您想要收到类似“X个server上CPU负载过高”这样的告警
        + 首先为 Server X 创建一个主机条目
        + 其次创建一个用于监控其 CPU的监控项
        + 最后创建一个触发器（trigger），用来触发 CPU负载过高这个动作（action），并将其发送到您的邮箱里
        + 虽然这些步骤看起来很繁琐，但是使用模板的话，实际操作非常简单。也正是由于这种设计，使得 Zabbix 的配置变得更加灵活易用。
* 定义(Zabbix中常用术语的含义)
    - [2. 定义](https://www.zabbix.com/documentation/4.0/zh/manual/definitions)
    - 主机（host）
        + 你想要监控的联网设备，有IP/DNS。
    - 主机组（host group)
        + 主机的逻辑组；可能包含主机和模板。
    - 监控项（item）
        + 你想要从主机接收的特定数据，一个度量（metrics）/指标数据
    - 值预处理（value preprocessing）
        + 存入数据库之前，转化/预处理接收到的指标数据
    - 触发器（trigger）
        + 触发器是一个逻辑表达式，用来定义问题阈值和“评估”监控项接收到的数据
        + 当接收到的数据高于阈值时，触发器从“OK”变成“Problem”状态。当接收到的数据低于阈值时，触发器保留/返回“OK”的状态。
    - 事件（event）
        + 发生的需要注意的事件，例如触发器状态改变、自动发现/监控代理自动注册
    - 异常（problems）
        + 处在“异常”状态的触发器
    - 异常状态更新（problem update）
        + Zabbix提供的异常管理选项，例如添加评论、确认异常、改变严重级别或者手动关闭等。
    - 动作（action）
        + 预先定义的应对事件的动作
        + 一个动作由操作(例如发出通知)和条件(什么时间进行操作)组成
    - 远程命令（remote command）
        +  预定义好的，满足特定条件的情况下，可以在被监控主机上自动执行的命令。
    - 模版（template）
        + 被应用到一个或多个主机上的一整套实体组合（如监控项，触发器，图形，聚合图形，应用，LLD，Web场景等）。
        + 模版的应用使得主机上的监控任务部署快捷方便；也可以使监控任务的批量修改更加简单。模版是直接关联到每台单独的主机上。
    - 仪表板（dashboard）
        + 自定义的web前端模块中，用于重要的概要和可视化信息展示的单元， 我们称之为组件（widget）。
    - 组件（widget）
        + Dashboard中用来展示某种信息和数据的可视化组件（概览、map、图表、时钟等）。
    - Zabbix API
        + Zabbix API允许用户使用JSON RPC协议来创建、更新和获取Zabbix对象（如主机、监控项、图表等）信息或者执行任何其他的自定义的任务
    - Zabbix server
        + Zabbix软件的核心进程，执行监控操作，与Zabbix proxies和Agents进行交互、触发器计算、发送告警通知；也是数据的中央存储库
    - Zabbix agent
        + 部署在监控对象上的进程，能够主动监控本地资源和应用
    - Zabbix proxy
        + 代替Zabbix Server采集数据，从而分担Zabbix Server负载的进程
    - agent自动注册（agent auto-registration）
        + Zabbix agent自己自动注册为一个主机，并且开始监控的自动执行进程。

#### 安装

- 四种方法
    + 从 发行包 安装；
    + 下载最新的归档源码包并 编译它；
    + 从 容器 中安装；
    + 下载 Zabbix 应用。
- 从发行包(二进制包)安装(CentOS 7)
    + [Red Hat Enterprise Linux/CentOS](https://www.zabbix.com/documentation/4.0/zh/manual/installation/install_from_packages/rhel_centos)
    + 添加 Zabbix 软件仓库
        * 安装软件仓库配置包，这个包包含了 yum（软件包管理器）的配置文件。
            - `rpm -ivh http://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm`
        * 前端安装的先决条件，Zabbix 前端需要额外的基础安装包。 您需要在运行 Zabbix 前端的系统中启用可选 rpms 的软件仓库：
            - `yum-config-manager --enable rhel-7-server-optional-rpms`
    + 安装 Server/proxy/前端 (自己只安装server和web前端)
        * 安装 Zabbix server（适用于 RHEL7，在 RHEL 6 上弃用）并使用 MySQL 数据库(若用pg数据库则mysql替换为pgsql)：
            - `yum install zabbix-server-mysql -y`
        * 安装 Zabbix 前端（适用于 RHEL 7，在 RHEL 6 上弃用）并使用 MySQL 数据库：
            - `yum install zabbix-web-mysql -y`
    + 创建数据库
        * 对于 Zabbix server 和 proxy 守护进程而言，数据库是必须的。而运行 Zabbix agent 是不需要的。如果 Zabbix server 和 Zabbix proxy 安装在相同的主机，它们必须创建不同名字的数据库！
    + 导入数据
        * 使用 MySQL 来导入 Zabbix server 的初始数据库 schema 和数据
            - 创建数据库`create database zabbix character set utf8 collate utf8_bin;`
            - 创建zabbix用户 `create user zabbix identified by 'zabbix';`，选择zabbix.*赋全部权限：`grant all on zabbix.* to zabbix;`
                + 若grant时出现，Access denied for user 'root'@'%' to database 'zabbix'，则可能root没有权限
                + `select host,user,grant_priv,super_priv from mysql.user;`查询`grant_priv`和`super_priv`权限，若没有权限则`update mysql.user set grant_priv='Y' where user='root' and host='%';`(发现环境中grant_priv是'N')，然后`flush privileges;`，再重启mysql
            - `zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix`
                + `zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -h 192.168.xxx.xxx -uroot -p zabbix`
                    * 使用root，连接远程环境的mysql
                    * 后面的`zabbix`是指定数据库
    + 为 Zabbix server/proxy 配置数据库(本地proxy不使用)
        * `vi /etc/zabbix/zabbix_server.conf`
            - DBHost=localhost(设置ip)
            - DBName=zabbix
            - DBUser=zabbix
            - DBPassword=<password>
    + 启动 Zabbix server 进程
        * `service zabbix-server start`
    + 设置开机启动
        * `systemctl enable zabbix-server`
    + Zabbix 前端配置
        * 对于 RHEL 7 和更高版本，Zabbix 前端的 Apache 配置文件位于 `/etc/httpd/conf.d/zabbix.conf`。
        * 取消 "date.timezone" 注释，并设置当前时区，ll /etc/localtime查看链接的时区文件为`../usr/share/zoneinfo/Asia/Shanghai`，则设置为"Asia/Shanghai"
    + SELinux 配置
        * getenforce本地关闭了SELinux，若开启需要参考进行配置
        * 可以开放防火墙：zabbix-agent zabbix-server
            - `firewall-cmd --permanent --add-service=zabbix-server`
            - `firewall-cmd --permanent --add-service=zabbix-agent`
            - `firewall-cmd --reload`
    + 前端和SELinux配置完成后重新启动Apache web服务器
        * `service httpd restart`，若没有服务则`yum install httpd`
        * 设置httpd开机启动 `systemctl enable httpd`
    + 安装 Agent
        * 安装：`yum install zabbix-agent -y`
        * 启动：`service zabbix-agent start`
        * Ubuntu 18.04 (bionic)(监测ubuntu的主机，需要安装agent):
            - `apt install zabbix-agent`
            - `service zabbix-agent start` (`systemctl list-unit-files`查看zabbix-agent.service是自动开机启动的)
            - [参考](https://www.zabbix.com/documentation/4.0/zh/manual/installation/install_from_packages/debian_ubuntu)
        * agent相关的配置
            - 参考下面的章节：`+ 新建主机`
    + 安装完成，按链接检查前端安装项，数据库地址、用户密码等
        * [前端安装步骤](https://www.zabbix.com/documentation/4.0/manual/installation/install#installing_frontend)
        * 确认配置完成后，会出来登录页面，默认用户： Admin, 密码： zabbix
    + 注意：
        + 由于服务器操作的安全性要求和任务关键性，`UNIX`(类 Unix) 是唯一能够始终如一地提供必要性能、容错和弹性的操作系统。
            * 所以server只能在UNIX(Linux)上，agent则可以为Linux或Windows
        + Windows安装使用agent
            * Windows安装zabbix监控(agent)，可参考官网：[Windows 下的Zabbix agent](https://www.zabbix.com/documentation/4.0/zh/manual/appendix/install/windows_agent)
            * 下载地址：[官网下载agent地址](https://www.zabbix.com/cn/download_agents)
                - `zabbix_agentd.exe --start`
                - `zabbix_agentd.exe --stop`
            * 监控windows下的mysql：
                - 上面链接下载安装，安装时可以输入配置的server和激活远程命令执行(也可后续在配置文件中修改后重启服务)
                    + `控制面板-系统和安全-管理工具-服务(双击打开)-找到对应服务名(zabbix...)` 可以手动重启，或者上面的命令重启
                - 配置监控项、触发器、动作(和Linux一致)
                    + 关于动作：可以找到对应的服务名(控制面板-系统和安全-管理工具-服务(双击打开)-找到对应服务名)，然后`net start 服务名`进行启动
                    + 比如我的环境下，`服务`中mysql服务名是：`MySQL80`，则远程执行命令设置：`net start mysql80`(大小写均可)
            * 默认安装位置(Windows10)：`C:\Program Files\Zabbix Agent`，若要修改配置则到该目录找`zabbix_agentd.conf`，修改后参考上面的方式重启zabbix_agentd.exe
        + 参考：[支持的平台](https://www.zabbix.com/documentation/4.0/zh/manual/concepts/server)
        + Zabbix server 需要 UTF-8 语言环境，以便可以正确解释某些文本项。

#### 入门使用

- [5. 快速入门](https://www.zabbix.com/documentation/4.0/zh/manual/quickstart/login)
- 配置
    + 可以切换中文：点击右上角人体图标->User->Language->选Chinese
    + Admin登录，可以看到`配置（Configuration）` and `管理（Administration）` 菜单(权限)
    + 增加用户
        * 管理（Administration） → 用户（Users）
        * `媒介`
            - 默认情况下，没有为新增的用户定义媒介（media，即通知发送方式)
            - 如需要创建，可以到'媒介（Media）'标签下（用户->报警媒介)，然后点击增加（Add）
        * `添加权限`
            - 默认的情况下，一个新用户没有全选访问任何主机
            - 在Zabbix中，主机的访问权限是被分配到用户组，而不是单个用户。
            - 修改用户组后在用户列表界面的用户类型可体现，超级管理员/管理员/用户
    + 新建主机
        * Zabbix中的主机（Host）是一个你想要监控的网络实体（物理的，或者虚拟的）。
        * 配置（Configuration） → 主机（Hosts）菜单，查看已配置的主机信息。
            - 默认已有一个名为'Zabbix server'的预先定义好的主机。
            - 建议不要修改该默认名称(默认就是Zabbix server)，**该名称和zabbix_agentd.conf里面的Hostname要一样**，否则会报“cannot send list of active checks to "127.0.0.1": host [Zabbix server] not found”
        * 主机名称，可以使用字母数字、空格、点"."、中划线"-"、下划线"_"。
        * 选择一个或者多个组
        * 输入主机的IP地址。
            - 注意如果要监控的主机是Zabbix server(服务端在的那台设备)的IP地址，它必须是Zabbix agent配置文件中‘Server’参数的值。
            - 要监控的主机上，zabbix_agentd.conf配置文件中的`Server`或者`ServerActive`配置加上zabbix server的ip地址(该地址可以有多个，用`,`分隔)并重启服务
                + `ServerActive`用于主动检查，被监测的机器自己检查变化，并上报给zabbix服务端，可以参考应用：`* 日志文件监控`
            - 配置文件可以配置日志位置，和日志等级，出现问题时可以开启debug等级排查问题`DebugLevel`设置为4(注意日志会很多)
        * 上面的说明偏文本，实际配置一台要监测的主机，要改的配置示例：
            - `Server=` 改成zabbix服务端的ip，(如果要使用主动检查的功能如主动上报日志变化，则`ServerActive`中配置zabbix服务端的ip)
            - `EnableRemoteCommands=`配置为1 允许服务端远程执行命令，`LogRemoteCommands=`配置为1(客户端日志中把执行的命令打印出来)
            - 如果要支持zabbix服务端远程执行命令，需要把zabbix用户加到sudo列表中，参考章节：`* 动作`
    + 新建监控项
        * 监控项是Zabbix中获得数据的基础。没有监控项，就没有数据——因为一个主机中只有监控项定义了单一的指标或者需要获得的数据。
        * 所有的监控项都是依赖于主机的。当我们要配置一个监控项时，先要进入 `配置` → `主机` 页面查找到新建的主机。
        * 新主机的`监控项`列数字没有或者是0，点击`监控项`进入->创建监控项（Create item）
            - `名称（Name）` 输入 CPU Load 作为值
            - `值（Key）` 手动输入 `system.cpu.load` 作为值(下拉框出来的`system.cpu.load[<cpu>,<mode>]` **会提示无效参数**)
            - `信息类型（Type of information）` float
            - 减少监控项历史保留的天数，7或者14天。对于数据库而言，最佳实践是避免数据库保留过多的历史数据。
            - 当完成后，点击添加（Add）。新的监控项将出现在监控项列表中。点击列表中的详细（Details）以查看具体细节。
            - (如果`Key`参数错误，修改后点击`现在检查`，再点更新，日志提示"xxxname:system.cpu.load" became supported)
        * 查看数据
            - 监控/或监测（Monitoring）(页面最上面一栏) → 最新数据（Latest data）, 在过滤器中选择刚才新建的主机，然后点击应用（Apply)。
                + **注意**，添加监控项时若不存储历史记录，则最新数据里看不到数据(并不保存)，若不需要数据存很久可配置短一点(1d,1h,10s等，时间格式参考章节：搜 `- 时间格式`，试了下貌似h级别不行)
            - 如果你在没有看到类似截图中的监控项信息，请确认：
                + 输入的监控项'值（Key）' 和 '信息类型（Type of information）'正确
                + agent和server都在运行状态
                + 主机状态为'监控（Monitored）'并且它的可用性图标是绿色的
                + 在主机的下拉菜单中已经选择了对应主机，且监控项处于启用状态
    + 新建触发器
        * 如果收到的数据超过了这个定义好的级别，触发器将被“触发”，或者进入“异常（Problem）”状态
        * 如果数据再次`恢复`到合理的范围，触发器将会到“正常（Ok）”状态。
        * `配置` -> `主机` -> 选择一个主机找到`触发器`列点击进入 -> 右上角`创建触发器`
            - `名称（Name）` 输入："CPU load too high on 'xdCentOS_50.118' for 3 minutes"作为值。这个值会作为触发器的名称被现实在列表和其他地方
            - `表达式（Expression）` 输入：`{xdCentOS_50.118:system.cpu.load.avg(3m)}>2`
                + 此处，监控项值(system.cpu.load)用于指出具体的监控项。如果3分钟内，CPU负载的平均值超过2，那么就触发了问题的阈值。
                + 界面提供了`表达式构造器`用于便捷构造表达式
            - 完成后，点击添加（Add）。
        * 显示触发器状态
            - 如果CPU负载超过了你在触发器中定义的阈值，这个问题将显示在`监控/监测（Monitoring）` → `问题（Problems）`中。
            - 闪烁意味着这个触发器状态最近30分钟内发生过变化。
    + 获取问题通知
        * 当监控项收集了数据后，触发器会根据异常状态触发报警。
        * 根据一些报警机制，它也会通知我们一些重要的事件，而不需要我们直接在Zabbix前端进行查看。
        * 这就是`通知（Notifications）`的功能。`E-mail`是最常用的异常通知发送方式。
            - `E-mail配置` `管理（Administration）` → `报警媒介类型（Media types）`→ `Email` 进行配置
                + SMTP服务器地址设置
                    * 使用外部SMTP服务器(e.g. 注册qq邮箱开启SMTP获取授权码，可参考：[Zabbix使用QQ邮箱通知](https://www.cnblogs.com/yinzhengjie/p/10389897.html))
                    * 自己尝试Linux搭建SMTP服务器，postfix/dovecot，尝试失败。。。
                + 现在你已经配置了'Email'作为一种可用的媒体类型。一个媒体类型必须通过发送地址来关联用户(如同我们在配置一个新用户中做的)，否则它将无法生效。
                + 最后自己注册一个126邮箱用于邮件发送
                    * 登录邮箱->设置->POP3/SMTP/IMAP，开启SMTP(选一个支持的即可，我选的IMAP/SMTP服务)，会生成一个授权码
                    * SMTP服务器: smtp.126.com，用户名/密码/授权码，web上设置的用户名为邮箱，密码填授权码
            - `新建动作`
                + 发送通知是Zabbix中动作（actions）执行的操作之一
                + 因此，为了`建立一个通知`，前往`配置（Configuration）` → `动作（Actions）`，然后点击`创建动作（Create action）`。
                + `新建操作` 我们还需要定义这个动作具体做了什么 —— 即在 `操作（Operations）标签页`中执行的操作。
                    * 点击"新的"，进行操作定义，选用户组或者用户
        * 可以在`报表（Reports）` → `动作日志（Action log）`中检查动作日志
* 模板
    - [6 新建模板](https://www.zabbix.com/documentation/4.0/zh/manual/quickstart/template)
    - 在之前的章节中学会了如何配置监控项、触发器，以及如何从主机上获得问题的通知。
    - 虽然这些步骤提供了很大的灵活性，但仍然需要很多步骤才能完成。如果我们需要配置上千台主机，一些自动化操作会带来更多便利性。
    - 模板（templates）功能可以实现这一点。模板允许对有用的监控项、触发器和其他对象进行分组，只需要一步就可以对监控主机应用模板，以达到反复重用的目的。
    - `创建模板`：配置（Configuration） → 模板（Templates）→ 创建模板，创建后添加监控、添加触发器等信息
        + 创建时需要选择群组，模板必须属于一个组。
    - `链接模版到主机`：配置 -> 主机 -> 选择一个主机 -> 点击模板页 -> 选择某个或者某些模板
    - `取消链接模板`
        + Unlink - 取消链接模板，但保留它的监控项、触发器和图表(主机的监控项等信息还在)
        + Unlink and clear - 取消链接模板并删除所有它的监控项、触发器和图表

#### 配置

* 监控项
    - 监控项键值(Key)的格式：形如 `icmpping[,,200,,500]`
        + 允许字符：`0-9a-zA-Z_-.`
            * 另外参数相关的字符：`[ , ]`
        + 参数：监控项的键值可以有多个逗号分隔的参数；
            * 参数也可以为空，此时使用默认值。
            * 如果指定了后面的其它参数，则**该参数前面的参数位置**必须添加对应数量的逗号。
        + 每个key参数可以是带引号、无引号的字符串或数组
            * 仅支持用双引号，不支持单引号。
    - 时间格式
        + 时间间隔包含
            * `md` - month days
            * `wd`或者`w` - week days
            * `h` - hours
            * `m` - minutes
            * `s` – seconds
        + 自定义时间间隔
            * `灵活间隔`，包含：间隔和周期
                - e.g. 间隔`10` 周期`1-5,09:00-18:00` 监控项将在工作日时间内每10秒检查一次。(间隔设置0则表示周期内不做检查)
            * `调度间隔`，在特定时间检查监控项
                - 格式：`md<filter>wd<filter>h<filter>m<filter>s<filter>`
                    + filter定义为： `[<from>[-<to>]][/<step>]`
                - e.g. `wd1-5h9`      每周一至周五9:00
                - e.g. `h9m0-59/30`   在9:00，9:30执行(即0-59分，间隔30m)
                - 更多示例见：[示例](https://www.zabbix.com/documentation/4.0/zh/manual/config/items/item/custom_intervals)
* 日志文件监控
    - [日志文件监控](https://www.zabbix.com/documentation/4.0/zh/manual/config/items/itemtypes/log_items)
    - Zabbix可以集中监控和分析 支持/不支持日志轮询的日志文件。
    - 当日志文件**包含某些字符串或字符串模式**时，可以使用通知来警告用户。
        + 被监控日志文件的大小限制取决于 大文件支持(32为操作系统上能力能达到 2 GB)。
    - 配置要求(/etc/zabbix/zabbix_agentd.conf)：
        + 'Hostname'参数与前端的主机名一致
            * **说明**：设置成主动监控项在的那个主机名，并不是指"Zabbix server"这台
        + 'ServerActive'参数中的服务器被指定用于处理主动检查
            * 指定 Zabbix server 或者 Zabbix proxy 的地址
        + 关于配置文件的各项说明可参考：[Zabbix agent (UNIX)](https://www.zabbix.com/documentation/4.0/manual/appendix/config/zabbix_agentd)
    - zabbix用户需要有对应日志文件的权限
        + 安装zabbix时，自动添加了zabbix用户(/sbin/nologin类型，即不可登录)，使用时报错了(提示无权限)：active check "log[/home/xxxfile,”xdtest”,,,skip,,]" is not supported: Cannot obtain information for file "/home/xxxfile": [13] Permission denied
        + **解决：** `usermod -a -G root zabbix` 把zabbix加入到root组中(文件权限为`-rw-r--r-- 1 root root`，同组和不同组用户看起来都有读权限，但还是把zabbix加到root组吧，还不明确是什么原因导致权限拒绝了)
            * (试过`visudo`添加zabbix到sudoers中并不行，貌似添加了只是支持以root权限运行命令而已："Allow root to run any commands anywhere"，还是建议加一下，后面介绍的远程执行命令需要有执行权限`zabbix ALL=NOPASSWD: ALL`)
            * (试过chown把文件和目录改为zabbix用户和zabbix组也不行)
    - 监控项配置
        + `类型(Type)`
            * 选择`Zabbix agent (active)` (对应中文界面：`Zabbix客户端(主动式)`，翻译应该翻译为代理)
        + `键值(Key)`
            * log相关的各个参数说明可参考(log项)：[1 Zabbix客户端](https://www.zabbix.com/documentation/4.0/zh/manual/config/items/itemtypes/zabbix_agent#supported_item_keys)
            * `log[file,<regexp>,<encoding>,<maxlines>,<mode>,<output>,<maxdelay>]`
                - `file`：日志文件完整路径和名称
                - `encoding`：编码标识符，"UTF-8"/
                - `maxlines`：Agent将发送到Zabbix服务器或代理的每秒最大行数。 此参数覆盖zabbix_agentd.conf中的“MaxLinesPerSecond”值(默认20行)
                - `mode`：all (默认值)； skip 跳过历史的数据（即只管日志新增的内容）
                - `output`：输出格式模板，可以自定义输出的格式。 `regexp`正则表达式中格式化匹配的位置可以用`\N`(N可以取值0-9)表示，会替换为对应的位置。
                    + e.g. `regexp表达式`为"task run [0-9.]+ sec, processed ([0-9]+) records"，且日志行为"2015-11-13 10:08:26 task run 6.08 sec, processed 6080 records, 0 errors"，则`\1`表示`6080`
                - `maxdelay`：最大延迟（秒数，浮点型），若>0.0，则可以只分析maxdelay秒内的行
                - 示例：
                    + `log[/var/log/syslog]`
                    + `log[/var/log/syslog,error]` 匹配关键字error
                    + `log[/home/zabbix/logs/logfile,,,100]` 每秒最烦发送100行(指定参数前面的参数位置置空并用,隔开，其后不必)
            * `logrt[file_regexp, <regexp>,<encoding>,<maxlines>,<mode>,<output>,<maxdelay>]`
                - `file_regexp` 文件名以及正则表达式定义的文件名的绝对路径，可以匹配满足一定模式的文件列表
                - 规则文本：`{logrt[/root/^log_[0-9]{4}-[0-12]{2}-[1-31]{2}.txt$]`
                - 关键字：`logrt["/root/^log_[0-9]{4}-[0-12]{2}-[1-31]{2}.txt","========error"]`
            * `log.count[file,<regexp>,<encoding>,<maxproclines>,<mode>,<maxdelay>]`
                - `maxproclines`：Agent每秒将分析的最大行数。默认值为 10*'MaxLinesPerSecond'(即若配置文件保持默认则分析为200行)
            * logrt.count
        + `更新间隔Update interval (in sec)`
            * 该参数定义了Zabbix代理检查日志文件中任何更改的频率。将其设置为`1秒`将确保你能尽快的获得新记录。
    - 注意事项
        + 代理从上次停止的点开始读取日志文件。(所以是否设置mode为skip关系不大?)
        + 在代理刚刚启动或已收到以前被禁用或不支持的监控项的情况下，*已经分析的*字节数（大小计数器）和最后修改时间（时间计数器）*存储在Zabbix数据库中*并发送到代理，以确保代理从此开始读取日志文件。
        + 每当日志文件变得小于代理已知的日志大小计数器时，计数器将重置为零，代理从开始位置读取日志文件，将时间计数器考虑在内。
        + Zabbix agent每 `Update interval` 秒处理一次日志文件的新记录。
        + 对于大于256kB的日志文件记录，只有前256kB与正则表达式匹配，而其余部分将被忽略。(1MB的日志更新就识别不到了？**待测**)
        + 'maxdelay'>0可能导致 忽略重要的日志文件记录和错过的报警 ，只有在必要时才使用
        + 默认情况下，日志监控项将跟踪出现在日志文件中的所有新行。日志文件中写入大量的消息时，所有这些消息将被完全分析，可配置“maxlines”参数进行限制。
            - 但依旧存在两个问题：
                + 1. 向服务器报告大量潜在的不太有用的消息，消耗数据库中的空间。
                + 2. 由于每秒分析的行数有限，代理可能会滞后于最新的日志记录数小时。
            - 解决方案是通过`maxdelay`参数
                + 如果指定'maxdelay'> 0，在每次检查处理字节数时，将测量剩余字节数和处理时间(代理会根据这些数字估计剩余需要的秒数)。
                + 如果延迟不超过“maxdelay”，那么代理将像往常一样继续分析日志文件。
                + 如果延迟大于“maxdelay”，那么代理将通过“跳转”到一个新的*估计位置*来忽略日志文件的一个块， 以便在“maxdelay”秒内分析剩下的行。
                + **不推荐设置 'maxdelay' < 'update interval'（这可能会导致频繁的“jumps”）**
                + e.g. `log[/root/xxx,,,,skip,,5]`
    - 若有两个监控项监控同一个日志文件，有时只生效一个(现象比较奇怪，尝试了很久，对同一文件尽量不要有多个日志监控项)
* 触发器
    - [3 触发器](https://www.zabbix.com/documentation/4.0/zh/manual/config/triggers)
        + [2 触发器表达式](https://www.zabbix.com/documentation/4.0/zh/manual/config/triggers/expression)
        + 支持的函数：[1 Supported trigger functions](https://www.zabbix.com/documentation/4.0/manual/appendix/triggers/functions)
            * `avg (sec|#num,<time_shift>)` 平均值
                - 参数一表示最大计算周期，可取值1h,3s等形式；和#5形式，表示最近5个值，`avg(1h)`表示一小时平均值
                - 参数二表示时间偏移，如 `avg(1h,1d)`表示1天前的一小时平均值
            * `last (<sec|#num>,<time_shift>)` 最近的值
                - 参数一表示最大计算周期
                    + `last()`和`last(#1)`等价，最近一个值，`last(#3)`最近第三个值(不是最近三个)
                - 参数二时间偏移，和avg中一致，表示时间偏移前的值
* 动作
    - 若要远程操作，则需agent配置文件里配置选项 `EnableRemoteCommands=1`，是否日志记录也可选择开启`LogRemoteCommands=1`
    - `sudo sh /root/xxx.sh` 执行命令时需要指定`sudo`
        + 且在`/etc/sudoers`文件中加上(通过`visudo`编辑)：`zabbix  ALL=(ALL)   NOPASSWD: ALL`
        + 默认情况下，Zabbix用户没有权限重新启动系统服务。
    - 远程命令不适用于主动模式Zabbix代理
    - 可以设置循环多个通知或操作(通过步骤列来控制次数)，参考下面链接示例(但每个操作之间默认时间60s-604800s之间，如果要立即执行操作和通知，目前自己方式是新建两个动作分别处理。。。)
    - [2 远程命令](https://www.zabbix.com/documentation/4.0/zh/manual/config/notifications/action/operation/remote_command)
* 宏
    - 支持的宏：[1 宏使用场景](https://www.zabbix.com/documentation/4.0/zh/manual/appendix/macros/supported_by_location)
        + 监控项: {ITEM.KEY1}
        + 触发器内容: {TRIGGER.EXPRESSION}
        + 时间和日期: {EVENT.TIME} on {EVENT.DATE}
        + 事件名: {EVENT.NAME}
        + 主机: {HOST.NAME}
        + 严重程度: {EVENT.SEVERITY}
        + 事件 ID: {EVENT.ID}
