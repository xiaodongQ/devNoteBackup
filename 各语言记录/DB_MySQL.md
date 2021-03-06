## MySQL

* 查看数据库连接
    - `show status like 'Threads%';`
    - `show variables like '%max_connections%';`

* Windows下部署
    - 步骤
        + 下载mysql zip
        + 没有data目录，如果没有data目录，安装后`net start mysql`启动的时候就会报这个错：MySQL 服务无法启动。
        + 需要使用命令生成data文件夹
            * 到bin目录，`mysqld -install`
            * `mysqld --initialize --user=root --console`初始化后可看到新的data目录(可能会产生一个默认生成的密码)
            * 启动`net start mysql`
            * 登录`mysql -uroot -p` 若生成默认密码则用其登录，若没有回车即可进入mysql控制台
        + `mysqld -remove mysql` 卸载安装的mysql
        + 如果是通过安装包安装的，则可以查看启动的命令：`控制面板-系统和安全-管理工具-服务(双击打开)-找到MySQL-属性(右键)`
    - Windows下检查mysql是否开机启动：
        + 控制面板-系统和安全-管理工具-服务(双击打开)-找到MySQL-属性(右键)-启动类型为"自动"则为开机启动
    - mysql远程连接mysql时报错： `Host * is not allowed to connect to this MySQL server`
        + `mysql -h 192.xxx.xxx.xxx -u root -p` 报上述错误(检查防火墙对mysql的端口是否开放，有这个报错防火墙应该是支持入栈规则)
        + 到本机上终端连接后执行：`select user,host from mysql.user;`，可以看到`host`列都是`localhost`
        + `create user root identified by 'xxxx';`，可以看到host为`%`，且有两个`root`用户
        + `grant all on *.* to root;`

* mysql配置文件
    - 网上有各种版本的名称，`my.ini`，`my.cnf`，`mysqld.cnf`
    - 自己docker安装的mysql，配置文件是 `/etc/my.cnf`，`/etc/my.cnf.d/`目录中是空的
