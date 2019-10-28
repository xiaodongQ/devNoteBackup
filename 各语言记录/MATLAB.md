# MATLAB

参考：

[MATLAB教程](https://www.w3cschool.cn/matlab/) (该文档很多翻译问题，了解基础后日常检索和函数查询使用以下链接)

MathWorks文档：
[MATLAB](https://ww2.mathworks.cn/help/matlab/index.html?s_tid=CRUX_lftnav)

## MATLAB介绍

* MATLAB（矩阵实验室）是由美国MathWorks公司开发的第四代高层次的编程语言和交互式环境数值计算，可视化和编程；
* MATLAB允许矩阵操作、绘制函数和数据、算法实现、创建用户界面；
* MATLAB能和在其他语言，包括C、C++、Java和Fortran语言编写的程序接口；
* MATLAB可以分析数据、开发算法、建立模型和应用程序；
* MATLAB拥有众多的内置命令和数学函数，可以帮助您在数学计算，绘图和执行数值计算方法。

## MATLAB语法

* MATLAB 是一种解释型的环境。也就是说，只要你给MATLAB一个命令，它就会马上开始执行。
    - 在命令行窗口输入表达式，回车 e.g. `5+5`, `3^2`, `sin(pi/2)`, `7/0`(Inf)

* MATLAB命名变量
    - 字母后由任意数量的字母，数字或下划线
    - 注意MATLAB中是区分大小写的。

* MATLAB使用`save`命令保存工作区中的所有变量，然后在当前目录中作为一个扩展名为`.mat`的文件。
    - 该文件可以随时重新加载，然后使用`load`命令。

### MATLAB常用的运算符和特殊字符

#### 运算符

标识一些相较其他语言比较不同的。

句点字符 (.) 将数组运算与矩阵运算区别开来，由于矩阵运算和数组运算在加法和减法的运算上相同，
因此没有必要使用字符组合 .+ 和 .-。

```cpp
运算符 目的
+   加；加法运算符
-   减；减法运算符
*   标量和矩阵乘法运算符            // 矩阵A*B 矩阵 A 和 B 的线性代数乘积
.*  数组乘法运算符                 // 数组按元素乘法 A.*B 表示 A 和 B 的逐元素乘积
^   标量和矩阵求幂运算符            //
.^  数组求幂运算符                 //
\   矩阵左除                       //
/   矩阵右除
.\  阵列左除                       //
./  阵列右除                       //
:   向量生成；子阵提取              //
( )  下标运算；参数定义
[ ] 矩阵生成                       //
.   点乘运算，常与其他运算符联合使用
…   续行标志；行连续运算符          //
,   分行符（该行结果不显示）        //
;   语句结束；分行符（该行结果显示）
%   注释标志                       // 注释
_   引用符号和转置运算符            //
._  非共轭转置运算符                //
=   赋值运算符
```

* MATLAB中分号（;）表示语句结束；但是，如果想抑制和隐藏 MATLAB 输出表达，表达后添加一个分号。
    - 如输入`x = 2+3`回车，命令行窗口会立即打印`x=5`结果，添加分号则不会打印

* 关系运算符: ~=不等于 ==等于 >=大于或等于 >大于 <=小于或等于 <小于

* 逻辑运算符:
    - and、or 和 not（分别用符号 &、| 和 ~ 表示），对应位置的元素依次操作，置 1 或 0
    - 短路 &&: 逻辑 AND 运算; ||: 逻辑 OR 运算
    - 符号 & 和 && 在 MATLAB® 中执行不同的运算。在此介绍的按元素 AND 运算符是 &。短路 AND 运算符是 &&

* 集合操作符: MATLAB提供各种功能集合运算，如集，交集和测试组成员等。
    - `intersect(A,B)` 两个数组的交集；返回A和B所共有的值。
    - `ismember(A,B)` 是否包含
    - `union` 两个数组的并集
    - `setxor` 异或
    - `issorted(A)` 如果A的元素按排序顺序返回逻辑1（true）


#### 特殊变量和常量

Name    Meaning
ans 默认的变量名，以应答最近依次操作运算结果
eps 浮点数的相对误差
i,j 虚数单位，定义为 i2 = j2 = -1
Inf 代表无穷大
NaN 代表不定值（不是数字）
pi  圆周率


### 基本类型

* 每个MATLAB变量可以是数组或者矩阵。
    - `x = 3`, 创建了一个1-1的矩阵
* MATLAB注意事项：
    - 在使用变量之前，必须进行赋值。
    - 当系统接收到一个变量之后，这个变量可以被引用。 `x = 7 * 8; y = x * 7.89`
    - 若结果不分配给任何变量，系统会分配给一个变量命名ans，变量 ans 可以被继续使用: `2; 500/ans`
    - 多个任务可以在同一行 `a = 2; b = 7; c = a * b`

* 向量
    - 向量是一维数组中的数字。
    - MATLAB允许创建两种类型的矢量：
        + 行向量 `r = [7 8 9 10 11]`, `r(3)`为9, `r(2:3)`为8 9
        + 列向量 `c = [7;  8;  9;  10; 11]`, `c(1)`为7
    - 加/减法 当进行两个向量的加法与减法的时候，这两个向量的元素必须有相同的类型和数量
    - 乘法`.*` 原先的向量的每个元素乘以数字
    - 向量转置 转置操作使用一个单引号（'）来表示。`r = [ 1 2 3 4 ]; tr = r'`
    - 追加向量 允许在原有的向量中附加向量，共同创造新的向量`r = [r1,r2]` `r = [r1;r2]`
    - 向量的模 `|v| = √(v1^2 + v2^2 + … + vn^2)` 平方和取根号
    - 向量点积 `dot(a, b);`
        + 两个向量的点积a=(a1, a2, …, an) and b=(b1, b2, …, bn) 由以下给定:`a.b = ∑(ai.bi)`
    - 等差元素向量 `v = [f : n : l]`
        + 当一个向量中的元素过多，同时向量的各元素有等差的规律，此时采用直接输入法将过于繁琐。针对该种情况 ，可以使用冒号(:) 来生成等差元素向量。
        + e.g. `v = [1: 2: 10];` 输出：1 3 5 7 9
* 矩阵
    - 矩阵是一个二维数字阵列。
    - 创建一个矩阵每行输入空格或逗号分隔的元素序列 e.g. 3×3的矩阵：`m = [1 2 3; 4 5 6; 7 8 9]`
        + 同行元素之间用空格（或 “,”）隔开；
        + 行与行之间用 “;”（或回车符）隔开；
        + 矩阵的元素可以是数值、变量、表达式或函数；
        + 矩阵的尺寸不必预先定义。
    - 引用m行和n列的一个元素：`mx(m, n)` 注意行列是从1开始算
        + e.g. `a = [ 1 5 3 2; 2 4 5 3; 6 2 5 4; 7 8 3 4]`; a(2,3)为5
        + 引用m列中的所有元素 `a(:,3)`为 (3 5 5 3)' (第三列，为列向量)
        + 第 n 列的 m 个元素 `a(:, 2:3)`，第二列到第三列的元素
    - 删除行或列矩阵：分配一组空方括号 [ ] 给该行或列，`a(1, :) = []`删除第一行
    - 串联矩阵 `,`水平串联 `;`垂直串联
        + [A, B] 水平方向，([A 5] 报“要串联的数组的维度不一致”，空格也可串联，但要保证维度一致)
        + [A; B] 垂直方向
    - 矩阵的行列式 `d=det(A)`
    - 逆矩阵 逆矩阵的计算使用 inv 函数：逆矩阵A是`inv(A)`
    - 矩阵运算
        + 参考：[数组与矩阵运算](https://ww2.mathworks.cn/help/matlab/matlab_prog/array-vs-matrix-operations.html)
        + `*` 矩阵乘法 C = A*B 表示矩阵 A 和 B 的线性代数乘积。**A 的列数必须与 B 的行数相等**。 mtimes
        + `\` 矩阵左除 x = A\B 是方程 Ax = B 的解。矩阵 A 和 B 必须拥有相同的行数。 mldivide
        + `/` 矩阵右除 x = B/A 是方程 xA = B 的解。矩阵 A 和 B 必须拥有相同的列数。用左除运算符表示的话，B/A = (A'\B')'。 mrdivide
        + `^` 矩阵幂 A^B 表示 A 的 B 次幂（如果 B 为标量）。对于 B 的其他值，计算包含特征值和特征向量。mpower
        + `'` 复共轭转置 A' 表示 A 的线性代数转置。对于复矩阵，这是复共轭转置。 ctranspose
* 数组
    - 参考：[数组与矩阵运算](https://ww2.mathworks.cn/help/matlab/matlab_prog/array-vs-matrix-operations.html)
    - 在MATLAB中所有的数据类型的变量是多维数组，向量是一个一维数组，矩阵是一个二维数组。
    - `+`  加法 A+B 表示将 A 和 B 加在一起。 plus
    - `+`  一元加法 uplus
    - `-`  减法 A-B 表示从 A 中减去 B minus
    - `-`  一元减法 -A 表示对 A 的元素求反。 uminus
    - `.*` 按元素乘法 A.*B 表示 A 和 B 的逐元素乘积。 times
    - `.^` 按元素求幂 A.^B 表示包含元素 A(i,j) 的 B(i,j) 次幂的矩阵。 power
    - `./` 数组右除   A./B 表示包含元素 A(i,j)/B(i,j) 的矩阵。 rdivide
    - `.\` 数组左除   A.\B 表示包含元素 B(i,j)/A(i,j) 的矩阵。 ldivide
    - `.'` 数组转置   A.' 表示 A 的数组转置。对于复矩阵，这不涉及共轭。 transpose

注意矩阵运算和数组运算是有区分的：

* 矩阵运算遵循线性代数的法则，与多维数组不兼容。
* 运算双方所需输入的大小和形状取决于所执行的运算。
* 对于非标量输入，矩阵运算符与对应的数组运算符计算出的答案通常不同。

matlab数组和矩阵的区别：
    - 数组中的元素可以是字符等
    - 矩阵中的只能是数
    - 因为矩阵是一个数学概念（线性代数里的），数组是个计算机上的概念。
    - 所有 MATLAB 变量都是多维数组，与数据类型无关。
    - 矩阵是指通常用来进行线性代数运算的二维数组。


### 命令

* 管理会话的命令
    - `who` 使用 `who` 命令显示所有已经使用的变量名
    - `whos` 命令则显示多一点有关变量的信息，大小、字节、类型等
    - `clear` 命令删除所有（或指定）从内存中的变量（S）
    - `clc` 清除命令窗口。
* MATLAB的系统命令
    - cd、date、pwd
    - `dir`/`ls` 列出当前目录中的所有文件。
    - `load` 加载一个文件中的变量。
    - `type 文件名` 显示一个文件的​​内容。
* 输入输出命令
    - `format` 如果想更精确，需要使用 `format` 命令。
        + `format long`  显示小数点后16位，15位定点表示
        + `format short` 显示小数点后4位，四位十进制数（默认）
        + 还有其他形式`format +`, `format short e`等等
    - `disp` 显示一个数组或字符串的内容。 `arr=[1 2 0 3]; disp(a)`
    - `fscanf` 按格式从文件读取数据
    - `fprintf` 执行格式化写入到屏幕或文件
        + `fscanf`和`fprintf`行为类似C scanf和printf函数，%s,%d,%f,%e,%g
    - `input` 显示提示并等待输入 `input('please enter value name\n)`，按提示输入已存在变量，则会打印值
    - `;` 禁止显示网版印刷
* 向量，矩阵和阵列命令
    - `find`  查找非零元素的索引, `arr=[1 2 0 3]; find(arr)则1 2 3`
    - `length` 计算元素数量
    - `max` `min`
        + 如果 A 是向量，则 max(A) 返回 A 的最大值。
        + 如果 A 为矩阵，则 max(A) 是包含每一列的最大值的行向量。
        + `max(A,B)`，假设B是标量 B = 5;，则每元素会与B进行比较，取大的那个
            * A = [1 7 3; 6 2 9], C = max(A,B), 输出 [5 7 5; 6 5 9]
    - `size` 计算数组大小`arr=[1 2 0 3]; size(arr)则1 4 (一行四列)`
    - `sort` 排序每个列
    - `sum` 每列相加
    - `ones`全1矩阵  `zeros`生成零矩阵
    - cross矩阵交叉乘积 dot矩阵点积 det数组的行列式 inv矩阵的逆 pinv矩阵的伪逆 rank秩 rref行最简形 cell创建单元数组
* 绘图命令
    - axis 人工选择坐标轴尺寸
    - fplot 智能绘图功能
    - grid显示网格线
    - plot生成XY图
    - plotyy
        + 参考：[Matlab的plotyy用法总结](http://blog.sciencenet.cn/blog-427568-719786.html)
        + 很多时候，我们需要将两组或者多组数据量级相差很大的数据绘制在同一张图中以便观察，但往往数据较小的曲线会被较大的曲线淹没。e.g. `x=-1:.01:20; x=x'; y1=sin(x);z1=100*cos(x); plot(x,y1,x,z1);`
        + 解决这类问题的办法是用双纵坐标绘图，使得大小不同的数据分别属于不同的纵坐标。
        + 纵坐标绘图的函数plotyy基本用法是： `plotyy(x1,y1,x2,y2)`，即可将(x1,y1)绘制在左侧纵坐标轴中，将(x2,y2)图以右侧的纵坐标为纵轴。
        + 刚才的代码适用于两条曲线，如果要多条曲线，需要将这些曲线分成两组 `y2=sin(x+.5); z2=exp(x/4); z3=x.^2; plotyy(x,[y1,y2],x,[z1,z2,z3]);`
            * 图画出来了，但我们也搞不清哪条曲线是什么了。此时可以借助图例 `legend({'sin(x)','sin(x+.5)','100cos(x)','exp(x/4)','x^2'});`
        + `legend`中的次序与plotyy中绘图数据的排列次序相同
    - title 把文字置于顶部
    - xlabel/ylabel将文本标签添加到x/y轴
    - close关闭当前的绘图 `close all`关闭所有绘图

### M文件

* MATLAB允许写两个程序文件：
    - 脚本文件 .m 扩展程序文件。
        + 在这些文件中写的一系列命令，一起执行。
        + 脚本不接受输入 和 不返回在工作区中的数据操作的任何输出。
    - 函数文件 .m 扩展程序文件。
        + 函数可以接受输入和返回输出。

### 数据

* 默认情况下，MATLAB存储所有数值变量为双精度浮点值。
* MATLAB不需要任何类型声明或维度语句。
    - 当MATLAB遇到新的变量名称时，它将创建变量并分配适当的内存空间。
* 如果变量已经存在，则MATLAB将使用新内容替换原始内容，并在必要时分配新的存储空间。

* MATLAB提供15种基本数据类型
    - 分别是8种整型数据、单精度浮点型、双精度浮点型、逻辑型、字符串型、单元数组、结构体类型和函数句柄。(int8 uint8 int16 uint64 single char 单元格阵列 结构体...)
    - 字符串用单引号`'`
        + 字符串比较 strcmp strcmpi忽略大小写 strncmp strncmpi
        + 改变大小写 lower upper strtrim删除前导空格和尾随空格
    - 结构体
        + 结构体是使用被称为字段的数据容器将相关数据组合在一起的一种数据类型。
        + 每个字段都可以包含任意类型或任意大小的数据。
        + 结构体数组具有下列属性
            * 数组中的所有结构体都具有相同数目的字段。
            * 所有结构体都具有相同的字段名称。
            * 不同结构体中的同名字段可包含不同类型或大小的数据。
        + [创建结构体数组](https://ww2.mathworks.cn/help/matlab/matlab_prog/create-a-structure-array.html)
        + 定义结构体(直接赋值，会自动定义)：

```
patient(1).name = 'John Doe';
patient(1).billing = 127.00;
patient(1).test = [79, 75, 73; 180, 178, 177.5; 220, 210, 205];

>> patient 执行返回：
patient =
  包含以下字段的 struct:
       name: 'John Doe'
    billing: 127
       test: [3×3 double]
```


* 数据类型转换
    - MATLAB提供了各种用于将一种数据类型转换为另一种数据类型的函数。
        + char int2str mat2str str2double hex2dec struct2cell...
        + num2cell 将数组转换为相同大小的元胞数组
            * [num2cell](https://ww2.mathworks.cn/help/matlab/ref/num2cell.html)
            * `C = num2cell(A)`通过将 A 的每个元素放置于 C 的一个单独元胞中，来将数组 A 转换为元胞数组 C。
            * `C = num2cell(A,dim)`  dim 指定每个元胞包含 A 的哪个维度
                - e.g.假设是两维数组，1时每个元胞包含一个列向量 2时包含行向量； 三维数组则可指定1/2/3
            * 关于维度优先
                - MATLAB中是列优先，[matlab中的矩阵是行优先还是列优先？](https://blog.csdn.net/stpeace/article/details/8478811)
                - `a = ones(3, 5); sum(a) % 每一列的元素相加, y方向`,输出 3  3  3  3  3

**MATLAB列维度优先，与C/C++等语言不同**，生成的数组结构 参考：[num2cell](https://ww2.mathworks.cn/help/matlab/ref/num2cell.html)

4*7*3的数组A，y方向是4,x方向是7，z方向是3, 一维(4)是y方向

注意下例中取维度的结果：

```
test=[1 2 3; 4 5 6]
输出：
test =
     1     2     3
     4     5     6

num2cell(test) //转换为相同大小的元胞数组
输出：
ans =
  2×3 cell 数组
    {[1]}    {[2]}    {[3]}
    {[4]}    {[5]}    {[6]}

num2cell(test, 1) //指定每个元胞包含test的第一维的数据，注意此处是取各列的数据，列优先
输出：
ans =
  1×3 cell 数组
    {2×1 double}    {2×1 double}    {2×1 double}

num2cell(test, 2) //指定每个元胞包含test的第二维的数据，注意此处是取各行的数据，列优先
输出：
ans =
  2×1 cell 数组
    {1×3 double}
    {1×3 double}
```


* 数据类型确定
    - MATLAB提供了用于识别变量数据类型的各种函数。
        + is检测状态 isa确定输入是否是指定类的对象 iscell确定输入是单元格数组 ischar isfloat...
        + e.g. `x = 3.2; isinteger(x)`


* 元胞数组: 每个元胞都可以包含任意类型的数据
    - [cell](https://ww2.mathworks.cn/help/matlab/ref/cell.html)
    - 可以认为它和c语言里面的结构体、c++里面的对象很类似
    - 使用元胞数组构造运算符 {} 创建该数组`C = {'2017-08-16',[56 67 78]}`
    - 扩展元胞数组：`C(2,:) = {'2017-08-17',[58 69 79]}; C(3,:) = {'2017-08-18',[60 68 81]}`
    - 使用圆括号 () 对元胞数组进行索引，使用花括号 {} 对元胞的内容进行索引
    - 使用圆括号 () 进行索引时，将得到一个作为该元胞数组子集的元胞数组。
        + 对 C 的第一行进行索引。 `C(1,:)`, 输出：`{'2017-08-16'}    {1×3 double}`
    - 使用花括号 {} 进行索引时，将得到指定元胞中包含的数据段。
        + `C{1,2}` 表示元胞数组的第1行，第2列的元胞中包含的内容 `56 67 78`

### 条件控制

* if...end

```sh
    a = 10;
    % check the condition using if statement
    if a < 20
       fprintf('a is less than 20\n');
    end
    fprintf('value of a is : %d\n', a);
```

* if...else...end
* if...elseif...elseif...else...end
* switch(switch_expression)语句
    - switch_expression 是一个标量或字符串

```cpp
    grade = 'B';
    switch(grade)
    case 'A'
        fprintf('Excellent!\n');
    case 'B'
        fprintf('Well done\n');
    otherwise
        fprintf('Invalid grade\n');
    end
```

### 循环

* while

```sh
    while <expression>
        <statements>
    end
```

* for
    - for 循环的值有下述三种形式
        + `initval:endval` 递增1
        + `initval:step:endval` 按每次迭代中的值步骤递增
        + `valArray` 列向量索引

```sh
#1
    for a = 10:20
        fprintf('value of a: %d\n', a);
    end

#2
    for a = 1.0: -0.1: 0.0
       disp(a)
    end

#3
    for a = [24,18,17,23,28]
       disp(a)
    end
```

* `break;` `continue;`

### 函数

* 在MATLAB中，函数定义在单独的文件。
* 文件函数的文件名应该是相同的。
* 函数在自己的工作空间进行操作，被称为本地工作区；
* 函数可以接受多个输入参数和可能返回多个输出参数。
* 每个函数的第一行要以 `function` 关键字开始。
    - 语法： `function [out1,out2, ..., outN] = myfun(in1,in2,in3, ..., inN)`

e.g. 五个数字作为参数并返回最大的数字

```sh
function max = mymax(n1, n2, n3, n4, n5)
%This function calculates the maximum of the
% five numbers given as input
max = n1;
if(n2 > max)
    max = n2;
end
if(n3 > max)
   max = n3;
end
if(n4 > max)
    max = n4;
end
if(n5 > max)
    max = n5;
end
```

MATLAB匿名函数：

* 在MATLAB命令行或在一个函数或脚本可以定义一个匿名函数。`f = @(arglist)expression`
* 这种方式，可以创建简单的函数，而不必为他们创建一个文件。

e.g. 编写一个匿名函数 power，需要两个数字作为输入并返回x的n次幂。

```sh
power = @(x, n) x.^n;
result1 = power(2, 3)
```

多返回值 e.g. 写一个 quadratic 函数来计算一元二次方程的根。

```sh
function [x1,x2] = quadratic(a,b,c)
    %this function returns the roots of
    % a quadratic equation.
    % It takes 3 input arguments
    % which are the co-efficients of x2, x and the
    %constant term
    % It returns the roots
    d = disc(a,b,c);
    x1 = (-b + d) / (2*a);
    x2 = (-b - d) / (2*a);
end

function dis = disc(a,b,c)
    dis = sqrt(b^2 - 4*a*c);
end
```

### 数据导入导出

* MATLAB中导入数据的方式有两种
    - 在命令行通过代码把数据导进去
        + `importdata` 函数允许加载各种不同格式的数据文件`A = importdata(filename)`
    - 通过MATLAB的数据导入向导导入数据

* MATLAB 中数据导出可以将数据写入文件
    - 还可以将数据导出到 Excel

### MATLAB绘图

e.g. 绘制简单的函数 y = x , x 值的范围从0到100，增量为5。

```sh
x = [0:5:100];
y = x;
plot(x, y)
```

e.g. 绘制函数 y = x^2

```
x = [1 2 3 4 5 6 7 8 9 10];
x = [-100:20:100];
y = x.^2;
plot(x, y)
```

* 可以在 MATLAB 中添加标题，调整 x 轴和 y 轴(xlabel 和 ylabel)，网格线，并沿标签美化图形
    - MATLAB设置轴刻度 `axis([xmin xmax ymin ymax])`
    - 设置线型、标记符号和颜色 `plot(X,Y,LineSpec)`，参考：[plot二维线图](https://ww2.mathworks.cn/help/matlab/ref/plot.html?searchHighlight=plot&s_tid=doc_srchtitle#btzitot-LineSpec)

e.g.

```sh
x = [0:0.01:10];
y = sin(x);
plot(x, y), xlabel('x'), ylabel('Sin(x)'), title('Sin(x) Graph'),
grid on, axis equal
```

* 在MATLAB中可以绘制多个图形

```sh
x = [0 : 0.01: 10];
y = sin(x);
f = cos(x);
plot(x, y, x, f, '.-'), legend('Sin(x)', 'Cos(x)')
```

* 颜色，线性

```
Color   Code
White   w
Black   k
Blue    b
Red     r
Cyan    c
Green   g
Magenta m
Yellow  y
```

```
plot(x, y, x, f, '.-k')       设置第二个线型为 .-，颜色为黑色
plot(x, y, '--+g', x, f, '.-r') 设置第一个线型为虚线，颜色为绿色；第二个.-红色
```

### 图形

* 如何绘制二维条形图
    - MATLAB 中使用 `bar` 命令绘制一个二维条形图

e.g. 如果有一个包含10名学生的教室，这些学生获得的分数的百分比是75，58，90，87，50，85，92，75，60和95，使用这个数据，我们将绘制条形图。

```sh
x = [1:10];
y = [75, 58, 90, 87, 50, 85, 92, 75, 60, 95];
bar(x,y), xlabel('Student'),ylabel('Score'),
title('First Sem:')
print -deps graph.eps
```

* 如何绘制等值(contour)线
    - MATLAB 提供了一个轮廓绘制等高线图的函数`meshgrid`。

绘制函数 g = f(x, y), where −5 ≤ x ≤ 5, −3 ≤ y ≤ 3，这两个值的增量为0.1。这些变量设置语法为：
`[x,y] = meshgrid(–5:0.1:5, –3:0.1:3);`

需要分配功能，具体函数：x2 + y2

```sh
[x,y] = meshgrid(-5:0.1:5,-3:0.1:3); %independent variables
g = x.^2 + y.^2;                     % our function
contour(x,y,g)                       % call the contour function
print -deps graph.eps
```

* 如何绘制三维图
    - 使用 `surf` 命令来创建曲面图

e.g. 建立一个三维地图函数表面 `g = xe^(-(x^2 + y^2))`

```sh
[x,y] = meshgrid(-2:.2:2);
g = x .* exp(-x.^2 - y.^2);
surf(x, y, g)
print -deps graph.eps
```

### MATLAB代数

* MATLAB 中使用 `solve` 命令求解代数方程组
    - e.g. `solve(x-5==0)` 返回 ans=5，注意不用单引号，且相等用`==`
    - `solve('x-5=0')`语法已移除，会报语法错误
    - 使用`syms`定义变量，并用`solve(2*x == 1,x)`来替换`solve('2*x == 1','x')`

参考：[solve Syntax](https://ww2.mathworks.cn/help/symbolic/solve.html)

>**Support for character vector or string inputs has been removed. Instead, use syms to declare variables and replace inputs such as solve('2*x == 1','x') with solve(2*x == 1,x).**

* solve 语法
    - `S = solve(eqn,var)` 解关于var变量(variable)的方程(equation)eqn, 如果没有指定var，则`symvar`函数会确定一个求解的变量，e.g. `solve(x + 1 == 2, x)`解关于x的方程x + 1 = 2
* solve 示例

- 解二次方程(Quadratic Equation)

```sh
#定义变量
syms a b c x
#定义方程
eqn = a*x^2 + b*x + c == 0;
#解方程
S = solve(eqn)

输出：
S =
 -(b + (b^2 - 4*a*c)^(1/2))/(2*a)
 -(b - (b^2 - 4*a*c)^(1/2))/(2*a)
```

若指定解关于a的方程：

```
Sa = solve(eqn,a)

输出：
Sa =
-(c + b*x)/x^2
```

- 解方程组()

e.g. 求解方程组

```
5x + 9y = 5
3x – 6y = 4
```

```sh
# 定义x和y，注意没有,
syms x y
# 接收两个返回值，指定是求解x和y的方程
[solx, soly] = solve(5*x + 9*y == 5, 3*x - 6*y == 4, x, y)

#也可以单独定义方程组：
eqns = [5*x + 9*y == 5, 3*x - 6*y == 4]
[solx, soly] = solve(eqns, [x,y])
#S = solve(eqns,[u v])，则S是一个结构体，需要S.x,S.y来单独访问
```

* expand 和 collect
    - MATLAB中 expand 和 collect 命令用于扩展，并分别收集一个方程。

### MATLAB微积分

`limit` 命令计算极限

### MATLAB多项式

`polyval` 函数用于计算多项式

```
p = [1 7 0  -5 9];
polyval(p,4)
```

计算矩阵多项式 `polyvalm` 函数

计算多项式的根`roots`

`poly` 函数是根函数，并返回多项式的系数的倒数

### MATLAB变换

拉普拉斯和傅里叶变换

傅立叶变换实现了时间与频率的转换；`FT = fourier(f)`
拉普拉斯变换可以将时域函数变换复频域函数，简化微积分方程计算。 `laplace(f(t))`

## 快捷键和注意项

```
* 在编辑器(Editor)中：
    1) 【Tab】（或【Ctrl+]】）――增加缩进（对多行有效）
    2) 【Ctrl+[】 (或shift+tab－－减少缩进（对多行有效）

    3) 【Ctrl+I】－－自动缩进（即自动排版，对多行有效）
    4) 【Ctrl+R】――注释（对多行有效）
    5) 【Ctrl+T】――去掉注释（对多行有效）

* 续行号...不能放在等号后面使用，不能放在变量名中间使用，起作用时默认显蓝色
* 多行注释，%{和%}配套使用，且都需要单独成行
%{
%}
* 等号=用于赋值
* 双等号==表示数学意义上的等号
* 主窗口里面，输入时，换行用Shift+Enter
* 矩阵中用圆括号表示下标，单元数组用大括号表示下标
* which+函数名 证实该函数是否在当前路径

函数式M文件中，变量都是局部变量
脚本式M文件中，变量都是全局变量
```

## 部分函数功能说明


* `NaN(n, 1)` % 返回非数字的标量表示，由NaN值组成的n行，1列的矩阵
* Y = `prctile(X,p)` %计算X中大于p%的值，p必须介于0和100之间
    - 百分位数（Percentile）?
        + 参考[百分位数](https://wiki.mbalib.com/wiki/%E7%99%BE%E5%88%86%E4%BD%8D%E6%95%B0)
        + 百分位数又称百分位分数(percentile)，是一种相对地位量数，它是次数分布(也称频数分布:按某标准分组后，统计出各个组内含个体的个数)中的一个点。
        + 百分位通常用第几百分位来表示，如第五百分位，它表示在所有测量数据中，测量值的累计频次达5%
        + 百分位数则是对应于百分位的实际数值。
* Y = `abs(X)` 返回数组 X 中每个元素的绝对值。
    - 如果 X 是复数，则 abs(X) 返回复数的模。
* Y = `ceil(X)` 朝正无穷大四舍五入
    - 将 X 的每个元素四舍五入到大于或等于该元素的最接近整数。
    - >> X = [-1.9 -0.2 3.4; 5.6 7 2.4+3.6i];
    - >> Y = ceil(X)
    - -1.0000 + 0.0000i   0.0000 + 0.0000i   4.0000 + 0.0000i
    - 6.0000 + 0.0000i   7.0000 + 0.0000i   3.0000 + 4.0000i
* `Y = floor(X)` 朝负无穷大四舍五入
    - 将 X 的每个元素四舍五入到小于或等于该元素的最接近整数
* `find` 查找非零元素的索引，即矩阵中非0元素的下标
    - `k = find(X)` 返回一个包含数组 X 中每个非零元素的线性索引的向量。
    - 如果 X 为多维数组，则 find 返回由结果的线性索引组成的列向量。
        + e.g. `X = [1 0 2; 0 1 1; 0 0 4]`, `k = find(X)`, 返回列向量 [1 5 7 8 9]'(注意按列开始算索引)
    - `k = find(X,n)` 前n个
    - `k = find(X,n,direction)` 前n个还是后n个，由direction指定，默认是'first'，'last'是后n个
    - `[row,col] = find(xxx)` 可以接收结果的行和列，分别打印row，col
    - `[row,col,v]` 另外可以接收向量的值
* `exp` `Y = exp(X)` 为数组 X 中的每个元素返回指数 e^x， 计算 1 的指数，它是欧拉数 e `exp(1)` ans = 2.7183
* `expm` `Y = expm(X)` 计算 X 的矩阵指数。
    - 对于逐个元素的指数运算，请使用 exp。
* `diff` 差分和近似导数
    - `Y = diff(X)` 向量元素之间的差分
        + 如果 X 是长度为 m 的向量，则 Y = diff(X) 返回长度为 m-1 的向量。
        + Y 的元素是 X 相邻元素之间的差分。`Y = [X(2)-X(1) X(3)-X(2) ... X(m)-X(m-1)]`
        + e.g. `X = [1 1 2 3 5 8 13 21]; Y = diff(X)`   输出(系数降1)： 0  1  1  2  3  5  8
    - `Y = diff(X)` 矩阵行之间的差分
        + 如果 X 是不为空的非向量 p×m 矩阵，则 Y = diff(X) 返回大小为 (p-1)×m 的矩阵
        + 其元素是 X 的行之间的差分 `Y = [X(2,:)-X(1,:); X(3,:)-X(2,:); ... X(p,:)-X(p-1,:)]`
        + e.g. `X = [1 1 1; 5 5 5; 25 25 25]; Y = diff(X)` 输出如下(y方向第二行-第一行，行-1)：
        +   4     4     4
        +   20    20    20
    - `Y = diff(X,n)` 多阶差分
        + 通过递归应用 diff(X) 运算符 n 次来计算第 n 个差分。
        + 在实际操作中，这表示 diff(X,2) 与 diff(diff(X)) 相同。 (系数-2)
    - `Y = diff(X,n,dim)` 矩阵列之间的差分
        + 是沿 dim 指定的维计算的第 n 个差分。dim 输入是一个正整数标量。
        + MATLAB列优先，若指定维度是2，则为相邻两列的各个行元素对应相减
            * e.g. `X = [1 3 5;7 11 13;17 19 23]; Y = diff(X,1,2)` 输出如下(x方向相邻元素相减，列-1)：
            * 2     2
            * 4     2
            * 2     4
        + 若指定维度1
            * e.g. `X = [1 3 5;7 11 13;17 19 23]; Y = diff(X,1,1)` 输出如下(y方向上相邻相减，行-1)：
            * 6     8     8
            * 10    8    10
    - 前面num2cell介绍过MATLAB列维度优先，与C/C++等语言不同，生成的数组结构：
        + `4*7*3`的数组A，y方向是4,x方向是7，z方向是3，一维(4)是在y方向
    - [diff](https://ww2.mathworks.cn/help/matlab/ref/diff.html?searchHighlight=diff&s_tid=doc_srchtitle)
* `sum` 数组元素总和
    - 如果 A 是向量，则 `sum(A)` 返回元素之和。(不论是行还是列向量)
    - 如果 A 是矩阵，则 `sum(A)` 将返回包含每列总和的行向量。
    - `S = sum(A,'all') ` 计算 A 的所有元素的总和。此语法适用于 MATLAB® R2018b 及更高版本(现在用的R2018a)
    - `S = sum(A,dim)` 沿维度 dim 返回总和。例如，如果 A 为矩阵，则 `sum(A,2)` 是包含每一行总和的列向量
    - `sum(___,nanflag)`指定计算中包括还是忽略 NaN 值，'includenan'、'omitnan'
    - `sum(___,outtype)` 指定计算中数据类型，'default'、'double' 或 'native'
* `mean` 数组的均值
    - 如果 A 是向量，则 mean(A) 返回元素均值。
    - 如果 A 为矩阵，那么 mean(A) 返回包含每列均值的行向量。 (y方向上)
    - `M = mean(A,'all')` 计算 A 的所有元素的均值。此语法适用于 MATLAB® R2018b 及更高版本
    - `M = mean(A,dim)` 返回维度 dim 上的均值。例如，如果 A 为矩阵，则 `mean(A,2)` 是包含每一行均值的列向量。
    - `mean(___,nanflag)`指定计算中包括还是忽略 NaN 值，`mean(___,outtype)` 指定计算中数据类型
* 函数返回值接收时，用`~`表示忽略一个返回参数
* `histc` 直方图的 bin 计数（不推荐；使用 histcounts）
    - `bincounts = histc(x,binranges)` 计算 x 中位于各指定 bin 范围内的值的数量。
        + 如果 x 是一个向量，则 histc 将以直方图中 bin 计数的向量形式返回 bincounts
        + 如果 x 是一个矩阵，则 histc 可沿 x 的每列进行运算，并以每列的直方图 bin 计数的矩阵形式返回 bincounts。
    - 要绘制直方图，请使用 bar(binranges,bincounts,'histc')。
    - `[bincounts,ind]= histc(___)` 返回 ind，一个与 x 大小相同的数组，该数组表示在 x 中排序的每一项的直方图数量。 还不明确ind啥东西
        + bincounts 包含各 bin 中的值数量(即范围段的元素数量)。 ind 表示 bin 数量。
    - binranges(bin 范围)，指定为单调非降值的向量或沿各连续列排列的单调非降值的矩阵。
        + 如果 binranges 包含复数值，则 histc 忽略虚部和只使用实部。
        + 每个 bin 包括左端点，但不包括右端点
        + 如果 binranges 等于向量 `[0,5,10,13]`，则 histc 可创建四个bin，[0,5),[5,10),[10,13),[13,13]
    - [histc](https://ww2.mathworks.cn/help/matlab/ref/histc.html?searchHighlight=histc&s_tid=doc_srchtitle)
* `exist name` 或者 `exist(name)` 以数字形式返回 name 的类型，不同返回代表不同类型
    - 0不存在, 1工作区中的变量, 2是扩展名为 .m、.mlx、 或 .mlapp 的文件`exist './data/ORCL_20130515.mat'`
    - 3 MEX 文件, 4 Simulink 模型或库文件, 5内置 MATLAB 函数, 6 P 代码文件, 7文件夹, 8类
    - `exist name searchType`或者 `exist(name,searchType)`可以指定搜索的类型，searchType取值如下
        + builtin 只检查内置函数。 可能返回 5、0
        + class   只检查类。 8、0
        + dir     只检查文件夹。 7、0
        + file    只检查文件或文件夹。 2、3、4、6、7、0
        + var     只检查变量。 1、0
    - [exist](https://ww2.mathworks.cn/help/matlab/ref/exist.html?searchHighlight=exist&s_tid=doc_srchtitle)
* `sign` （求符号函数）返回数组各元素的符号(正1, 0, 负-1，复数比较特殊需计算)
    - `Y = sign(x)` 返回与 x 大小相同的数组 Y
        + x 为复数时 值为 x./abs(x)
* `all` 确定数组元素是否 全部为非零
    - `B = all(A)` 如果 A 为向量
        + 当所有元素为非零时，all(A) 返回逻辑 1 (true)
        + 当一个或多个元素为零时，返回逻辑 0 (false)。
    - 如果 A 为非空矩阵，all(A) 将 A 的各列视为向量，返回包含逻辑 1 和 0 的行向量。
        + e.g. `A = [0 0 3;0 0 3;0 0 3]` B=all(A) 返回， 0 0 1
    - `B = all(A,dim)` 沿着 dim 维测试元素。dim 输入是一个正整数标量。dim指定维度
* `rng` 控制随机数生成
    - 要将 rng 函数（而非 rand 或 randn）与 'seed'、'state' 或 'twister' 输入结合使用
        + `X = randi(imax)`   返回一个介于 1 和 imax 之间的伪随机整数标量
        + `X = randi(imax,n)` 返回 n×n 矩阵，其中包含从区间 [1,imax] 的均匀离散分布中得到的伪随机整数
        + `randi(10,3,4)` 返回一个由介于 1 和 10([1,10]) 之间的伪随机整数组成的 3×4 数组
    - rng(seed) 使用非负整数 seed 为随机数生成器提供种子，以使 rand、randi 和 randn 生成可预测的数字序列
* `size` 数组大小
    - sz = size(A)
* `properties` 类属性名称
    - `properties(ClassName)` 显示 MATLAB® 类的可见公共属性的名称，包括继承的属性。
* `repmat` 重复数组副本
    - `B = repmat(A,n)` 返回一个数组，该数组在其行维度和列维度包含 A 的 n 个副本
    - A 为矩阵时，B 大小为 size(A)*n。
    - `B = repmat(A,r1,...,rN)` 指定一个标量列表 r1,..,rN，这些标量用于描述 A 的副本在每个维度中如何排列。
        + 当 A 具有 N 维时，B 的大小为 `size(A).*[r1...rN]`。 (`.*` 逐元素乘积)
        + e.g. repmat([1 2; 3 4],2,3) 返回一个 4×6 的矩阵。 (注意，r1...为标量数字)
    - `B = repmat(A,r)` 使用行向量 r 指定重复方案。r是行向量，效果和 r1,...,rN 一样
* `numel` 数组元素的数目
    - `n = numel(A)` 返回数组 A 中的元素数目n
    - 等同于 prod(size(A))


### 语法功能

* A(1, 3)，A中第1行第3列的元素
* [A, B] = funTest()，[A, B]表示接收多个返回
* [A, B] 表示串联，串联矩阵: `,`水平串联 `;`垂直串联
* 返回满足条件的矩阵元素：

```
a = [1 2 3; 5 6 7]
输出：
a =
     1     2     3
     5     6     7

idx = (a>=2)
输出：
idx =
  2×3 logical 数组
   0   1   1
   1   1   1

a(idx)
输出(注意返回的是列向量，索引从列开始算)：
ans =
     5
     2
     6
     3
     7
```

* 类 使用面向对象编程创建可在 MATLAB® 中使用的新对象类型
    - 创建类可以简化涉及与特殊类型的数据交互的专用数据结构体或大量函数的编程任务。
    - MATLAB 类支持函数和运算符重载、对属性和方法的控制访问、引用和值语义以及事件和侦听程序。
    - 类定义 [类组件](https://ww2.mathworks.cn/help/matlab/matlab_oop/class-components.html)
        + `classdef...end` - 所有类组件的定义
    - 属性
        + `properties...end` - 属性名称声明、属性特性设定、默认值赋值
    - 方法(函数)
        + `methods...end` - 方法签名、方法属性和函数代码的声明
        + 包含属性定义，其中包括可选的初始值
    - 事件
        + `events...end` - 事件名称和属性的声明
    - 枚举类
        + `enumeration...end` - 枚举类的枚举成员和枚举值的声明。

```
classdef示例，以下 classdef 定义名为 MyClass 的类，该类是 handle 类的子类，但它不能用于派生子类：
classdef (Sealed) MyClass < handle
   ...
end

properties示例，以下类定义名为 Prop1 的属性，该属性具有私有访问权限，默认值等于 date 函数的输出。
classdef MyClass
   properties (SetAccess = private)
      Prop1 = date
   end
   ...
end

methods示例，
classdef MyClass
   methods (Access = private)
      function obj = myMethod(obj)
      ...
      end
   end
end

events示例，例如，以下类定义名为 StateChange 的事件，ListenAccess 设置为 protected：
classdef EventSource
   events (ListenAccess = protected)
      StateChanged
   end
   ...
end

enumeration示例，例如，以下类定义两个枚举成员，分别代表逻辑值 false 和 true：
classdef Boolean < logical
   enumeration
      No  (0)
      Yes (1)
   end
end
```


### 数学知识

* 差分(difference)
    - 又名差分函数或差分运算，差分的结果反映了离散量之间的一种变化，是研究离散数学的一种工具
    - 差分对应离散，微分对应连续。
    - 差分又分为前向差分、向后差分及中心差分三种
        + 函数的前向差分通常简称为函数的差分, `Delta(f(x(k))) = f(x(k+1)) - f(x(k))`
        + 前向差分会将多项式阶数降低1
        + 向后差分 `Delta(f(x(k))) = f(x(k)) - f(x(k-1))` (差分方程)
            * 等差数列是最简单形式的差分方程Da(n) = d;
        + 中心差分 `Delta(f(x(k))) = 1/2 * (f(x(k+1)) - f(x(k-1)))`
    - 设变量y依赖于自变量t ,当t变到t + 1时,因变量y = y(t)的改变量Dy(t)= y(t+1) - y(t)称为函数y(t)在点t处步长为1的(一阶)差分，记作Dy1= y(t+1)- y(t)，简称为函数y(t)的(一阶)差分,并称D为差分算子。

* 微分(Differentiation)
    - 微分在数学中的定义：由函数B=f(A)，得到A、B两个数集，在A中当dx靠近自己时，函数在dx处的极限叫作函数在dx处的微分
    - 微分的中心思想是无穷分割。微分是函数改变量的线性主要部分。微积分的基本概念之一。

### MATLAB转C++

matlab coder
[matlab程序转C/C++代码](https://blog.csdn.net/libing403/article/details/77344839)

数组设置为无限长

[生成的示例 C/C++ 主函数的结构](https://ww2.mathworks.cn/help/coder/ug/structure-of-example-cc-main-function.html)

[MATLAB Coder 对生成代码进行优化](https://ww2.mathworks.cn/help/coder/ug/matlab-coder-optimizations-in-generated-cc-code.html)