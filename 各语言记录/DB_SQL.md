## 数据库

* 关系型数据库遵循ACID规则
    - 1、A (Atomicity) 原子性
        + 事务里的所有操作要么全部做完，要么都不做
    - 2、C (Consistency) 一致性
        + 数据库要一直处于一致的状态，事务的运行不会改变数据库原本的一致性约束。
        + 例如现有完整性约束a+b=10，如果一个事务改变了a，那么必须得改变b，使得事务结束后依然满足a+b=10，否则事务失败。
    - 3、I (Isolation) 隔离性
        + 指并发的事务之间不会互相影响
    - 4、D (Durability) 持久性
        + 指一旦事务提交后，它所做的修改将会永久的保存在数据库上，即使出现宕机也不会丢失。

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

### `count(1)` and `count(字段)` `count(*)`

[`count(1)`、`count(*)`与`count(列名)`的执行区别](https://blog.csdn.net/iFuMI/article/details/77920767)

`count(*)`和`count(1)`执行的效率是完全一样的。
`count(*)`的执行效率比`count(col)`高，因此可以用`count(*)`的时候就不要去用`count(col)`。
`count(col)`的执行效率比`count(distinct col)`高，不过这个结论的意义不大，这两种方法也是看需要去用。

如果想统计行数, 就用 `count(*)` 避开麻烦.

2. `count(1)` and `count(字段)`
两者的主要区别是
（1） count(1) 会统计表中的所有的记录数，包含字段为null 的记录。
（2） count(字段) 会统计该字段在表中出现的次数，忽略字段为null 的情况。即不统计字段为null 的记录。 

## distinct

* `SELECT distinct name,class,age FROM xdtest.student;`
    - distinct要放在开头
    - 可以作用于多列，此处即为name,class,age三者都不同时才会过滤

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

## group by

* `group by` 结合聚合函数，可根据一个或多个列对结果集进行分组。
    - 参考链接中的示例：[SQL GROUP BY 语句](https://www.runoob.com/sql/sql-groupby.html)
    - `SELECT site_id, SUM(access_log.count) AS nums FROM access_log GROUP BY site_id;` 统计每个站点site_id对应的访问次数
    - `select site_id, count(1) as num from access_log [where...] group by site_id;` 各个site_id的数据库记录条数
* `having`
    - e.g. `select city, count(*),age from dbo.user where departmentID=2 group by city,age` (可以先where)
    - Having用于对where和group by查询出来的分组经行过滤，查出满足条件的分组结果(mysql中实验不用where也可用having)
        + WHERE 是用于在初始表中筛选查询，HAVING用于在WHERE和GROUP BY 结果分组中查询
        + Having 子句中的每一个元素也必须出现在select列表中
        + Having语句可以使用聚合函数，而where不使用
    - 在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用
        + `SELECT Customer,SUM(OrderPrice) FROM Orders GROUP BY Customer HAVING SUM(OrderPrice)<2000`

* 算每个账户中，盈利>0的数量和总数量
    - 不用inner则为隐式内连接，使用显式的内连接来避免隐式带来的不确定性(内连接则只会查询交集，profit<=0的记录不会返回)
    - 使用`outer join`则会把b表全部查出来，不过a表字段为null。a表位置应该调整为`sum(profit>0)`
    - 下面的几种方式，使用d方式

```sql
# a. 内连接只查交集(会缺失profit<=0)
select a.account,a.ac,b.bc from (SELECT account,count(1) ac FROM deal where profit>0 group by account) as a 
inner join (select account,count(1) bc from deal group by account) as b 
on a.account=b.account

# b. 右(外)连接返回右边全部(b有a无的记录，对应a中的字段为null)
select a.account,a.ac,b.bc from (SELECT account,count(1) ac FROM deal where profit>0 group by account) as a 
right outer join (select account,count(1) bc from deal group by account) as b 
on a.account=b.account

# c. 查询profit>0的数量时：count(1)调整为sum(profit>0)，b表操作也可调整为sum(1)算总量
select a.account,a.ac,b.bc from (SELECT account,sum(profit>0) ac FROM deal group by account) as a 
inner join (select account,count(1) bc from deal group by account) as b 
on a.account=b.account
```

```sql
# d. 不用连接
select account,sum(profit>0) c1, count(*) c2 from deal group by account;

# 算好占比
select new.*, new.c1/new.c2 from (select account,sum(profit>0) c1, count(*) c2 from deal group by account) as new;
```

## 视图

* 视图(view)
    - [12丨视图在SQL中的作用是什么，它是怎样工作的？](https://time.geekbang.org/column/article/105643)
    - 视图作为一张虚拟表(视图不具有数据)，帮我们封装了底层与数据表的接口
        + 它相当于是一张表或多张表的数据结果集。
        + 视图的这一特点，可以帮我们简化复杂的 SQL 查询，比如在编写视图后，我们就可以直接重用它，而不需要考虑视图中包含的基础查询的细节
    - 通常情况下，小型项目的数据库可以不使用视图，但是在大型项目中，以及数据表比较复杂的情况下，视图的价值就凸显出来了，它可以帮助我们把经常查询的结果集放到虚拟表中，提升使用效率
    - 创建视图：`CREATE VIEW view_name AS SELECT column1, column2 FROM table WHERE condition`
    - 当视图创建之后，它就相当于一个虚拟表，可以直接使用
        + `SELECT * FROM player_above_avg_height` (`player_above_avg_height`是创建好的一个视图)
        + 当我们创建好一张视图之后，还可以在它的基础上继续创建视图
    - 修改视图：`ALTER VIEW view_name AS SELECT column1, column2 FROM table WHERE condition`
    - 删除视图：`DROP VIEW view_name`

## 存储过程

* 存储过程(Stored Procedure)
    - [13丨什么是存储过程，在实际项目中用得多么？](https://time.geekbang.org/column/article/106250)
    - SQL 的存储过程和视图一样，都是对 SQL 代码进行封装，可以反复利用
        + 不过它和视图不同，视图是虚拟表，通常不对底层数据表直接操作，而存储过程是程序化的 SQL，可以直接操作底层数据表，相比于面向集合的操作方式，能够实现一些更复杂的数据处理
    - 存储过程可以说是由SQL语句和流控制语句构成的语句集合，它和我们之前学到的函数一样，可以接收输入参数，也可以返回输出参数给调用者，返回计算结果
        + 一旦存储过程被创建出来，使用它就像使用函数一样简单，我们直接通过调用存储过程名即可
        + 流控制语句
            * `BEGIN…END`，中间的语句以`;`作为结束符
                - `DECLARE` 用来声明变量，使用于 BEGIN…END 语句中间，而且需要在其他语句使用之前进行变量的声明
                - `SET` 赋值语句，用于对变量进行赋值
                - `SELECT…INTO` 把从数据表中查询的结果存放到变量中，也就是为变量赋值
            * `IF…THEN…ENDIF` 条件判断语句
            * `CASE` 用于多条件的分支判断
                - `CASE WHEN expression1 THEN ... `
                       `WHEN expression2 THEN ... `
                       `... `
                       `ELSE ` --ELSE语句可以加，也可以不加。加的话代表的所有条件都不满足时采用的方式。
                  `END`
            * `LOOP、LEAVE 和 ITERATE` LOOP 是循环语句，使用 LEAVE 可以跳出循环，使用 ITERATE 则可以进入下一次循环
            * `REPEAT…UNTIL…END REPEAT` 循环语句
            * `WHILE…DO…END WHILE` 循环语句
    - 创建：`CREATE PROCEDURE 存储过程名称([参数列表]) BEGIN 需要执行的语句 END`
        + 使用方式：`CALL add_num(50);`
    - 删除存储过程：`DROP PROCEDURE`
    - 更新：`ALTER PROCEDURE`
    - 优点
        + 存储过程可以一次编译多次使用
        + 其次它可以减少开发工作量
        + 还有一点，存储过程的安全性强，我们在设定存储过程的时候可以设置对用户的使用权限，这样就和视图一样具有较强的安全性
        + 最后它可以减少网络传输量，因为代码封装到存储过程中，每次使用只需要调用存储过程即可，这样就减少了网络传输量
        + 基于上面这些优点，不少大公司都要求大型项目使用存储过程，比如微软、IBM 等公司。但是国内的阿里并不推荐开发人员使用存储过程
    - 存储过程虽然有诸如上面的好处，但缺点也是很明显的
        + 它的可移植性差，存储过程不能跨数据库移植，比如在 MySQL、Oracle 和 SQL Server 里编写的存储过程，在换成其他数据库时都需要重新编写
        + 其次调试困难，只有少数 DBMS 支持存储过程的调试
        + 此外，存储过程的版本管理也很困难，比如数据表索引发生变化了，可能会导致存储过程失效
        + 最后它不适合高并发的场景，高并发的场景需要减少数据库的压力，有时数据库会采用分库分表的方式，而且对可扩展性要求很高，在这种情况下，存储过程会变得难以维护，增加数据库的压力，显然就不适用了

## LeetCode

* [182. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)
    - 编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱
    - `select distinct a.Email from Person a, Person b where a.Id!=b.Id and a.Email=b.Email`
        + 执行用时 :196 ms, 在所有 MySQL 提交中击败了42.57%的用户
        + 内存消耗 :0B, 在所有 MySQL 提交中击败了100.00%的用户
    - `select Stat.Email from (select Email,count(*) as count from Person group by Email) as Stat where Stat.count > 1`
        + 使用`group by`查询每个Email的`count(*)`，再查>0的记录
        + 执行用时 :167 ms, 在所有 MySQL 提交中击败了57.42%的用户
        + 内存消耗 :0B, 在所有 MySQL 提交中击败了100.00%的用户
    - `select Email as count from Person group by Email having count(*)>1`
        + 使用`group by`, `having`
        + 执行用时 :147 ms, 在所有 MySQL 提交中击败了96.06%的用户
        + 内存消耗 :0B, 在所有 MySQL 提交中击败了100.00%的用户
* [595. 大的国家](https://leetcode-cn.com/problems/big-countries/)
    - `where` `or`条件查询 VS 单个`where`后通过`union`连接
    - 通常情况下, 多个索引列时，用`UNION`替换`WHERE`子句中的`OR`将会起到较好的效果. 对索引列使用`OR`将造成全表扫描
        + 若坚持要用`OR`, 那就需要返回记录最少的索引列写在最前面
        + 防止`or`导致索引失效
    - 另外几种索引失效的可能：[规避MySQL中的索引失效](https://juejin.im/post/5df8830cf265da339565e184?hmsr=coffeephp.com)
        + 若where语句中索引列使用了负向查询，可能会导致索引失效
            * 负向查询包括：NOT、!=、<>、!<、!>、NOT IN、NOT LIKE等
            * 字段设为not null并提供默认值
        + 在索引列上使用内置函数，一定会导致索引失效
            * 尽量在应用程序中进行计算和转换
        + 对索引列进行运算，一定会导致索引失效
        + 联合索引中，where中索引列违背最左匹配原则，一定会导致索引失效
            * 当创建一个联合索引的时候，如(k1,k2,k3)，相当于创建了(k1)、(k1,k2)和(k1,k2,k3)三个索引，这就是最左匹配原则
            * k2=2/k3=3/k2=2 and k3=3并不会命中索引，k1=1 and k3=3只会命中k1
            * 建立组合索引，必须把区分度高的字段放在前面
    - 对于相同的列查多个值，用`in`来替换`or`，e.g. `id=1 or id=2`调整为`id in(1,2)`
    - 可以用`explain sql语句`来查看执行过程，`key`列可以看到本次是否用到索引，据此判断索引是否失效
* `case...when...end`语句
    - `update salary set sex=case sex when 'm' then 'f' else 'm' end` 条件更新
    - [627. 交换工资](https://leetcode-cn.com/problems/swap-salary/)
* `mod(id, 2) = 1` 字段值为奇数，或`id%2=1`，`mod`效率更高
* [181. 超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/)
    - `select a.Name as Employee from Employee a join Employee b on a.ManagerId=b.Id and a.Salary>b.Salary` 268ms
    - `select a.Name as Employee from Employee a, Employee b where a.ManagerId=b.Id and a.Salary>b.Salary` 301ms
        + 使用`join...on`，避免产生笛卡尔积，产生冗余的数据
        + ON语句的执行是在JOIN语句之前，先会判断是否满足on再进行join
        + 当两张表的数据量比较大，又需要连接查询时，应该使用 FROM table1 JOIN table2 ON xxx的语法，避免使用 FROM table1,table2 WHERE xxx 的语法，因为后者会在内存中先生成一张数据量比较大的笛卡尔积表，增加了内存的开销
        + sql执行流程顺序：from->on->join->where->group by->select->having->order by->limit
        + [SQL与笛卡尔积](https://juejin.im/post/5c1c5301e51d451cdc394d13)
    - `select Name as Employee from Employee a where a.ManagerId is not null and a.Salary>(select Salary from Employee b where b.Id=a.ManagerId)` 779ms，这个方式太直线思维了。。。
* [183. 从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)
    - `select a.Name as Customers from Customers a left join Orders b on a.Id=b.CustomerId where b.CustomerId is null` 359ms，注意`on`后面再接`where`过滤，而不是用`and`作为`join...on`的条件
    - `not in` 411ms
* [1179. 重新格式化部门表](https://leetcode-cn.com/problems/reformat-department-table/)
    - 每个id在各个月份的收入，使用`case 列 when 列值 then 需要的列值 end`，配合内置函数

```sql
# Write your MySQL query statement below
select id, 
sum(case month when 'Jan' then revenue end) Jan_Revenue,
sum(case month when 'Feb' then revenue end) Feb_Revenue,
sum(case month when 'Mar' then revenue end) Mar_Revenue,
sum(case month when 'Apr' then revenue end) Apr_Revenue,
sum(case month when 'May' then revenue end) May_Revenue,
sum(case month when 'Jun' then revenue end) Jun_Revenue,
sum(case month when 'Jul' then revenue end) Jul_Revenue,
sum(case month when 'Aug' then revenue end) Aug_Revenue,
sum(case month when 'Sep' then revenue end) Sep_Revenue,
sum(case month when 'Oct' then revenue end) Oct_Revenue,
sum(case month when 'Nov' then revenue end) Nov_Revenue,
sum(case month when 'Dec' then revenue end) Dec_Revenue 
 from Department group by id
```

* [196. 删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)
    - 删除重复并保留最小的记录
    - `delete p1 from Person p1,Person p2 where p1.Email=p2.Email and p1.Id>p2.Id` p1记录大于所有p2表Id的记录
        + 1949ms
    - `delete p1 from Person p1 inner join Person p2 on p1.Email=p2.Email where p1.Id>p2.Id` 用join快一点
        + 1540ms
    - `delete from Person where Id not in (select id from (select min(Id) as id from Person group by Email) as temp)` 
        + 1177ms
        + 先用group by找出每个email最小的id，作为一个临时表，再删除Person表中id不存在于这个临时表中的id的记录
* [197. 上升的温度](https://leetcode-cn.com/problems/rising-temperature/)
    - `select w1.Id from Weather w1 inner join Weather w2 on datediff(w1.RecordDate, w2.RecordDate)=1 and w1.Temperature > w2.Temperature`
    - `datediff()`函数保证日期间隔，id和日期都不一定是按顺序递增的，可能id=1对应的日期晚于id=2的日期，所以不能依赖于id的顺序
    - 前面参数减后面参数，`select datediff('2015-01-01', '2015-01-02')`返回-1，`select datediff('2015-01-02', '2015-01-01')`返回1
* [596. 超过5名学生的课](https://leetcode-cn.com/problems/classes-more-than-5-students/)
    - `select class from courses group by class having count(distinct student)>=5` 216ms, 34.41%
    - 需要加上`distinct`，记录里面并没有保证同一个人选同一门课只有一条记录
* [176. 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)
    - `select (select distinct Salary from Employee order by Salary desc limit 1 offset 1) as SecondHighestSalary`
        + 使用`limit...offset...`，为防止offset没有记录(null问题)，再用`select ...`，并用`as`对列命名
        + 解决null问题，还可用`ifnull(sql, NULL)`来处理
        + 206ms,15.41%
    - `select max(Salary) SecondHighestSalary from Employee where Salary < (select max(Salary) from Employee)`
        + 190ms,19.36%，上面的order by得先排序，更耗时