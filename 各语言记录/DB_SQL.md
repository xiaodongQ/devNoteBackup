## 数据库

### DML、DDL、DCL

```sql
DML（data manipulation language）数据操作语言
    SELECT、UPDATE、INSERT、DELETE
    CALL, call a PL/SQL or Java subprogram
    EXPLAIN PLAN, explain access path to data(Oracle RDBMS)
    LOCK TABLE, control concurrency 锁，用于控制并发
    用来对数据库里的数据进行操作的语言
    SQL中处理数据等操作统称为数据操纵语言

DDL（data definition language）数据定义语言
    用于定义和管理 SQL 数据库中的所有对象的语言
    CREATE
    ALTER
    DROP  删除
    TRUNCATE
        Truncate table 表名 速度快,而且效率高,因为:
            TRUNCATE TABLE 在功能上与不带 WHERE 子句的 DELETE 语句相同：二者均删除表中的全部行。
            但 TRUNCATE TABLE 比 DELETE 速度快，且使用的系统和事务日志资源少。
            DELETE 语句每次删除一行，并在事务日志中为所删除的每行记录一项。
            TRUNCATE TABLE 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。
        TRUNCATE TABLE 删除表中的所有行，但表结构及其列、约束、索引等保持不变。
            新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 DELETE。
        如果要删除表定义及其数据，请使用 DROP TABLE 语句。

        对于由 FOREIGN KEY 约束引用的表，不能使用 TRUNCATE TABLE，而应使用不带 WHERE 子句的 DELETE 语句。
            由于 TRUNCATE TABLE 不记录在日志中，所以它不能激活触发器。
        TRUNCATE TABLE 不能用于参与了索引视图的表。
    COMMENT 注释
    GRANT   授权
    REVOKE  收回已经授予的权限

DCL（Data Control Language）数据控制语言
    用来设置或更改数据库用户或角色权限的语句，包括（grant,deny,revoke等）语句。
    用来授予或回收访问数据库的某种特权，并控制数据库操纵事务发生的时间及效果，对数据库实行监视等。
    在默认状态下，只有sysadmin,dbcreator,db_owner或db_securityadmin等人员才有权力执行DCL
```

### sql

#### 删表记录方式区别

```sql
1. 删除整张表的数据：
delete from table_name;
    DML语言，每次删除一行，都在事务日志中为所删除的每行记录一项。产生rollback，事务提交之后才生效;如果有相应的 trigger,执行的时候将被触发，如果删除大数据量的表速度会很慢。
    删除表中数据而不删除表的结构(定义)，同时也不释放空间。

2. truncate table table_name;
    DDL语言，操作立即生效,自动提交，原数据不放到rollback segment中,不能回滚. 操作不触发trigger.
    将表中数据全部删除，在功能上和不带where子句的delete语句相同
    删除内容、释放空间但不删除表的结构(定义)。

3. drop table table_name;
    DDL语言，删除表的结构，以及被依赖的约束(constrain),触发器(trigger),索引(index);
    立即执行，执行速度最快
    删除内容和定义，释放空间。

执行速度：
　　drop > truncate > delete

在没有备份情况下，谨慎使用 drop 与 truncate。要删除表结构使用drop;
```

#### ON DUPLICATE KEY UPDATE

在高并发项目中，使用多线程录入数据有可能造成重复录入

判断数据库是否已存在此主键，如果存在会将录入操作更改为更新操作

常规方式：先查询，有则更新，没有就添加。

```sql
select count(player_id) from player_count where player_id = 1;//查询统计表中是否有记录
insert into player_count(player_id,count,name) value(1,2,”张三”);//判读如果没有记录就执行insert 操作
//如果存在，执行录入操作
update from player_count set count=1 ，name=‘张三’ where player_id=1;
```

ON DUPLICATE KEY UPDATE

```sql
insert into player_count(player_id,count,name) value(1,1,”张三”)
 on duplicate key update
 count= 2,
 name=”张三”;
```

### count(1) and count(字段) count(*)

