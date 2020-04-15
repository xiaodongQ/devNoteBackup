## boost

官网：
https://www.boost.org/

文档：
[Boost 1.71.0 Library Documentation](https://www.boost.org/doc/libs/1_71_0/)

10个Boost库被包含在c++标准委员会的库技术报告(TR1)和新的c++ 11标准中。除了来自TR1的Boost库之外，c++ 11还包含了其他几个Boost库。在c++ 17中提出了更多的Boost库以实现标准化。

>Ten Boost libraries are included in the C++ Standards Committee's Library Technical Report (TR1) and in the new C++11 Standard. C++11 also includes several more Boost libraries in addition to those from TR1. More Boost libraries are proposed for standardization in C++17.

### 编译安装

[windows下编译和安装boost库](https://www.cnblogs.com/cmranger/p/4759223.html)

源码包解压后的目录结构：

```sh
BOOST_1_55_0    #boost根目录，存放配置脚本和说明文件
├─boost         #所有boost库头文件，90%以上的boost库源码都在这里
├─doc           #文档
├─libs          #所有组件的示例/测试/编译代码和说明文档
├─more          #库作者相关的文档
├─status        #可用于测试boost库的各个组件
└─tools         #b2、quickbook等一些自带的工具
```

boost子目录是最为重要的目录。以头文件的形式分门别类存放了要使用的库代码

boost库大多数组件不需要编译链接就可以使用，在自己的工程中直接包含头文件即可。  
如要使用boost xpressive正则库(与regex库不同，该正则库不需要编译)，只需要在自己的源代码中包含头文件`#include <boost/xpressive/xpressive_dynamic.hpp>`即可。

* `bootstrap.bat` (linux下执行./bootstrap.sh) 该脚本为boost.build系统运行准备环境，是编译前的配置工作。
* `b2.exe`和`bjam.exe` (执行完提示使用./b2)
    - 执行完该脚本后，在boost源码安装包的根目录会生成`b2.exe`和`bjam.exe`两个可执行文件
    - 这两个文件是一样的，只是名字不同。
    - `b2 --show-libraries`命令可查看所有必须编译才能使用的库
* 完全安装boost
    - `bjam --buildtype=complete`
    - windows本机安装(全量,可以缩减只用date_time,test)用了：`./b2.exe install --prefix=F:/boost_1_70_0/mingw --build-type=complete toolset=gcc threading=multi` （--build-type=complete在unix用不了，需要单独编译动态静态和调试版本)
* 定制安装boost
    - 完整编译boost费时费力，而且这些库在并不可能在开发中全部用到，因此只需编译需要的库即可
    - `bjam --show-libraries` 查看所有必须编译才能使用的库
    - e.g. 单独编译安装regex库(建议stage)，`bjam stage --with-regex link=static runtime-link=shared threading=multi`
        + 编译结果在stage\lib目录下生成regex库(Debug/Release两个版本)
        + 本linux仅装date_time和test:
            * 动态：`./b2 stage --stagedir=/home/xd/local/boost --with-date_time --with-test link=shared runtime-link=shared threading=multi`
            * 静态：`./b2 stage --stagedir=/home/xd/local/boost --with-date_time --with-test link=static runtime-link=static threading=multi`
            * `./b2 install --prefix=/usr/local` 安装会把`boost_1_70_0/boost`(156MB)里面大部分(154MB)内容拷贝到指定的prefix下面的include中，不如手动创建include后将boost(包含boost)拷贝过去
                - 部署一个环境时安装忘记指定`--prefix=/usr/local`了，创建`/usr/local/boost`路径，并创建lib和include目录将默认安装的lib中和boost/include中所有信息移动到了该路径的子目录下
                - 注意头文件的路径层次为 `/usr/local/boost/include/boost`
                - 如上include中还有一级boost目录，否则编译踩坑：
                    + 代码中包含头文件`#include <boost/shared_ptr.hpp>`
                    + 编译时指定的路径`-I/usr/local/boost/include`
                    + 所以编译会到 /usr/local/boost/include/boost 中找 shared_ptr.hpp

* 关于link和runtime-link的组合关系：
    - [link 和 runtime-link，搭配shared 和 static](https://blog.csdn.net/yasi_xi/article/details/8660549)
    - **假设：** 一个库A依赖于库B，我们自己的程序client依赖于库A
    - link=static、runtime-link=static  client通过A.a (A.lib)静态包含A；A通过B.a (B.lib)静态包含B；不关 .so .dll的事
    - link=static、runtime-link=shared  client通过A.a (A.lib)静态包含A；在运行时，client要动态调用B.so (B.dll)
    - link=shared、runtime-link=shared  client会包含A.a (A.lib)；A会包含 B.a (B.lib)；但都只保存动态库的真正实现的stub，运行时通过stub去动态加载 A.so (A.dll), B.so (B.dll) 中的实现
        + stub，桩代码，满足形式要求但没有实现实际功能的占坑/代理代码
    - link=shared、runtime-link=static  client会包含A.a (A.lib)，但只包含真正实现的stub；A通过B.a (B.lib)静态包含B；运行时，client会动态调用A.so (A.dll)

参数含义：

```
--prefix/--stagedir=<PREFIX>   编译后安装路径，默认C:\Boost
--build-type=<type> 编译类型，可选minimal（最小）、complete（完整），默认minimal。
--with-<library>    加入此参数，代表只编译的库。 假如只安装regex `--with-regex`
--without-<library> 加入此参数，代表忽略编译的库。
toolset             指定编译器，win下默认msvc，用MinGW则选择gcc。可选的如borland、gcc、msvc（VC6）、msvc-9.0（VS2008）等
stage/install：     stage表示只生成库（dll和lib），install还会生成包含头文件的include目录
                可以直接用stage，install生成的include目录实际就是boost安装包解压缩后的boost目录，新建include把文件放入即可
link=static         指定生成静态regex库，可指定为shared生成动态链接库
                    一般boost库可能都是以static方式编译，因为最终发布程序带着boost的dll感觉会比较累赘。
threading=multi     指定生成多线程库，可指定单线程single，一般都写多线程程序
runtime-link=shared 指定动态链接C和C++ 运行库
stagedir/prefix：   stage时使用--stagedir，install时使用--prefix，表示编译生成文件的路径
                    推荐给不同的IDE指定不同的目录，如VS2008对应的是E:\SDK\boost\bin\vc9，VC6对应的是E:\SDK\boost\bin\vc6，否则都生成到一个目录下面，难以管理
debug/release：     编译debug/release版本。一般都是程序的debug版本对应库的debug版本，所以两个都编译
```

### date time

#### boost::gregorian::date

[Gregorian Date System](https://www.boost.org/doc/libs/1_71_0/doc/html/date_time/gregorian.html)

Gregorian(英 /grɪ'gɔ:rɪən/) Date：公历日期

Gregorian Date System提供了一个基于公历日期的编程系统  
实现的日历是一种“预期公历”，可以追溯到1582年首次采用公历之前。当前的实现支持1400-1-01到9999-12-31之间的日期  
> The implemented calendar is a "proleptic Gregorian calendar" which extends dates back prior to(可以追溯到) the Gregorian Calendar's first adoption in 1582.The current implementation supports dates in the range 1400-Jan-01 to 9999-Dec-31

##### 使用

需要包含头文件：  
`#include "boost/date_time/gregorian/gregorian.hpp" //include all types plus i/o` (一般包含这个)  
或者  
`#include "boost/date_time/gregorian/gregorian_types.hpp" //no i/o just types` (只有类型)

```cpp
// Construct
date d(2002,Jan,10);

std::string ds("2002/1/25");
date d(from_string(ds));

std::string ds("20020125");
date d(from_undelimited_string(ds));

date d(day_clock::local_day());         // Get the local day based on the time zone settings of the computer.
date d(day_clock::universal_day());     // Get the UTC day.

// Accessors
date d(2002,Jan,10);
d.year(); // --> 2002
d.month(); // --> 1
d.day(); // --> 10                    //Get the day part of the date.
d.day_of_week(); // --> Thursday      //Get the day of the week (Sunday, Monday, etc.)
d.day_of_year(); // --> 10            //Get the day of the year. Number from 1 to 366
d.end_of_month(); // --> 2000-Jan-31  //Returns a date object set to the last day of the calling objects current month

// Convert to String
std::string to_simple_string(date d); //To YYYY-mmm-DD string where mmm is a 3 char month name. "2002-Jan-01"
std::string to_iso_string(date d);    //To YYYYMMDD where all components are integers. "20020131"
std::string to_iso_extended_string(date d); //To YYYY-MM-DD where all components are integers. "2002-01-31"
```

#### 相关类

boost::gregorian::date_duration:  
类boost::gregorian::date_duration是一个简单的日计数，用于计算gregorian::date

months: `months single(1);`  
years: `years single(1);`  
weeks: `weeks single(1);`

date_period:  
类boost::gregorian::date_period提供了两个日期之间范围的直接表示

```cpp
date_period dp(date(2005,Jan,1), days(3));
dp.shift(days(3));                         //Add duration to both begin and end.
// dp == 2005-Jan-04 to 2005-Jan-07

date_period dp(date(2005,Jan,2), days(2));
dp.expand(days(1));                        //Subtract duration from begin and add duration to end. 从开始减去持续时间，从结束添加持续时间
// dp == 2005-Jan-01 to 2005-Jan-04

std::string to_simple_string(date_period dp); //To [YYYY-mmm-DD/YYYY-mmm-DD] string where mmm is 3 char month name. [2002-Jan-01/2002-Jan-31]
```

#### boost::posix_time::ptime

[Posix Time](https://www.boost.org/doc/libs/1_71_0/doc/html/date_time/posix_time.html)

定义了具有纳秒/微秒分辨率和稳定计算性能的非调整时间系统

`#include "boost/date_time/posix_time/posix_time.hpp" //include all types plus i/o`  
or  
`#include "boost/date_time/posix_time/posix_time_types.hpp" //no i/o just types`

```cpp
// Construct
ptime t1(date(2002,Jan,10), time_duration(1,2,3));

std::string ts("2002-01-20 23:59:59.000");
ptime t(time_from_string(ts));

std::string ts("20020131T235959");
ptime t(from_iso_string(ts));

ptime t(second_clock::local_time());

// Accessors
date d(2002,Jan,10);
ptime t(d, hour(1));
t.date() // --> 2002-Jan-10;
t.time_of_day() // --> 01:00:00;

boost/date_time/posix_time/time_formatters.hpp
boost::posix_time::to_simple_string(ptime)
std::string to_simple_string(ptime); // 2002-Jan-01 10:00:01.123456789
```

```cpp
const boost::posix_time::ptime dateToFormat;

time_facet *facet = new time_facet("%Y-%m-%d %H:%M:%S");  //new不用手动释放？，ostringstream中会释放？
std::ostringstream oss;
oss.imbue(std::locale(oss.getloc(), facet));

oss << dateToFormat;

return oss.str();
```

时间叠加(链接中有示例)：

重载了 ptime operator+(days)

```cpp
date d(2002,Jan,1);
// 加minutes (通过vscode跳转到minutes定义的hpp文件，可看到还有hours, seconds等, 都是继承自time_duration)
// boost::posix_time::minutes类定义的文件：boost-1_70\boost\date_time\posix_time\posix_time_duration.hpp
ptime t(d,minutes(5));
// days定义的文件：boost-1_70\boost\date_time\gregorian\greg_duration.hpp
// boost::gregorian::days 是date_duration的别名：typedef date_duration days;
days dd(1);
ptime t2 = t + dd;
```


`boost::posix_time::time_duration`

```cpp
using namespace boost::posix_time;
time_duration td(1,2,3,4); //01:02:03.000000004 when resolution is nano seconds
time_duration td(1,2,3,4); //01:02:03.000004 when resolution is micro seconds

```

`boost::posix_time::time_period`

类`boost::posix_time::time_period`提供了两个时间范围的直接表示。

`boost::posix_time::iterators`

时间迭代器提供了一种遍历时间的迭代机制。

#### Date Time Input/Output

定义输入输出格式(格式化的 %S %M 这类信息定义在此处)：  
[Date Time IO System](https://www.boost.org/doc/libs/1_71_0/doc/html/date_time/date_time_io.html)

* date_facet
    - `boost::date_time::date_facet` 该类为日期类型提供基于I/O方面的格式
    - 这个类允许使用格式字符串来格式化日期, A,a,B,b,x,Y-b-d等等，其他各种具体格式参考上面的链接
    - `%y`, `%Y`  年，%y表示两位"05", %Y四位"2005"
    - `%m` 数字表示月份, Month name as a decimal 01 to 12, "01" => January
    - `%b`, `%B`,月份名称 %b是简略名"Feb", %B "February"
    - `%d` Day of the month as decimal 01 to 31
    - `%w` 数字表示星期, Weekday as decimal number 0 to 6, "0" => Sunday
    - `%a`, `%A` 星期名称 "Mon" 表示星期一, %A 完整的星期名 "Monday"
    - `%D !` 相当于一个组合，Equivalent to %m/%d/%y
    - 还有一些表示固定的搭配格式
* time_facet
    - `boost::date_time::time_facet` 指定时间方面的IO格式
    - `boost::date_time::time_facet`是`boost::date_time::date_facet`的扩展。
    - `time_facet`在`posix_time`名称空间中被类型定义为`time_facet`和`wtime_facet`
    - 它在`local_time`名称空间中被类型定义为`local_time_facet`和`wlocal_time_facet`
    - `%H:%M:%S%F` 输入时默认格式，Default time_duration format for input. "13:14:15.003400"
    - `%S` 秒"59"
    - `%f` "13:15:16.000000"
    - `%Y-%m-%d %H:%M:%S%F%Q`  "2005-10-15 13:12:11-07:00"
    - `%T !` The time in 24-hour notation (%H:%M:%S)

## Boost单元测试框架

[了解 Boost 单元测试框架](https://www.ibm.com/developerworks/cn/aix/library/au-ctools1_boost/index.html)

Boost 的优势就在于白箱测试：由开发人员编写测试，对类和函数进行语义检查。
这个过程极其重要，因为代码以后的维护者可能会破坏原来的逻辑，这时单元测试就会失败。
通过使用白箱测试，常常很容易找到出错的地方，不必使用调试器。

不需要单独的主函数，代码也不使用任何链接库：作为 Boost 一部分的 unit_test.hpp 头文件中包含所需的所有定义。

[C++（boost）单元测试](https://www.jianshu.com/p/9a87918023fb)

Boost test库提供了一个用于单元测试的基于命令行界面的测试套件UTF:Unit Test Framework，具有单元测试、检测内存泄露、监控程序运行的功能。

* 测试用例是一个包含多个测试断言的函数；是可以被独立执行测试的最小单元。
* 测试套件是测试用例的容器，可以嵌套，包含一个或多个测试用例，将多个测试用例分组管理，共享安装、清理代码。
* 测试夹具（test fixture） 测试安装和测试清理好比c++中的构造函数和析构函数，“测试夹具”实现了自动的测试安装和测试清理。

[Boost Test Library](https://www.boost.org/doc/libs/1_55_0/libs/test/doc/html/index.html)


`BOOST_AUTO_TEST_CASE` 创建测试用例
`BOOST_TEST_MESSAGE`   打印消息

四个断言宏可以使用：
`BOOST_CHECK(predicate)`      // 断言表达式通过，如不通过不影响程序继续执行
`BOOST_REQUIRE(predicate)`    // 断言表达式必须通过，如不通过程序终止
`BOOST_ERROR(message)`        // 给出一个错误信息，程序继续执行
`BOOST_FAIL(message)`         // 给出一个错误信息，程序终止执行

## lexical_cast

`#include <boost/lexical_cast.hpp> `

[boost之lexical_cast简易说明](http://www.habadog.com/2011/05/07/boost-lexical_cast-intro/)

lexical_cast使用统一的接口实现字符串与目标类型之间的转换。

lexical_cast依赖于字符流std::stringstream，其原理相当简单：把源类型读入到字符流中，再写到目标类型中

```cpp
//字符串->数值
int a = lexical_cast<int>("123");
double b = lexical_cast<double>("123.12");

//数值->字符串
const double d = 123.12;
string s = boost::lexical_cast<string>(d);

//如果转换发生了意外，lexical_cast会抛出一个bad_lexical_cast异常，因此程序中需要对其进行捕捉
try
{
    i = boost::lexical_cast<int>("xyz");
}
catch(boost::bad_lexical_cast& e)
{
    cout<<e.what()<<endl;
    return 1;
}
```

* 和C++11中的stou/i/l
    - [Is boost::lexical_cast redundant with c++11 stoi, stof and family?](https://stackoverflow.com/questions/23582089/is-boostlexical-cast-redundant-with-c11-stoi-stof-and-family)
    - 处理更多类型的转换，包括迭代器对、数组、C字符串等
    - 提供相同的通用接口(sto*对不同类型有不同的名称)
    - 是本地语言环境敏感的(`sto*`/`to_string`只是部分的，例如`lexical_cast`可以处理数千个分隔符，而stoul通常不能)

## std::chrono

* [C++11 std::chrono库详解](https://www.cnblogs.com/jwk000/p/3560086.html)
    - chrono是一个time library, 源于boost，现在已经是C++标准，提供高精度的时间控制
    - 要使用chrono库，需要`#include<chrono>`，其所有实现均在std::chrono namespace下
        + 定义了一些辅助类型(通过duration类)表示时间段，直接使用即可(类似于Go里面的Duration)
            * std::chrono::seconds 秒，其定义为`duration</*至少 35 位的有符号整数类型*/>`
            * std::chrono::minutes 分
            * std::chrono::nanoseconds 纳秒、std::chrono::microseconds 微秒、std::chrono::milliseconds 毫秒
* 另可参考
    - Boost.Chrono: [Chapter 7. Boost.Chrono 2.0.8](https://www.boost.org/doc/libs/1_71_0/doc/html/chrono.html)
    - cppreference: [标准库头文件 <chrono>](https://zh.cppreference.com/w/cpp/header/chrono)

* 获取当前的毫秒时间戳

```cpp
#include <chrono>

int64_t getCurrentLocalTimeStamp()
{
    std::chrono::time_point<std::chrono::system_clock, std::chrono::milliseconds> tp =
        std::chrono::time_point_cast<std::chrono::milliseconds>(std::chrono::system_clock::now());
    auto tmp = std::chrono::duration_cast<std::chrono::milliseconds>(tp.time_since_epoch());
    return tmp.count();

    // return std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::system_clock::now().time_since_epoch()).count();
}
```

* 已知毫秒时间戳，想获取年月日时分秒毫秒
    - 假设为`millistamp`，则毫秒为`millistamp%1000`，
    - 其他部分`millistamp/1000`换成秒数后再`struct tm *localtime(const time_t *timep);`获取到`tm`结构，再取对应数据
    - 一直找chrono中的解决方法(还没搞清楚...)，没想到这种。。。