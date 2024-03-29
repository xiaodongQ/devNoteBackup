# shell

## 循环

Shell 流程控制：
[for 循环](https://www.runoob.com/linux/linux-shell-process-control.html)

```sh
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

## 函数

[Shell 函数](https://www.runoob.com/linux/linux-shell-func.html)

shell中函数的定义格式如下：

```sh
[ function ] funname [()]
{
    action;
    [return int;]
}
```

函数返回值在调用该函数后通过 $? 来获得。

注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。
**调用函数仅使用其函数名即可**。

```sh
demoFun(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"
```

##  参数

$0 是命令本身

## Shell判断字符串包含关系的几种方法

[Shell判断字符串包含关系的几种方法](https://www.cnblogs.com/AndyStudy/p/6064834.html)

A是否包含B判断 (*注意下面示例用的是[[]]，而不是[]*)

方法一：利用grep查找

```sh
result=$(echo $strA | grep "${strB}")
if [[ "$result" != "" ]]
then
    echo "包含"
else
    echo "不包含"
fi
```

方法二：利用通配符

```sh
if [[ $A == *$B* ]]    # 注意，用的是[[]]，并且字符串不加""
then
   echo "包含"
else
    echo "不包含"
fi
```

## 数组

* [Shell数组：shell数组的定义、数组长度](http://c.biancheng.net/cpp/view/7002.html)
* 在Shell中，用括号`()`来表示数组，数组元素用“空格”符号(` `)分割开
* 定义形式如下

```sh
# 数组定义
array_name=(value0 value1 value2 value3)

# 或者
array_name=(
value0
value1
value2
value3
)

# 还可以单独定义数组的各个分量
# 可以不使用连续的下标，而且下标的范围没有限制
array_name[0]=value0
array_name[2]=value2
```

* 读取数组
    - 读取数组元素值的一般格式：`${array_name[index]}`
        + e.g. `valuen=${array_name[2]}` (下标从0开始)
    - 使用 `@` 或 `*` 可以获取数组中的所有元素
        + `${array_name[*]}`
        + `${array_name[@]}`
* 获取数组长度
    - 获取数组长度的方法与获取字符串长度的方法相同
        + `length=${#array_name[@]}` (`()`里多了一个`#`)
        + 或者 `length=${#array_name[*]}`
    - 获取数组单个元素的长度
        + `lengthn=${#array_name[n]}`

## awk

* 利用`awk`计算所有tar文件的总大小
    - `tar_file=($(ls -tr *.tar.gz))`
    - `tar_file_size=$(echo ${tar_file[@]} | xargs stat -c %s)`
        + 其中`tar_file`为一个数组，`${tar_file[@]}`表示数组所有元素
        + `tar_file_size`为所有元素的大小组成的数组
        + `stat -c ` 后面接格式标志，可获取不同信息
            * `%s` 获取字节大小
            * `%y` 修改时间
            * `man stat` 查看各个格式标志
    - `total_size=$(echo ${tar_file_size[@]} | awk '{for(i=0;i<NF;i++) sum+=$i; print sum}')`
        + 通过`awk`计算求和
        + 注意此处作为`awk`的参数，for后面只有一层括号`()`
            * 一般shell中两层 `for((i=0; i<5; i++))`

## expr

* [Linux expr命令](https://www.runoob.com/linux/linux-comm-expr.html)
* `expr` 命令是一个手工命令行计数器，用于在UNIX/LINUX下求表达式变量的值，一般用于整数值，也可用于字符串
    - 计算字串长度
        + `expr length “this is a test”`
            * 结果：`14` (即`this is a test`的长度)
    - 抓取字串
        + `expr substr “this is a test” 3 5`
            * 结果：`is is` (从下标3开始，取5个字符)
            * 注意，下标是从`1`开始算(而不是`0`)
    - 抓取第一个字符数字串出现的位置
        + `expr index "sarasara"  a`
            * 结果：`2`
            * 下标从`1`开始算
    - 整数运算
        + `expr 14 % 9`
            * 结果：`5`，求余
        + `expr 10 + 10`
        + `expr 30 / 3 / 2`
        + `expr 30 \* 3`
            * 使用乘号时，必须用反斜线屏蔽其特定含义
            * 因为shell可能会误解显示星号的意义
            * `expr 30 * 3`会报语法错误：`expr: Syntax error`

## shell中各种括号的作用和区别

[shell 中各种括号的作用()、(())、[]、[[]]、{}](https://www.runoob.com/w3cnote/linux-shell-brackets-features.html)

### 单小括号() 和 双小括号(())

1. 单小括号()
    - 命令组
        + 括号中的命令将会**新开一个子shell顺序执行**，所以括号中的变量不能够被脚本余下的部分使用，多个命令分号隔开 `(cmd1;cmd2;cmd3)`
    - 命令替换
        + 等同于`cmd`，shell扫描一遍命令行，发现了**$(cmd)结构**，便将$(cmd)中的cmd执行一次，得到其标准输出，再将此输出放到原来命令。
            e.g. 获取pid+3赋值给a，
            `a=$((3+$(pidof crond))); echo $a`
    - 用于初始化数组
        + 如：`array=(a b c d)`

```sh
        arr=(1 2 3)
        echo $arr          # 并打不出数组所有成员，只会打印1
        i=0
        while(( $i <= 3 )) # i需要先定义
        do
            echo ${arr[i]} # 此处需要用{}，用()和不用任何括号都不行
            ((i++))
        done
```

2. 双小括号(())
    - 整数扩展
        + 不支持浮点型，((exp))结构扩展并计算一个算术表达式的值
    - 只要括号中的运算符、表达式符合C语言运算规则，都可用在`$((exp))`中，甚至是三目运算符。 `i=4; a=$((i>4?9:10)); echo $a`，
        作不同进位(如二进制、八进制、十六进制)运算时，输出结果全都自动转化成了十进制。如：`echo $((16#5f)) 结果为95 (16进位转十进制)`
    - 单纯用 (( )) 也可重定义变量值，比如 a=5; ((a++)) 可将 $a 重定义为6
    - 常用于算术运算比较，双括号中的变量可以不使用$符号前缀。 括号内支持多个表达式用逗号分开。 只要括号中的表达式符合C语言运算规则

```sh
    比如可以直接使用for((i=0;i<5;i++)),
    如果不使用双括号, 则为
    for i in `seq 0 4` 或者 for i in {0..4}。

    再如可以直接使用if (($i<5)), 或者 if ((i<5))
    如果不使用双括号, 则为if [ $i -lt 5 ]。

    a=3
    if ((i<5, a++)), 执行完后，a=4
```

### 单中括号[] 和 双中括号[[]]

1. 单中括号[]
    - bash 的内部命令，[和test是等同的。 新版的Bash中要求必须有]匹配。
    - Test和[]中可用的比较运算符只有==和!=，两者都是用于字符串比较的
        + 不可用于整数比较，整数比较只能使用-eq，-gt这种形式。
        + 无论是字符串比较还是整数比较都不支持大于号小于号。 如果实在想用，对于字符串比较可以使用转义形式，如果比较"ab"和"bc"：[ ab \< bc ]，结果为真，也就是返回状态为0。
        + [ ]中的逻辑与和逻辑或使用-a 和-o 表示。(**[]内部用，各[]之间可以&&**)
    - 字符范围。用作正则表达式的一部分，描述一个匹配的字符范围 e.g. `[0-3]`。 **作为test用途的中括号内不能使用正则**。
    - 在一个array 结构的上下文中，中括号用来引用数组中每个元素的编号。 e.g. `${arr[i]}`

2. 双中括号[[]]
    - [[是 bash 程序语言的关键字。并不是一个命令，[[ ]] 结构比[ ]结构更加通用。
        + 在[[和]]之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换。
    - 支持字符串的模式匹配，使用=~操作符时甚至支持shell的正则表达式。
        + 比如`[[ hello == hell? ]]`，结果为真。[[ ]] 中匹配字符串或通配符，**不需要引号**(也不能用，用引号时当做引号字符)。
    - 使用[[ ... ]]条件判断结构，而不是[ ... ]，能够防止脚本中的许多逻辑错误。
        + 比如，&&、||、<和> 操作符能够正常存在于[[ ]]条件判断结构中，但是如果出现在[ ]结构中的话，会报错。
        + 比如可以直接使用`if [[ $a != 1 && $a != 2 ]]`, 如果不使用双括号, 则为`if [ $a -ne 1] && [ $a != 2 ]`或者`if [ $a -ne 1 -a $a != 2 ]`。

```sh
if ($i<5)                       # 报错 没有那个文件或目录
if [ $i -lt 5 ]
if [ $a -ne 1 -a $a != 2 ]      # 且, -ne比较数字，!=用于字符串比较
if [ $a -ne 1] && [ $a != 2 ]   # 同[ -a ]
if [[ $a != 1 && $a != 2 ]]     # [[]] 内部可以使用&&

for i in $(seq 0 4);do echo $i;done    # $()相当于``
for i in `seq 0 4`;do echo $i;done     #
for ((i=0;i<5;i++));do echo $i;done    # (()) 可以使用符合C语言运算规则的表达式
for i in {0..4};do echo $i;done        #
```

### 大括号、花括号 {}

1. 大括号拓展。(通配(globbing))将对大括号中的文件名做扩展。 在大括号中，不允许有空白，除非这个空白被引用或转义。
    - 第一种：对大括号中的以逗号分割的文件列表进行拓展。`touch {a,b}.txt 结果为a.txt b.txt` **{a,b}之间不能有空格，有空格会变成 "{a," 和 "b}.txt"两个文件**
    - 第二种：对大括号中以点点（..）分割的顺序文件列表起拓展作用 `touch {a..d}.txt 结果为a.txt b.txt c.txt d.txt`

2. 代码块，又被称为内部组，这个结构事实上创建了一个匿名函数 。
    - 与小括号中的命令不同，大括号内的命令不会新开一个子shell运行，即脚本余下部分仍可使用括号内变量。
    - 括号内的命令间用分号隔开，最后一个也必须有分号。`{ cmd1;cmd2;cmd3;}` (注意与但括号的区别`(cmd1;cmd2;cmd3)`)
    - {}的第一个命令和左括号之间必须要有一个空格。

3. 符号$后的括号
    - ${a} 变量a的值, 在不引起歧义的情况下可以省略大括号
    - $(cmd) 命令替换，和`cmd`效果相同，结果为shell命令cmd的输出
    - $((expression)) 和`exprexpression`效果相同, 计算数学表达式exp的数值, 其中exp只要符合C语言的运算规则即可, 甚至三目运算符和逻辑表达式都可以计算。

```
几种特殊的替换结构
${var:-string} 若变量var为空，则用在命令行中用string来替换${var:-string}，否则变量var不为空时，则用变量var的值来替换${var:-string}；
${var:+string} ${var:+string}的替换规则和上面的相反，即只有当var不是空的时候才替换成string，若var为空时则不替换或者说是替换成变量 var的值，即空值。
${var:=string} 对于${var:=string}的替换规则和${var:-string}是一样的，所不同之处是${var:=string}若var为空时，用string替换${var:=string}的同时，把string赋给变量var
${var:?string} 若变量var不为空，则用变量var的值来替换${var:?string}；若变量var为空，则把string输出到标准错误中，并从脚本中退出。我们可利用此特性来检查是否设置了变量的值。

四种模式匹配替换结构(#和%中的单一符号是最小匹配，两个相同符号是最大匹配。)
${var%pattern}  这种模式时，shell在var中查找，看它是否以给的模式pattern结尾，如果是，就从命令行把var中的内容去掉右边最短的匹配模式
${var%%pattern  这种模式时，shell在var中查找，看它是否以给的模式pattern结尾，如果是，就从命令行把var中的内容去掉右边最长的匹配模式
${var#pattern}  这种模式时，shell在var中查找，看它是否以给的模式pattern开始，如果是，就从命令行把var中的内容去掉左边最短的匹配模式
${var##pattern} 这种模式时，shell在var中查找，看它是否以给的模式pattern开始，如果是，就从命令行把var中的内容去掉左边最短的匹配模式
这四种模式中都不会改变variable的值
```

[shell if [[ ]]和[ ]区别 || &&](https://www.cnblogs.com/aaron-agu/p/5700650.html)

* []和test，两者是一样的，在命令行里test expr和[ expr ]的效果相同。

    test中可用的比较运算符只有==和!=，两者都是用于字符串比较的，不可用于整数比较(>、<)，整数比较只能使用-eq, -gt这种形式。

    无论是字符串比较还是整数比较都千万不要使用大于号小于号。当然，如果你实在想用也是可以的，对于字符串比较可以使用尖括号的转义形式， 如果比较"ab"和"bc"：[ ab \< bc ]，结果为真，也就是返回状态为0.

* [[ ]]

    比test强大，支持字符串的模式匹配

    字符串比较时可以把右边的作为一个模式
    （这是右边的字符串**不加双引号的情况下**。如果右边的字符串加了双引号，则认为是一个文本字符串。）

    **注意：使用[]和[[]]的时候不要吝啬空格，每一项两边都要有空格，[[ 1 == 2 ]]的结果为“假”，但[[ 1==2 ]]的结果为“真”！**

```
    字符串比较

    =   等于,如:if [ "$a" = "$b" ]
    ==  等于,如:if [ "$a" == "$b" ],与=等价

       注意:==的功能在[[]]和[]中的行为是不同的,如下:
       1 [[ $a == z* ]]    # 如果$a以"z"开头(模式匹配)那么将为true
       2 [[ $a == "z*" ]]  # 如果$a等于z*(字符匹配),那么结果为true
       3
       4 [ $a == z* ]      # File globbing(通配、文件名替换) 和word splitting(单词分割)将会发生(不会如预期通配符一样起作用)
       5 [ "$a" == "z*" ]  # 如果$a等于z*(字符匹配),那么结果为true
       一点解释,关于File globbing是一种关于文件的速记法,比如"*.c"就是,再如~也是.
       但是file globbing并不是严格的正则表达式,虽然绝大多数情况下结构比较像.

       (将*/?/[]这些shell元字符扩展为文件名的过程就称作globbing，通配)
```

* let和(())

    - 两者也是一样的(或者说基本上是一样的，双括号比let稍弱一些)。
    - 主要进行算术运算，也比较适合进行整数比较(可直接使用>、<)
    - 可以`直接使用变量名`如var而不需要$var这样的形式。
        + e.g. `let "totalms=totalms+ms"` 或者 `((totalms=totalms+ms))` 表示将变量`ms`的值累加到`totalms`变量

tips：

1. 首先，尽管很相似，但是从概念上讲，二者是不同层次的东西。

    "[["，是关键字，许多shell(如ash bsh)并不支持这种方式。

    "["是一条命令， 与test等价，大多数shell都支持。

2. [[]]结构比Bash版本的[]更通用。

    用[[ ... ]]测试结构比用[ ... ]更能防止脚本里的许多逻辑错误。

        &&,||,<和>操作符能在一个[[]]测试里通过，但在[]结构会发生错误。

3. [ ... ]为shell命令，所以在其中的表达式应是它的命令行参数，

    所以串比较操作符">" 与"<"必须转义，否则就变成IO改向操作符了(`比较"ab"和"bc"：[ ab \< bc ]`)。
    在[[中"<"与">"不需转义；

在bash中，数字的比较最好使用 (( ))，虽说可以使用 [[ ]]，但若在其内使用运算符 >、>=、<、<=、==、!= 时，其结果经常是错误的，

不过若在 [[ ]] 中使用 [ ] 中的运算符“-eq、-ne、-le、-lt、-gt、-ge”等，还尚未发现有错。

因此诸如$ [[ " a" != “b” && 4 > 3 ]] 这类组合（见上）也不可以在bash中使用，其出错率很高。

例：[[ "a" != "b" && 10 > 2 ]] 判断结果就不正常。

诸如 [ 2 \< 10 ]、[[ 2 < 10 ]] 都是不要使用。使用算术扩展最好用 (( 99+1 == 100 )) ，而不要使用[[ 99+1 -eq 100 ]] 。


## 实现自动化过期日志备份

日志命名以 `xxx_log_2019-10-12.txt` 方式命名，每日假设会重新生成对应日期的文件

实现功能：

1. 将以日期命名的文件进行移动，备份到指定备份路径
2. 定期每日00:00执行移动备份
3. 脚本执行过程记录日志文件，日志文件大小有100MB限制处理，超过100MB清空重新记录

不完善功能：

1. 日志备份路径，文件总大小没有控制

    不管不顾可能最后会将磁盘占满。 应根据实际需求，超额进行报警或者自动删除部分规则的文件。

2. 会重新生成对应日期的日志文件只是前提假设

    若为7*24小时程序，删除日志文件可能不会重新生成，移动日志后后续日志则记不到文件中。 脚本中应进行保护判断。

## 日志清理脚本

```sh
#log_daily_backup.sh

#!/bin/bash

LOG_DIR=/home/xd/log
LOG_DIR_BACKUP=backup

AUTO_OPS_LOG_FILE="/home/xd/auto_ops/ops.log"

# 2019-10-14形式
CUR_DATE=`date "+%Y-%m-%d"`
echo -e "\n`date`, check backup log start...\n" >> $AUTO_OPS_LOG_FILE

# 添加必要保护，有该日志路径才继续，防止变量为空导致的非预期目录操作
if [ ! -d ${LOG_DIR} ]; then
    echo "`date`, $LOG_DIR not exist, exit!" >> $AUTO_OPS_LOG_FILE
    exit -1
fi

cd ${LOG_DIR}
if [ ! -d $LOG_DIR_BACKUP ]; then
    echo "`date`, $LOG_DIR_BACKUP not exist! create..." >> $AUTO_OPS_LOG_FILE
    mkdir $LOG_DIR_BACKUP
fi

# 命名格式都是xxx_log_2019-10-12.txt形式
function log_backup()
{
    for file in `ls`
    do
        if [ ${file} != "${LOG_DIR_BACKUP}" ] && [[ ${file} != *_${CUR_DATE}.txt ]]; then
            echo "`date`, matched file[$file], not like *_${CUR_DATE}.txt, mv to backup" >> $AUTO_OPS_LOG_FILE
            mv ${file} ${LOG_DIR_BACKUP}/ -f 2>&1 | tee -a $AUTO_OPS_LOG_FILE
        fi
    done
}

function check_log_size()
{
    OPS_LOG_FILE_SIZE=`ls -l ${AUTO_OPS_LOG_FILE} | awk -F' ' '{ print $5}'`
    MAX_LOG_SIZE=$((1024*1024*100))
    echo "`date`, file[${AUTO_OPS_LOG_FILE}] size:${OPS_LOG_FILE_SIZE}" >> $AUTO_OPS_LOG_FILE
    # 大于100M则重新生成
    if [ $OPS_LOG_FILE_SIZE -gt $MAX_LOG_SIZE ]; then
        echo "`date`, file[${AUTO_OPS_LOG_FILE}] size:${OPS_LOG_FILE_SIZE} over $((${MAX_LOG_SIZE}/1024/1024)) MB, recover" > $AUTO_OPS_LOG_FILE
    fi
}

log_backup
check_log_size

echo "`date`, end" >> $AUTO_OPS_LOG_FILE
```

## crontab文件

关于crontab，参考前面贴出来的 [crontab学习使用笔记](https://xiaodongq.github.io/2015/08/18/crontab%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)

```sh
#crontab.xd

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
  0  0  *  *  * sh /home/xd/auto_ops/log_daily_backup.sh
```

用 `crontab crontab.xd` 进行定期任务安装加载；

`crontab -r` 进行该用户的定期任务移除；

`crontab -l` 查看当前用户已安装的crontab定期任务


## 编码规范

参考Google 的 Shell 脚本编程规范:
[Shell Style Guide](https://google.github.io/styleguide/shell.xml)

* 必须使用bash，以`#!/bin/bash`开头
* Shell仅用于一些简单的包装脚本，超过100行的脚本最好用Python来代替
* 所有错误信息都打印到STDERR，建议如下格式

```sh
err() {
  echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: $@" >&2
}

if ! do_something; then
  err "Unable to do_something"
  exit "${E_DID_NOTHING}"
fi
```

* 每个文件必须有一个顶层的注释，其中包含内容的简要概述

## 通配符

* [`Shell脚本中$0、$?、$!、$$、$*、$#、$@`](https://www.cnblogs.com/zhangjiansheng/p/8318042.html)
* `$@`
    - 所有参数列表
    - 若用括号括起来：`"$@"`，则表示`"$1" "$2" … "$n"` 形式
* `$*`
    - 所有参数列表
    - 若用括号括起来：`"$*"`，则表示`"$1 $2 … $n"` 形式
    - 相对于`"$@"`每个参数都单独用`"`，此处用一个`"`把所有参数都括起来
* `$?`
    - 最后运行的命令的结束代码
    - Shell函数可通过return返回数值，此时可通过`$?`获取
* `$#`
    - Shell的参数个数(不含Shell命令自身，e.g. `./test.sh`，`$#`为0)
* `$0`
    - Shell本身
* `$1`
    - 是传递给该Shell脚本的第一个参数
* `$$`
    - Shell本身的PID
* `$!`
    - Shell最后运行的后台进程的PID


## `bc` 使用 bc 命令实现高级数学运算

* `bc`
    - [Linux bc 命令](https://www.runoob.com/linux/linux-comm-bc.html)
    - `echo "1.212*3" | bc`
    - `echo "scale=2;3/8" | bc`，结果为：0.37
        + 也可以引用变量 `res=$(echo "scale=2;$val+4.5"|bc)`
    - `echo "obase=10;ibase=2;$abc" | bc`，十进制转换为二进制
    - `echo "sqrt(100)" | bc`、`echo "10^10" | bc`，计算开方和平方

## 把各个子目录的文件移动到当前目录，并加各子目录的前缀

* 源于clone github上的示例项目时，各目录按章节区分"ch1 ch2 ch3"形式，每次找文件要点开目录查找

```sh
#!/bin/bash

for dir in `ls -l|grep "^d"|awk -F' ' '{print $9}'`
do
    echo $dir
    for file in `ls $dir`
    do
        echo "name: $file"
        echo "dst: "${dir}_$file""
        mv $dir/$file "${dir}_$file"
    done
done
```

## dirname

* 用 `dirname` 替换shell脚本中的`cd 绝对路径`
    - `dirname NAME` 从文件名剥离非目录的后缀
    - 打印去除了`/`后面部分的NAME;如果NAME没有包含`/`,则输出`.`(表示当前目录)
    - 添加 `cd $(dirname $0)` 即可保证进入到脚本所在的路径
        + `sh ./test.sh` 会 `cd .`
        + `sh /home/xd/workspace/test.sh` 会 `cd /home/xd/workspace`

## find

查找目录 时跳过指定目录，使用prune(英 /pruːn/   删除；减少)
  (注意顺序，-path接源路径，后面跟-prune，再跟-o，后面再跟其他过滤选项，-print不能少)：
  `find . -path './util' -prune -o -type d -print`

过滤多个目录：
  `find . \( -path './util' -o -path './tradebot \) -prune -o -type d -print`

  -o 类似于 or,  或者;
  -a 类似于 and, 且

  (1) grep指定h文件类型查找hello字符串：
find . -type f -name '*.h' | xargs grep "hello"

查看端口 lsof -i:5000

* 排除目录下所有以md结尾的文件：
`find . -type f ! -name "*.md"`

* 排除多个：
`find . -type f ! -name "*.md" ! -name "*.o"`

* 正则表达式：
`find . -regex '.*\.md\|.*\.h\|.*\.cpp'`

## 统计文本行数

语法：wc [选项] 文件…

说明：该命令统计给定文件中的字节数、字数、行数。如果没有给出文件名，则从标准输入读取。

    该命令各选项含义如下：

    　　- c 统计字节数
    　　- l 统计行数
    　　- w 统计字数

* `wc -lcw Makefile`

* 统计src目录下所有cpp文件代码行数(子目录也会统计)

`find src/ -name "*.cpp" |xargs cat|wc -l`

* 统计当前目录及子目录下文件行数
`find . -type f |xargs cat|wc -l`

* 统计当前目录及子目录下.h和.cpp文件行数
`find . -type f -name "*.h" -o -name "*.cpp" |xargs cat|wc -l`
`find . -type f -name "*.h" |xargs cat|wc -l`

* 统计src目录下所有cpp文件代码行数(过滤空行)

`find src/ -name "*.cpp" |xargs cat | grep -v ^$ | wc -l`

## unzip

unzip zip文件

## 目录排序

du -s -d1|sort -n      (h会影响排序，仅按数字来排的)

## grep 查找后去重

`grep "#include <boost" -rn *|awk -F' ' '{print $2}'|sort|uniq`  (注意要先sort，要不仅会去重相邻的)

* `grep -o targetStr_1\|targetStr_2\|targetStr_3…… filename | wc -l` 统计关键字个数
    - `\|` 过滤多个
    - `-o` 只显示字符串，不显示整行
        + `grep -o` 一条数据里面有多个相同，会统计实际的次数
        + `grep` 一条数据里面有多个相同，只会统计一次次数

* `grep`
    - `grep -E` 正则匹配多个关键字
        + `grep -E "abc|123"` 会匹配包含 `abc` 或 `123`的字符串
    - `grep -w pattern files` 只匹配整个单词，而不是字符串的一部分

## sort uniq 文件去重

对文件排序：  
sort test.csv (可>输出到新文件)

* `sort`
    - 实例
        + 文件words存放英文单词，格式为每行一个英文单词（单词可以重复），统计这个文件中出现次数最多的前10个单词
        + `cat words.txt | sort | uniq -c | sort -k1,1nr | head -10`
            * `uniq -c` 行首显示出现次数
            * `sort -k1,1nr`
                - `-k1,1` 只按第一个域
                    + 另外：`-k 1.2` 从第一个域的第二个字符，到本域最后一个字符为止的字符串排序
                    + 如`baidu 100`，`-k1.1`从`b`排序，`-k1.2`从`a`位置开始排序
                - `-n` 按照字符串的数值顺序比较
                - `-r` 颠倒比较的结果
            * 这几个选项使用，可以再参考：[sort命令](https://man.linuxde.net/sort)

## echo 不换行

* echo -e 允许对下面列出的加反斜线转义的字符进行解释

```sh
  \n    换行符
  \c    禁止尾随的换行符
  \t    水平制表符
  等等

  echo -e "hello\n"  在原来基础上多加一个换行
  echo -e "hello\c"  不换行
```

* `echo -n "hello"` 也可指定不换行(-n 不输出行尾的换行符)

## 正则表达式

非：  volume:[^0] 匹配"volume:"后接非0的行


## shell文件包含

* 脚本内容中包含
    - `. xdtest.sh` 或者 `source xdtest.sh`

## shell默认值

* 形式一：`${a-defaultvalue}`
    - a如果没有定义，则表达式返回默认值，否则返回a的值
    - e.g.
        + `ret1=${a-"/usr/local"}`，`echo "$ret1"`则输出`/usr/local`(a未定义)
        + 若有`a=""`，`echo "$ret1"`则输出空
* 形式二：`${a:-defaultvalue}` (多了一个`:`，相对上面多了一种空串的情况)
    - a没有定义或者a为空字符串，则表达式返回默认值，否则返回a的值
    - e.g.
        + `ret1=${a-"/usr/local"}`，`echo "$ret1"`则输出`/usr/local`(a未定义)
        + 若有`a=""`，`echo "$ret1"`则输出`/usr/local`(a为空)
        + 若有`a="123"`，`echo "$ret1"`则输出`123`(a非空)

## 字符串判断

* `[ -z STRING ]` 如果STRING的长度为0则返回为真
* `[ -n STRING ]` 如果STRING的长度非0则返回为真
* [Shell脚本IF条件判断和判断条件总结](https://www.jb51.net/article/56553.htm)

## tr

## google style guide

### 背景

* Shell应该仅仅被用于小功能或者简单的包装脚本。

    以下是一些准则：

    * 如果你主要是在调用其他的工具并且做一些相对很小数据量的操作，那么使用shell来完成任务是一种可接受的选择。
    * 如果你在乎性能，那么请选择其他工具，而不是使用shell。
    * 如果你发现你需要使用数据而不是变量赋值（如 ``${PHPESTATUS}`` ），那么你应该使用Python脚本。
    * 如果你将要编写的脚本会超过100行，那么你可能应该使用Python来编写，而不是Shell。请记住，当脚本行数增加，尽早使用另外一种语言重写你的脚本，以避免之后花更多的时间来重写。

### 注释

* 每个文件必须包含一个顶层注释，对其内容进行简要概述。版权声明和作者信息是可选的。

```sh
#!/bin/bash
#
# Perform hot backups of Oracle databases.
```

* 任何不是既明显又短的函数都必须被注释。任何库函数无论其长短和复杂性都必须被注释。

* 使用TODO注释临时的、短期解决方案的、或者足够好但不够完美的代码。

    TODOs应该包含全部大写的字符串TODO，接着是括号中你的用户名。冒号是可选的。最好在TODO条目之后加上 bug或者ticket 的序号。

```sh
# TODO(mrmonkey): Handle the unlikely edge cases (bug ####)
```

### 格式

* 请将 ``; do`` , ``; then`` 和 ``while`` , ``for`` , ``if`` 放在同一行。

```sh
    if [[ -n "${myvar}" ]]; then
        xxx
    else
        xxx
    fi
```

* 推荐用 ``${var}`` 而不是 ``$var``

* 传递参数时，请使用 ``$@`` 除非你有特殊原因需要使用 ``$*``

### 特性及错误

* 使用 ``$(command)`` 而不是反引号

    嵌套的反引号要求用反斜杠转义内部的反引号

* 推荐使用 ``[[ ... ]]`` ，而不是 ``[`` , ``test`` , 和 ``/usr/bin/[``

    因为在 ``[[`` 和 ``]]`` 之间不会有路径名称扩展或单词分割发生，所以使用 ``[[ ... ]]`` 能够减少错误

* 测试字符串

    * 为了避免对你测试的目的产生困惑，请明确使用`-z`或者`-n`
        * 建议 `if [[ -z "${my_var}" ]];`
        * 不建议 `if [[ "${my_var}" = "" ]];`

    * 建议 `if [[ "${my_var}" = "some_string" ]];`
        * 不建议 `if [[ "${my_var}X" = "some_stringX" ]];`

* 应该避免使用eval

### 命名约定

* 使用小写字母，并用下划线分隔单词

    函数名：my_func

    源文件名：make_template.sh

* 函数名之后必须有圆括号。关键词 ``function`` 是可选的，但必须在一个项目中保持一致。
* 大括号必须和函数名位于同一行（就像在Google的其他语言一样），并且函数名和圆括号之间没有空格。

```sh
    my_func() {
        ...
    }
```

* 使用 ``local`` 声明特定功能的变量。声明和赋值应该在不同行。

    当赋值的值由命令替换提供时，声明和赋值必须分开。因为内建的 ``local`` 不会从命令替换中传递退出码。

```sh
    # 传入值无需返回值，所以不分开没关系
    local name="$1"

    # 通过命令获取的结果赋值，分开便于判断命令返回值
    local my_var
    my_var="$(my_func)" || return
```

* 对于包含至少一个其他函数的足够长的脚本，需要称为 ``main`` 的函数。

    显然，对于仅仅是线性流的短脚本， ``main`` 是矫枉过正，因此是不需要的。

### 调用命令

* 可以在调用shell内建命令和调用另外的程序之间选择，请选择内建命令。

```sh
    # Prefer this:
    addition=$((${X} + ${Y}))
    substitution="${string/#foo/bar}"

    # Instead of this:
    addition="$(expr ${X} + ${Y})"
    substitution="$(echo "${string}" | sed -e 's/^foo/bar/')"
```