[count(1)、count(*)与count(列名)的执行区别](https://blog.csdn.net/iFuMI/article/details/77920767)

count(*)和count(1)执行的效率是完全一样的。
count(*)的执行效率比count(col)高，因此可以用count(*)的时候就不要去用count(col)。
count(col)的执行效率比count(distinct col)高，不过这个结论的意义不大，这两种方法也是看需要去用。

如果想统计行数, 就用 count(*) 避开麻烦.

2. count(1) and count(字段)
两者的主要区别是
（1） count(1) 会统计表中的所有的记录数，包含字段为null 的记录。
（2） count(字段) 会统计该字段在表中出现的次数，忽略字段为null 的情况。即不统计字段为null 的记录。 

## join

* 连接(join)
    - 内连接
        + `inner join on`
        + `select * from a_table a inner join b_table b on a.a_id = b.b_id;`
        + 组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集（阴影）部分。
    - 左(外)连接
        + `left join on` 或 `left outer join on`
        + `select * from a_table a left join b_table b on a.a_id = b.b_id;`
        + left join 是left outer join的简写，它的全称是左外连接，是外连接中的一种。
        + 左(外)连接，**左表**(`a_table`)的记录将会**全部**表示出来，而右表(`b_table`)只会显示符合搜索条件的记录。
        + 右表记录不足的地方均为NULL
    - 右(外)连接
        + `right join on` 或 `right outer join on`
        + `select * from a_table a right outer join b_table b on a.a_id = b.b_id;`
        + right join是right outer join的简写，它的全称是右外连接，是外连接中的一种。
        + 与左(外)连接相反，右(外)连接，左表(`a_table`)只会显示符合搜索条件的记录，而**右表**(`b_table`)的记录将会**全部**表示出来。
    - 全(外)连接
        + 结合左，右外连接的结果
        + 注意：MySQL不支持全连接(`FULL JOIN`)，可以使用`UNION ALL`来实现两个表左、右连接结合的结果
        + e.g. 查询学生表中所有的学生姓名和课程表中的所有课程：
            * `select students.name,course.kecheng from students full join course on students.stuID=course.stuID`
    - [图解MySQL 内连接、外连接、左连接、右连接、全连接](https://blog.csdn.net/plg17/article/details/78758593)

* `显式inner join` 和 `隐式inner join`
    - 显式内连接：`select * from table1 a inner join table2 b on a.id=b.id`
    - 隐式内连接：`select * from table1 a, table2 b where a.id=b.id`
    - 性能上几乎相同，使用显式的内连接来避免隐式带来的不确定性
    - [Explicit vs implicit SQL joins](https://stackoverflow.com/questions/44917/explicit-vs-implicit-sql-joins)

* join的性能问题
    - 参考：[](https://www.cnblogs.com/BeginMan/p/3754322.html)(搜`七.性能优化`小节)
    - 尽量用`inner join`.避免 `LEFT JOIN` 和 `RIGHT JOIN`
        + on 与 where 的执行顺序
            * ON 条件（“A LEFT JOIN B ON 条件表达式”中的ON）用来决定如何从 B 表中检索数据行。如果 B 表中没有任何一行数据匹配 ON 的条件,将会额外生成一行所有列为 NULL 的数据
            * 在`匹配阶段` WHERE 子句的条件都`不会`被使用。仅在`匹配阶段完成以后`，WHERE 子句条件才会被使用。它将从匹配阶段产生的数据中检索过滤。
        + 所以在使用Left (right) join的时候，一定要在先给出尽可能多的匹配满足条件，减少Where的执行(参考链接中的示例)
    - 尽量避免子查询，而用join
        + 参考链接中的示例

## 两数据表归档

* 场景：有两个相同结构的表 `table1`, `table1_his`，每日将`table1`的数据归档到`table1_his`，并清空`table1`表
    - 表中存在自增id，xxx表示感兴趣字段(e.g. "name,id,class")
    - 使用not exists语法：`insert into table1_his(xxx) select xxx from table1 as a where not exists(select name from table1_his as b where a.key=b.key);`
        + `where a.key=b.key` 过滤是为了防止已经插入数据后，再次执行归档会插入重复记录
            * 此处的key是笼统表示，表示能区分两表数据是否同一记录的条件
        + (虽然定时操作只执行一次，但可能多个测试服务的场景，或无法保证一定只执行一次)
        + `not exists`中`select`的内容并不重要，重要的是`not exists` true还是false
* exists子查询可参考：[SQL 子查询 EXISTS 和 NOT EXISTS](https://blog.csdn.net/qq_27571221/article/details/53090467)