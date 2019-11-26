## linux mysql操作

* Mysql启动、停止、重启常用命令
    - `service mysqld status` 查看状态
    -  启动
        + 使用service 启动： `service mysqld start`
    - `netstat -anp|grep mysqld` 监听端口3306和33060
    - `service mysqld stop` 停止
    - `service mysqld restart` 重启
* 登录
    - 遇到问题：ERROR 1045 (28000): Access denied for user 'root'@'localhost'
    - 跳过密码认证过程来重置密码
        + vim /etc/my.cnf
        + 在 mysqld 后面任意位置，添加`skip-grant-tables`用来跳过密码验证的过程
        + 保存配置重启mysqld，`service mysqld restart`
        + 命令行输入`mysql`即可登录
    - [重置密码解决MySQL for Linux错误 ERROR 1045 (28000)](https://www.cnblogs.com/gumuzi/p/5711495.html)

* 备份
    - `mysqldump ---user [user name] ---password=[password] [database name] > [dump file]`

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

[27.8 MySQL C API](https://dev.mysql.com/doc/refman/5.7/en/c-api.html)

implemented in the libmysqlclient library

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

