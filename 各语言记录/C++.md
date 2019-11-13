## C++


### const成员函数

参考zh.cppreference.com：
[const、volatile 及引用限定的成员函数](https://zh.cppreference.com/w/cpp/language/member_functions#const.E3.80.81volatile_.E5.8F.8A.E5.BC.95.E7.94.A8.E9.99.90.E5.AE.9A.E7.9A.84.E6.88.90.E5.91.98.E5.87.BD.E6.95.B0)

const、volatile 及引用限定的成员函数

* 非静态成员函数可声明为带有 const、volatile 或 const volatile 限定符（这些限定符出现在函数声明中的形参列表之后）。
* cv 限定性不同的函数具有不同类型，从而可以相互重载。
* 在 cv 限定的函数体内，this 指针被 cv 限定，**const 成员函数中，只能正常地调用其他 const 成员函数。**

```cpp
#include <vector>
struct Array {
    std::vector<int> data;
    Array(int sz) : data(sz) {}
    // const 成员函数
    int operator[](int idx) const {
                          // this 具有类型 const Array*
        return data[idx]; // 变换为 (*this).data[idx];
    }
    // non-const member function
    int& operator[](int idx) {
                          // this 具有类型 Array*
        return data[idx]; // 变换为 (*this).data[idx]
    }
};
int main()
{
    Array a(10);
    a[1] = 1; // OK：a[1] 的类型是 int&
    const Array ca(10);
    ca[1] = 2; // 错误：ca[1] 的类型是 int
}
```

* 单独定义简单类进行编译演示const变量及const成员函数

印证：

1. const 成员函数中，只能正常地调用其他 const 成员函数
2. 定义的const 变量，只能访问非const成员函数

自定义一个类中，成员函数以const在形参列表后修饰：

>const 成员函数中，只能正常地调用其他 const 成员函数

```cpp
#include <iostream>
using namespace std;

class Apple
{
public:
    void print() const;
    void func2();
    void func3() const;
    Apple():color(1){}
private:
    int color;
};

void Apple::print() const
{
    cout << __FUNCTION__ << endl;
    // func2(); //编译报错：丢弃了类型限定 [-fpermissive]
    func3();
    cout << this->color << endl;
}

void Apple::func2()
{
    cout << __FUNCTION__ << endl;
}

void Apple::func3() const
{
    cout << __FUNCTION__ << endl;
}

int main(int argc, const char *argv[])
{
    Apple apple;
    apple.print(); // 以下调用都正常
    apple.func2();
    apple.func3();

    const Apple apple2;
    // apple2.func2();   //编译报错：将‘const Apple’作为‘void Apple::func2()’的‘this’实参时丢弃了类型限定 [-fpermissive]
}
```

```
[➜ /home/xd/workspace/src ]$ ./a.out
print
func3
1
```

放开print()函数中的 `func2()` 注释，编译报错：

```sh
[➜ /home/xd/workspace/src ]$ g++ test_const_func.cpp
test_const_func.cpp: 在成员函数‘void Apple::print() const’中:
test_const_func.cpp:19:8: 错误：将‘const Apple’作为‘void Apple::func2()’的‘this’实参时丢弃了类型限定 [-fpermissive]
```

### 重载、重写

[C++重载重写和多态区别](https://www.cnblogs.com/zhaodun/p/6984479.html)

* 重载overload
    - 在同一个类中，函数名相同，参数列表不同，编译器会根据这些函数的不同参数列表，将同名的函数名称做修饰，从而生成一些不同名称的预处理函数，
    - 未体现多态。
* 重写override
    - 也叫覆盖，子类重新定义父类中有相同名称相同参数的虚函数，
    - 主要是在继承关系中出现的，被重写的函数**必须是virtual**的，
    - 重写函数的访问修饰符可以不同，尽管virtual是private的，子类中重写函数改为public,protected也可以，
    - 体现了多态。
* 重定义redefining
    - 也叫隐藏，子类重新定义父类中有相同名称的非虚函数，参数列表可以相同可以不同，会覆盖其父类的方法，
    - 未体现多态。

### 虚基类、虚函数、纯虚函数

[虚基类、虚函数和纯虚基类](https://blog.csdn.net/lovemysea/article/details/5298589)

* 在继承的类的前面加上virtual关键字表示被继承的类是一个**虚基类**，它的被继承成员在派生类中只保留一个实例
    - 一个类可以在一个类族中既被用作虚基类，也被用作非虚基类

```cpp
class A
{
public:
    int iValue;
};

class B: virtual public A    //注意是virtual public ，virtual 在 public 前面；
{
public:
    void bPrintf(){cout<<"This is class B"<<endl;};
};
```

* 在基类的成员函数前加virtual关键字表示这个函数是一个**虚函数**
    - 所谓虚函数就是在编译的时候不确定要调用哪个函数，而是动态决定将要调用哪个函数
    - 要实现虚函数必须派生类的函数名与基类相同，参数名参数类型等也要与基类相同。
    - 派生类中的virtual关键字可以省略，也表示这是一个虚函数

```cpp
class A
{
public:
    virtual void funPrint(){cout<<"funPrint of class A"<<endl;};
};

class B:public A
{
public:
    virtual void funPrint(){cout<<"funPrint of class B"<<endl;};
};
```

* **纯虚函数**
    - 与其叫纯虚函数还不如叫抽象类,它 只是 声明一个函数但不实现它，让派生类去实现它

```cpp
class Vehicle
{
public:
    virtual void PrintTyre() = 0; //纯虚函数是这样定义的
};

class Camion:public Vehicle
{
public:
    virtual void PrintTyre(){cout<<"Camion tyre four"<<endl;};
};
```

### 抽象类

[C++的抽象类详解](https://blog.csdn.net/lidiya007/article/details/53220786)

在基类中仅仅给出声明，不对虚函数实现定义，而是在派生类中实现。这个虚函数称为**纯虚函数。**
普通函数如果仅仅给出它的声明而没有实现它的函数体，这是编译不过的。纯虚函数没有函数体。
    纯虚函数需要在声明之后加个=0；

```cpp
class <基类名>
{
    virtual <类型><函数名>(<参数表>)=0; ......
};
```

含有纯虚函数的类被称为**抽象类**

抽象类只能作为派生类的基类，不能定义对象，但可以定义指针。

继承于抽象类的派生类如果不能实现基类中所有的纯虚函数，那么这个派生类也就成了抽象类。

### override 和 final

C++11 新特性：显式 override 和 final

#### final

当在虚函数声明或定义中使用时，final 确保函数为虚并指定其不可被派生类覆盖。
当在类定义中使用时，final 指定此类不可在另一类的定义中的 基类说明符列表 中出现（换言之，不能派生于它）。
    当不希望某个类被继承，或不希望某个虚函数被重写，在类名和虚函数后添加final关键字
    添加final关键字后被继承或重写，编译器会报错

```cpp
struct Base
{
    virtual void foo();
};

struct A : Base
{
    void foo() final; // Base::foo 被覆盖而 A::foo 是最终覆盖函数
    void bar() final; // 错误：非虚函数不能被覆盖或是 final
};

struct B final : A // struct B 为 final
{
    void foo() override; // 错误：foo 不能被覆盖，因为它在 A 中是 final
};

struct C : B // 错误：B 为 final
{
};
```

#### override

指定一个虚函数覆盖另一个虚函数。
在成员函数声明或定义中，override 确保该函数为虚函数并覆盖某个基类中的虚函数。
    为了减少程序的运行时错误，养成重写虚函数加上override的习惯
    如果不使用override，当将foo()写成了f00()，结果是编译器并不会报错，因为它并不知道你的目的是重写虚函数，而是把它当成了新的函数

```cpp
struct A
{
    virtual void foo();
    void bar();
};

struct B : A
{
    void foo() const override; // 错误：B::foo 不覆盖 A::foo
                               // （签名不匹配）
    void foo() override; // OK：B::foo 覆盖 A::foo
    void bar() override; // 错误：A::bar 非虚
};
```

### 模板

模板是创建泛型类或函数的蓝图或公式。

### 使用const, enum, inline代替#define

字符串常量定义方法:

```cpp
const char * const STR = "Here is a string."；
const std::string STR("Here is a string.")
```

类中定义
`static const int MAX = 5;`  .h中进行声明

在这种情况，要注意声明和定义的区别，在类中的变量都是声明。对于static变量，如果需要定义，需要在类外定义个变量（不要放在头文件）：
`const int ClassName::MAX;`, cpp文件中进行定义

## vector

### 查找

vector没有find()成员函数，其find是依靠algorithm来实现的

```cpp
#include <algorithm>
vector<int>::iterator it = find(vec.begin(), vec.end(), 6);
```

### 删除

使用`vector.erase(迭代器)`删除成员，如果是循环注意指针陷阱

注意：remove不是vector的成员函数，而是<algorithm>中实现

`remove(v.begin(), v.end(), 99);`

[STL Vector remove()和erase()的使用](https://blog.csdn.net/yockie/article/details/7859330)

remove只是将成员前移，原来的位置的值还在。 e.g. 9 10 8 7, remove 10则变成9 8 7 7

从一个容器中remove元素不会改变容器中元素的个数

如果你真的要删除东西的话，你应该在remove后面接上erase。

v.erase(remove(v.begin(), v.end(), 99), v.end());  // 真的删除所有等于99的元素

把remove的返回值作为erase区间形式第一个参数传递很常见，这是个惯用法。事实上，remove和erase是亲密联盟，这两个整合到list成员函数remove中。这是STL中唯一名叫remove又能从容器中除去元素的函数：

>对于list，调用remove成员函数比应用erase-remove惯用法更高效

```cpp
list<int> li;   // 建立一个list
    // 放一些值进去
li.remove(99);   // 除去所有等于99的元素：
    // 真的删除元素，
    // 所以它的大小可能改变了
```

## list

```cpp
遍历，使用迭代器，不能使用下标
iter = list1.begin(); != end(); iter++

List.push_back(info);  添加到末尾
List.pop_back();       删除末尾元素
List.pop_front();      删除第一个元素

iter = List.insert(iter , info);  插入后iter指向新插入的元素
iter = std::find(List.begin(),List.end(), info);  查找
```

## map

迭代器遍历
erase(iter)前，

```cpp
//错误
for(iter=begin; ...; iter++)
{
    erase(iter)
}
//错误
for(iter=begin; ...; )
{
    erase(iter)
    iter++
}

//正确
for(iter=begin; ...; )
{
    erase(iter++)
}

同:
temp = iter;
iter++;
erase(temp);

//分支
for(vector<int>::iterator iter=veci.begin(); iter!=veci.end(); )
{
    if( *iter == 3)
        iter = veci.erase(iter);
    else
        iter ++;
}
```

### 新增记录

(1)   map的变量名[key] = value;
(2)   PeopleMap.insert(map<int, string>::value_type(111, “zhang wu” ));
(3)   PeopleMap.insert(pair<int, string>(222,"zhang liu"));      //常用的
(4)   PeopleMap.insert(make_pair<int,  string>(222, "zhang liu")); //常用的

## unordered_map

[C++ unordered_map](https://www.jianshu.com/p/56bb01df8ac7)

hash_map ≈ unordered_map
>从 C++ 11 开始，hash_map 实现已被添加到标准库中。但为了防止与已开发的代码存在冲突，决定使用替代名称 unordered_map。这个名字其实更具描述性，因为它暗示了该类元素的无序性。

### 和map比较

新版的hash_map都是unordered_map了，这里只说unordered_map和map.
    运行效率方面：unordered_map最高，而map效率较低但 提供了稳定效率和有序的序列。
    占用内存方面：map内存占用略低，unordered_map内存占用略高,而且是线性成比例的。

需要无序容器，快速查找删除，不担心略高的内存时用unordered_map；有序容器稳定查找删除效率，内存很在意时候用map。

map的内部实现是二叉平衡树(红黑树)；hash_map内部是一个hash_table

### auto

C++11中引入的auto主要有两种用途：自动类型推断和返回值占位。

auto在C++98中的标识临时变量的语义，由于使用极少且多余，在C++11中已被删除。

前后两个标准的auto，完全是两个概念。


## C++11 for 循环新用法

```cpp
// auto
for (auto it = arr.begin(); it != arr.end(); it++)
{
    std::cout << *it << std::endl;
}

// for_each
#include<algorithm>
void func(int n)
{
    std::cout << n << std::endl;
}
std::for_each(arr.begin(), arr.end(), func);

// auto 只读
std::vector<int> arr = {0,1,2,3,4};
for (auto n : arr)    //n类型是int
{
    std::cout << n << std::endl;
}
// 可写 (声明为引用)
for (auto &n : arr)   //n类型是int&，引用，循环内对成员做修改操作不需要像操作指针来解引用*
{
    n = 10;
    std::cout << n << std::endl;
}
```

[【C++11】新特性——auto的使用](https://blog.csdn.net/huang_xw/article/details/8760403)

[基于范围的 for 循环 (C++11 起)](https://zh.cppreference.com/w/cpp/language/range-for)

## string
### stod, stoi

`#include <string>`
[C++字符串转换](https://blog.csdn.net/baidu_34884208/article/details/88342844)
std::stod

to_string

### compare

== 0
> 0
< 0

str1从str1[0]开始，3个字符，与str2整体比较:
`str1.compare(0, 3, str2)`

`"6001234.compare(0, 3, "600")`
`"6001234.compare(0, 3, "60087")`  注意是"600"和"60087"比较

## std::mutex

```cpp
#include <mutex>
```

std::recursive_mutex 可重入，lock/unlock数量匹配

[C++11中std::mutex的使用](https://blog.csdn.net/fengbingchun/article/details/73521630)

std::mutex mtx;
mtx.try_lock()异步
mtx.lock()
mtx.unlock()

### linux下 pthread_mutex_t
在C/C++中（linux下）就需要使用pthread库中提供的互斥锁，并且设置锁的属性为递归锁:

  pthread_mutex_t  Mutex;
  pthread_mutexattr_t attr;
  pthread_mutexattr_init(&attr);
  pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
  pthread_mutex_init(&Mutex, &attr);

## 操作符优先级

[C++操作符的优先级 及其记忆方法](https://www.cnblogs.com/ywl925/p/3710246.html)

() [] -> . :: ++(后置自增) --                                       从左到右结合

! ~ ++(前置自增) -- - + *(解引用) &(取地址) (type)(类型转换) sizeof   从右到左结合

## 精度

[float与double的范围和精度](https://www.cnblogs.com/BradMiller/archive/2010/11/25/1887945.html)

精度
  float和double的精度是由尾数的位数来决定的。浮点数在内存中是按科学计数法来存储的，其*整数部分*始终是一个隐含着的“1”，由于它是不变的，故*不能对精度造成影响*。
  float：2^23 = 8388608，一共七位，这意味着最多能有7位有效数字，但绝对能保证的为6位，也即*float的精度为6~7位有效数字*； 小数位
  double：2^52 = 4503599627370496，一共16位，同理，*double的精度为15~16位*。


## #和##宏定义

    #是“字符串化”的意思
    出现在宏定义中的#是把跟在后面的参数转换成一个字符串
`#define D(a) cout << #a "=[" << a << "]" << endl;`

    “##”是一种分隔连接方式，它的作用是先分隔，然后进行强制连接。
`#define A1(name, type)  type name_##type##_type`
    “name_”、“type”、以及“_type”，这中间只有“type”是在宏前面出现过的，所以它可以被宏替换。
    A1(a1, int);  /* 等价于: int name_int_type; */

## 文件流 fstream

从文件读取流和向文件写入流
要在 C++ 中进行文件处理，必须在 C++ 源代码文件中包含头文件 `<iostream> 和 <fstream>`

    ofstream    该数据类型表示输出文件流，用于创建文件并向文件写入信息。
    ifstream    该数据类型表示输入文件流，用于从文件读取信息。
    fstream 该数据类型通常表示文件流，且同时具有 ofstream 和 ifstream 两种功能，这意味着它可以创建文件，向文件写入信息，从文件读取信息。

### RAII (Resource Acquisition Is Initialization) 资源获取即初始化

cppreference:
[RAII](https://zh.cppreference.com/w/cpp/language/raii)

[C++11实现模板化(通用化)RAII机制](https://blog.csdn.net/10km/article/details/49847271)

RAII(Resource Acquisition Is Initialization)，是一种 C++ 编程技术，它将必须在使用前请求的资源（分配的堆内存、执行线程、打开的套接字、打开的文件、锁定的互斥体、磁盘空间、数据库连接等——任何存在受限供给中的事物）的生命周期绑定与一个对象的生存期相绑定。

根据 RAII 对象的生存期在退出作用域时结束这一基本状况，此技术的另一名称是*作用域界定的资源管理*（ Scope-Bound Resource Management，SBRM）。

RAII 可总结如下:

* 将每个资源封装入一个类，其中
    - 构造函数请求资源，并建立所有类不变式，或在它无法完成时抛出异常，
    - 析构函数释放资源并决不抛出异常；
* 始终经由 RAII 类的实例使用满足要求的资源，该资源
    - 自身拥有自动存储期或临时生存期，或
    - 具有与自动或临时对象的生存期绑定的生存期

拥有 open()/close()、lock()/unlock()，或 init()/copyFrom()/destroy() 成员函数的类是非 RAII 类的典型的例子：

```cpp
std::mutex m;

void bad()
{
    m.lock();                    // 请求互斥体
    f();                         // 若 f() 抛异常，则互斥体永远不被释放
    if(!everything_ok()) return; // 提早返回，互斥体永远不被释放
    m.unlock();                  // 若 bad() 抵达此语句，互斥才被释放
}

void good()
{
    std::lock_guard<std::mutex> lk(m); // RAII类：互斥体的请求即是初始化
    f();                               // 若 f() 抛异常，则释放互斥体
    if(!everything_ok()) return;       // 提早返回，互斥体被释放
}                                      // 若 good() 正常返回，则释放互斥体
```

C++ 标准库：

遵循 RAII 管理其自身的资源：

* std::string、std::vector、std::thread，以及多数其他类在构造函数中获取其资源（错误时抛出异常），并在其析构函数中释放之（决不抛出），而不要求显式清理。

另外，标准库提供几种 RAII 包装器以管理用户提供的资源：

* std::unique_ptr 及 std::shared_ptr 用于管理动态分配的内存，或以用户提供的删除器管理任何以普通指针表示的资源；
* std::lock_guard、std::unique_lock、std::shared_lock 用于管理互斥体。

> RAII 不适用于并非在使用前请求的资源：CPU 时间、核心，以及缓存容量、熵池容量、网络带宽、电力消费、栈内存等。

```cpp
#include <mutex> // class mutex;
std::mutex g_mutexThing;

int doSomething(const string &input, string &output)
{
    int iRetVal = -1;

    do
    {
        g_mutexThing.lock();
        if (!condition1)
        {
            LOGGER_ERROR("error1");
            g_mutexThing.unlock();
            break;
        }
        if (!condition2)
        {
            LOGGER_ERROR("error1");
            g_mutexThing.unlock();
            break;
        }
        g_mutexThing.unlock();

        iRetVal = 0;
    } while (0);

    return iRetVal;
}
```

* 类 lock_guard 是互斥封装器，为在作用域块期间占有互斥提供便利 RAII 风格机制。

```cpp
#include <mutex> // class mutex; class lock_guard;
std::mutex g_mutexThing;

int doSomething2(const string &input, string &output)
{
    int iRetVal = -1;

    do
    {
        std::lock_guard<std::mutex> lockGuard(g_mutexThing);
        if (!condition1)
        {
            LOGGER_ERROR("error1");
            break;
        }
        if (!condition2)
        {
            LOGGER_ERROR("error1");
            break;
        }

        iRetVal = 0;
    } while (0);

    return iRetVal;
}
```

* 若只需要在某个小范围执行加锁，则用{}定义语句块，临时变量lockGuard在语句块结束时则析构销毁，析构中进行解锁。

```cpp
#include <mutex> // class mutex; class lock_guard;
std::mutex g_mutexThing;

int doSomething3(const string &input, string &output)
{
    int iRetVal = -1;

    do
    {
        {
            std::lock_guard<std::mutex> lockGuard(g_mutexThing);
            if (!condition1)
            {
                LOGGER_ERROR("error1");
                break;
            }
        }

        func1();

        iRetVal = 0;
    } while (0);

    return iRetVal;
}
```

### 标准库头文件 <mutex>

cppreference.com:
[标准库头文件 <mutex>](https://zh.cppreference.com/w/cpp/header/mutex)

mutex(互斥量/互斥锁)包含在C++的线程支持库中，C++包含线程、互斥、条件变量和future的内建支持。(C++11开始支持)

C++0x/C++11提供了对thread, mutex, condition_variable这些concurrency相关特性的支持

该标准头文件包含各类型的互斥量的类和函数，截取一部分：

```cpp
namespace std {
    class mutex;                                // 基本互斥锁 (C++11新增)
    class recursive_mutex;                      // 能被同一线程递归锁定的互斥设施(C++11)
    class timed_mutex;                          // 有时限锁定的互斥锁(C++11)
    class recursive_timed_mutex;                // 能被同一线程递归锁定的互斥设施，并实现有时限锁定(C++11)

    template <class Mutex> class lock_guard;    // 严格基于作用域的互斥体所有权包装器(C++11)
    template <class Mutex> class unique_lock;   // 可移动的互斥体所有权包装器(C++11)

    template< class... MutexTypes >class scoped_lock;  // 用于多个互斥体的免死锁 RAII 封装器(C++17)
    //...
}
```

关于读写锁：
C++是从14之后的版本才正式支持共享互斥量，也就是实现读写锁的结构。 <shared_mutex>头文件中。

关于linux下的posix读写锁类型，另外再做介绍。

关于该头文件中各个类型的版本支持，在cppreference.com已经有说明了，也可参考：
[C++雾中风景12:聊聊C++中的Mutex，以及拯救生产力的Boost](https://www.cnblogs.com/happenlee/p/9747743.html)

> 在C++98中没有thread, mutex, condition_variable这些与concurrency相关的特性支持
> [漫话C++0x（五）—- thread, mutex, condition_variable](https://www.cnblogs.com/lidabo/p/3949465.html)

### C++各版本

百度百科：[c++0x](https://baike.baidu.com/item/c%2B%2B0x)

* 1998年是C++标准委员会成立的第一年，以后每5年视实际需要更新一次标准。
* 2009年，C++标准有了一次更新，一般称该草案为C++0x。
* C++0x是C++11标准成为正式标准之前的草案临时名字。
* 后来，2011年，C++新标准标准正式通过，更名为ISO/IEC 14882:2011，简称C++11。

维基百科：[C++11](https://zh.wikipedia.org/wiki/C%2B%2B11#cite_note-1)

* C++11，先前被称作C++0x，即ISO/IEC 14882:2011，是C++编程语言的一个标准。
    - 它取代第二版标准ISO/IEC 14882:2003
    （第一版ISO/IEC 14882:1998公开于1998年，
      第二版于2003年更新，分别通称C++98以及C++03，两者差异很小），且已被C++14取代
* C++14 旨在作为C++11的一个小扩展，主要提供漏洞修复和小的改进。
    - 2014年8月18日，经过C++标准委员投票，C++14标准获得一致通过。ISO/IEC 14882:2014
* C++17 又称C++1z，是继 C++14 之后，C++ 编程语言 ISO/IEC 标准的下一次修订的非正式名称。
    - 官方名称 ISO/IEC 14882:2017
    - 基于 C++ 11，C++ 17 旨在简化该语言的日常使用，使开发者可以更简单地编写和维护代码。
    - C++ 17是对 C++ 语言的重大更新
    - [C++ 17 标准正式发布](https://blog.csdn.net/csdnnews/article/details/78737012)

相比于C++03，C++11标准包含核心语言的新机能，而且扩展C++标准程序库，并入了大部分的C++ Technical Report 1程序库（数学的特殊函数除外）。

关于C++11的版本发布过程...：

>上一个版本的C++国际标准是2003年发布的，所以叫C++ 03。然后C++国际标准委员会在研究C++ 03的下一个版本的时候，一开始计划是07年发布，所以最初这个标准叫C++ 07。但是到06年的时候，官方觉得07年肯定完不成C++ 07，而且官方觉得08年可能也完不成。最后干脆叫C++ 0x。x的意思是不知道到底能在07还是08还是09年完成。结果2010年的时候也没完成，最后在2011年终于完成了C++标准。所以最终定名为C++11。  参考：[c++ 0x和c++ 11是什么关系？0x又是什么意思？](https://www.zhihu.com/question/20141092/answer/21463744)

#### C++11新特性

参考：

[C++11 新特性](https://www.jianshu.com/p/3ac281aa457f)

##### 关键字及新语法

* auto 关键字及用法
    - C++11 之前，auto 具有存储期说明符的语义。auto在C++98中的标识临时变量的语义，由于使用极少且多余，在C++11中已被删除。前后两个标准的auto，完全是两个概念。
* nullptr 关键字及用法
    - 引入nullptr，是因为重载函数处理 NULL 的时候会出问题，二义性

```cpp
    void foo(int);   //(1)
    void foo(void*); //(2)

    foo(NULL);    // 重载决议选择 (1)，但调用者希望是 (2)
    foo(nullptr); // 调用(2)
```

* for 循环语法
    - for ( 范围声明 : 范围表达式 ) 循环语句

##### STL 容器

* std::array
    - std::array 提供了静态数组，编译时确定大小、更轻量、更效率，当然也比 std::vector 有更多局限性。
* std::forward_list
    - 单向链表
* std::unordered_map
* std::unordered_set

##### 多线程

* std::thread
    - 在 C++11 以前，C++ 的多线程编程均需依赖系统或第三方接口实现
    - 一定程度上影响了代码的移植性。
    - C++11 中，引入了 boost 库中多线程的部分内容，形成标准后的接口与 boost 库基本没有变化，这样方便了使用者切换使用 C++ 标准接口。
* std::atomic
    - 从实现上，可以理解为这些原子类型内部自己加了锁。
* std::condition_variable

##### 智能指针内存管理

* std::shared_ptr
* std::weak_ptr

##### 其他

* std::function、std::bind 封装可执行对象
* lambda 表达式
    - lambda 表达式用于定义并创建匿名的函数对象，以简化编程工作。

#### C++11 编译器支持：

参考的知乎问答：

[C++11编译器的支持](https://zhuanlan.zhihu.com/p/27010179)

* 编译器对C++0x和C++11的支持
* GCC编译器对C++11的特性支持
    - codecvt用于编码转换，在GCC 5时引入，在GCC 7（C++17）时废弃。
    - GCC 4.9时正则表达式
    - GCC 4.8时引入了类成员变量函数返回值的左值、右值引用
    - GCC 4.7时正式启用-std=c++11，之前都是使用-std=c++0x
    - GCC 4.6时引入了range based for，即for each。
    - GCC 4.5时引入了lambda表达式，大大方便了函数式编程。
    - stoi/stod和to_string系列函数其实很早就引入了GCC（< 4.5）

参考zh.cppreference.com整理的对于各个标准特性的支持情况(包含C++11,C++14,17等等)：

[C++ 编译器支持情况表](https://zh.cppreference.com/w/cpp/compiler_support#cpp11)

选取GCC中个人目前注意的几个：

* auto, 4.4
    - C++0x/C++11 为 auto 关键字定义了完全不同的语义,4.5开始支持 参考：[GCC 4.5 中的 C++0x 特性支持](https://www.ibm.com/developerworks/cn/aix/library/au-gcc/index.html)
* nullptr, 4.6
* 范围 for 循环, 4.6
    - for ( 范围声明 : 范围表达式 ) 循环语句
* noexcept, 4.6
    - 指定函数是否抛出异常。 `void f() noexcept; // 函数 f() 不抛出`
* override 与 final, 4.7
    - override 指定一个虚函数覆盖另一个虚函数。 [override 说明符](https://zh.cppreference.com/w/cpp/language/override)
    - final 指定某个虚函数不能在子类中被覆盖，或者某个类不能被子类继承。 [final 说明符](https://zh.cppreference.com/w/cpp/language/final)
* decltype 4.8.1
    - 检查实体的声明类型，或表达式的类型和值类别。

### C各版本

参考：

维基百科：[C语言](https://zh.wikipedia.org/wiki/C%E8%AF%AD%E8%A8%80#C99)

* C语言早期
    - 最早由丹尼斯·里奇（Dennis Ritchie）为了在PDP-11电脑上运行的Unix系统所设计出来的编程语言
    - 第一次发展在1969年到1973年之间。
    - 在PDP-11出现后，丹尼斯·里奇与肯·汤普逊着手将Unix移植到PDP-11上
    - 1973年，Unix操作系统的核心正式用C语言改写，这是C语言第一次应用在操作系统的核心编写上。
    - 1975年C语言开始移植到其他机器上使用。史蒂芬·强生实现了一套“可移植编译器”
* K&R C
    - 1978年，丹尼斯·里奇和布莱恩·柯林汉合作出版了《C程序设计语言》的第一版。 “K&R C”（柯里C）。
* C89
    - 1989年，C语言被美国国家标准协会（ANSI）标准化，这个版本又称为C89
    - 标准化的一个目的是扩展K&R C，增加了一些新特性。
* C90
    - 1990年，国际标准化组织（ISO）规定国际标准的C语言
    - 通过对ANSI标准的少量修改，最终制定了 ISO 9899:1990，又称为C90。
    - 随后，ANSI亦接受国际标准C，并不再发展新的C标准。
* C99
    - 1994年为C语言创建了一个新标准，但是只修正了一些C89标准中的细节和增加更多更广的国际字符集支持。
    - 不过，这个标准引出了1999年ISO 9899:1999的发表。它通常被称为C99。
    - C99被ANSI于2000年3月采用。
* C11
    - 2011年12月8日，ISO正式发布了新的C语言的新标准C11，之前被称为C1X
    - 官方名称为ISO/IEC 9899:2011
    - 新的标准提高了对C++的兼容性，并增加了一些新的特性。
    - 这些新特性包括泛型宏、多线程、带边界检查的函数、匿名结构等。
* C18
    - C18没有引入新的语言特性，只对C11进行了补充和修正。


K&R C语言到ANSI/ISO标准C语言 (C89/C90)的改进包括：

* 增加了真正的标准库
* 新的预处理命令与特性
* 函数原型允许在函数申明中指定参数类型
* 一些新的关键字，包括 const、volatile 与 signed
* 宽字符、宽字符串与多字节字符
* 对约定规则、声明和类型检查的许多小改动与澄清

#### C99部分新特性

(只截取了部分本人关注的)：

* 支持不定长的数组，声明时使用 int a[var] 的形式。
* 变量声明不必放在语句块的开头，for 语句提倡写成 for(int i=0;i<100;++i) 的形式
* 允许采用（type_name）{xx,xx,xx} 类似于 C++ 的构造函数的形式构造匿名的结构体。
* 除了已有的 `__line__` `__file__` 以外，增加了 `__func__` 得到当前的函数名。
* 取消了函数返回类型默认为 int 的规定。
* 增加和修改了一些标准头文件(定义bool的<stdbool.h>、定义复数的<complex.h>、<time.h>里增加了 struct tmx，对 struct tm 做了扩展。)

> 但是各个公司对C99的支持所表现出来的兴趣不同。当GCC和其它一些商业编译器支持C99的大部分特性的时候[4]，微软和Borland却似乎对此不感兴趣。


`__FILE__, __LINE__, __FUNCTION__`

#### C11

参考：

维基百科：[C11](https://zh.wikipedia.org/wiki/C11)

* C11：
    - C11（也被称为C1X）指ISO标准ISO/IEC 9899:2011，是当前最新的C语言标准。
    - 在它之前的C语言标准为C99 (1999年ISO 9899:1999)，C99被ANSI于2000年3月采用。
    - 这次修订新增了被主流C语言编译器(如GCC,Clang,Visual C++等)增加的内容
    - 引入了内存模型以更好的执行多线程
    - 之前C99的一些被推迟的计划在C11中增加了，但是对C99仍保留向后兼容。

* 编译器支持
    - GCC从4.6版本开始，已经可以支持一些C11的特性，但多线程相关的库直到2019年还未出现稳定的实现，等于没有编译器可以完整的支持C11
    - GCC 4.6 版本使用参数-std=c1x ，4.7版本以后使用参数-std=c11
    - Clang则是自3.1版开始支持，并在LLVM 3.6版之后默认使用C11的语法
    - 但另一个主流编译器，微软的 Visual Studio 则是自 C99 开始就没有支持新的C语言版本了。

> 虽然 gcc 与 clang 支持C11的语法，却没有实现strcat_s()等边界检查函数以及线程相关<threads.h>库。gcc的支持者狂热的四处宣称这些库是GNU C库的责任而不是gcc的责任——尽管gcc和GNU C库都是GNU项目的子项目。


### explicit说明符

参考：

[explicit 说明符](https://zh.cppreference.com/w/cpp/language/explicit)

* 指定构造函数 或 转换函数(C++11 起)为显式，即它不能用于隐式转换和复制初始化。
    - 声明时不带函数说明符 explicit 的拥有单个无默认值形参的 (C++11 前)构造函数被称作转换构造函数
    - 构造函数（除了复制/移动）和用户定义转换函数都可以是函数模板；explicit 的含义不变。

#### 隐式转换

* 为什么c++需要隐式类型转换
    - c++多态的特性，就是通过父类的对象实现对子类的封装，以父类的类型返回之类对象。
    - c++中使用父类的地方一定可以使用子类代替，这也得益于隐式类型转换。
    - c++是一种强类型的语言，有着非常严格的类型检查，采用隐式类型转换会使程序员更方便快捷一点。
    - 但是在享受方便的时候，风险也紧跟其后。

##### 隐式类型转换和显式类型转换

参考(注意其中Sales_data的示例没有把combine函数实现放进去，只做了声明，编译需要加实现)：

[C++类型转换：隐式类型转换、类类型转换、显式类型转换](https://segmentfault.com/a/1190000016582440)

在C++语言中，类型转换有两种方式，隐式类型转换和显式类型转换。

* 隐式类型转换针对不同的类型有不同的转换方式，总体可以分为两种类型，算术类型和类类型。
    - 算术类型转换的设计原则就是尽可能避免损失精度。
        + 将小整数类型转换成较大的整数类型
        + 有符号类型转换为无符号类型
        + 在条件判断中，非布尔类型自动转换为布尔类型
        + 类类型转换
    - 类类型转换
        + 如果一个类的某个构造函数只接受一个参数，且没有被声明为explicit，则它实际上定义了将这个参数的类型转换为此类类型的隐式转换机制，我们把这种构造函数称为**转换构造函数**。
            * e.g. C类型字符串转换为string类型 `string s = "hello world"`，因为string类有一个构造函数`string(const char *s)`，所以"hello world"字符串可以自动转换为一个string类型的临时变量，然后将这个临时变量的值复制到s中。
            * e.g. class A中只有一个成员变量string b, 则定义变量时，`A(string("abc"))`也是成功的
            * 只有一次的隐式类类型转换是可行的，`A("abc")`时错误的，存在两次隐式转换
        + 如果不想隐式转换，以防用户误操作
            * 可以通过将构造函数声明为explicit来阻止隐式转换。
            * explicit构造函数只能用于直接初始化。不允许编译器执行其它默认操作（比如类型转换，赋值初始化）。
            * **关键字explicit只对一个实参的构造函数有效。**
    - 反向转换(转换函数)
        + 既然基本数据类型通过隐式转换为类，那么也可以做相反的转换，使用operator 类型转换操作符 参考：[C++ 类的隐式转换与explicit](https://blog.csdn.net/wysnkyd/article/details/82712289)，其中"反向转换(转换函数)"


* 显式类型转换
    - C风格的强制转换 `type val = (type)(expression);`
    - C++命名的强制类型转换，C++提供了4个命名的强制类型转换
        + `static_cast<type>(expression);` 很像 C 语言中的旧式类型转换，是非安全的
        + `dynamic_cast` 主要用来在继承体系中的安全向下转型,是实现多态的一种方式。
        + `const_cast` 可去除对象的常量性（const）还可以去除对象的易变性（volatile）
        + `reinterpret_cast` 用来执行低级转型，如将执行一个 int 的指针强转为 int
            * 其转换结果与编译平台息息相关，不具有可移植性，因此在一般的代码中不常见到它。


## 空悬指针 和 野指针

参考：

[空悬指针和野指针](https://blog.csdn.net/zhangxiao93/article/details/51768649)

* 空悬指针(dangling pointer)：指向已经销毁的对象或已经回收的地址。
    - 指向临时变量的指针，脱离临时变量的作用域时
    - free指针后，未将指针置NULL时
    - 函数返回临时变量的地址时

* 野指针(wild pointers)：没有初始化的指针就是野指针
    - 指针定义未初始化时

## 智能指针

参考：

zh.cppreference.com：
[动态内存管理](https://zh.cppreference.com/w/cpp/memory)

cnblogs：
[C++11中智能指针的原理、使用、实现](https://www.cnblogs.com/wxquare/p/4759020.html)

陈硕：
[C++ 工程实践(8)：值语义](https://www.cnblogs.com/Solstice/archive/2011/08/16/2141515.html)

> C++ 要求凡是能放入标准容器的类型必须具有值语义。 值语义/value semantics 和 对象语义/object semantics

> 因此，在写一个 class 的时候，先让它继承 boost::noncopyable，几乎总是正确的。

关于不可拷贝类，参考：[C++ 编写一个不可复制的类](https://blog.csdn.net/flyfish1986/article/details/43305363)，介绍C++11之前和C++11中，以及Boost中的实现使用。 (`### 不可拷贝类`中说明)



## shared_ptr 和 unique_ptr (均C++11 起)

[shared_ptr 和 unique_ptr区别和联系](https://blog.csdn.net/MissXy_/article/details/81029794)

`#include <memory>` 都在该头文件中

### shared_ptr

 shared_ptr 允许多个指针指向同一个对象

```cpp
shared_ptr<string> p1;      //可以指向string
shared_ptr<list<int>> p2;   //可以指向int的list
```

智能指针的使用方式与普通指针类似，解引用一个智能指针返回它指向的对象。

#### shared_ptr 独有操作

make_shared<T> (args)   返回一个shared_ptr，指向一个动态分配的类型为T的对象
shared_ptr<T> p(q)  p是shared_ptr q的拷贝；此操作会增加q中的计数器
p=q p和q都是shared_ptr，所保存的指针必须能相互转换。此操作会递减p的引用计数，递增q的引用计数；若p的引用计数变为0，则将其管理的原内存释放
p.unique()  若p.use_count()为1，返回true；否则返回false
p.use_count()   返回与p 共享对象的智能指针数量；可能很慢，主要用于调试

### unique_ptr
unique_ptr 独占所指向的对象。与shared_ptr不同，某个时刻只能有一个unique_ptr指向一个给定对象。当unique_ptr被销毁时，它所指向的对象也被销毁。

与shared_ptr不同，没有类似make_shared的标准库函数返回一个unique_ptr。所以初始化unique_ptr必须采用直接初始化形式，如下：

```cpp
unique_ptr<double> p1;              //可以指向一个double的unique_ptr
unique_ptr<int> p2(new int(42));    //p2指向一个值为42的int
```

由于unique_ptr拥有它指向的对象，因此unique_ptr不支持普通的拷贝或赋值操作：

```cpp
unique_ptr<string> p1(new string("hello"));
unique_ptr<string> p2(p1);          //错误：unique_ptr不支持拷贝
unique_ptr<string> p3;
p3 = p2;                           //错误：unique_ptr不支持赋值
```

#### unique_ptr 独有操作

unique_ptr<T> u1    空unique_ptr，可以指向类型为T的对象。u1会使用delete来释放它的指针
unique_ptr<T,D> u2  同上。u2会使用一个类型为D的可调用对象来释放它的指针
u=nullptr   释放u指向的对象，将u置为空
u.release() u放弃对指针的控制权，返回指针，并将u置空
u.reset()   释放u指向的对象
u.reset(q)  如果提供了内置指针q，令u指向这个对象；否则将u置空

```cpp
unique_ptr<string> u1(new string("hello ls"));
//将所有权从u1转移给u2;
unique_ptr<string> u2(u1.release());    //release将u1置空
unique_ptr<string> u3(new string("u3"));
//将所有权从u3转移给u2
u2.reset(u3.release());                 //reset释放了u2原来指向的内存
```

### shared_ptr 和 unique_ptr 共有操作

shared_ptr<T> sp    空智能指针，可以指向类型为T的对象
unique_ptr<T> up    同上
*p  解引用p，获得它指向的对象
p->mem  等价于(*p).mem
p.get() 返回p中保存的指针，要小心使用。
swap(p,q)   交换p和q中的指针
p.swap(q)   同上


## 不可拷贝类

参考：

[C++ 编写一个不可复制的类](https://blog.csdn.net/flyfish1986/article/details/43305363)

其中介绍C++11之前和C++11中，以及Boost中的实现使用

>Effective C++:条款06
若不想使用编译器自动生成的函数，就该明确拒绝 .
Explicitly disallow the use of complier-generated functions you do not want.

### C++11前的写法

定义如下基类再进行继承(noncopyable类名可自定义)

```cpp
class noncopyable
{
protected:
    noncopyable() {}
    ~noncopyable() {}
private:
    noncopyable(const noncopyable&);
    noncopyable& operator=(const noncopyable&);
};

class Example:private noncopyable{
    ...
};
```

### C++11的写法

定义类，并使用delete关键字限定 拷贝构造函数 和 拷贝赋值运算符(复制赋值运算符)

```cpp
class Example
{
protected:
    constexpr Example() = default;
    ~Example() = default;
    Example(const Example&) = delete;
    Example& operator=(const Example&) = delete;
};
```

关于类定义时自动产生的几个默认成员函数，以下进行说明

#### 特殊成员函数

参考：

[特殊成员函数](https://www.cnblogs.com/xinxue/p/5503836.html)

* C++98 编译器会隐式的产生四个函数：缺省构造函数，析构函数，拷贝构造函数 和 拷贝赋值运算符，它们称为**特殊成员函数 (special member function)**
* 在 C++11 中，除了上面四个外，特殊成员函数还有两个：移动构造函数 和 移动赋值运算符
    - [复制赋值运算符](https://zh.cppreference.com/w/cpp/language/copy_assignment)
    - [移动赋值运算符](https://zh.cppreference.com/w/cpp/language/move_assignment) (C++11 起)

参考伪代码：

```cpp
class DataOnly {
public:
    DataOnly ()                  // default constructor 缺省构造函数
    ~DataOnly ()                 // destructor 析构函数

    DataOnly (const DataOnly & rhs)            // copy constructor 拷贝构造函数
    DataOnly & operator=(const DataOnly & rhs) // copy assignment operator 拷贝赋值算子/运算符

    DataOnly (const DataOnly && rhs)         // C++11, move constructor 移动构造函数
    DataOnly & operator=(DataOnly && rhs)    // C++11, move assignment operator 移动赋值算子/运算符
};
```

* 右值引用和移动语义，移动构造函数和移动赋值运算符的说明


参考 知乎专栏[Modern C++学习笔记]：

[C++右值引用](https://zhuanlan.zhihu.com/p/54050093)

几个概念：

* 移动语义
    - 将内存的所有权从一个对象转移到另外一个对象，高效的移动用来替换效率低下的复制。
    - 对象的移动语义需要实现移动构造函数（move constructor）和移动赋值运算符（move assignment operator）。

* 左值引用
    - 智能绑定在左值上 (const引用例外: `int const& i = 42;`)
    - 函数入参 `func(int &input)`

* 右值引用
    - C++11标准添加了右值引用(rvalue reference)
    - 这种引用只能绑定右值，不能绑定左值，它使用两个&&来声明

```cpp
int&& i=42;
int j=42;
int&& k=j;  // 编译失败，j不是右值 编译报错："无法将左值‘int’绑定到‘int&&’"
```

```cpp
int x = 20;   // 左值
int&& rx = x * 2;  // x*2的结果是一个右值，rx延长其生命周期
int y = rx + 2;   // 因此你可以重用它：42
rx = 100;         // 一旦你初始化一个右值引用变量，该变量就成为了一个左值，可以被赋值
```

这点很重要，**初始化之后的右值引用将变成一个左值，如果是non-const还可以被赋值！**

```cpp
// 接收左值
void fun(int& lref)
{
    cout << "l-value reference\n";
}
// 接收右值
void fun(int&& rref)
{
    cout << "r-value reference\n";
}

// 其实它不仅可以接收左值，而且可以接收右值（如果你没有提供接收右值引用的重载版本(注意该前提)）
// 如果注释void fun(int&& rref)，fun(10);运行时，也会调用该函数
void fun(const int& clref)
{
    cout << "l-value const reference\n";
}

int main()
{
    int x = 10;
    fun(x);  // output: l-value reference
    fun(10); // output: r-value reference，若注释void fun(int&& rref)，则l-value const reference
    const int y = 10;
    fun(10);  // output: l-value const reference
}
```

* 一旦你已经**自己创建了复制构造函数与复制赋值运算符后**，编译器**不会创建默认的移动构造函数和移动赋值运算符**，这点要注意。
* 最好的话，这个4个函数一旦自己实现一个，就应该养成实现另外3个的习惯。
* 这就是移动语义，用移动而不是复制来避免无必要的资源浪费，从而提升程序的运行效率。
* 其实在C++11中，STL的容器都实现了移动构造函数与移动赋值运算符，这将大大优化STL容器。

* 有时候你需要将一个左值也进行移动语义（因为你已经知道这个左值后面不再使用），那么就必须提供一个机制来将左值转化为右值。
    - std::move就是专为此而生。注意示例中，move之后v1变空了。

```cpp
vector<int> v1{1, 2, 3, 4};
vector<int> v2 = v1;             // 此时调用复制构造函数，v2是v1的副本
vector<int> v3 = std::move(v1);  // 此时调用移动构造函数，v3与v1交换：v1为空，v3为{1, 2, 3, 4}
```

移动构造函数示例(参考[拷贝构造函数和移动构造函数](https://www.jianshu.com/p/f5d48a7f5a52)：

```cpp
#include <iostream>
using namespace std;

class A {
public:
    int x;
    A(int x) : x(x)
    {
        cout << "Constructor" << endl;
    }
    A(A& a) : x(a.x)
    {
        cout << "Copy Constructor" << endl;
    }
    A& operator=(A& a)
    {
        x = a.x;
        cout << "Copy Assignment operator" << endl;
        return *this;
    }
    A(A&& a) : x(a.x)
    {
        cout << "Move Constructor" << endl;
    }
    A& operator=(A&& a)
    {
        x = a.x;
        cout << "Move Assignment operator" << endl;
        return *this;
    }
};

A GetA()
{
    return A(1);
}

A&& MoveA()
{
    return A(1);
}

int main()
{
    cout << "-------------------------1-------------------------" << endl;
    A a(1);
    cout << "-------------------------2-------------------------" << endl;
    A b = a;
    cout << "-------------------------3-------------------------" << endl;
    A c(a);
    cout << "-------------------------4-------------------------" << endl;
    b = a;
    cout << "-------------------------5-------------------------" << endl;
    A d = A(1);
    cout << "-------------------------6-------------------------" << endl;
    A e = std::move(a);
    cout << "-------------------------7-------------------------" << endl;
    A f = GetA();
    cout << "-------------------------8-------------------------" << endl;
    A&& g = MoveA();
    cout << "-------------------------9-------------------------" << endl;
    d = A(1);
}
```

* 编译，添加选项`-fno-elide-constructors`

```sh
[➜ /home/xd/workspace/src ]$ g++ test_mv_constructor.cpp -std=c++11 -fno-elide-constructors
[➜ /home/xd/workspace/src ]$ ./a.out
-------------------------1-------------------------
Constructor
-------------------------2-------------------------
Copy Constructor
-------------------------3-------------------------
Copy Constructor
-------------------------4-------------------------
Copy Assignment operator
-------------------------5-------------------------
Constructor
Move Constructor
-------------------------6-------------------------
Move Constructor
-------------------------7-------------------------
Constructor
Move Constructor
Move Constructor
-------------------------8-------------------------
Constructor
-------------------------9-------------------------
Constructor
Move Assignment operator
```

上面编译添加了编译选项`-fno-elide-constructors`来禁止初始化变量时临时变量优化，而不禁止，5和7被优化了

>-fno-elide-constructors
The C++ standard allows an implementation to omit creating a temporary which is only used to initialize another object of the same type. Specifying this option disables that optimization, and forces G++ to call the copy constructor in all cases.
当被用来初始化另一个相同类型的另外对象时，省略产生临时变量。 可以禁止此项优化，来强制使g++在所有的cases中调用copy constructor。

```sh
[➜ /home/xd/workspace/src ]$ g++ test_mv_constructor.cpp -std=c++11
[➜ /home/xd/workspace/src ]$ ./a.out
-------------------------1-------------------------
Constructor
-------------------------2-------------------------
Copy Constructor
-------------------------3-------------------------
Copy Constructor
-------------------------4-------------------------
Copy Assignment operator
-------------------------5-------------------------
Constructor
-------------------------6-------------------------
Move Constructor
-------------------------7-------------------------
Constructor
-------------------------8-------------------------
Constructor
-------------------------9-------------------------
Constructor
Move Assignment operator
```

### Boost的实现用法

- Boost不仅将两种方法结合，还防止无意识的参数相关查找(protection from unintended ADL)
- ADL(Argument Dependent Lookup) 参考：[ADL（编程用语）](https://baike.baidu.com/item/ADL/18763333)
    + 完全限定名：带完整命名空间的路径标识
    + 限定域 和 无限定域，限定的作用域包含：类域、名字空间域、全局域
    + 也称Koenig查找，当编译器对无限定域的函数调用进行名字查找时，查找函数时，除了当前名字空间域以外，也会把函数参数类型所处的名字空间加入查找的范围

```cpp
namespace boost {
    namespace noncopyable_
    {
        class noncopyable
        {
        };
    }
    typedef noncopyable_::noncopyable noncopyable;
}
```

### 禁止默认构造

[如何禁止C++默认生成成员函数](https://www.cnblogs.com/zhaohuaipeng/p/5323839.html)

c++的默认行为:
当我们创建空类时，c++默认给我们生成了四种成员函数：
    构造函数
    析构函数
    拷贝构造函数(copy)
    重载=的拷贝函数(copy assignment)

```cpp
    因此，当你写下如下的代码：

    class Empty{};
    那么编译器会自动生成：

    class Empty{
    public:
        Empty(){...}                              //default构造函数
        Empty(const Empty& rhs){...}              //copy构造函数
        ~Empty(){...}                             //析构函数
        Empty& operator=(const Empty& rhs){...}   //copy assignment操作符
    };
```

拒绝使用默认的 拷贝构造函数 和 copy assignment操作符

```cpp
    //member函数和友元函数仍然能调用私有成员函数。
    //member成员函数和友元函数调用时，在编译阶段没问题，在链接阶段会报错。
    class HomeForSale{
    public:
        ...
    private:
        HomeForSale(const HomeForSale&);
        HomeForSale& operator=(const HomeForSale&);
    };
```

由于上述做法链接阶段报错：

```cpp
    //为此建立一个基类
    class Uncopyable{
    protected:
        Uncopyable(){}                            //允许derived对象的构造和解析
        ~Uncopyable(){}
    private:
        Uncopyable(const Uncopyable&);            //但阻止copying
        Uncopyable& operator=(const Uncopyable&);
    };

    //为了阻止HomeForSale被拷贝，我们只需继承Uncopyable：
    class HomeForSale:private Uncopyable{
        ...
    };
```

C++11中，当类中含有不能默认初始化的成员变量时，可以禁止默认构造函数的生成，

```cpp
    C++11提出更简单的解决方案：delete。

    class HomeForSale{
    public:
        HomeForSale(const HomeForSale&) = delete;
        HomeForSale& operator=(const HomeForSale&) = delete;
    };
```

### 定义局部变量和new, 带括号和不带括号的区别

参考：

[类名与括号](https://blog.csdn.net/misayaaaaa/article/details/89971212)

**注意： `A a();` 这种形式是函数声明，并没有定义局部变量**

**注意：构造函数的形参和成员变量同名时，函数体内赋值需要`this`或`类::`限定**

```cpp
#include<iostream>
using namespace std;

class A
{
public:
    A()
    {
        cout << "A()" << endl;
    }
    A(int a):a(a)
    {
        // a = a;
        // this->a = a;
        // A::a = a;
        cout << "A(int a)" << endl;
    }
    int a;
};

int main()
{
    //栈上
    //warning C4930 : “A a(void)” : 未调用原型函数(是否是有意用变量定义的 ? )
    A a();//这里声明了一个函数，没有传入的参数，返回值为类类型
    cout << "===========" << endl;
    A b;//默认调用“对象名()”这个构造函数构造对象
    //xd运行结果并没有初始化为0
    cout << "=====A b;======" << b.a << endl;
    A c(1);//默认调用相应的构造函数构造对象
    cout << "=====A c(1);======" << c.a << endl;

    //堆上,加括号不加括号无差别，都调用默认的构造函数
    A *d = new A();
    cout << "=====new A();=====" << d->a << endl;
    A *e = new A;
    cout << "=====new A;=====" << e->a << endl;

    //对于内置类型而言,加括号是进行了初始化，不加是未进行初始化
    //xd运行结果都是初始化了的，new int[3000]结果也一样，所以不明确是否为编译器优化了
    int *f = new int();
    int *g = new int;

    cout << *f << endl;
    cout << *g << endl;
    return 0;
}
```

```
[➜ /home/xd/workspace/src ]$ ./a.out
===========
A()
=====A b;======2           //注意此处并没有初始化为0
A(int a)
=====A c(1);======1
A()
=====new A();=====0
A()
=====new A;=====0
0
0
```

### std::numeric_limits

* 查询各种算术类型属性的标准化方式（例如 int 类型的最大可能值是 std::numeric_limits<int>::max() ）
    - cppreference：[std::numeric_limits](https://zh.cppreference.com/w/cpp/types/numeric_limits)

* 在库编译平台提供基础算术类型的极值(数值极限，最大值、最小值)等属性信息
    - [C++/C++11中std::numeric_limits的使用](https://blog.csdn.net/fengbingchun/article/details/77922558)
    - 比较常用的使用是对于给定的基础类型用来判断在当前系统上的最大值、最小值。若使用此类，需包含<limits>头文件。
    - 数学上极值的定义：若函数f(x)在x₀的一个邻域D有定义，且对D中除x₀的所有点，都有f(x)<f(x₀)，则称f(x₀)是函数f(x)的一个极大值；若都有f(x)>f(x₀)，则称极小值。

和C语言中`CHAR_MIN`、`CHAR_MAX`、`INT_MAX`等宏定义效果一样，表示 char类型或其他数值类型的数值最小和最大值

### std::distance

[std::distance](https://zh.cppreference.com/w/cpp/iterator/distance)

定义于头文件 <iterator>，返回两个迭代器间的距离

```
template< class InputIt >
typename std::iterator_traits<InputIt>::difference_type
    distance( InputIt first, InputIt last );
```

返回从 first 到 last 的路程

```cpp
std::vector<int> v{ 3, 1, 4 };
std::distance(v.begin(), v.end())  // 3，end()指向最后一个元素的下一个位置
std::distance(v.end(), v.begin())  // -3
```

### std::advance

std::advance用来对迭代器做偏移操作

// std::advance用来对迭代器做偏移操作, 相当于splitIter=splitIter+splitPos
std::advance(splitIter,splitPos);

### 输入输出流 iostream

imbue

`std::locale imbue( const std::locale& loc );`

设置本地环境
设置流的关联本地环境为给定值。

std::locale 定义于头文件 <locale>

C++ 输入/输出库的每个流对象与一个 std::locale 对象关联，并用其平面分析及格式化所有数据。

### stringstream

#### std::ostringstream

```cpp
const boost::posix_time::ptime dateToFormat;

time_facet *facet = new time_facet("%Y-%m-%d %H:%M:%S");
std::ostringstream oss;
oss.imbue(std::locale(oss.getloc(), facet));

oss << dateToFormat;

return oss.str();
```

#### std::stringstream

```cpp
std::stringstream ssstarttime;
ssstarttime << dateToFormat;
std::string sTime = ssstarttime.str();
```


### 重载()

#### functor 仿函数
[深入理解仿函数(functor或function object)](https://blog.csdn.net/kezunhai/article/details/38514099)

* functor 仿函数
    - 仿函数(functor)又称之为函数对象(function object)，其实就是重载了operator()操作符的struct或class
    - 仿函数(functor)使一个类的使用看上去象一个函数，这个类就有了类似函数的行为，就是一个仿函数类了
    - 仿函数比一般的函数灵活
    - 仿函数有类型识别，可以作为模板参数
    - 执行速度上仿函数比函数和指针要更快的。

```cpp
struct IntLess
{
    bool operator()(int _left, int _right) const
    {
        return _left<_right;
    }
};
```

除了在stl里，别的地方你很少会看到仿函数的身影。而在stl里仿函数最常用的就是作为函数的参数，或者模板的参数。

#### list unique()

从容器移除所有相邻的重复元素。只留下相等元素组中的第一个元素。

如果要去重所有的：  
在使用unique之前(不论是list的unique还是泛型的unique)，先对容器内的元素进行排序，因为unique()是比较相邻的元素. 去掉相邻元素中重复的

类似linux中的shell命令 uniq，如果不排序，那么还是可能有重复的元素，uniq只移除相邻的重复
e.g. `a a b a a`，uniq之后是 `a b a`

