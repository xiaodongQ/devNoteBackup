## boost

官网：
https://www.boost.org/

文档：
[Boost 1.71.0 Library Documentation](https://www.boost.org/doc/libs/1_71_0/)

10个Boost库被包含在c++标准委员会的库技术报告(TR1)和新的c++ 11标准中。除了来自TR1的Boost库之外，c++ 11还包含了其他几个Boost库。在c++ 17中提出了更多的Boost库以实现标准化。

>Ten Boost libraries are included in the C++ Standards Committee's Library Technical Report (TR1) and in the new C++11 Standard. C++11 also includes several more Boost libraries in addition to those from TR1. More Boost libraries are proposed for standardization in C++17.

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

std::string to_simple_string(ptime); // 2002-Jan-01 10:00:01.123456789
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

[Date Time IO System](https://www.boost.org/doc/libs/1_71_0/doc/html/date_time/date_time_io.html)

* Time Facet: `boost::date_time::time_facet`
    - `boost::date_time::time_facet`是`boost::date_time::date_facet`的扩展。
    - `time_facet`在`posix_time`名称空间中被类型定义为`time_facet`和`wtime_facet`
    - 它在`local_time`名称空间中被类型定义为`local_time_facet`和`wlocal_time_facet`

* date_facet
    - [Class template date_facet](https://www.boost.org/doc/libs/1_71_0/doc/html/boost/date_time/date_facet.html)
    - `boost::date_time::date_facet` 该类为日期类型提供基于I/O方面的格式
    - 这个类允许使用格式字符串来格式化日期, A,a,B,b,x,Y-b-d