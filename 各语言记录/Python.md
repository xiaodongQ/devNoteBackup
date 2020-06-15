## python3

[Python 3 教程](https://www.runoob.com/python3/python3-tutorial.html)

Python的3.0版本，常被称为Python 3000，或简称Py3k。
相对于Python的早期版本，这是一个较大的升级。为了不带入过多的累赘，Python 3.0在设计的时候没有考虑向下兼容。

[Python2.x与3​​.x版本区别](https://www.runoob.com/python/python-2x-3x.html)
    许多针对早期Python版本设计的程式都无法在Python 3.0上正常执行。
    Python 2.6作为一个过渡版本，基本使用了Python 2.x的语法和库，同时考虑了向Python 3.0的迁移，允许使用部分Python 3.0的语法与函数。
    新的Python程式建议使用Python 3.0版本的语法。
    Python3 print语句没有了，取而代之的是print()函数

* 升级Python3.x
    - [CentOS7升级Python3](https://blog.csdn.net/weixin_41798704/article/details/88238222)
    - https://www.python.org/ftp/python 下载3.x对应版本
    - `./configure` `make` `make install`
    - 修改链接
        + `whereis python`
        + 把`/usr/bin/python`链接文件备份，新建一个链接文件到python3
            * `ln -s xxx/python3 /usr/bin/python`
    - 将python升级为3.x后，yum不能正常使用，需要编辑 yum 的配置文件
        + 为了使yum命令正常使用(yum是一个python脚本)，需要将其配置的python依然指向2.x版本，把`# /usr/bin/python`修改为`# /usr/bin/python2.7`
        + `/usr/bin/yum`
        + `/usr/libexec/urlgrabber-ext-down`

### 安装pip

* [install pip](https://pip.pypa.io/en/stable/installing/)
    - `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py`
    - 然后 `python get-pip.py`
    - 如果是Windows下安装，安装在 $python_path/Scripts 下面(安装完会提示)，在系统环境变量path中添加一下路径

### linux终端
tab自动补全:
新建 .pythonstartup.py

```python
# Add auto-completion and a stored history file of commands to your Python
# interactive interpreter. Requires Python 2.0+, readline. Autocomplete is
# bound to the Esc key by default (you can change it - see readline docs).
#
# Store the file in ~/.pystartup, and set an environment variable to point
# to it:  "export PYTHONSTARTUP=~/.pystartup" in bash.
import atexit
import os
import readline
import rlcompleter
readline.parse_and_bind('tab: complete')
historyPath = os.path.expanduser("~/.pyhistory")
def save_history(historyPath=historyPath):
    import readline
    readline.write_history_file(historyPath)
if os.path.exists(historyPath):
    readline.read_history_file(historyPath)
atexit.register(save_history)
del os, atexit, readline, rlcompleter, save_history, historyPath
```

bashrc中，export PYTHONSTARTUP=~/.pythonstartup.py

## Python3 基础语法

默认情况下，Python 3 源码文件以 UTF-8 编码
也可以为源码文件指定不同的编码：    `# -*- coding: cp-1252 -*-`  //Windows-1252

### 注释
Python中单行注释以 # 开头
多行注释可以用多个 # 号，还有 ''' 和 """：

```python
    # 第一个注释
    # 第二个注释

    '''
    第三注释
    第四注释
    '''

    """
    第五注释
    第六注释
    """
```

### 行与缩进

python最具特色的就是使用缩进来表示代码块，不需要使用大括号 {} 。
缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进空格数。
    缩进不一致，会导致运行错误

```python
if True:
    print ("True")
else:
    print ("False")
```


Python 通常是一行写完一条语句，但如果语句很长，我们可以使用反斜杠(\)来实现多行语句
    在 [], {}, 或 () 中的多行语句，不需要使用反斜杠(\)

```python
total = item_one + \
        item_two + \
        item_three

total = ['item_one', 'item_two', 'item_three',
        'item_four', 'item_five']
```

### 数字类型

python中数字有四种类型：整数、布尔型、浮点数和复数。
    int (整数), 如 1, 只有一种整数类型 int，表示为长整型，没有 python2 中的 Long。
    bool (布尔), 如 True。
    float (浮点数), 如 1.23、3E-2
    complex (复数), 如 1 + 2j、 1.1 + 2.2j

### 字符串

python中单引号和双引号使用完全相同。
使用三引号('''或""")可以指定一个多行字符串。

```python
    word = '字符串'
    sentence = "这是一个句子。"
    paragraph = """这是一个段落，
    可以由多行组成"""
```

转义符 '\'
反斜杠可以用来转义，使用r可以让反斜杠不发生转义 (这里的 r 指 raw，即 raw string)
    `如 r"this is a line with \n" 则\n会显示，并不是换行。`

```python
    print('hello\nrunoob')      # 使用反斜杠(\)+n转义特殊字符
    print(r'hello\nrunoob')     # 在字符串前面添加一个 r，表示原始字符串，不会发生转义
```

按字面意义级联字符串，如"this " "is " "string"会被自动转换为this is string。   *??*
字符串可以用 + 运算符连接在一起，用 * 运算符重复
    `print(str * 2)             # 输出字符串两次`
    `print(str + '你好')        # 连接字符串`
Python 中的字符串有两种索引方式，从左往右以 0 开始，从右往左以 -1 开始
    `print(str[0:-1])           # 输出第一个到倒数第二个的所有字符`
    `print(str[0])              # 输出字符串第一个字符`
    `print(str[2:5])            # 输出从第三个开始到第五个的字符`
    `print(str[2:])             # 输出从第三个开始的后的所有字符`

Python 没有单独的字符类型，一个字符就是长度为 1 的字符串 **没有char**

### 空行

空行与代码缩进不同，空行并不是Python语法的一部分。

书写时不插入空行，Python解释器运行也不会出错。但是空行的作用在于分隔两段不同功能或含义的代码，便于日后代码的维护或重构。

记住：空行也是程序代码的一部分。


### 语句分割

Python可以在同一行中使用多条语句，语句之间使用分号(;)分割
    `import sys; x = 'runoob'; sys.stdout.write(x + '\n')`

**每条语句不需要用;结尾**

缩进相同的一组语句构成一个代码块，我们称之*代码组*。

print 默认输出是换行的，如果要实现不换行需要在变量末尾加上 end=""

```python
    """ 换行输出
    """
    print( x )
    # 不换行输出
    print( x, end=" " )
```

### import 与 from...import

在 python 用 import 或者 from...import 来导入相应的模块。

```python
将整个模块(somemodule)导入，格式为： import somemodule
从某个模块中导入某个函数,格式为： from somemodule import somefunction
从某个模块中导入多个函数,格式为： from somemodule import firstfunc, secondfunc, thirdfunc
将某个模块中的全部函数导入，格式为： from somemodule import *
```

```python
导入 sys 模块:
import sys

导入 sys 模块的 argv,path 成员
from sys import argv,path  #  导入特定的成员
print('path:',path) # 因为已经导入path成员，所以此处引用时不需要加sys.path
```

## Python3 基本数据类型

Python 中的变量不需要声明。每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。
    `counter = 100          # 整型变量`
    `miles   = 1000.0       # 浮点型变量`
    `name    = "runoob"     # 字符串`

在 Python 中，变量就是变量，它没有类型，我们所说的"类型"是变量所指的内存中对象的类型。

可以为多个对象指定多个变量
    `a, b, c = 1, 2, "runoob"`

### 标准数据类型

不可变数据（3 个）：Number（数字）、String（字符串）、Tuple（元组）；
可变数据（3 个）：List（列表）、Dictionary（字典）、Set（集合）。

* Number（数字）

    int、float、bool、complex（复数）
    > 注意：在 Python2 中是没有布尔型的，它用数字 0 表示 False，用 1 表示 True。到 Python3 中，把 True 和 False 定义成关键字了，但它们的值还是 1 和 0，它们可以和数字相加。

    两种除法运算符及乘方运算符：

```python
    >>> 2 / 4  # 除法，得到一个浮点数
    0.5
    >>> 2 // 4 # 除法，得到一个整数
    0
    >>> 2 ** 5 # 乘方
    32
```

* String（字符串）
    与 C 字符串不同的是，Python 字符串不能被改变。向一个索引位置赋值，比如word[0] = 'm' **会导致错误。**

```python
    >>>word = 'Python'
    word[0] = 'm'      # 会导致错误
```

    Python中的字符串有两种索引方式，从左往右以0开始，从右往左以-1开始。

```python
    >>>word = 'Python'
    >>> print(word[0], word[5])
    P n
    >>> print(word[-1], word[-6]) ？？？
    n P
```

* List（列表）
    List（列表） 是 Python 中使用最频繁的数据类型。
    索引值以 0 为开始值，-1 为从末尾的开始位置。
    写在方括号 [] 之间
    `tinylist = [123, 'runoob']`

* Tuple（元组）
    元组（tuple）与列表类似，不同之处在于元组的元素不能修改。
    元组写在小括号 () 里
    `tinytuple = (123, 'runoob')`

* Set（集合）
    集合（set）是由一个或数个形态各异的大小整体组成的，构成集合的事物或对象称作*元素*或是*成员*。
    基本功能是进行成员关系测试和删除重复元素。
    可以使用大括号 { } 或者 set() 函数创建集合
        注意：创建一个空集合必须用 set() 而不是 { }，因为 { } 是用来创建一个空字典。
        `student = {'Tom', 'Jim', 'Mary', 'Tom', 'Jack', 'Rose'}`
        `print(student)   # 输出集合，重复的元素被自动去掉`

    set可以进行集合运算
        a = set('abracadabra')
        b = set('alacazam')
        print(a)         # 集合会自动去掉重复元素, {'b', 'a', 'c', 'r', 'd'}
        print(a - b)     # a 和 b 的差集 {'b', 'd', 'r'}
        print(a | b)     # a 和 b 的并集 {'l', 'r', 'a', 'c', 'z', 'm', 'b', 'd'}
        print(a & b)     # a 和 b 的交集 {'a', 'c'}
        print(a ^ b)     # a 和 b 中不同时存在的元素，类似异或 {'l', 'r', 'z', 'm', 'b', 'd'}
* Dictionary（字典）
    字典（dictionary）是Python中另一个非常有用的内置数据类型。
    列表是有序的对象集合，字典是无序的对象集合。
    两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。
    字典是一种映射类型，字典用 { } 标识，它是一个无序的 键(key) : 值(value) 的集合。
        键(key)必须使用不可变类型
        在同一个字典中，键(key)必须是唯一的。

```python
    dict = {}     #空字典，注意此处{}不是集合
    dict['one'] = "1 - 菜鸟教程"

    tinydict = {'name': 'runoob','code':1, 'site': 'www.runoob.com'} #字典元素为key:value，集合为单个类型成员
```
    构造函数 dict() 可以直接从键值对序列中构建字典

```python
    >>>dict([('Runoob', 1), ('Google', 2), ('Taobao', 3)])
    {'Taobao': 3, 'Runoob': 1, 'Google': 2}

    >>> dict(Runoob=1, Google=2, Taobao=3)
    {'Runoob': 1, 'Google': 2, 'Taobao': 3}
```

    另外，字典类型也有一些内置的函数，例如clear()、keys()、values()等

Python数据类型转换 [Python数据类型转换](https://www.runoob.com/python3/python3-data-type.html)
    int()
    float
    dict(d)
    list(s)
    tuple(s)
    repr(x)
    oct(x)
    hex(x)

## 控制语句

### 条件控制

Python 中用 elif 代替了 else if，所以if语句的关键字为：if – elif – else。
    每个条件后面要使用冒号 :
    使用缩进来划分语句块
    在Python中没有switch – case语句

```python
if condition_1:
    statement_block_1
elif condition_2:
    statement_block_2
else:
    statement_block_3
```

e.g.

```python
#!/usr/bin/python3

# 该实例演示了数字猜谜游戏
number = 7
guess = -1
print("数字猜谜游戏!")
while guess != number:
    guess = int(input("请输入你猜的数字："))

    if guess == number:
        print("恭喜，你猜对了！")
    elif guess < number:
        print("猜的数字小了...")
    elif guess > number:
        print("猜的数字大了...")
```

### 循环语句

Python中的循环语句有 for 和 while
    在 Python 中没有 do..while 循环。

```python
    #!/usr/bin/env python3

    n = 100
    sum = 0
    counter = 1
    while counter <= n:
        sum = sum + counter
        counter += 1

    print("1 到 %d 之和为: %d" % (n,sum))
```

* while 循环使用 else 语句

```python
#!/usr/bin/python3

count = 0
while count < 5:
   print (count, " 小于 5")
   count = count + 1
else:
   print (count, " 大于或等于 5")
```

* for语句
    break 语句用于跳出当前循环体

```python
for <variable> in <sequence>:
    <statements>
else:
    <statements>
```

e.g.

```python
#!/usr/bin/python3

sites = ["Baidu", "Google","Runoob","Taobao"]
for site in sites:
    if site == "Runoob":
        print("菜鸟教程!")
        break
    print("循环数据 " + site)
else:
    print("没有循环数据!")
print("完成循环!")
```

* 内置range()函数 生成数列
    `for i in range(5):`         0-4
    `for i in range(5,9) :`      5-8
    `for i in range(0, 10, 3) :` 指定步长3，[0,3,6,9]
    `for i in range(-10, -100, -30) :` [-10,-40,-70]

#### break和continue语句及循环中的else子句
如果你从 for 或 while 循环中终止，任何对应的循环 else 块将不执行
循环语句可以有 else 子句，它在穷尽列表(以for循环)或条件变为 false (以while循环)导致循环终止时被执行,但循环被break终止时不执行。

```python
#!/usr/bin/python3

for n in range(2, 10):
    for x in range(2, n):
        if n % x == 0:
            print(n, '等于', x, '*', n//x)
            break
    else:
        # 循环中没有找到元素
        print(n, ' 是质数')
```

### pass 语句
Python pass是空语句，是为了保持程序结构的完整性。
    pass 不做任何事情，一般用做占位语句

```python
    if letter == 'o':
        pass
        print ('执行 pass 块')
```

## Python3 迭代器与生成器

* 迭代是Python最强大的功能之一，是访问集合元素的一种方式。
* 迭代器有两个基本的方法：`iter()` 和 `next()`。

```python
>>> list=[1,2,3,4]
>>> it = iter(list)    # 创建迭代器对象
>>> print (next(it))   # 输出迭代器的下一个元素
1
>>> print (next(it))
2
```

* 遍历：
* 常规for语句进行遍历: it创建后，for in it循环中不用再对it操作

```python
list=[1,2,3,4]
it = iter(list)    # 创建迭代器对象
for x in it:
    print (x, end=" ")
```

* next函数遍历：

```python
list=[1,2,3,4]
it = iter(list)    # 创建迭代器对象

while True:
    try:
        print (next(it))
    except StopIteration:
        sys.exit()
```

把一个类作为一个迭代器使用需要在类中实现两个方法 `__iter__()` 与 `__next__()` 。

## 函数

* 规则
    - 函数代码块以 `def` 关键词开头，后接函数标识符名称和圆括号 `()`
    - 任何传入参数和自变量必须放在圆括号中间，圆括号之间可以用于定义参数
    - 函数的第一行语句可以选择性地使用文档字符串，用于存放函数说明
    - 函数内容以冒号`:`起始，并且缩进
    - `return [表达式]` 结束函数，选择性地返回一个值给调用方。不带表达式的`return`相当于返回 `None`

```python
#!/usr/bin/python3

# 计算面积函数
def area(width, height):
    return width * height

def print_welcome(name):
    print("Welcome", name)

print_welcome("Runoob")
w = 4
h = 5
print("width =", w, " height =", h, " area =", area(w, h))
```

* 可更改(mutable)与不可更改(immutable)对象
    - 在 python 中，`strings`, `tuples`, 和 `numbers` 是不可更改的对象，而 `list`,`dict` 等则是可以修改的对象
    - 不可变类型示例：
        + 变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变a的值，相当于新生成了a
    - 可变类型示例：
        + 变量赋值 la=[1,2,3,4] 后再赋值 la[2]=5 则是将 list la 的第三个元素值更改，本身la没有动，只是其内部的一部分值被修改了


## 安装包问题

* Pip 安装pymongo失败
    - "WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ReadTimeoutError("HTTPSConnectionPool(host='pypi.org', port=443): Read timed out. (read timeout=15)")': /simple/pymongo/"
    - `pip install pymongo -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com`
    - [Pip 安装pymongo失败](https://blog.csdn.net/lwc5411117/article/details/79659794)


## python 编译成C++

* [nuitka](http://nuitka.net/doc/user-manual.html)
  - Nuitka是一个用Python写的Python编译器，可以无缝替换或者扩展 CPython包含的所有解释和编译指令
  - Nuitka把Python模块翻译成C语言级别的程序，使用 libpython库和它的静态C文件，和CPython以同样的方式执行
  - 所有的优化都是为了避免不必要的开销
* 使用
  - Windows下安装
      + 安装C编译器，MinGW
          * 下载链接：http://mingw-w64.org/doku.php/download/mingw-builds
          * MinGW的bin目录添加到Path环境变量
          * `gcc --version` 查看是否安装正常
      + 安装Python
      + 安装Nuitka
          * `pip install nuitka`
          * 检查是否安装正常，`nuitka --version`
  - 编写测试代码并用nuitka编译
      + 测试运行：`python hello.py`
      + 用nuitka编译：`python -m nuitka --standalone --mingw64 hello.py`
          * `--show-progress --show-scons` 可以查看全部输出
      + 执行`hello.dist`目录下的 `hello.exe`程序
  - 若要分发副本，则拷贝`hello.dist`目录

```python
# hello.py

def talk(message):
    return "Talk " + message

def main():
    print(talk("Hello World"))

if __name__ == "__main__":
    main()
```

* 使用用例
    - 1. 编译时嵌入所有模块
        + 若想递归地编译整个程序，而不是单个包含main的程序，可：
        + `python -m nuitka --follow-imports program.py`
            * 若有模块插件目录通过import找不到，则可以手动指定：`--include-plugin-directory=plugin_dir`，也可以设置`PYTHONPATH`(推荐)
        + 执行完之后，在Windows上生成的二进制文件名是`program.exe`，其他系统上是`program.bin`
            * 注意，二进制文件还是依赖于CPython并且会使用按照好的C扩展，若要支持能拷贝到任何机器，则使用`--standalone`并且复制生成的`program.dist`目录(其中包含对应平台的二进制执行文件)
    - 2. 编译单独的扩展模块
        + `python -m nuitka --module some_module.py`
        + 生成的`some_module.so`使用时可以替换`some_module.py`
    - 3. 编译整个package并嵌入所有模块
        + `python -m nuitka --module some_package --include-package=some_package`

* [Python打包exe的王炸-Nuitka](https://zhuanlan.zhihu.com/p/133303836)
    - 参考链接中的完整命令：`nuitka --mingw64 --windows-disable-console --standalone --show-progress --show-memory --plugin-enable=qt-plugins --plugin-enable=pylint-warnings --recurse-all --recurse-not-to=numpy,jinja2 --output-dir=out index.py`
    - 执行：`nuitka --mingw64 --standalone --show-progress --show-memory --plugin-enable=pylint-warnings --recurse-all --recurse-not-to=numpy,pandas --output-dir=out005_2 005Strategy.py`
        + 编译耗时八九分钟； dist目录：143MB
        + 下面几个也进行了编译(较慢)：pymongo,google,grpc
        + 需要手动添加到dist目录的模块：numpy,pandas,(pytz,datautil 被依赖也需要复制),xlrd,(openpyxl, 又依赖：jdcal.py,et_xmlfile)
            * 这些目录和文件在python的pip install安装包放的路径，一般在：python安装路径\Lib\site-packages
        + PyQT,Numpy,Scipy,Pandas,Opencv,OpenpyXL等pyd的模块不编译，交给python3x.dll来调用，避免模块依赖失败
    - 选项含义
        + --nofollow-imports 所有的import全部不使用，交给python3x.dll执行
        + --follow-import-to=need 需要编译成C/C++的import
        + --mingw64 #默认为已经安装的vs2017去编译，否则就按指定的比如mingw
        + --standalone 独立文件，这是必须的
        + --windows-disable-console 没有CMD控制窗口
        + --recurse-all 所有的资源文件 这个也选上
        + -recurse-not-to=numpy,jinja2 不编译的模块，防止速度会更慢
        + --output-dir=out 生成exe到out文件夹下面去
        + --show-progress 显示编译的进度，很直观
        + --show-memory 显示内存的占用
        + --plugin-enable=pylint-warnings 报警信息
        + --plugin-enable=qt-plugins 需要加载的PyQT插件
        + 可以通过`nuitka --help`查看各命令说明

Numpy等类似c程式和pyd的调用还是忽略编译好，编译后反而更慢

* [python py、pyc、pyo、pyd文件区别](https://blog.csdn.net/willhuo/article/details/49886663)
    - py是源文件，pyc是源文件编译后的文件，pyo是源文件优化编译后的文件，pyd是其他语言写的python库

### 使用pyinstaller

* [Python 程序打包成 exe 可执行文件](https://www.cnblogs.com/valorchang/p/11357358.html)
    - 使用pyinstaller
    - 发布方式：
        + .py 文件：对于开源项目或者源码没那么重要的，直接提供源码，需要使用者自行安装 Python 并且安装依赖的各种库。（Python 官方的各种安装包就是这样做的）
        + .pyc 文件：有些公司或个人因为机密或者各种原因，不愿意源码被运行者看到，可以使用 pyc 文件发布，pyc 文件是 Python 解释器可以识别的二进制码，故发布后也是跨平台的，需要使用者安装相应版本的 Python 和依赖库
        + 可执行文件：对于非码农用户或者一些小白用户，你让他装个 Python 同时还要折腾一堆依赖库，那简直是个灾难。对于此类用户，最简单的方式就是提供一个可执行文件，只需要把用法告诉他即可。比较麻烦的是需要针对不同平台需要打包不同的可执行文件（Windows, Linux, Mac,…）。
    - pyinstaller安装使用
        + `pip install pyinstaller`，`pyinstaller --version`
        + pyinstaller 的语法：`pyinstaller [options] script [script…] | specfile`
        + 最简单的用法：在`myscript.py`同目录下执行命令，e.g. `pyinstaller mycript.py`
            * 会看到新增加了两个目录 `build` 和 `dist`
            * `dist` 下面的文件就是可以发布的可执行文件
                - `dist`目录下面有一堆文件，各种动态库文件和myscript可执行文件
                - 需要打包 dist 下面的所有东西才能发布，万一丢掉一个动态库就无法运行了
            *  pyInstaller 支持单文件模式：`pyinstaller -F mycript.py`
                - `dist`下面只有一个可执行文件，这个单文件就可以发布了
                - 一个简单的helloworld程序，hello.exe前后大小对比：1.38MB，加`-F`:6.16MB
        + 在执行 pyInstaller 命令的时候，会在和脚本相同目录下，生成一个.spec 文件，该文件会告诉 pyinstaller 如何处理你的所有脚本，同时包含了命令选项。
            * 一般我们不用去理会这个文件，若需要打包数据文件，或者给打包的二进制增加一些 Python 的运行时选项时…一些高级打包选项时，需要手动编辑.spec 文件。
    - PyInstaller 原理简介
        + PyInstaller 其实就是把 python 解析器和你自己的脚本打包成一个可执行的文件，和编译成真正的机器码完全是两回事，所以千万不要指望成打包成一个可执行文件会提高运行效率，相反可能会降低运行效率，好处就是在运行者的机器上不用安装 python 和你的脚本依赖的库
        + 需要注意的是，PyInstaller 打包的执行文件，只能在和打包机器系统同样的环境下。也就是说，不具备可移植性，若需要在不同系统上运行，就必须针对该平台进行打包
