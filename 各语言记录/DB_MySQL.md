## Windows MySQL部署

* 步骤
    - 下载mysql zip
    - 没有data目录，如果没有data目录，安装后`net start mysql`启动的时候就会报这个错：MySQL 服务无法启动。
    - 需要使用命令生成data文件夹
        + 到bin目录，`mysqld -install`
        + `mysqld --initialize --user=root --console`初始化后可看到新的data目录(可能会产生一个默认生成的密码)
        + 启动`net start mysql`
        + 登录`mysql -uroot -p` 若生成默认密码则用其登录，若没有回车即可进入mysql控制台
    - `mysqld -remove mysql` 卸载安装的mysql
    - 如果是通过安装包安装的，则可以查看启动的命令：`控制面板-系统和安全-管理工具-服务(双击打开)-找到MySQL-属性(右键)`

* Windows下检查mysql是否开机启动：
    - 控制面板-系统和安全-管理工具-服务(双击打开)-找到MySQL-属性(右键)-启动类型为"自动"则为开机启动
* mysql远程连接mysql时报错： `Host * is not allowed to connect to this MySQL server`
    - `mysql -h 192.xxx.xxx.xxx -u root -p` 报上述错误(检查防火墙对mysql的端口是否开放，有这个报错防火墙应该是支持入栈规则)
    - 到本机上终端连接后执行：`select user,host from mysql.user;`，可以看到`host`列都是`localhost`
    - `create user root identified by 'xxxx';`，可以看到host为`%`，且有两个`root`用户
    - `grant all on *.* to root;`


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
        + 初始化root用户时，可能会初始化一个随机密码，查找错误日志文件(CentOS下为/var/log/mysqld.log)，其中打印了初始随机密码
        + MySQL安装程序包含初始化数据目录，包含MySQL中定义的授权表
            * 具体参考：[2.10.1 Initializing the Data Directory](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html)
    - `mysql.user` 授权表定义了MySQL初始用户和其权限，定义了`'root'@'localhost'`超级用户
    - 给初始化的root用户配置密码：
        + 登录：
            * 若有初始密码(本地环境有) `mysql -u root -p` 则从错误日志中找到初始密码(参考上面的说明)
            * 若没有密码 `mysql -u root --skip-password`
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
        + 查看用户表： `select user,host from mysql.user;`
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
        + `select user()`
    - 数据库
        + 查看所有数据库 `show databases;`
        + 创建 `create database menagerie;`
            * unix下数据库名和表名大小写敏感，windows下则不敏感(所以不管何种情况，推荐都使用相同字符)
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