* Linux下通过容器部署
    - [ Basic Steps for MySQL Server Deployment with Docker](https://dev.mysql.com/doc/refman/8.0/en/docker-mysql-getting-started.html)
        + 下载docker镜像(不是必要的，操作该步可保证本地包是对应版本最新的)
            * `docker pull mysql/mysql-server:tag`，tag是镜像版本(例如5.5, 5.6, 5.7, 8.0, or latest)
            * 此处我选择latest，直接省略:tag即可，默认也是latest，最新版本
        + 运行实例
            * `docker run --name=mysql1 -p 3306:3306 --restart=always -v /home/data/mysql/conf/my.cnf:/etc/my.cnf -v /home/data/mysql/data:/var/lib/mysql -d mysql/mysql-server`，若为其他版本则 mysql/mysql-server:对应tag
                - 如果docker镜像没有通过之前的`docker pull` or `docker run`命令下载，则会自动下载，并运行
                - `-d`表示后台运行容器(container)
                - `-p 8080:80`可以将容器中的端口(80)映射到服务器上的端口(8080)，这样外部就可以通过该端口8080访问容器中的服务
                    + 若没有该映射(`-p 3306:3306`)，则其他机器访问不到mysql的3306端口
                - `--restart=always` 每次docker服务重启后容器也自动重启
                - 可以`-v`指定映射关系，配置文件、数据目录等
                    + e.g. `-v /opt/mysql/data:/var/lib/mysql` 映射数据目录
                    + e.g. `-v /opt/mysql/config/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf`：映射配置文件
                    + 可在`docker inspect xxxid`结果中的Binds中看到映射关系
                    + 启动时可能要`--privileged`(自己没做映射，没尝试)，使用该参数，container内的root拥有真正的root权限，否则，container内的root只是外部的一个普通用户权限
                        * 若要映射可参考：[Docker安装MySql-挂载外部数据和配置](https://www.cnblogs.com/0oliumino0/p/10538207.html)
                    + 指定时区：`-v /etc/localtime:/etc/localtime` (貌似没用，还是进入容器复制文件修改了`cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`)
            * `docker logs mysql1` 查看容器的输出
        + mysql启动后，会给root生成一个随机密码，从日志中查找密码：
            * `docker logs mysql1 2>&1 | grep GENERATED`
        + `docker exec -it mysql1 mysql -uroot -p` 启动一个客户端连接到mysql
            * 修改默认的root@localhost用户的密码(若不修改则无法运行命令，会提示修改)
                - `alter user root@localhost identified by '新密码';`
                - root@localhost和root用户有区别，关于用户名和密码，可以参考笔记中下面的章节：搜"用户名和密码"
                - 创建用户`create user root identified by 'xd123456';`，创建后指定权限：`grant all on *.* to root;`
    - 关于mysql容器相关的其他操作：
        + 启动一个进入到容器中的shell终端 `docker exec -it mysql1 bash`
            * 进入容器后操作和linux一致，mysql路径：/var/lib/mysql
        + 停止容器
            * `docker stop mysql1`
            * 会发送一个SIGTERM信号给mysqld进程，因此服务能优雅关闭
            * 注意，容器中的主进程停止的话，容器也会停止。 若mysqld停止，则包含其的容器也会停止
        + 再次启动容器 `docker start mysql1`
        + 重启容器 `docker restart mysql1`
        + 删除容器(两步，先停止再删除)
            * `docker stop mysql1`
            * `docker rm mysql1`
            * 如果要同时删除该数据库的数据(若该数据库还要继续使用则不要去删数据)，则`docker rm`加上`-v`选项
                - 由于docker容器原则上是短暂的，其数据和配置容易丢失，所以docker卷提供了一种mount机制来持久化数据
                - `docker inspect mysql1`来查看mount挂载的信息，"Mounts"部分(`docker inspect`返回docker对象的低层次信息)
                    + 数据挂载目录一般在容器外的 "/var/lib/docker/volumes/xxxxxx" 下面
                - [Persisting Data and Configuration Changes](https://dev.mysql.com/doc/refman/8.0/en/docker-mysql-more-topics.html#docker-persisting-data-configuration)
        + 在运行docker容器时可以加如下参数来保证每次docker服务重启后容器也自动重启
            * `docker run --restart=always`
            * 如果已经启动了则可以使用如下命令
                - `docker update --restart=always <CONTAINER ID>`

* Linux下安装Mysql 8.0
    - 自己的CentOS虚拟机比较早安装，参考下面链接直接安装的mysql8.0，建议参考官网：[Installing MySQL on Linux Using the MySQL Yum Repository](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html)
    - 本次安装参考：[CentOS 7 安装 Mysql 8.0 教程](https://blog.csdn.net/danykk/article/details/80137223)
    - 1）配置Mysql 8.0安装源
        + sudo rpm -Uvh https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
    - 2）安装Mysql 8.0
        + sudo yum --enablerepo=mysql80-community install mysql-community-server
* 客户端
    - `apt-get install mysql-client` 安装客户端(Ubuntu)
    - `sudo apt-get install libmysqlclient-dev` 若开发时需要用客户端连接，则需要该包
    - centos下：`yum install mysql-devel` C/C++驱动

## linux mysql操作

* docker容器部署的MySQL
    - `docker ps` 查看容器实例(e.g. 实例名查看得知为 mysql1)
    - `docker exec -it mysql1 bash` 进入docker
    - `ls /var/lib/mysql` 容器中mysql数据文件
    - `docker start mysql1` 启动mysql容器
    - `docker stop mysql1` 停止
    - `docker rm mysql1` 删除，(删除前先进行一下stop)

* 使用systemd管理mysql服务(docker部署的mysql用不了这种方式，容器方式单独说明)
    - 官网文档：[Managing MySQL Server with systemd](https://dev.mysql.com/doc/refman/8.0/en/using-systemd.html)
        + 包含安装步骤
    - MySQL启动、停止、重启常用命令
        * `systemctl {start/stop/restart/status} mysqld`
        * 或者 `service mysqld {start|stop|restart|status}`
        * `netstat -anp|grep mysqld` 监听端口3306和33060
    - 可以创建 /etc/systemd/system/mysqld.service.d 文件，进行服务的配置(最大文件打开数、nice值、core文件个数限制等)
    - `systemctl daemon-reload` 上述配置文件修改，需要告诉系统要重新生效，并`systemctl restart mysqld`重启服务
    - `my.cnf` mysql实例配置文件
        + 其中可以配置多个mysql数据库实例，各自独立，若为多个则用`mysqld@`进行区分配置(@是systemctl支持的唯一分隔符)，并用systemctl进行管理，[参考](https://dev.mysql.com/doc/refman/8.0/en/using-systemd.html#systemd-overview)
            * e.g. `systemctl start mysqld@replica01` 而不是用mysqld
        + 路径：/etc/my.cnf (CentOS为该路径，其他系统可能路径有出入/etc/mysql/my.cnf等)
        + 其中指定数据文件路径、端口、错误日志路径、socket等
* 连接并登录服务器
    - `mysql -h host -u user -p`，然后输入密码
        + 如果是本地mysql服务，则`mysql -u user -p`
        + 可以指定数据库：`msyql -h host -u user -p -D xdtest`
    - 进程未运行报错
        + 如果报错：ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock'
        + 说明进程未运行，检查并启动
    - 其他一些使用过程中的常见错误
    -   + [B.4.2 Common Errors When Using MySQL Programs](https://dev.mysql.com/doc/refman/8.0/en/common-errors.html)
* 用户名和密码
    - [6.2.1 Account User Names and Passwords](https://dev.mysql.com/doc/refman/8.0/en/user-names.html)
    - MySQL将用户存储在表 `user` 中，MySQL用户名不能超过32字符长度(操作系统最大长度有不同的限制)
    - MySQL安装程序会使用初始用户`root`填充授权表
        + [2.10.4 Securing the Initial MySQL Account](https://dev.mysql.com/doc/refman/8.0/en/default-privileges.html)
        + 初始化root用户时，可能会初始化一个随机密码，查找错误日志文件(`CentOS下为/var/log/mysqld.log`)，其中打印了初始随机密码，查找关键字"GENERATED"
        + MySQL安装程序包含初始化数据目录，包含MySQL中定义的授权表
            * 具体参考：[2.10.1 Initializing the Data Directory](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html)
    - `mysql.user` 授权表定义了MySQL初始用户和其权限，定义了`'root'@'localhost'`超级用户(该用户只允许本地使用)
        + `root@localhost`和`root`并不相同，对于用户中的主机部分，"用户的格式"章节做了说明
    - 给初始化的root用户配置密码：
        + 登录：
            * 若有初始密码(本地环境有) `mysql -u root -p` 则从错误日志中找到初始密码(GENERATED，参考上面用户名和密码的说明)
            * 若没有密码 `mysql -u root --skip-password`
            * 关于版本问题报错
                - 报错：plugin 'caching_sha2_password' cannot be loaded: /usr/lib64/mysql/plugin/caching_sha2_password.so
                - mysql8 之前的版本中加密规则是mysql_native_password,而在mysql8之后,加密规则是caching_sha2_password, 解决问题方法有两种,一种是升级navicat驱动,一种是把mysql用户登录密码加密规则还原成mysql_native_password
                - 修改加密规则：
                    + `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';` 更新一下用户的密码
                    + `FLUSH PRIVILEGES;` 刷新权限
                - 参考：[MySQL 连接出现 Authentication plugin 'caching_sha2_password' cannot be loaded](https://www.cnblogs.com/zhurong/p/9898675.html)
        + 修改密码：`ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';`
            * [6.4.3 The Password Validation Component](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html)
            * 默认密码策略长度需要至少8个字符，否则报错Your password does not satisfy the current policy requirements
            * 默认密码策略允许配置，修改`validate_password.policy`的值，8.0.4版本引入，有三个等级(默认为MEDIUM等级)
                - 0 or LOW      Length
                - 1 or MEDIUM   Length; numeric, lowercase/uppercase, and special characters
                - 2 or STRONG   Length; numeric, lowercase/uppercase, and special characters; dictionary file
            * 需要修改默认密码才能执行其他sql语句
                - `alter user 'root'@'localhost' identified by 'Root@123';`
            * 密码相关的变量
                - 查看：`show variables like 'validate_password.%';`
                - 修改设置：e.g. `set global validate_password_policy=0;` 此处是把密码策略改成LOW级别，只校验长度>=8
        + 密码忘记重置密码(WindowS和Unix/类Unix，通用方法)
            * [B.4.3.2 How to Reset the Root Password](https://dev.mysql.com/doc/refman/8.0/en/resetting-permissions.html)
            * 跳过密码认证过程来重置密码(使用mysqld/mysqld_safe命令行手动启动mysql，加--skip-grant-tables选项。 或者改配置文件重新启动)
                - vim /etc/my.cnf
                - 在 mysqld 后面任意位置，添加`skip-grant-tables`用来跳过密码验证的过程
                - 保存配置重启mysqld，`service mysqld restart`
                - 命令行输入`mysql`即可登录
                - 进行密码修改操作 e.g. `alter user 'root'@'localhost' identified by 'Root@123';`
                - [重置密码解决MySQL for Linux错误 ERROR 1045 (28000)](https://www.cnblogs.com/gumuzi/p/5711495.html)
    - 创建新用户并分配权限/删除用户
        + [6.2.8 Adding Accounts, Assigning Privileges, and Dropping Accounts](https://dev.mysql.com/doc/refman/8.0/en/creating-accounts.html)
        + 创建：`create user 'finley'@'localhost' identified by 'password';`
        + 分配权限
            * 创建后指定权限(注意是两条语句)：
                - `grant all(全部权限，也可指定select/insert/update等等权限) on *.*(数据库.表) to 'finley'@'localhost';`
                - e.g. `CREATE USER xd@localhost IDENTIFIED BY 'xd123456';`，创建后指定权限：`GRANT ALL ON *.* to xd@localhost;`
            * 撤销权限
                - `revoke all on *.* from 'finley'@'localhost';`
        + *查看用户表*： `select user,host from mysql.user;`
        + 查看指定用户权限： `show grants for root@localhost;`
        + 删除用户：`drop user finley;`
        + 用户的格式 root@% 和 root@localhost的区别
            * 用户名格式：`'user_name'@'host_name'`
                - 两部分，host_name指定允许什么样的主机用`user_name`连接
                - 只有`user_name`的用户名，等价于：`'user_name'@'%'`，e.g. me 等价于 me@%
                - 若去掉引号是合法的标识符则可去掉引号
                - 若用户名或者host包含特殊符号，如`-`，则需要引号；
                - 包含通配符也需要引号：`'test-user'@'%.com'`
                - 注意引号范围，'me'@'localhost'和'me@localhost'，后者表示'me@localhost'@'%'
            * `mysql.user`用户表里，User和Host是分开存的，两个一起组成了主键
                - `desc mysql.user;`可以查看
                - 所以root(root@%) 和 root@localhost 是不同的mysql.user表记录，可能会存在User列同名的记录
            * `host_name`
                - @ 后面即可以是主机名也可以是ip地址
                - `%`和`_`通配符可以用在host_name中，表示 `like`语法
                    + `'192.51.100.*'` 可表示网络地址下(/24)的所有主机(注意此时@后面是需要引号`'`的)
                    + `'%.myusql.com'` 可表示任何在`mysql.com`域下的地址
            * [6.2.4 Specifying Account Names](https://dev.mysql.com/doc/refman/8.0/en/account-names.html)
* SQL使用
    - [3.3.1 Creating and Selecting a Database](https://dev.mysql.com/doc/refman/8.0/en/creating-database.html)
    - *关键字*大小写不敏感，以分号`;`(semicolon)结束语句，字符串使用 `'` 和 `"` 都可以
    - 查询
        + `select user();`
    - 数据库
        + 查看所有数据库 `show databases;`
        + 创建 `create database menagerie;`
            * unix下数据库名和表名大小写敏感，windows下则不敏感(所以不管何种情况，推荐都使用相同字符)
            * 在MySQL的语法操作中（MySQL5.0.2之后），可以使用`CREATE DATABASE`和`CREATE SCHEMA`来创建数据库，两者在功能上是一致的(不过其他数据库两种有所不同)
                - [MySQL中CREATE DATABASE和CREATE SCHEMA区别](http://blog.useasp.net/archive/2013/05/21/The-difference-between-create-database-and-create-schema-in-mysql.aspx)
        + 使用 `use menagerie`
            * 每次使用都要进入数据库，可以连接时就选择：`mysql -h host -u user -p menagerie`
            * 此处menagerie并不指密码，若要指定密码-p后面直接接密码`-ppassword`，不要空格(不推荐)
        + 查看当前选择的是哪个数据库 `select database()`，选择某个数据库后才会打印
    - 表
        + 查看所有表 `show tables;`
        + 创建
            * `create table pet (name varchar(20), owner varchar(20), species varchar(20), sex char(1), birth date, death date);`
        + 查看指定表 `describe pet;` 或 `desc pet;` 或者 `explain pet;`
        + 速记select查指定表或者所有数据库用`()`不带`s`,show不用括号带s

* 备份
    - 使用客户端工具进行导出和导入
        + MySQL Workbench：Server->Data Export/Import->选择数据库->选择导出到文件/项目目录->Export Progress页进行导出(点击Start Export)
    - `mysqldump ---user [user name] ---password=[password] [database name] > [dump file]`

e.g.示例(暂未测试)

```sh
#!/bin/bash
### MySQL Server Login Info ###
MUSER="root"
MPASS="MYSQL-ROOT-PASSWORD"
MHOST="localhost"
MYSQL="$(which mysql)"
MYSQLDUMP="$(which mysqldump)"
BAK="/backup/mysql"
GZIP="$(which gzip)"
### FTP SERVER Login info ###
FTPU="FTP-SERVER-USER-NAME"
FTPP="FTP-SERVER-PASSWORD"
FTPS="FTP-SERVER-IP-ADDRESS"
NOW=$(date +"%d-%m-%Y")

### See comments below ###
### [ ! -d $BAK ] && mkdir -p $BAK || /bin/rm -f $BAK/* ###
[ ! -d "$BAK" ] && mkdir -p "$BAK"

DBS="$($MYSQL -u $MUSER -h $MHOST -p$MPASS -Bse 'show databases')"
for db in $DBS
do
 FILE=$BAK/$db.$NOW-$(date +"%T").gz
 $MYSQLDUMP -u $MUSER -h $MHOST -p$MPASS $db | $GZIP -9 > $FILE
done

lftp -u $FTPU,$FTPP -e "mkdir /mysql/$NOW;cd /mysql/$NOW; mput /backup/mysql/*; quit" $FTPS
```

* MySQL 中使用浮点数和定点数来表示小数
    - [MySQL FLOAT、DOUBLE、DECIMAL（小数类型）](http://c.biancheng.net/view/2424.html)
    - 浮点类型有两种，分别是单精度浮点数（`FLOAT`）和双精度浮点数（`DOUBLE`）；定点类型只有一种，就是 `DECIMAL`
    - 浮点类型和定点类型都可以用`(M, D)`来表示，其中`M`称为精度，表示总共的位数；`D`称为标度，表示小数的位数
        + `M` 和 `D` 在 FLOAT 和DOUBLE 中是可选的，`FLOAT` 和 `DOUBLE` 类型将被保存为硬件所支持的最大精度。`DECIMAL` 的默认 D 值为 0、M 值为 10
    - 浮点数相对于定点数的优点是在长度一定的情况下，浮点数能够表示更大的范围；缺点是会引起精度问题
    - 小结：在 MySQL 中，定点数以字符串形式存储，在对精度要求比较高的时候（如货币、科学数据），使用 `DECIMAL` 的类型比较好，另外两个浮点数进行减法和比较运算时也容易出问题，所以在使用浮点数时需要注意，并尽量避免做浮点数比较
* double精度问题(存金额)
    - 使用 `DECIMAL(M,D)`
        + 注意：如果用这种形式，`DECIMAL(20,5)`，整数为只有20-5位，即M表示该值的总共长度，D表示小数点后面的长度
            * DECIMAL在不指定精度时，默认整数为10，小数为0，即`DECIMAL`默认为`DECIMAL(10,0)`
        + M is the maximum number of digits (the precision). It has a range of 1 to 65
        + D is the number of digits to the right of the decimal point (the scale). It has a range of 0 to 30 and must be no larger than M.
        + If D is omitted, the default is 0. If M is omitted, the default is 10
    - [ DECIMAL Data Type Characteristics](https://dev.mysql.com/doc/refman/5.7/en/precision-math-decimal-characteristics.html)

## 27.8 MySQL C API

* [27.8 MySQL C API](https://dev.mysql.com/doc/refman/5.7/en/c-api.html)
    - 实现在库 libmysqlclient 里面(`libmysqlclient.so`、`libmysqlclient.a`)
    - 可以用 `mysql_config` 来查看编译需要的选项
        + 查看其内容是一个shell脚本，里面会获取mysql安装的位数，并执行根据位数拼接得到的脚本，e.g. `mysql_config-64`，而`mysql_config-64`中定义了各个路径/库名等信息，根据传入的选项进行echo输出(不传选项则会打印提示各选项会获取的内容)
        + `mysql_config --include`, -I头文件路径
        + `mysql_config --libs`, 编译需要链接的库名、库路径、依赖库等编译选项
        + `--cflags`, 返回为："-I/usr/include/mysql -m64 "
            * -m选项用来指定位数
            * -m32 int、long、指针都指定为4字节(32bits)
            * -m64 int指定为4字节，long、指针指定为8字节(64bits)
        + `--cxxflags`, 返回为： "-I/usr/include/mysql -m64 "
* 若Linux无 MySQL C/C++驱动，则需安装mysql-devel包(会提供`mysql_config`和客户程序需要的相关.so库)
    - `yum install mysql-devel`
    - `unsigned int mysql_errno(MYSQL *mysql)` 获取错误码，系统`errno`并不会设置
    - `const char *mysql_error(MYSQL *mysql)` 获取错误信息

[27.8.6 C API Function Overview](https://dev.mysql.com/doc/refman/5.7/en/c-api-function-overview.html)

```c
mysql_init
    if (NULL == mysql_init(&node->fd)
mysql_options
    if (0 == mysql_options(&node->fd, MYSQL_SET_CHARSET_NAME, "utf8"))
mysql_real_connect
    (node->mysql_sock = mysql_real_connect(&node->fd, sp->ip, sp->user, sp->passwd, sp->db_name, sp->port, NULL, 0)))
mysql_close
     mysql_close(sp->sql_pool[index].mysql_sock);
mysql_ping
    Checks whether the connection to the server is working.
    mysql_ping(sp->sql_pool[index].mysql_sock);

mysql_real_query
    int mysql_real_query(MYSQL *mysql, const char *stmt_str, unsigned long length)
    mysql_real_query() executes the SQL statement pointed to by stmt_str, a string length bytes long.
    Return Values
    Zero for success. Nonzero if an error occurred.

mysql_query()
    mysql_query() cannot be used for statements that contain binary data;不能包含二进制
        (Binary data may contain the \0 character, which mysql_query() interprets as the end of the statement string.)
     In addition, mysql_real_query() is faster than mysql_query() because it does not call strlen() on the statement string.

mysql_field_count()
    Returns the number of columns for the most recent query on the connection.
    unsigned int
    If you want to know whether the statement mysql_real_query() returns a result set, you can use mysql_field_count() to check for this

mysql_store_result
    After invoking mysql_query() or mysql_real_query(), you must call mysql_store_result() or mysql_use_result() for every statement that successfully produces a result set

    You must also call mysql_free_result() after you are done with the result set.需要释放资源

    check whether mysql_error() returns a nonempty string, mysql_errno() returns nonzero, or mysql_field_count() returns zero.

mysql_free_result
    void mysql_free_result(MYSQL_RES *result)
    frees the memory allocated for a result set by mysql_store_result(), mysql_use_result(),

mysql_fetch_row
    MYSQL_ROW mysql_fetch_row(MYSQL_RES *result)
    You can call mysql_fetch_row() to fetch rows from the result set, or mysql_row_seek() and mysql_row_tell() to obtain or set the current row position within the result set.

mysql_num_rows()
    my_ulonglong mysql_num_rows(MYSQL_RES *result)
    Returns the number of rows in the result set.
    The use of mysql_num_rows() depends on whether you use mysql_store_result() or mysql_use_result() to return the result set. If you use mysql_store_result(), mysql_num_rows() may be called immediately. If you use mysql_use_result(), mysql_num_rows() does not return the correct value until all the rows in the result set have been retrieved.
mysql_affected_rows().
    For statements such as INSERT, UPDATE, or DELETE, the number of affected rows can be obtained with mysql_affected_rows().
    select用mysql_num_rows

mysql_error
    const char *mysql_error(MYSQL *mysql)
        For the connection specified by mysql, mysql_error() returns a null-terminated string containing the error message for the most recently invoked API function that failed. If a function did not fail, the return value of mysql_error() may be the previous error or an empty string to indicate no error.
    For functions that reset mysql_error(), either of these two tests can be used to check for an error:(对于会设置error的函数，使用以下任一个检查是否有错误产生)
        if(*mysql_error(&mysql))
        {
          // an error occurred
        }

        if(mysql_error(&mysql)[0])
        {
          // an error occurred
        }

mysql_errno()
    For the connection specified by mysql, mysql_errno() returns the error code for the most recently invoked API function that can succeed or fail. A return value of zero means that no error occurred.
    Some functions such as mysql_fetch_row() do not set mysql_errno() if they succeed
```

## binlog

* binlog
    - binlog是一个二进制格式的文件，用于记录用户对数据库更新的SQL语句信息，例如更改数据库表和更改内容的SQL语句都会记录到binlog里，但是对库表等内容的查询不会记录(`binlog.000035`形式命名)
    - 默认情况下，binlog日志是二进制格式的，不能使用查看文本工具的命令（比如，cat，vi等）查看，而使用`mysqlbinlog`解析查看。
    - 主要作用是用于数据库的主从复制及数据的增量恢复
    - `show variables like 'log_bin%';` 查看数据库是否开启binlog(`log_bin`变量为`ON`则是开启状态)
        + `show binary logs;` 查看现有的binlog文件
    - 清理和关闭binlog
        + `PURGE`指令删除binlog
            * `purge binary logs before '2019-11-25 13:09:51';` 将指定时间之前的binlog清掉
            * `purge binary logs to 'bin.000055'` 将bin.000055之前的binlog清掉
            * `purge master logs before date_sub(current_date, interval 1 day);` 删除1天前的binlog
        + 可设置binlog过期时间
            * `show variables like '%expire%';` 查看时间
            * `binlog_expire_logs_seconds`，默认2592000（30天）过期
            * e.g. 可设置2天，`set global binlog_expire_logs_seconds=60*60*24*2;`
        + `binlog_expire_logs_seconds`设置之后不会立即清除过期的，触发条件是
            * binlog大小超过`max_binlog_size` (`show variables like 'max_binlog_size';`查看大小)
            * 手动执行flush logs
            * 重新启动时(MySQL将会new一个新文件用于记录binlog)
        + [mysql清理和关闭binlog日志](https://www.jianshu.com/p/e4cfbcdb5fc1)
    - 可以在配置文件`my.cnf`中开启：
        + `log-bin = 目录如/data/3306/mysql-bin`
    - [MySQL Binlog详解](https://www.cnblogs.com/xhyan/p/6530861.html)
* [初探 MySQL 的 Binlog](https://segmentfault.com/a/1190000003072963)
    - Binlog描述了数据库的改动，如建表、数据改动等，也包括一些潜在改动，比如 DELETE FROM ran WHERE bing = luan，然而一条数据都没被删掉的这种情况
    - 更新类操作多的话，binlog可能很大，此次了解binlog就是因为binlog快40GB，导致磁盘占满。。。

* 查看每个数据库大小
    - `select table_schema, concat(round(sum(data_length/1024/1024),2),'MB') as data from information_schema.tables group by table_schema;`

## MySQL锁

* [MySQL锁详解](https://www.cnblogs.com/luyucheng/p/6297752.html)
    - 数据库锁定机制简单来说，就是数据库为了保证数据的一致性，而使各种共享资源在被并发访问变得有序所设计的一种规则
    - MySQL各存储引擎使用了三种类型（级别）的锁定机制：`表级锁定`，`行级锁定`和`页级锁定`
        + `表级锁定（table-level）`
            * 表级别的锁定是MySQL各存储引擎中最大颗粒度的锁定机制
            * 该锁定机制最大的特点是实现逻辑非常简单，带来的系统负面影响最小，可以很好的避免死锁问题
            * 不过，锁定颗粒度大所带来最大的负面影响就是出现锁定资源争用的概率也会最高，致使并发度大打折扣
            * 使用表级锁定的主要是`MyISAM`，MEMORY，CSV等一些非事务性存储引擎
        + `行级锁定（row-level）`
            * 行级锁定最大的特点就是锁定对象的颗粒度很小，也是目前各大数据库管理软件所实现的锁定颗粒度最小的
            * 由于锁定颗粒度很小，所以发生锁定资源争用的概率也最小，能够给予应用程序尽可能大的并发处理能力而提高一些需要高并发应用系统的整体性能
            * 由于锁定资源的颗粒度很小，所以每次获取锁和释放锁需要做的事情也更多，带来的消耗自然也就更大了。此外，行级锁定也最容易发生死锁
            * 使用行级锁定的主要是`InnoDB`存储引擎
        + `页级锁定（page-level）`
            * 页级锁定是MySQL中比较独特的一种锁定级别，在其他数据库管理软件中也并不是太常见
            * 页级锁定的特点是锁定颗粒度介于行级锁定与表级锁之间，所以获取锁定所需要的资源开销，以及所能提供的并发处理能力也同样是介于上面二者之间。另外，页级锁定和行级锁定一样，会发生死锁
            * 使用页级锁定的主要是BerkeleyDB存储引擎
    - 总的来说，MySQL这3种锁的特性可大致归纳如下：
        + 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低；
        + 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高；   
        + 页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。
        + 适用：从锁的角度来说，表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用，如一些在线事务处理（OLTP）系统。
    - MySQL表级锁的锁模
        + MySQL的表级锁有两种模式：表`共享读锁`（Table Read Lock）和表`独占写锁`（Table Write Lock）
        + 由于`MyISAM`存储引擎使用的锁定机制完全是由MySQL提供的`表级锁定`实现，所以下面以`MyISAM`存储引擎作为示例存储引擎：
        + 锁模式的兼容性：
            * 对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求
            * 对MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作
            * MyISAM表的读操作与写操作之间，以及写操作之间是串行的。当一个线程获得对一个表的写锁后，只有持有锁的线程可以对表进行更新操作。其他线程的读、写操作都会等待，直到锁被释放为止
        + MyISAM在执行`查询`语句（SELECT）前，会自动给涉及的所有表加`读锁`，在执行`更新`操作（UPDATE、DELETE、INSERT等）前，会自动给涉及的表加`写锁`，这个过程并不需要用户干预，因此，用户一般*不需要*直接用`LOCK TABLE`命令给MyISAM表显式加锁    
        + MyISAM表锁优化建议
            * 在优化MyISAM存储引擎锁定问题的时候，最关键的就是如何让其提高并发度。由于锁定级别是不可能改变的了，所以我们首先需要尽可能让锁定的时间变短，然后就是让可能并发进行的操作尽可能的并发
    - 行级锁定
        + 行级锁定不是MySQL自己实现的锁定方式，而是由其他存储引擎自己所实现的，如广为大家所知的`InnoDB`存储引擎，以及MySQL的分布式存储引擎`NDBCluster`等都是实现了行级锁定
        + 考虑到行级锁定君由各个存储引擎自行实现，而且具体实现也各有差别，而`InnoDB`是目前事务型存储引擎中使用最为广泛的存储引擎，所以这里主要分析一下InnoDB的锁定特性
        + InnoDB的锁定模式实际上可以分为四种：`共享锁（S）`，`排他锁（X）`，`意向共享锁（IS）`和`意向排他锁（IX）`
            * 如果一个事务请求的锁模式与当前的锁兼容(查看链接)，InnoDB就将请求的锁授予该事务；反之，如果两者不兼容，该事务就要等待锁释放
            * `意向锁`是InnoDB自动加的，不需用户干预。对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加`排他锁（X)`；对于普通SELECT语句，InnoDB不会加任何锁
            * 事务可以通过以下语句显式给记录集加共享锁或排他锁：
                - 共享锁（S）：`SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE`
                    + 主要用在需要数据依存关系时来确认某行记录是否存在，并确保没有人对这个记录进行UPDATE或者DELETE操作
                    + 但是如果当前事务也需要对该记录进行更新操作，则很有可能造成死锁，对于锁定行记录后需要进行更新操作的应用，应该使用`SELECT... FOR UPDATE`方式获得排他锁
                - 排他锁（X)：`SELECT * FROM table_name WHERE ... FOR UPDATE`
        + InnoDB行锁实现方式
            * InnoDB行锁是通过给索引上的索引项加锁来实现的，只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁
            * 在实际应用中，要特别注意InnoDB行锁的这一特性，不然的话，可能导致大量的锁冲突，从而影响并发性能
