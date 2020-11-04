[TOC]

## C++

* 设置vim语法支持C++11
    - `let g:syntastic_cpp_compiler_options = ' -std=c++11 -stdlib=libc++'`
* 迭代器循环中，谨记continue时对迭代器进行更新！

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

>对于传入的const容器，可以定义const迭代器进行遍历：const_iterator

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

搜索查看本笔记中的章节：`## 模板`

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

* 对于vector/list
    - front() 第一个元素的const_reference(const引用，非指针)
    - back() 最后一个元素

### 删除

使用`vector.erase(迭代器)`删除成员，如果是循环注意指针陷阱

* vector可使用`iter = veci.erase(iter);`方式循环删除成员，而不能用`veci.erase(iter++)`方式(list、map可以)
    - [STL的erase()陷阱-迭代器失效总结](https://www.cnblogs.com/blueoverflow/p/4923523.html#_lab2_0_0)
    - 对于关联容器(如`map`, `set`, `multimap`, `multiset`)，删除当前的iterator，仅仅会使当前的iterator失效，只要在`erase`时，递增当前iterator即可
        + 这是因为map之类的容器，使用了红黑树来实现，插入、删除一个结点不会对其他结点造成影响
    - 对于序列式容器(如vector,deque)，删除当前的iterator会使后面所有元素的iterator都失效。
        + 这是因为`vetor`,`deque`使用了连续分配的内存，删除一个元素导致后面所有的元素会向前移动一个位置。
        + 还好`erase`方法可以返回下一个有效的iterator
    - 对于`list`来说，它使用了不连续分配的内存，并且它的erase方法也会返回下一个有效的iterator，因此两种方法都可使用


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

### assign 赋值

* vector assign()
    - 用法1: `void assign(const_iterator first,const_iterator last);` 把first到last的成员都赋值给调用者，[first, last)
        + e.g. `v2.assign(v1.begin(),v1.begin()+9);` 下标[0, 8]的成员, (即[0,9))
        + 或用`std::advance`来偏移迭代器
    - 用法2: `void assign(size_type n,const T& x = T());` n个成员都赋值为x
        + e.g. `v2.assign(10,7);` 10个成员，都赋值为7

### 性能优化

[C++ vector 性能优化](https://blog.csdn.net/ShellDawn/article/details/87906632)

* 若提前知道空间大小，提前分配会提高性能
    - `vector<int> v; v.reserve(10000);` reserve预留空间，但不创建对象
        + 区别于`resize()`，resize()会创建对象。
    - 对于vector和string，在需要更多空间时，会做与realloc等效的事情
        + 分配一个新内存块，其容量时当前容量的数倍。多数实现中，vector和string容量提升因子在1.5至2之间
        + 将原内存中数据拷贝至新内存(很费时间)
        + 释放原有内存对象
        + 释放原有内存
* 彻底释放vector内存时，使用`swap()`，而不是clear()或erase():
    - `vector<int>().swap(v); printf("%lu\n", v.capacity());` 容量会变为0
    - `v.clear()` 容量并不会改变
* vector填充时，赋值(=/assignment)最快，insert()次之，push_back()最慢。
    - assignment知道要拷贝的 vector 有多大，然后只需要通过内存管理一次性拷贝 vector 内部的缓存
* 遍历vector时，下标最快
* 避免在vector前部插入元素
    - 避免：`vector<int> v; v.insert(v.begin(), 1);`
    - 避免：`list<int> li; li.push_front(1);`

* 在向 vector 插入元素的时候使用 emplace_back() 而不是 push_back() ?

* [C++性能优化技术导论](https://blog.csdn.net/iteye_15898/article/details/82236319)
    - 为了精确的计算一段代码的耗时，我们需要极高精度的时间函数。`gettimeofday`是其中一个不错的选择，它的精度在1us，每秒可以调用几十万次
        + 1us内cpu可以处理几千甚至上万条指令。对于代码长度少于百行的函数来说，其单次执行时间很可能小于1us
        + 目前最精确的计时方式是cpu自己提供的指令：`rdtsc`。它可以精确到一个时钟周期（1条指令需要消耗cpu几个时钟周期）
    - 系统在调度程序的时候，可能会把程序放到不同的cpu核心上面运行，而每个cpu核心上面运行的周期不同，从而导致了采用rdtsc时，计算的结果不正确。解决方案是调用linux系统的`sched_setaffinity`来强制进程只在固定的cpu核心上运行
    - 除了基本的计算执行次数和时间外，还有如下几种分析性能的策略：
        + 基于概率
            * 通过不断的中断程序，查看程序中断的位置所在的函数，出现次数最多的函数即为耗时最严重的函数
        + 基于事件
            * 当发生一次cpu硬件事件的时候，某些cup会通知进程。如果事件包括L1失效多少次这种，我们就能知道程序跑的慢的原因
        + 避免干扰
            * 性能测试最忌讳外界干扰。比如，内存不足，读内存变成了磁盘操作
    - 性能分析工具-`callgrind`
        + valgrind系列工具因为免费，所以在linux系统上面最常见。
        + callgrind是valgrind工具里面的一员，它的主要功能是模拟cpu的cache，能够计算多级cache的有效、失效次数，以及每个函数的调用耗时统计
            * callgrind的实现机理（基于外部中断）决定了它有不少缺点。比如，会导致程序严重变慢、不支持高度优化的程序、耗时统计的结果误差较大等等。
    - `gprof`是g++自带的性能分析工具（gnuprofile）。它通过内嵌代码到各个函数里面来计算函数耗时。按理说它应该对高度优化代码很有效，但实际上它对-O2的代码并不友好，这个可能和它的实现位置有关系（在代码优化之后）

## list

* 删除成员
    - `clist.erase(iter)` 迭代器
    - `clist.erase(clist.begin(),clist.end());` 删除所有
    - `clist.clear()` 从容器擦除所有元素。此调用后 size() 返回零

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

* 删除成员
    - `cmap.erase("bfff");` key
    - `cmap.erase(iter);` 迭代器
    - `cmap.erase(cmap.begin(),cmap.end());` 删除所有
    - `cmap.clear()`从容器擦除所有元素。此调用后 size() 返回零

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
    // 注意list,set,map可用该方式，vector、deque不行！
    // (vector可使用iter = veci.erase(iter);方式)
    // [STL的erase()陷阱-迭代器失效总结](https://www.cnblogs.com/blueoverflow/p/4923523.html#_lab2_0_0)
    // 对于关联容器(如map, set, multimap,multiset)，删除当前的iterator，仅仅会使当前的iterator失效，只要在erase时，递增当前iterator即可。这是因为map之类的容器，使用了红黑树来实现，插入、删除一个结点不会对其他结点造成影响
    // 对于序列式容器(如vector,deque)，删除当前的iterator会使后面所有元素的iterator都失效。这是因为vetor,deque使用了连续分配的内存，删除一个元素导致后面所有的元素会向前移动一个位置。还好erase方法可以返回下一个有效的iterator
    // 对于list来说，它使用了不连续分配的内存，并且它的erase方法也会返回下一个有效的iterator，因此两种方法都可使用
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

## STL 相关算法

* `std::find`
    - [std::find, std::find_if, std::find_if_not](https://zh.cppreference.com/w/cpp/algorithm/find)
    - 定义于头文件 `<algorithm>`
    - `find`
        + `template< class InputIt, class T >`
        + `InputIt find( InputIt first, InputIt last, const T& value );` (C++20前)
        + 返回指向首个满足条件(此处条件为等于value)的迭代器，或若找不到这种元素则为 last
            * e.g. `vector<string> vec;`, `auto iter = find(vec.begin(), vec.end(), "c");`
    - `find_if` / `find_if_not`
        + `template< class InputIt, class UnaryPredicate >`
        + `InputIt find_if( InputIt first, InputIt last, UnaryPredicate p );`
        + 返回指向首个满足条件的迭代器，或若找不到这种元素则为 last
            * 此处条件为谓词p，其返回值为一个bool类型的值
            * predicate翻译成了谓词，可以有多种形式
                - 函数(function)谓词
                    + e.g. `bool isLarger (const std::string &s1, const std::string &s2) {return s1.size() > s2.size();}`
                    + `std::stable_sort(sv.begin(), sv.end(), isLarger);`
                - 函数指针(function pointer)谓词
                    + 建立一个函数指针, 传入算法, 使用指针代替函数名, 用法类似函数谓词
                    + e.g. `bool (*pf) (const std::string &s1, const std::string &s2);`
                    + `pf = &isLarger;`
                    + `std::stable_sort(sv.begin(), sv.end(), *pf);`
                - Lambda表达式(lambda expression)谓词
                    + Lambda表达式格式: `[capture list] (parameter list) -> return type { function body }`
                    + e.g. `std::stable_sort(sv.begin(), sv.end(), [](const std::string& s1, const std::string& s2){ return s1.size()>s2.size(); });`
                - 函数对象(Function Object)谓词
                    + 类中重载函数的调用`()`, 使类可以被调用, 并且传入算法谓词中, 进行使用
                    + `bool operator() (const std::string& a, const std::string& b) { return a.size() > b.size();}`
                - 库定义的函数对象(Library-Defined Function Object)谓词
                    + 使用标准库定义的函数对象, 充当算法中的谓词, 包含在`#include<functional>`,包含基本的算法和逻辑操作
                    + e.g. `std::stable_sort(sv.begin(), sv.end(), std::less<std::string>());`
                - [C++ - 算法(algorithm) 的 谓词(predicate) 详解](https://blog.csdn.net/caroline_wendy/article/details/15378055)

* `std::copy`
    - [std::copy, std::copy_if](https://zh.cppreference.com/w/cpp/algorithm/copy)
    - 定义于头文件 `<algorithm>`
    - `copy`
        + `template< class InputIt, class OutputIt >`
        + `OutputIt copy( InputIt first, InputIt last, OutputIt d_first );`
        + 复制 [first, last) 所定义的范围中的元素到始于 d_first 的另一范围
* `insert`
    - vector
        + 在 pos 前插入 value
        + iterator insert( iterator pos, const T& value ); (C++11 前)
        + iterator insert( const_iterator pos, const T& value ); (C++11 起)
        + iterator insert( const_iterator pos, InputIt first, InputIt last ); (`template< class InputIt >`)

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

// auto 只读 (注意若遍历map要修改内容，也需要用`auto &`)
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

* `std::string`类型的定义为：`std::basic_string<char>`，模板类，模板形参为`char`
    - [std::basic_string](https://zh.cppreference.com/w/cpp/string/basic_string)
    - `string`构造函数：
        + 通过`char`构造string：`string (size_t n, char c);`
            * e.g. 通过'0'定义"0"：`string(1, '0')`
            * 也可以先定义再push_back：`string s1; s1.push_back('0')`，`char`的容器
    - `std::basic_string` 满足 具分配器容器 (AllocatorAwareContainer) 、序列容器 (SequenceContainer) 及连续容器 (ContiguousContainer) (C++17 起)的要求
* stod, stoi, to_string
    - `#include <string>`
    - `std::stod`
    - `to_string`
    - [C++字符串转换](https://blog.csdn.net/baidu_34884208/article/details/88342844)
* compare
    - `== 0`、`> 0`、`< 0`
    - str1从str1[0]开始，3个字符，与str2整体比较:
        + `str1.compare(0, 3, str2)`
    - e.g. `"6001234.compare(0, 3, "600")`
    - e.g. `"6001234.compare(0, 3, "60087")`  注意是"600"和"60087"比较
* 截取/查找/替换字符串
    - 截取子串
        + `s.substr(pos, n)` 截取s中从pos开始（包括0）的n个字符的子串，并返回
        + `s.substr(pos)` 截取s中从从pos开始（包括0）到末尾的所有字符的子串，并返回
    - 替换子串
        + `s.replace(pos, n, s1)` 用s1替换s中从pos开始（包括0）的n个字符的子串
    - 查找子串
        + `s.find(s1)` 查找s中第一次出现s1的位置，并返回（包括0）
            * 若找不到子串则返回为 `std::string::npos`
            * `static const size_type npos = -1;` 其值为-1，
                - 虽然定义使用 `-1` ，由于有符号到无符号隐式转换，且 `size_type`是无符号整数类型，`npos`的值是其所能保有的最大正值。
                - 这是指定任何无符号类型的最大值的可移植方式。
        + `s.rfind(s1)` 最后一次
    - [C++基础-string截取、替换、查找子串函数](https://www.cnblogs.com/catgatp/p/6407788.html)

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

* 相关编译错误：使用了被删除的函数‘std::mutex& std::mutex::operator=(const std::mutex&)’
    - 定义了类的map，其中类成员包含一个std::mutex成员
    - 由于std::mutex的拷贝构造函数和赋值运算符被禁用了： `mutex( const mutex& ) = delete;`
    - `std::mutex& std::mutex::operator=(const std::mutex&) = delete`
    - 而 std::vector 和 std::map 都是要求 类型 必须包含拷贝构造函数，所以就报错了。
    - 可以把 `std::mutex _mutex` 改成 `std::shared_ptr<std::mutex> _mutex`，使用时`std::lock_guard<std::mutex> _lock{*解引用};`
    - [std::mutex 引起的 C2280 尝试引用已删除的函数](https://www.cnblogs.com/lzpong/p/10138872.html)

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

### 输入输出流 iostream

* imbue
    - 设置本地环境，设置流的关联本地环境为给定值。
    - `std::locale imbue( const std::locale& loc );`
    - std::locale 定义于头文件 <locale>
    - C++ 输入/输出库的每个流对象与一个 std::locale 对象关联，并用其平面分析及格式化所有数据。
    - 设置输出格式示例：

```cpp
    const boost::posix_time::ptime dateToFormat;

    time_facet *facet = new time_facet("%Y-%m-%d %H:%M:%S");
    std::ostringstream oss;
    oss.imbue(std::locale(oss.getloc(), facet));

    oss << dateToFormat;

    return oss.str();
```

* std::stringstream 示例

```cpp
    std::stringstream ssstarttime;
    ssstarttime << dateToFormat;
    std::string sTime = ssstarttime.str();
```

## 输入输出流

* [C++ 流（stream）总结](https://blog.csdn.net/luguifang2011/article/details/40979231)
    - 使用流插入运算符`<<`向文件写入信息，使用流提取运算符`>>`从文件读取信息
    - 标准I/O流：内存与标准输入输出设备之间信息的传递
        + `istream`、`ostream`、`iostream`
            * `std::cin` 是 `istream` 类的对象(声明在 `<iostream>` 头文件中)
                - e.g. `cin >> str1` 从标准输入中读取string到str1中
                - 会忽略开头所有的空白字符（空格、换行、制表符等）
                - 读取字符直至再次遇到空白字符，读取终止
            * `std::cout` 是 `ostream` 类的对象
        + `getline(istream, string)` 读取整行文本
            * 不忽略开头的空白字符
            * 读取字符直至遇到换行符，如果第一个字符是换行符，则返回空string
            * 返回时丢弃换行符，换行符不存储在string中
            * e.g. `getline(cin,s);` 从`std::cin`标准输入读取数据到s(根据s的类型自动推导)
    - 文件I/O流：内存与外部文件之间信息的传递
        + `ifstream`、`ofstream`、`fstream` (`<fstream>`头文件)
    - 字符串I/O流：内存变量与表示字符串流的字符数组之间信息的传递
        + `istringstream`、`ostringstream`、`stringstream` (`<sstream>`头文件)
        + 经常用作格式转换，使用`stringstream`对象简化类型转换
            * 若想用`sprintf()`函数将一个变量从int类型转换到字符串类型。
                - 为了正确地完成这个任务，你必须确保证目标缓冲区有足够大空间以容纳转换完的字符串。
                - 此外，还必须使用正确的格式化符。如果使用了不正确的格式化符，会导致非预知的后果
                - 错误占位符示例：
                    + `int n=10000;`, `char s[10];`, `sprintf(s,"%f",n)` 会导致程序崩溃
            * 相同示例对于`stringstream`
                - 由于n和s的类型在编译期就确定了，所以编译器拥有足够的信息来判断需要哪些转换
                - `<sstream>`使用`string`对象来代替字符数组。这样可以避免缓冲区溢出的危险。而且，传入参数和目标对象的类型被自动推导出来，即使使用了不正确的格式化符也没有危险
                - 示例：
                    + `stringstream ss;`
                    + `string result="10000";`, `int n=0;`, `ss<<result;`(写入到ss), `ss>>n`(从ss读取到n中)
            * 如果打算在多次转换中使用同一个`stringstream`对象，记住再每次转换前要使用`clear()`方法
                - 在多次转换中重复使用同一个`stringstream`（而不是每次都创建一个新的对象）对象最大的好处在于效率。`stringstream`对象的构造和析构函数通常是非常耗费CPU时间的
            * 配合模板可以定义一个通用的转换模板，用于任意类型之间的转换，如下面两个示例

```cpp
template<class T>
void to_string1(string & result,const T& t)
{
    ostringstream oss;//创建一个流
    oss<<t;//把值传递如流中
    result=oss.str();//获取转换后的字符转并将其写入result
}

```

* 利用`stringstream`进行通用类型转换

```cpp
template<class out_type,class in_value>
out_type convert(const in_value & t)
{
    stringstream stream;
    stream<<t;//向流中传值
    out_type result;//这里存储转换结果
    stream>>result;//向result中写入值
    return result;
}

// 使用
    double d;
    string s = "12.56";
    d = convert<double>(s);//d等于12.56
    string salary=convert<string>(9000.0); //salary等于"9000"
```


* [C++ 文件和流](https://www.w3cschool.cn/cpp/cpp-files-streams.html)
    - 用于从文件读取流和向文件写入流
    - 要在 C++ 中进行文件处理，必须在 C++ 源代码文件中包含头文件 `<iostream> 和 <fstream>`
    - `ofstream`
        + 该数据类型表示输出文件流，用于创建文件并向文件写入信息
        + 注意这里的`o`和`i`，自己老是容易搞混淆是读取还是写入，速记：`o`输出到指定流，即写，`i`流的输入来源，即读
    - `ifstream`
        + 该数据类型表示输入文件流，用于从文件读取信息
    - `fstream`
        + 该数据类型通常表示文件流，且同时具有 ofstream 和 ifstream 两种功能，这意味着它可以创建文件，向文件写入信息，从文件读取信息
* 操作
    - 打开`open()`
        + 在从文件读取信息或者向文件写入信息之前，必须先打开文件
        + `ofstream` 和 `fstream` 对象都可以用来打开文件进行*写*操作
        + 如果只需要打开文件进行*读*操作，则使用 `ifstream` 对象
        + `void open(const char *filename, ios::openmode mode);`
            * `open()` 函数是 fstream、ifstream 和 ofstream 对象的一个成员
                - (指的是这些流的成员函数，不是指系统函数，`xstream.open(xxx)`的方式调用)
            * 第二个参数`mode`是打开模式
                - `ios::app` 追加模式
                - `ios::ate` 文件打开后定位到文件末尾
                - `ios::in` 打开文件用于读取
                - `ios::out` 打开文件用于写入
                - `ios::trunc` 截断
            * 可以把以上两种或两种以上的模式结合使用
                - 用于写：`outfile.open("file.dat", ios::out | ios::trunc );` (`ofstream outfile;`)
                - 用于读写：`afile.open("file.dat", ios::out | ios::in );` (`fstream afile;`)
    - 关闭`close()`
    - 写入、读取
        + 使用流插入运算符`<<`向文件写入信息
            * 示例
                - `ofstream outfile;` 创建ofstream并以写模式(ofstream默认)打开文件
                - `outfile.open("afile.dat");`
                - `outfile << data << endl;` 将data写入文件(`char data[100];`)
                - `outfile.close();` 关闭文件
        + 使用流提取运算符`>>`从文件读取信息
            * 示例
                - `ifstream infile;` 以读模式打开文件
                - `infile.open("afile.dat");`
                - `infile >> data;` 读取内容到data中(`char data[100];`)
                - `infile.close();` 关闭文件
    - 文件位置指针
        + 文件位置指针是一个整数值，指定了从文件的起始位置到指针所在位置的字节数
        + `istream` 和`ostream` 都提供了用于重新定位文件位置指针的成员函数
            * `istream` 的 `seekg` ("seek get")
            * `ostream` 的 `seekp` ("seek put")
        + 参数
            * `seekg`和`seekp`的参数通常是一个长整型
            * 第二个参数可以用于指定查找方向
                - `ios::beg` 默认的，从流的开头开始定位
                - `ios::cur` 从流的当前位置开始定位
                - `ios::end` 从流的末尾开始定位
        + e.g.
            * `fileObject.seekg(n);` 从流的开头起，向后n个字节(默认`ios::beg`)
            * `fileObject.seekg(n, ios::cur);` 当前位置向后移动n字节
            * `fileObject.seekg(n, ios::end);` 从fileObject末尾往回移 n 个字节
            * `fileObject.seekg(0, ios::end);` 定位到末尾

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

* `std::unique_ptr` 及 `std::shared_ptr` 用于管理动态分配的内存，或以用户提供的删除器管理任何以普通指针表示的资源；
* `std::lock_guard`、`std::unique_lock`、`std::shared_lock` 用于管理互斥体。

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

* `std::unique_lock`
    - 参考：[C++11 并发指南三(Lock 详解)](https://www.cnblogs.com/haippy/p/3346477.html)
        + C++11 标准为我们提供了两种基本的锁类型
            * `std::lock_guard`，与 Mutex RAII 相关，方便线程对互斥量上锁。
            * `std::unique_lock`，与 Mutex RAII 相关，方便线程对互斥量上锁，但提供了`更好的上锁和解锁控制`。
                - 类 `unique_lock` 是通用互斥包装器，允许延迟锁定、锁定的有时限尝试、递归锁定、所有权转移和与条件变量一同使用。
        * 构造(可以选择以哪种方式锁定提供的互斥锁m)
            - 默认构造函数 `unique_lock() noexcept;`
                + 新创建的 unique_lock 对象不管理任何 Mutex 对象。
            - 单参数构造(此处传的是锁，不是拷贝构造) `explicit unique_lock( mutex_type& m );`
                + 和lock_gurad方式一样，创建且调用m.lock()
            - `unique_lock( mutex_type& m, std::try_to_lock_t t );`
                + 尝试加锁，m.try_lock()不成功也不阻塞
            - `unique_lock( mutex_type& m, std::defer_lock_t t ) noexcept;`
                + 延迟加锁，初始化后并不调用lock
            - `unique_lock( mutex_type& m, std::adopt_lock_t t );`
                + 当前线程已经获得锁，创建unique_lock对象来管理m，创建后的对象接手锁的所有权
            - 另外几个的构造函数说明
                + `unique_lock(mutex_type& m, const chrono::duration<Rep,Period>& rel_time);`
                    * 创建并试图通过调用 `m.try_lock_for(rel_time)` 来锁住 Mutex 对象一段时间
                + `unique_lock(mutex_type& m, const chrono::time_point<Clock,Duration>& abs_time);`
                    * 创建并试图通过调用 `m.try_lock_until(abs_time)` 来在某个时间点(abs_time)之前锁住 Mutex 对象
                + 拷贝构造被禁用，`unique_lock(const unique_lock&) = delete;`
                + 移动(move)构造 `unique_lock(unique_lock&& x);`
                    * 新创建的 unique_lock 对象获得了由 x 所管理的 Mutex 对象的所有权(包括当前 Mutex 的状态)。调用 `move` 构造之后， x 对象如同通过默认构造函数所创建的，就不再管理任何 Mutex 对象了
        * 操作函数
            - `lock()`              //阻塞等待加锁
            - `try_lock()`          //非阻塞等待加锁
            - `try_lock_for()`      //在一段时间内尝试加锁
            - `try_lock_until()`    //在某个时间点之前尝试加锁
            - `unlock()`            //解锁
    + `std::lock`
        * 定义：`void lock( Lockable1& lock1, Lockable2& lock2, LockableN&... lockn );` Lockablen等类型都是模板类
        * 锁定给定的*可锁定*对象lock1、lock2...，用免死锁算法避免死锁
            - 满足一定条件则为可锁定Lockable，需要满足m.lock()/m.unlock()/m.try_lock()等表现要求
            - 可锁定要求可参考：[C++ 具名要求： 可锁定 (Lockable)](https://zh.cppreference.com/w/cpp/named_req/Lockable)
        * [std::lock](https://zh.cppreference.com/w/cpp/thread/lock)
        * 可以看一下链接中的示例，`std::lock`获取第二个锁而不用担心死锁，可多个互斥量同时加锁。示例中列出了几个等价加锁的方式：
            - a. `std::lock(e1.m, e2.m);` `std::lock_guard<std::mutex> lk1(e1.m, std::adopt_lock);`，相同方式对e2.m加锁
                + std::adopt_lock方式，会接管锁，并lock
            - b. `std::unique_lock<std::mutex> lk1(e1.m, std::defer_lock);`，e2.m同样方式定义，再`std::lock(lk1, lk2);`
                + std::defer_lock方式，初始化后并不调用lock
            - c. C++17中有较优解法：`std::scoped_lock lk(e1.m, e2.m);`
    + `std::lock_guard`跟mutex本身是强关联的，也就是说`lock_guard`一旦存在，mutex就必须是锁定的，而条件变量中有过程是要求释放锁的。这个场景下可以用`unique_lock`

* `unique_lock`的示例:

```cpp
std::mutex foo,bar;

// 场景：两个银行账户相互转账，各自有锁，转账时同时锁定两把锁
void task_a () {
    // simultaneous lock (prevents deadlock) 同时加锁，避免死锁
    std::lock (foo,bar);
    // 创建unique_lock并接管锁foo，创建前已经获取了两个锁
    std::unique_lock<std::mutex> lck1 (foo,std::adopt_lock);
    // 创建unique_lock并接管锁bar
    std::unique_lock<std::mutex> lck2 (bar,std::adopt_lock);
    std::cout << "task a\n";
    // (unlocked automatically on destruction of lck1 and lck2)
}
```

* `adopt_lock`也可用于`lock_guard`，示例(创建该类型前已经获得锁)：

```cpp
void print_thread_id (int id) {
    // 创建前已经通过lock()获得锁，有下面的接管后不用再调用unlock
    mtx.lock();
    // 创建adopt_lock类型的包装，接管锁，且声明周期后会自动解锁
    std::lock_guard<std::mutex> lck(mtx, std::adopt_lock);
    std::cout << "thread #" << id << '\n';
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

* [C++ 编译器支持情况表](https://zh.cppreference.com/w/cpp/compiler_support#cpp11)
    - 另外`gcc`对于`C++17`的支持情况，很多特性都是在`gcc7`才支持，部分特性在`gcc5`和`gcc6`支持，所以若要用C++17特性，最好还是升级到`gcc7`
* gcc版本升级可参考：[为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本](https://www.vpser.net/manage/centos-6-upgrade-gcc.html)

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

### decltype

* `decltype`
    - [C++11特性：decltype关键字](https://www.cnblogs.com/QG-whz/p/4952980.html)
    - `typeid`
        + `typeid`用于查询一个变量的类型，这种类型查询在运行时进行
        + RTTI机制为每一个类型产生一个`type_info`类型的数据，而typeid查询返回的变量相应`type_info`数据，通过`name`成员函数返回类型的名称
            * 注意输出并不是`int`、`float`之类的名称
            * 要判断返回的类型，可以用 `if(typeid(xdvalue) == typeid(int))` 形式来判断
        + RTTI会导致运行时效率降低，且在泛型编程中，我们更需要的是编译时就要确定类型，RTTI并无法满足这样的要求
        + 编译时类型推导的出现正是为了泛型编程，在非泛型编程中，我们的类型都是确定的，根本不需要再进行推导
    - decltype与auto关键字一样，用于进行编译时类型推导
        + decltype的类型推导并不是像auto一样是从变量声明的初始化表达式获得变量的类型，
        + 而是总是以一个普通表达式作为参数，返回该表达式的类型,
        + 而且decltype并不会对表达式进行求值
        + e.g. `int i = 4;`, `decltype(i) a;` //推导结果为int。a的类型为int
    - 与`using`/`typedef`合用，用于定义类型
        + C++11里面，**推荐使用using**，而非typedef
            * using 可以用于模板别名，typedef 不可用于模板别名，且using可读性较好
                - 可读性e.g.
                    + `typedef void (*FP) (int, const std::string&);`
                    + `using FP = void (*) (int, const std::string&);`
                - 模板中使用e.g.
                    + `template <typename T>`, `using Vec = MyVector<T, MyAlloc<T>>;`, 使用: `Vec<int> vec;`
                    + 而用typedef，`typedef MyVector<T, MyAlloc<T>> Vec;`，编译的时候，将会得到类似：error: a typedef cannot be a template的错误信息
            * [Effective Modern C++ Note 02](https://zhuanlan.zhihu.com/p/21264013)
        + e.g. `using size_t = decltype(sizeof(0));`
        + e.g. `typedef decltype(vec.begin()) vectype;`
    - 泛型编程中结合auto，用于追踪函数的返回值类型
        + `template <typename _Tx, typename _Ty>`
            * `auto multiply(_Tx x, _Ty y)->decltype(_Tx*_Ty) {...}`
    - 推导规则
        + 如果e是一个没有带括号的标记符表达式或者类成员访问表达式，那么的decltype（e）就是e所命名的实体的类型
        + 否则 ，假设e的类型是T，如果e是一个将亡值，那么decltype（e）为T&&
        + 否则，假设e的类型是T，如果e是一个左值，那么decltype（e）为T&
        + 否则，假设e的类型是T，则decltype（e）为T

* 替代宏
    - [宏在C++中的替代解决方案](https://blog.csdn.net/aheroofeast/article/details/7932762)
    - `#define NUM 100`
        + 由于宏是预编译程序来处理，所以NUM这个名字不会加入到符号表中，如果出现编译错误时，提示信息中就不会出现NUM，而是100，为排除错误增加了额外的障碍
    - 替代方案就是使用const来定义常量，或者使用枚举enum
        + `const int NUM = 100;`
        + const常量放在头文件中，也不必担心存在多个实例的问题，对于const修饰的变量，编译器一般也会对其进行优化，不会出现多重定义的问题
    - C语言中还有一个特殊的常量定义：`NULL`。其一般的定义为` #define NULL 0`，指针的内容却是一个整型，这不符合常理。
        + 所以在C++11中使用`nullptr`代替了`NULL`

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
        + 类型转换的示例，参考：[C++ 四种强制类型转换](https://www.cnblogs.com/Allen-rg/p/6999360.html)
        + `static_cast<type>(expression);` 很像 C 语言中的旧式类型转换，是非安全的
            * e.g. `double result = static_cast<double>(a) / static_cast<double>(b);`，模板特化
            * 同 `double result = (double)a / (double)b;`
            * static_cast不能转换掉expression的const、volitale或者__unaligned属性。
        + `dynamic_cast` 主要用来在继承体系中的安全向下转型,是实现多态的一种方式。
            * 其他三种都是编译时完成的，dynamic_cast是运行时处理的，运行时要进行类型检查。
            * 不能用于内置的`基本数据类型`的强制转换
            * dynamic_cast转换如果成功的话`返回`的是`指向类的指针或引用`，转换失败的话则会返回`NULL`。
            * 使用dynamic_cast进行转换的，基类中`一定要有虚函数`，否则编译不通过。(通过虚函数实现多态)
                - 这是由于运行时类型检查需要运行时类型信息，而这个信息存储在类的虚函数表，只有定义了虚函数的类才有虚函数表。
            * 在类的转换时(参考链接示例)
                - 在类层次间进行上行转换时(子类指针转换为父类指针)，`dynamic_cast`和`static_cast`的效果是一样的
                - 在进行下行转换时，`dynamic_cast`具有类型检查的功能(转换失败返回NULL，所以应该判断)，比`static_cast`更安全
                    + 编译成功，运行时错误，dynamic_cast转换会失败，返回NULL
        + `const_cast` 可去除对象的常量性（const）还可以去除对象的易变性（volatile）
            * 在C语言中，const限定符通常被用来限定变量，用于表示该变量的值不能被修改。
            * 而const_cast则正是用于强制去掉这种不能被修改的常数特性
            * 但需要特别注意的是const_cast`不是用于去除变量的常量性`，而是去除`指向常数对象的指针或引用`的常量性，其去除常量性的**对象必须为指针或引用**。
            * 对指向`const常量`的`const指针`(或引用)去除const属性后(e.g.通过const_cast<int*>(p))，解引用后进行赋值操作，其语句行为是未定义的，即在标准C++规范中并没有明确规定这种语句的具体行为，其具体行为由编译器来自行决定如何处理。对于这种未定义行为的语句应该尽量予以避免！
            * const_cast的应用场景是函数返回是const指针或者引用，不过在调用该函数的外部，指针或引用指向的变量并不是const的，这种情况可以去除const属性进行操作(可参考上面链接中的示例)
            * 但是建议`不要`利用const_cast去掉指针或引用的常量性并且去修改原始变量的数值，这是一种`非常不好`的行为
        + `reinterpret_cast` 用来执行低级转型，如将执行一个 int 的指针强转为 int
            * 其转换结果与编译平台相关(指针长度：32位4字节、64位8字节)
            * 用法：`reinterpret_cast<type_id> (expression)`
            * 用途：改变指针或引用的类型(别的类型指针)、将指针或引用转换为一个足够长度的整形、将整型转换为指针或引用类型
                - 可以把一个指针转换成一个整数
                - 也可以把一个整数转换成一个指针

## 虚函数表

* 虚函数表
    - [C++ 虚函数表解析](https://blog.csdn.net/haoel/article/details/1948051/)
        + 注意参考链接中的示例是基于32位系统的，虚函数地址表成员强转为`(int*)`类型是ok的(地址表里都保存int 4字节的地址)，但是对于64位系统，`(int*)`就有问题了，应该是`(long*)`，虚函数表中保存的指针为8字节数据
        + 虚函数表中只保存虚函数的地址/指针，示例中的子类成员函数也是定义成virtual的
    - 虚函数（Virtual Function）是通过一张`虚函数表（Virtual Table）`来实现的。简称为V-Table。
        + 编译器会为每个有虚函数的类创建`一个`虚函数表，该虚函数表将被`该类的所有对象共享`
        + 类的每个虚成员占据虚函数表中的一行。如果类中有N个虚函数，那么其虚函数表将有`N*4或者8`(根据指针字节大小4或8)的大小。
        + 虚函数表放在应用程序的`常量区`
        + [vtbl（虚函数表）与vptr（虚函数表指针）](https://blog.csdn.net/micx0124/article/details/9838147)
    - 在这个表中，主要是一个类的虚函数`地址表`，这张表解决了继承、覆盖的问题。
    - 当我们用父类的指针来操作一个子类的时候，虚函数表指明了实际所应该调用的函数。
    - C++的编译器应该是保证`虚函数表的指针`存在于对象实例中`最前面的位置`(第一个指针，前4/8个字节)（这是为了保证取到虚函数表的有最高的性能——如果有多层继承或是多重继承的情况下）。这意味着我们通过对象实例的地址得到这张虚函数表，然后就可以遍历其中的函数指针，并调用相应的函数
    - 参考链接中的示例：
        + class Base含有下面的虚函数：`virtual void f() { cout << "Base::f" << endl; }`和`virtual void g() { cout << "Base::g" << endl; }`
        + 定义对象实例：`Base b;`，则虚函数表的指针(位于对象最前面)为 `(long*)(&b)`，(而不是 ~~(int*)(&b)~~)
            * 也可以用 `reinterpret_cast<unsigned long*>(&b)`，将对象指针转换为(长)整型的指针(指针长度：unsigned long)
            * 使用int时编译警告：将一个整数转换为大小不同的指针 [-Wint-to-pointer-cast]
        + 通过虚函数表的指针找到对应虚函数的地址(函数指针)，声明函数地址类型为： `typedef void (*Func)(void);`
        + 则可得虚函数成员f()(第一个虚函数)的地址为： `*((long*)*(long*)(&b) + 0)`，强转为函数指针`(Func) *((long*)*(long*)(&b) + 0)`
        + 第二个虚函数`g()`地址为：`(Func) *((long*)*(long*)(&b) + 1)` (强转为long*，64位系统下+1则偏移8字节)
    - 另外一种情况是`虚继承`，父类没有虚函数，但子类虚继承父类(继承时添加virtual关键字)，此时也会有一个虚表指针
        + (更复杂的情形是菱形继承，不过不推荐多重继承，所以菱形继承也随之少见。关于菱形继承，SO参考链接和下面` C++的开销分析`章节中也有介绍，不过多重虚继承的类型和虚表指针还有偏移看得脑壳疼。。。)
        + 相较与普通继承而包含虚函数，gdb查看有差别
            * 包含虚函数，为`_vptr.HaveVirtual = 0x400f10 <vtable for HaveVirtual+16>`
            * 虚继承，为`_vptr.TestCVirtual = 0x400ef8 <VTT for TestCVirtual>`
                - `virtual table table`，简写为VTT，是用于构造中的一个 vtalbes 的表
        + 虚继承：在继承定义中包含了virtual关键字的继承关系
        + 虚基类：在虚继承体系中的通过virtual继承而来的基类(称xx为xx子类的虚基类)
        + 一旦出现了虚基类，就必须在每一个继承类中都必须包含虚基类的初始化语句
        + [虚继承](https://baike.baidu.com/item/%E8%99%9A%E7%BB%A7%E6%89%BF)
        + [What is the VTT for a class?](https://stackoverflow.com/questions/6258559/what-is-the-vtt-for-a-class)
    - gdb 查看虚函数表
        + a. `b 行号` 打断点到对象定义后，`p b`打印对象
            * `set print object on` 可设置打印真实的类类型(即打印对象的指针时，打印派生的实际类而不是基类)
                - 有RTTI才生效，需要有虚函数才生效
                - 参考gdb onlinedoc(搜set print object on)：[gdb onlinedoc](https://sourceware.org/gdb/onlinedocs/gdb/Print-Settings.html)
        + b. 设置上面的object on后，`p b`打印前面多了一个`(真实类型)`
            * `$6 = {_vptr.Base = 0x400f90 <vtable for Base+16>}` 设置前
            * `$7 = (Base) {_vptr.Base = 0x400f90 <vtable for Base+16>}` 设置后
        + c. 查看虚表指针(`0x400f90`)中的内容(可以看到虚函数f()、g()、h()及对应地址)
            * `p /a *(void**)0x400f90@4` 或(`p/a`)
                - `$15 = {0x400ce6 <Base::f()>, 0x400d10 <Base::g()>, 0x400d3a <Base::h()>, 0x6465766972654437}`
                - `set print symbol-filename on` 可打印各虚函数在文件中的位置(文件和行号)
            * [GDB print 详解](https://blog.csdn.net/linuxheik/article/details/17380767)
                - `@` 是一个和数组有关的操作符
                - `::` 指定一个在文件或是一个函数中的变量。变量重名时，查看文件f2.c中的全局变量x的值：`(gdb) p 'f2.c'::x`
                - `p *array@len ` array:数组的首地址，len:数据的长度
                - `/a` 按十六进制格式显示变量。 还可以`p/c`字符格式、`p/d`十进制、`p/o`八进制、`p/f`浮点数等等
        + [gdb查看虚函数表、函数地址](https://www.cnblogs.com/johnnyflute/p/3675630.html)

示例(64位CentOS)：

```cpp
#include <iostream>
#include <cstdio>

using namespace std;

class Base {

public:
    virtual void f() { cout << "Base::f" << endl; }
    virtual void g() { cout << "Base::g" << endl; }
    virtual void h() { cout << "Base::h" << endl; }
    void i() { cout << "Base::i" << endl; }
};

class Derived final : Base {

public:
    virtual void haha() { cout << "Derived::haha" << endl;}
    void i() { cout << "Derived::i" << endl; }
};

typedef void (*Func)(void);

int main(int argc, char const *argv[])
{
    long *point = NULL;
    // point size: 8, sizeof(long):8, sizeof(int):4
    cout << "point size: " << sizeof(point)
        << ", sizeof(long):" << sizeof(long)
        << ", sizeof(int):" << sizeof(int)
        << endl;
    Base b;

    /*
        虚表地址:0x400f30
        第一个虚函数地址:0x400cb2
        第二个虚函数地址:0x400cdc
    */
    // 第一个指针中的内容(存放虚函数表地址)
    printf("虚表地址:%p\n", *(long *)(&b));
    // 虚函数表按顺序保存虚函数地址(普通成员的函数地址/指针不保存)
    printf("第一个虚函数地址:%p\n", *(long*)*(long*)(&b));
    printf("第二个虚函数地址:%p\n", *((long*)*(long*)(&b) + 1));

    cout << "======= 等价于 ======" << endl;
    /*
        虚表地址:0x400f30
        第一个虚函数地址:0x400cb2
        第二个虚函数地址:0x400cdc
    */
    unsigned long *pVt = reinterpret_cast<unsigned long*>(&b); // 对象地址转为unsigned long*类型
    pVt = (unsigned long*)*pVt; // 虚函数表指针
    printf("虚表地址:%p\n", pVt);
    printf("第一个虚函数地址:%p\n", pVt[0]);
    printf("第二个虚函数地址:%p\n", pVt[1]);
    cout << "==============\n" << endl;

    /*
        addr:0x400cb2, addr2:0x400cdc
        Base::f
        Base::g
    */
    Func addr =  (Func)*((long*)*(long*)(&b)+0);
    // Func addr2 = (Func)*((int*)*(int*)(&b)+1); // 用int*会得到NULL
    Func addr2 = (Func)*((long*)*(long*)(&b)+1);
    cout << "addr:" << (long*)addr << ", addr2:" << (long*)addr2 << endl;
    addr();
    addr2();

    cout << "==============111" << endl;
    /*
        addr: 0x400cb2, Base::f
        addr: 0x400cdc, Base::g
        addr: 0x400d06, Base::h
    */
    // 说明：若继承N个包含虚函数的类，则就有N个虚函数表(继承但不实现虚函数，子类虚函数顺序排列在表中；实现则替换地址值)
    long **pVtable = (long**)(&b);
    for (long i = 0; i<3 && NULL != (Func)pVtable[0][i]; i++)
    {
        addr = (Func)pVtable[0][i];
        printf("addr: %p, ", pVtable[0][i]);
        addr();
    }

    cout << "==============222" << endl;
    /*
        addr: 0x400cb2, Base::f
        addr: 0x400cdc, Base::g
        addr: 0x400d06, Base::h
        addr: 0x400d30, Derived::haha
    */
    Derived derived;
    pVtable = (long**)(&derived);
    for (long i = 0;  NULL != (Func)pVtable[0][i]; i++)
    {
        addr = (Func)pVtable[0][i];
        printf("addr: %p, ", pVtable[0][i]);
        addr();
    }

    return 0;
}
```

* C++的开销分析
    - 参考：[RTTI、虚函数和虚基类的实现方式、开销分析及使用指导](http://www.baiy.cn/doc/cpp/inside_rtti.htm)
    - 相对于传统的 C 语言，C++ 引入的额外开销体现在以下两个方面
    - `编译时开销`
        + 模板、类层次结构、强类型检查等新特性，以及大量使用了这些新特性的 STL 标准库都增加了编译器负担。
            * 但是应当看到，这些新机能在不降低，甚至(由于模板的内联能力)提升了程序执行效率的前提下，明显减轻了广大C++程序员的工作量。
        + 在使用这些特性的时候，也有不少优化技巧。
    - `运行时开销`
        + 相对与传统C程序而言，C++中有可能引入额外运行时开销的新特性包括：
        + `虚基类`
        + `虚函数`
            * 一般地讲，能用`虚函数`解决的问题就不要用 `"dynamic_cast"`，能够用 `"dynamic_cast"` 解决的就不要用 `"typeid"`
            * 参考链接中的`反面`示例，实现一个函数功能时根据传入的类引用，用了`typeid`做多个分支判断处理(最好的方式是父类中定义虚函数，子类去实现。然后函数入参传入基类指针或者引用来实现多态)
            * 虚函数是C++众多运行时多态特性中开销最小，也最常用的机制。
            * 应当注意在对性能有苛刻要求的场合，或者需要频繁调用，对性能影响较大的地方（比如每秒钟要调用成千上万次，而自身内容又很简单的事件处理函数）要慎用虚函数
            * 需要特别说明的一点是：虚函数的调用开销与通过函数指针的间接函数调用（例如：经典C程序中常见的，通过指向结构中的一个函数指针成员调用；以及调用DLL/SO中的函数等常见情况）是*相当的*。比起函数调用*本身*的开销（保存现场->传递参数->传递返回值->恢复现场）来说，一次指针间接引用是微不足道的。这就使得*在绝大部分可以使用函数的场合中都能够负担得起虚方法*的些微额外开销。
        + `RTTI`（`dynamic_cast`和`typeid`）
            * Run-Time Type Identification，运行时类型识别
            * 通过运行时类型信息程序能够使用`基类的指针或引用`来检查这些指针或引用所指的对象的`实际派生类型`。
        + `异常`
            * 对于大多数现代编译器来说，在正常情况（未抛出异常）下，try块中的代码执行效率和普通代码一样高
            * 而且由于不再需要使用传统上通过返回值或函数调用来判断错误的方式，代码的实际执行效率还可能进一步提高
            * 抛出和捕捉异常的效率也只是在某些情况下才会稍低于函数正常返回的效率，*何况对于一个编写良好的程序，抛出和捕捉异常的机会应该不多。* (持保留意见，内存申请判断，函数返回值判断 两个大头)
        + `对象的构造和析构`
            * 对于不需要初始化/销毁的类型，并没有构造和析构的开销，相反对于那些需要初始化/销毁的类型来说，即使用传统的C方式实现，也至少需要与之相当的开销。
            * 这里要注意的一点是尽量不要让构造和析构函数过于臃肿，特别是在一个类层次结构中更要注意。
            * 时刻保持你的构造、析构函数中只有最必要的初始化和销毁操作，把那些并不是每个（子）对象都需要执行的操作留给其他方法和派生类去解决
    - C++之所以 被广泛认为比C“低效”，其根本原因在于：由于程序员对某些特性的实现方式及其产生的开销不够了解，致使他们在错误的场合使用了错误的特性。而这些错误基本都集中在：
        + 把异常当作另一种流控机制，而不是仅将其用于错误处理中
        + 一个类和/或其基类的构造、析构函数过于臃肿，包含了很多非初始化/销毁范畴的代码
        + 滥用或不正确地使用RTTI、虚函数和虚基类机制

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

```cpp
//创建string的共享指针
shared_ptr<string> sp = make_shared<string>("make_shared");

//创建vector的共享指针
shared_ptr<vector<int> > spv = make_shared<vector<int> >(10, 2);
```

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
* 最好的话，这4个函数一旦自己实现一个，就应该养成实现另外3个的习惯。
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

// std::advance用来对迭代器做偏移操作, 类似于splitIter=splitIter+splitPos
std::advance(splitIter,splitPos);

### 运算符重载

* [C++ 中的运算符重载](https://www.runoob.com/cplusplus/cpp-overloading.html)
    - 可以重定义或重载大部分 C++ 内置的运算符，链接中列出了可重载运算符/不可重载运算符
        + 不可重载：
            * `.` 成员访问运算符
            * `.*` `->*` 成员指针访问运算符
                - [指针到成员运算符：.* 和 ->*](https://docs.microsoft.com/zh-cn/cpp/cpp/pointer-to-member-operators-dot-star-and-star?view=vs-2019)
                - 指针到成员运算符 .* 和 ->*返回表达式左侧指定对象的特定类成员的值
                    + 右侧必须指定该类的成员
                    + 仅指向指定的类成员而不是类对象
                - 如类`Testpm`中有一个成员函数`m_func1()`和成员变量`int m_num;`
                    + `Testpm ATestpm;`定义一个类实例(若`Testpm *pTestpm = new Testpm;`则下面使用`->*`访问)
                    + 在类外定义一个`指向类成员的指针`：
                        * `void (Testpm::*pmfn)() = &Testpm::m_func1;`
                        * 使用时：`(ATestpm.*pmfn)();`
                    + 也可以定义非函数类型成员的指针
                        * `int Testpm::*pmd = &Testpm::m_num;`
                        * `ATestpm.*pmd = 1;`
            * `::` 域运算符
            * `sizeof` 长度运算符
            * `?:` 条件运算符
            * `#` 预处理符号
    - 运算符重载实例
        + 对于`++`，分前缀版本(`++i`)和后缀版本(`i++`)
            * `Time operator++()`    // 前缀版本，返回++之后的数据
            * `Time operator++(int)` // 后缀版本，返回++之前的数据

### 重载()

类A重载()后(可以有不同的形参形式)，A(p1, p2) 形式即使用到重载的operator(P1,P2)形式

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


## 编译问题

* 警告：在此处初始化后被初始化 [-Wreorder]
    - 初始化列表中，初始化的顺序和成员定义的顺序不同，会报该警告(有时变量初始化会有依赖关系，一般保持顺序一致)

* std::bad_alloc
    - 最后找到问题是头文件中类更新了(类加了个字段)但是make的时候没有生效，导致库中的.h类结构和调用库程序用的.h类结构不一致，内存越界
        + lib.a静态库单独编译，提供.a和.h头文件给程序使用
        + 程序编译时-l链接.a库，并使用其提供的.h文件(并没有复制文件，静态库的源码放在程序目录中，编译时直接找目录中的文件)
        + 若库中的.h修改，除了库要重新编译外，建议外部编译时先make clean一下再编译
        + 对于makefile的依赖规则，还要再查询整理下。已存在.o时若.h修改，再次编译不用新的.h？但此处外部程序会用到.h中类的新增字段，不用新的.h也说不过去

```
terminate called after throwing an instance of 'std::bad_alloc'
  what():  std::bad_alloc

Program received signal SIGABRT, Aborted.
[Switching to Thread 0x7fffef70d700 (LWP 9323)]
0x00007ffff5573207 in raise () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install cyrus-sasl-lib-2.1.26-23.el7.x86_64 glibc-2.17-260.el7.x86_64 keyutils-libs-1.5.8-3.el7.x86_64 krb5-libs-1.15.1-34.el7.x86_64 libcom_err-1.42.9-13.el7.x86_64 libselinux-2.5-14.1.el7.x86_64 libuuid-2.23.2-59.el7.x86_64 nss-softokn-freebl-3.36.0-5.el7_5.x86_64 openssl-libs-1.0.2k-16.el7.x86_64 pcre-8.32-17.el7.x86_64 zlib-1.2.7-18.el7.x86_64
(gdb) 
(gdb) 
(gdb) bt
#0  0x00007ffff5573207 in raise () from /lib64/libc.so.6
#1  0x00007ffff55748f8 in abort () from /lib64/libc.so.6
#2  0x00007ffff609d445 in __gnu_cxx::__verbose_terminate_handler () at ../../../../libstdc++-v3/libsupc++/vterminate.cc:95
#3  0x00007ffff609b5d6 in __cxxabiv1::__terminate (handler=<optimized out>)
    at ../../../../libstdc++-v3/libsupc++/eh_terminate.cc:38
#4  0x00007ffff609b603 in std::terminate () at ../../../../libstdc++-v3/libsupc++/eh_terminate.cc:48
#5  0x00007ffff609b823 in __cxxabiv1::__cxa_throw (obj=0x7fffe40141a0, tinfo=0x7ffff6322b00 <typeinfo for std::bad_alloc>, 
    dest=0x7ffff6099cd0 <std::bad_alloc::~bad_alloc()>) at ../../../../libstdc++-v3/libsupc++/eh_throw.cc:87
#6  0x00007ffff609bd1d in operator new (sz=18446744073709551600) at ../../../../libstdc++-v3/libsupc++/new_op.cc:56
#7  0x0000000000427cb2 in __gnu_cxx::new_allocator<TestResult>::allocate (this=0x7fffef70caf0, __n=177372539170284150)
    at /usr/local/include/c++/4.8.5/ext/new_allocator.h:104
#8  0x00000000004242d5 in std::_Vector_base<TestResult, std::allocator<TestResult> >::_M_allocate (
    this=0x7fffef70caf0, __n=177372539170284150) at /usr/local/include/c++/4.8.5/bits/stl_vector.h:168
#9  0x00000000005512f0 in std::vector<TestResult, std::allocator<TestResult> >::_M_emplace_back_aux<TestResult const&> (this=0x7fffef70caf0) at /usr/local/include/c++/4.8.5/bits/vector.tcc:404
#10 0x000000000054c8ed in std::vector<TestResult, std::allocator<TestResult> >::push_back (
    this=0x7fffef70caf0, __x=...) at /usr/local/include/c++/4.8.5/bits/stl_vector.h:911
```


## 零拷贝

* 零拷贝
    - 参考：[Linux零拷贝原理](http://ifeve.com/linux%e9%9b%b6%e6%8b%b7%e8%b4%9d%e5%8e%9f%e7%90%86/)
    - 举例：`read`文件再`write`文件到socket，实现了几次拷贝？
        + 数据至少被复制了4次
        + 而且至少4次用户态/内核态上下文切换(模式切换就算一次)
        + 参考链接里面的框图所示的过程，比较直观
            * 第一步：`read`读系统调用会导致从用户模式到内核模式的上下文切换(*切换1*)。 第一个复制由DMA引擎执行，它读取磁盘中的文件内容并将其存储到内核地址空间缓冲区中(*复制1*）。
            * 第二步：将数据从`内核缓冲区`复制到`用户缓冲区`(*复制2*)， `read`系统调用返回。调用的返回导致了从`内核`返回到`用户模式`的上下文切换(*切换2*)，现在，数据存储在用户地址空间缓冲区中。
            * 第三步::write系统调用导致从用户模式到内核模式的上下文切换(*切换3*)，执行第三个复制(*复制3*)，将数据再次放入内核地址空间缓冲区中。但是这一次，数据被放入一个`不同的缓冲区`，这个缓冲区是与套接字相关联的。
            * 第四步：写系统调用返回，创建第四个上下文切换(*切换4*)。`DMA引擎`将数据从`内核缓冲区`传递到`协议engin`时(*复制4*)，第四个复制发生了独立和异步(以太网驱动程序只是有空闲，接受了要传输的数据，并不保证开始传输或者传输完成)的情况。
    - 如上例所示：*大量的数据复制并不是真正需要的*。可以消除一些重复，以减少开销并提高性能。
        + 为了消除开销，我们可以从消除内核和用户缓冲区之间的一些复制开始。
        + 消除复制的一种方法是跳过调用`read`，改而调用`mmap`。 (3次复制，4次上下文切换)
            * `tmp_buf = mmap(file, len);` 然后 `write(socket, tmp_buf, len);`
            * 过程参考链接中的框图：
                - 第一步:mmap系统调用将`文件内容`复制到DMA引擎的`内核缓冲区`中。然后在用户进程中`共享`缓冲区，而不需要在内核和用户内存空间之间执行任何复制(上下文切换还是有的)。
                - 第二步:write系统调用导致内核将数据从原始`内核缓冲区`复制到与`套接字关联的内核缓冲区`中
                - 第三步:当DMA引擎将数据从`内核套接字缓冲区`传递到`协议引擎`时，第三次复制发生。
            * 通过使用`mmap`而不是读取，我们将内核必须复制的数据量减少了一次(而不是原文链接中的一半，内核态之间也有一次拷贝)。
        + 这种使用`mmap+write`的改进方法时存在一些隐藏的缺陷：
            * 当内存映射一个文件，然后调用`write`，而文件被其他进程截断时，`write`调用将被总线信号`SIGBUS`中断(该信号默认杀死进程并转储内核)
            * 解决方式：
                - a. 为信号`SIGBUS`安装一个信号处理程序，并将errno设置为成功。 不过必须指出：这个方案将是个糟糕的方案，因为`SIGBUS`信号表明过程出现了严重的问题
                - b. `文件租赁`，这是解决这个问题的正确方法。
                    + 通过使用文件描述符上的租赁,你将在内核上租赁获取一个特定的文件。通过在文件描述符上使用租赁，可以在特定文件上使用内核进行租约。然后可以从内核请求读/写租约。
                    + 当另一个进程试图截断正在传输的文件时，内核会向您发送一个实时信号，即`RT_SIGNAL_LEASE`信号。在程序访问一个无效的地址并被SIGBUS信号杀死之前，您的write调用会被中断。write调用的返回值是在中断之前写入的字节数，而errno将被设置为成功。
                    + 读取前：`if(fcntl(fd, F_SETSIG, RT_SIGNAL_LEASE) == -1)`
                    + 读取结束：`if(fcntl(fd, F_SETLEASE, l_type))`
                    + 应该在mmaping文件之前得到你的租约，并且在你完成之后将租约撕毁。这是通过使用`F_UNLCK`的租约类型调用`fcntl F_SETLEASE`实现的。
        + 在内核版本2.1中，引入了`sendfile`系统调用，以简化网络和两个本地文件之间的数据传输。`sendfile`的引入不仅减少了数据复制(*3次*)，还减少了上下文切换(*2次*)。参考链接中的流程框图。
            * 第一步:sendfile系统调用将把文件内容复制到DMA引擎的内核缓冲区中(*复制1*)。然后将数据复制到与套接字相关联的内核缓冲区中(*复制2*)。
            * 步骤二:当DMA引擎将数据从内核套接字缓冲区传递到协议引擎时，第三次复制发生。
            * 代码：`sendfile(socket, file, len);`
            * 如果另一个进程截断了我们用`sendfile`系统调用发送的文件：
                - 如果我们不注册任何信号处理程序，sendfile调用只需返回它在被中断之前传输的字节数，而errno将被设置为成功。
    - 由上面的方案，已经能够避免内核生成几个复制，但是我们仍然只剩下一个复制。也可以避免：
        + 在硬件的帮助下。为了消除内核所做的所有数据复制，我们需要一个支持收集操作的网络接口。
        + 这意味着等待传输的数据不需要在`连续的内存`中，它可以分散在不同的内存位置。
        + 在内核版本2.4中，修改了套接字缓冲区描述符以适应这些需求——在Linux下称为**零拷贝**。
            * 这种方法不仅减少了多个上下文切换(*只有2次*)，还消除了处理器的数据复制(*也只有2次*)。参考链接中的流程框图。
            * 代码仍然是：`sendfile(socket, file, len);`
            * 支持集合的硬件可以从多个内存位置组装数据，从而消除另一个复制。
            * 第一步:sendfile系统调用将把文件内容复制到DMA引擎的`内核缓冲区`中。
            * 第二步:`没有`将数据复制到套接字缓冲区中。相反，只有`带有关于数据的位置和长度的信息的描述符`被`追加`到套接字缓冲区。DMA引擎`直接`将数据从`内核缓冲区`传递到`协议引擎`，从而消除剩余的最终复制。
    - DMA(Direct Memory Access) 直接内存存取
        + [DMA （直接存储器访问）](https://baike.baidu.com/item/DMA/2385376)
        + 是所有现代电脑的重要特色
        + 它允许不同速度的硬件装置来沟通，而不需要依赖于 CPU 的大量中断负载
        + 否则，CPU 需要从来源把每一片段的资料复制到暂存器，然后把它们再次写回到新的地方。在这个时间中，CPU 对于其他的工作来说就无法使用。
        + DMA 传输将数据从一个地址空间复制到另外一个地址空间。在实现DMA传输时，是由DMA控制器直接掌管总线，因此，存在着一个总线控制权转移问题。即DMA传输前，CPU要把总线控制权交给DMA控制器，而在结束DMA传输后，DMA控制器应立即把总线控制权再交回给CPU。一个完整的DMA传输过程必须经过DMA请求、DMA响应、DMA传输、DMA结束4个步骤。

## 现代C++实战30讲

* 极客时间课程：[现代C++实战30讲](https://time.geekbang.org/column/article/169268)

## C++管理资源

* 编译器会自动调用析构函数，包括在函数执行发生异常的情况。在发生异常时对析构函数的调用，还有一个专门的术语，叫栈展开（stack unwinding）

## 模板

* 参考：[C++ 模板详解（一）](https://www.cnblogs.com/gw811/archive/2012/10/25/2738929.html)
* 模板
    - 通常有两种形式：`函数模板`和`类模板`
        + 函数模板 针对仅参数类型不同的函数；
        + 类模板 针对仅数据成员和成员函数类型不同的类。
    - 使用模板的目的就是能够让程序员编写与类型无关的代码
    - 注意：模板的声明或定义只能在`全局`，`命名空间`或`类范围内`进行。即不能在局部范围，`函数内`进行，比如不能在main函数中声明或定义一个模板。

* 函数模板的写法如下：
    - `template <class 形参名1, class 形参名2, ...>`
    - `class` 关键字也可以用 `typename` 关键字替换 (在这里typename 和class没区别)
        + `template <typename 形参名1, typename 形参名2, ...>`
        + 建议还是用`typename`：`template <typename 形参名1, typename 形参名2, ...>`
        + 这些形参称为：*模板形参* / *模板参数*
    - 编译器由模板自动生成函数时，会用具体的类型名对模板中所有的类型参数进行替换，其他部分则原封不动地保留。
    - 编译器由模板自动生成函数的过程叫模板的*实例化*。
    - 由模板实例化而得到的函数称为*模板函数*。
    - 如：`template <class T>;`, `void Swap(T & x, T & y){}`
        + T 是*类型参数*，代表类型。 `类型模板参数`
    - 三种模板参数
        + [模板形参与模板实参](https://zh.cppreference.com/w/cpp/language/template_parameters)
        + 类型模板参数
            * 类型形参关键词：`typename` 或 `class`
                - e.g. `template<class T>` 无默认类型的类型模板实参
                    + `class My_vector { /* ... */ };`
                - e.g. `template<class T = void>` 有默认类型的类型模板实参
                    + `struct My_op_functor { /* ... */ };`
                - e.g. `template<typename... Ts>` 类型模板形参包
                    + `class My_tuple { /* ... */ };`
            * 对模板声明时(而不是定义)，形参的名字是可选的
                - 上面的`My_vector`模板声明，可为：`template<class> class My_vector;`
                - `template<class = void> struct My_op_functor;`
                - `template<typename...> class My_tuple;`
        + 非类型模板参数
        + 模板模板参数(以模板作为模板的参数)

```cpp
template <class 形参名1, class 形参名2, ...>
返回值类型 函数名(参数列表)
{
    函数体
}
```

* 类模板
    - 一但声明了类模板就可以用类模板的 形参名 声明类中的成员变量和成员函数
        + 即可以在类中使用内置类型的地方都可以使用模板形参名来声明
        + e.g. `template<class T> class A{public: T a; T b; T hy(T c, T &d);};`
            * 在类A中声明了两个类型为T的成员变量a和b，还声明了一个返回类型为T带两个参数类型为T的函数hy
    - 类模板对象的创建
        + 一个模板类A，则使用类模板创建对象的方法为A<int> m;
        + 对于类模板，模板形参的类型必须在类名后的尖括号中明确指定。比如`A<2> m`;
            * 用这种方法把模板形参设置为int是`错误`的（编译错误：error C2079: 'a' uses undefined class 'A<int>'），类模板形参不存在实参`推演`的问题。
            * 也就是说不能把整型值2推演为int 型传递给模板形参。要把类模板形参调置为int 型必须这样指定`A<int> m`
    - 在类模板外部定义成员函数的方法为：
        + `template<模板形参列表> 函数返回类型 类名<模板形参名>::函数名(参数列表){函数体}`
        + e.g. 两个模板形参T1，T2的类A中含有一个`void h()`函数，实现函数：`template<class T1,class T2> void A<T1,T2>::h(){}。`
        + 当在类外面定义类的成员时template后面的模板形参应与要定义的类的模板形参一致。

```cpp
template<class 形参名1, class 形参名2, ...>
class 类名
{
    ...
};
```

## 白杨的博客：[白杨的博客](http://baiy.cn/)

* 跨平台、分布式 C/C++ 开发，白杨：[白杨的博客](http://baiy.cn/)
    - 白杨：资深跨平台 C/C++ 开发，13 年以上创业经验，上海重玄科技创始人[白杨](https://cn.linkedin.com/in/%E6%9D%A8-%E7%99%BD-93063a45)
    - 看了一篇《C++编码规范与技术指导-何时处理异常》，直接将自己的错误处理代码风格(do...while(0))列为了反面教材，，(实际中设置错误码break退出然后返回给客户端，自我感觉是挺清晰的，主要便于资源统一释放而不用编写额外的RAII风格类。不过针对会抛异常可能导致段错误的块，确实需要额外捕获)
        + [代码风格与版式_异常](http://www.baiy.cn/doc/cpp/index.htm#%E4%BB%A3%E7%A0%81%E9%A3%8E%E6%A0%BC%E4%B8%8E%E7%89%88%E5%BC%8F_%E5%BC%82%E5%B8%B8)
* 上面记录虚函数表，涉及到本博客站中的一些C++的开销分析的文章，在笔记中搜索` C++的开销分析`
* 构造函数中的异常，若抛异常前有资源(指针、句柄)申请(析构中包含释放)，则对象没有完成创建也不会执行析构，就会泄漏资源
    - 更好的方法是使用一个满足“资源申请即初始化（`RAII`）”准则的类型（如：句柄类、灵巧指针类等等）来代替一般的资源申请与释放方式
    - e.g. 构造时由外面传入资源给指针(没传入时默认值是NULL)，析构中释放。并重载`operator=`(使用模板来支持通用性)，传入指针和自身相同则返回自身；若不同则释放自身资源，再用传入的资源。 (对于指针则可`直接使用智能指针`)
    - 参考链接中的示例：[构造函数中的异常](http://www.baiy.cn/doc/cpp/index.htm#%E4%BB%A3%E7%A0%81%E9%A3%8E%E6%A0%BC%E4%B8%8E%E7%89%88%E5%BC%8F_%E5%BC%82%E5%B8%B8)
    - 另外参考`new`失败时的异常，VC++6.0：[C++ New崩溃原理及解决方法](https://blog.csdn.net/chenqiai0/article/details/44601383)
* 其他规范摘录(作为一种风格的参考)
    - [文件结构_文件的组织结构](http://www.baiy.cn/doc/cpp/index.htm#%E6%96%87%E4%BB%B6%E7%BB%93%E6%9E%84_%E6%96%87%E4%BB%B6%E7%9A%84%E7%BB%84%E7%BB%87%E7%BB%93%E6%9E%84)
    - 类/结构
        + 类的名称都要以大写字母“C”开头，后跟一个或多个单词。为便于界定，每个单词的首字母要大写。
            * 特别地，由于界面与其它类概念上的巨大差别，规定界面类要以大写字母“I”开头。
        + 传统C结构体的名称全部由大写字母组成，单词间使用下划线界定，例如："SERVICE_STATUS", "DRIVER_INFO" ....
    - 函数
        + 函数的名称由一个或多个单词组成。为便于界定，每个单词的首字母要大写。
        + 函数名应当使用"动词"或者"动词＋名词"（动宾词组）的形式。例如："GetName()", "SetValue()", "Erase()", "Reserve()" ....
        + 保护成员函数的开头应当加上一个下划线`“_”`以示区别，例如："_SetState()" ....
        + 类似地，私有成员函数的开头应当加上两个下划线`“__”`，例如："__DestroyImp()" ....
        + 回调和事件处理函数习惯以单词“On”开头。例如："_OnTimer()", "OnExit()" ....
        + 回调函数以外的虚函数习惯以“Do”开头，如："DoRefresh()", "_DoEncryption()" ....
    - 变量
        + 变量名由作用域前缀＋类型前缀＋一个或多个单词组成。为便于界定，每个单词的首字母要大写。
        + 作用域前缀标明一个变量的可见范围。
            * 无  局部变量
            * m_  类的成员变量（member）
            * sm_ 类的静态成员变量（static member）
            * s_  静态变量（static）
            * g_  外部全局变量（global）
            * sg_ 静态全局变量（static global）
            * gg_ 进程或动态链接库间共享的全局变量（global global）
            * 除非不得已，否则应该尽可能少使用全局变量
* [C++0x（C++11）新特性点评](http://www.baiy.cn/doc/cpp/comments_of_cxx0x.htm)

## assert

* assert
    - assert`宏`(而不是函数) 其作用是如果它的条件返回错误，则终止程序执行
    - 如果`expression`其值为假（即为0），那么它先向`stderr`打印一条出错信息，然后通过调用 `abort` 来`终止程序`运行。
    - 使用assert的缺点是，频繁的调用会极大的影响程序的性能，增加额外的开销。(应该仅在Debug版本使用)
    - 在调试结束后，可以通过在包含`#include <assert.h>`的语句之前插入`#define NDEBUG`来禁用assert调用(编译时指定`-DNDEBUG`)

```cpp
// 原型
#include <assert.h>
void assert( int expression );

// 禁用，编译时选项 -DNDEBUG，或 包含头文件前指定
#include <stdio.h>
#define NDEBUG
#include <assert.h>
```

## 条件变量

* `std::condition_variable`
    - linux下的 `pthread_cond_t`，参考：[笔记](https://github.com/xiaodongQ/devNoteBackup/blob/master/%E5%90%84%E8%AF%AD%E8%A8%80%E8%AE%B0%E5%BD%95/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B.md)，(搜`## 条件变量(condition variable)`章节)
    - `#include <condition_variable>`
    - [C++11条件变量使用详解](https://blog.csdn.net/c_base_jin/article/details/89741247)
        + 注意里面的例子，在linux下编译时，添加`-std=c++11 -lpthread`
            * 不加`-pthread`会报错"Enable multithreading to use std::thread: Operation not permitted"
    - C++11引入，可以使用条件变量（condition_variable）实现多个线程间的同步操作；当条件不满足时，相关线程被一直阻塞，直到某种条件出现，这些线程才会被唤醒
    - API
        + 参考cpp reference：[std::condition_variable](https://zh.cppreference.com/w/cpp/thread/condition_variable)
        + `notify_one()` 通知一个等待的线程
        + `notify_all()` 通知所有等待的线程
        + `wait` 阻塞当前线程，直到条件变量被唤醒
            * `void wait( std::unique_lock<std::mutex>& lock );`
            * `void wait( std::unique_lock<std::mutex>& lock, Predicate pred );`
                - 其中Predicate为模板类`template< class Predicate >`
                - lock 参考：必须为当前线程所锁定
                - pred 参数：判定函数，若应该继续等待则返回false，即达到true的条件后就退出等待
                    + 判定函数的签名应等价于：`bool pred();` (可以使用lambda表达式)
                    + 中文cpp reference上此处翻译有点不通顺，参考en：[std::condition_variable::wait](https://en.cppreference.com/w/cpp/thread/condition_variable/wait)
                - 该`wait`等价于：`while(!pred()) { wait(lock);}`
            * 调用该函数前，需要获得锁
        + `wait_for` 阻塞当前线程，直到条件变量被唤醒，或到指定时限时长后结束阻塞
            * `std::cv_status wait_for( std::unique_lock<std::mutex>& lock, const std::chrono::duration<Rep, Period>& rel_time);` (其中Rep和Period都是模板的类型参数)
                - 若超出时限，则返回值为`std::cv_status::timeout`，否则为`std::cv_status::no_timeout`
            * `bool wait_for( std::unique_lock<std::mutex>& lock, const std::chrono::duration<Rep, Period>& rel_time, Predicate pred);` 相对上面的重载多了一个判定函数模板参数，且返回值为bool
                - 若超出时限后，判定函数pred仍然为false，则返回false，否则为true
            * [wait_for](https://zh.cppreference.com/w/cpp/thread/condition_variable/wait_for)
    - 利用线程间共享的全局变量进行同步，主要包括两个动作
        + 一个线程因等待"条件变量的条件成立"而挂起；
            * 等待条件成立使用的是`condition_variable类`成员`wait`、`wait_for` 或 `wait_until`
        + 另外一个线程使"条件成立"，给出信号，从而唤醒被等待的线程。
            * 给出信号使用的是`condition_variable类`成员`notify_one()`或者`notify_all()`函数
    - 为了防止竞争，条件变量的使用总是和一个互斥锁结合在一起；
        + 通常情况下这个锁是std::mutex
        + 并且管理这个锁 只能是 `std::unique_lock<std::mutex>` RAII模板类。
            * 以上两个类型的`wait`函数都在会阻塞时自动释放锁权限，即调用`unique_lock`的成员函数`unlock()`，以便其他线程能有机会获得锁，搜索："* `std::unique_lock`"

```cpp
#include <mutex>
#include <iostream>
#include <deque>
#include <thread>
#include <condition_variable>
using namespace std;

std::mutex g_cvMutex;
// 定义两个条件变量，生产和消费单独通知
std::condition_variable g_cv_produce, g_cv_consume;

//缓存区
std::deque<int> g_data_deque;
//缓存区最大数目
const int  MAX_NUM = 30;
//数据
int g_next_index = 0;

//生产者，消费者线程个数
const int PRODUCER_THREAD_NUM  = 3;
const int CONSUMER_THREAD_NUM = 3;

void producer_thread(int thread_id)
{
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
        //加锁
        std::unique_lock <std::mutex> lk(g_cvMutex);
         //当队列未满时，继续添加数据
        // while(g_data_deque.size() >= MAX_NUM)
        // {
        //     g_cv_produce.wait(lk);
        // }
        g_cv_produce.wait(lk, [](){ return g_data_deque.size() < MAX_NUM; });  //lambda表达式
        g_next_index++;
        g_data_deque.push_back(g_next_index);
        std::cout << "producer_thread: " << thread_id << " producer data: " << g_next_index;
        std::cout << " queue size: " << g_data_deque.size() << std::endl;
        //唤醒其他线程 
        g_cv_consume.notify_all();
        //自动释放锁
     }
}

void consumer_thread(int thread_id)
{
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(550));
        //加锁
        std::unique_lock <std::mutex> lk(g_cvMutex);
        //检测条件是否达成
        g_cv_consume.wait( lk,   []{ return !g_data_deque.empty(); });
        //互斥操作，消息数据
        int data = g_data_deque.front();
        g_data_deque.pop_front();
        std::cout << "\tconsumer_thread: " << thread_id << " consumer data: ";
        std::cout << data << " deque size: " << g_data_deque.size() << std::endl;
        //唤醒其他线程
        g_cv_produce.notify_all();
        //自动释放锁
    }
}


int main()
{
    std::thread arrRroducerThread[PRODUCER_THREAD_NUM];
    std::thread arrConsumerThread[CONSUMER_THREAD_NUM];

    for (int i = 0; i < PRODUCER_THREAD_NUM; i++)
    {
        arrRroducerThread[i] = std::thread(producer_thread, i);
    }

    for (int i = 0; i < CONSUMER_THREAD_NUM; i++)
    {
        arrConsumerThread[i] = std::thread(consumer_thread, i);
    }

    for (int i = 0; i < PRODUCER_THREAD_NUM; i++)
    {
        arrRroducerThread[i].join();
    }

    for (int i = 0; i < CONSUMER_THREAD_NUM; i++)
    {
        arrConsumerThread[i].join();
    }

    return 0;
}
```

* spdlog里实现的一个多生产者多消费者阻塞队列(multi producer-multi consumer blocking queue)
    - [mpmc_blocking_q](https://github.com/gabime/spdlog/blob/v1.x/include/spdlog/details/mpmc_blocking_q.h)
    - 源码如下(截取删除了mingw下的代码块，不同之处是加锁的范围不同，在mingw中过早释放貌似会有死锁)：
        + `circular_q`类实现了一个基于vector的循环队列，[circular_q.h源码](https://github.com/gabime/spdlog/blob/v1.x/include/spdlog/details/circular_q.h)
        + 另外spdlog中[details](https://github.com/gabime/spdlog/tree/v1.x/include/spdlog/details)目录下，还有不少值得分析学习的实现
            * 新建线程通过条件变量定期执行回调函数的功能类：[periodic_worker.h](https://github.com/gabime/spdlog/blob/v1.x/include/spdlog/details/periodic_worker.h)

* mpmc_blocking_q.h源码

```cpp
#include "spdlog/details/circular_q.h"

#include <condition_variable>
#include <mutex>

namespace spdlog {
namespace details {

template<typename T>
class mpmc_blocking_queue
{
public:
    using item_type = T;
    explicit mpmc_blocking_queue(size_t max_items)
        : q_(max_items)
    {
    }

#ifndef __MINGW32__
    // try to enqueue and block if no room left
    void enqueue(T &&item)
    {
        {
            std::unique_lock<std::mutex> lock(queue_mutex_);
            // 为队列full时等待，直到不为full
            pop_cv_.wait(lock, [this] { return !this->q_.full(); });
            // 传入的是右值引用
            q_.push_back(std::move(item));
        }
        push_cv_.notify_one();
    }

    // enqueue immediately. overrun oldest message in the queue if no room left.
    void enqueue_nowait(T &&item)
    {
        {
            std::unique_lock<std::mutex> lock(queue_mutex_);
            q_.push_back(std::move(item));
        }
        push_cv_.notify_one();
    }

    // try to dequeue item. if no item found. wait upto timeout and try again
    // Return true, if succeeded dequeue item, false otherwise
    bool dequeue_for(T &popped_item, std::chrono::milliseconds wait_duration)
    {
        {
            std::unique_lock<std::mutex> lock(queue_mutex_);
            if (!push_cv_.wait_for(lock, wait_duration, [this] { return !this->q_.empty(); }))
            {
                return false;
            }
            q_.pop_front(popped_item);
        }
        pop_cv_.notify_one();
        return true;
    }

#else
    // apparently mingw deadlocks if the mutex is released before cv.notify_one(),
    // so release the mutex at the very end each function.
#endif

    size_t overrun_counter()
    {
        std::unique_lock<std::mutex> lock(queue_mutex_);
        return q_.overrun_counter();
    }

private:
    std::mutex queue_mutex_;
    std::condition_variable push_cv_;
    std::condition_variable pop_cv_;
    spdlog::details::circular_q<T> q_;
};
} // namespace details
} // namespace spdlog
```

* periodic_worker.h源码
    - [periodic_worker.h](https://github.com/gabime/spdlog/blob/v1.x/include/spdlog/details/periodic_worker.h)

```cpp
#include <chrono>
#include <condition_variable>
#include <functional>
#include <mutex>
#include <thread>
namespace spdlog {
namespace details {

class periodic_worker
{
public:
    // 定期要执行的工作
    // 调用该函数后，类成员active_为true，此后每隔一定间隔interval执行一次回调函数callback_fun，
    // 若类成员active_变为false，则退出线程(不过类中只有析构的时候才会置为false)
    // 该类不允许拷贝构造和赋值，所以一直会定期执行，直到类的生命周期结束
    periodic_worker(const std::function<void()> &callback_fun, std::chrono::seconds interval)
    {
        // 传入定期间隔要>0秒，否则退出函数，不继续下面的创建线程
        // 此处会改变成员变量active_，感觉加个锁比较合适
        active_ = (interval > std::chrono::seconds::zero());
        if (!active_)
        {
            return;
        }

        // 创建线程
        // 使用lambda表达式定义一个匿名函数，捕获this并按值捕获函数传入的两个参数
        worker_thread_ = std::thread([this, callback_fun, interval]() {
            // 使用for形式的死循环，便于后续扩展
            for (;;)
            {
                std::unique_lock<std::mutex> lock(this->mutex_);
                // std::condition_variable::wait_for和wait的差别是：wait_for多了一个超时触发条件，达到超时也会结束等待，并需根据返回值区分处理
                // 判定函数返回true就退出等待
                // 分情况处理：
                    // 未达到超时的情况下：判定函数pred返回true (即active_为false)，则退出等待且wait_for返回值为true，
                        // 进入if语句块执行return，退出线程
                        // (这是由于wait_for到达时限时若pred仍然是false才返回false，其他情况都返回true，所以此处未到超时情况判定为true)
                    // 达到超时的情况下，则要再分支判断：
                        // 若pred判定为false(此处即active为true)，则wait_for返回false，不进入if语句块，往下执行callback_fun()回调
                        // 若pred判定为true(此处即active为false)，则执行return退出线程
                if (this->cv_.wait_for(lock, interval, [this] { return !this->active_; }))
                {
                    return; // active_ == false, so exit this thread
                }
                // 执行回调函数
                callback_fun();
            }
        });
    }

    periodic_worker(const periodic_worker &) = delete;
    periodic_worker &operator=(const periodic_worker &) = delete;

    // stop the worker thread and join it
    ~periodic_worker()
    {
        if (worker_thread_.joinable())
        {
            {
                std::lock_guard<std::mutex> lock(mutex_);
                active_ = false;
            }
            // 会解阻塞一个等待线程
            cv_.notify_one();
            worker_thread_.join();
        }
    }

private:
    bool active_;
    std::thread worker_thread_;
    std::mutex mutex_;
    std::condition_variable cv_;
};
} // namespace details
} // namespace spdlog
```


* 一个多生产者-多消费者的、无锁的 并发队列 (基于C++11)
    - star:4.4k fork:934 (20200509)
    - [cameron314/concurrentqueue](https://github.com/cameron314/concurrentqueue)

## 信号量

* [进程间通信之-信号量semaphore](https://blog.csdn.net/gatieme/article/details/50994533)
    - 信号量的使用主要是用来保护共享资源，使得资源在一个时刻只有一个进程（线程）所拥有

## lambda表达式

* lambda
    - [C++11 lambda表达式](http://c.biancheng.net/view/3741.html)
    - lambda 表达式是 C++11 最重要也最常用的一个特性之一，lambda 来源于函数式编程的概念
    - 优点
        + 声明式编程风格：就地匿名定义目标函数或函数对象，不需要额外写一个命名函数或者函数对象。以更直接的方式去写程序，好的可读性和可维护性
        + 简洁：不需要额外再写一个函数或者函数对象，避免了代码膨胀和功能分散，让开发者更加集中精力在手边的问题，同时也获取了更高的生产率
        + 在需要的时间和地点实现功能闭包，使程序更灵活
    - 概念和基本用法
        + lambda 表达式定义了一个匿名函数，并且可以捕获一定范围内的变量
            * 语法形式：`[ capture ] ( params ) opt -> ret { body; };`
                - capture 是捕获列表
                - params 是参数表
                - opt 是函数选项
                - ret 是返回值类型
                - body是函数体
            * e.g. `auto f = [](int a) -> int { return a + 1; };` 定义了一个功能闭包，用来将输入加 1 并返回
        + 返回值
            * 返回值是通过`返回类型后置`语法来定义的
                - [C++返回值类型后置（跟踪返回值类型）](http://c.biancheng.net/view/3727.html)
                - 返回类型后置（trailing-return-type，又称跟踪返回类型），C++11新增，将 decltype 和 auto 结合起来完成返回值类型的推导
                    + `template <typename T, typename U>`
                    + `auto add(T t, U u) -> decltype(t + u) {...}`
                - 返回值类型后置语法，是为了解决函数返回值类型依赖于参数而导致难以确定返回值类型的问题
                - 有了这种语法以后，对返回值类型的推导就可以用清晰的方式（直接通过参数做运算）描述出来，而不需要像 C++98/03 那样使用晦涩难懂的写法
                - 关于decltype关键字，查看章节：`### decltype`
            * C++11中允许省略lambda表达式的返回值(很多时候返回值比较明显)
                - `auto f = [](int a){ return a + 1; };`
                - 这样编译器就会根据 return 语句自动推导出返回值类型
            * 需要注意的是，初始化列表不能用于返回值的自动推导
                - `auto x1 = [](int i){ return i; };  // OK: return type is int`
                - `auto x2 = [](){ return { 1, 2 }; };  // error: 无法推导出返回值类型`
            * lambda 表达式在没有参数列表时，参数列表是可以省略的，下面的写法都是正确的：
                - `auto f1 = [](){ return 1; };`
                - `auto f2 = []{ return 1; };  // 省略空参数表`
        + 可以通过捕获列表捕获一定范围内的变量(即上面语法中的`[capture]`)
            - 捕获含义为在lambda函数体里是否能使用被捕获的变量
            - lambda 表达式的捕获列表精细地控制了 lambda 表达式能够访问的外部变量，以及如何访问这些变量
            - `[]` 不捕获任何变量
                + `auto f1 = []{ return a; };` // *error*，没有捕获外部变量
                    * lambda表达式外定义了：`int a = 0, b = 1;`
            - `[&]` 捕获外部作用域中所有变量，并作为引用在函数体中使用（按引用捕获）
                + `auto f2 = [&]{ return a++; };` // OK，捕获所有外部变量，并对a执行自加运算
            - `[=]` 捕获外部作用域中所有变量，并作为副本在函数体中使用（按值捕获）
                + `auto f3 = [=]{ return a; };`   // OK，捕获所有外部变量，并返回a
                + `auto f4 = [=]{ return a++; };` // *error*，a是以复制方式捕获的，无法修改
            - `[=，&foo]` 按值捕获外部作用域中所有变量，并按引用捕获 foo 变量
                + `auto f6 = [a, &b]{ return a + (b++); };` // OK，捕获a和b的引用，并对b做自加运算
                + `auto f7 = [=, &b]{ return a + (b++); };` // OK，捕获所有外部变量和b的引用，并对b做自加运算
            - `[bar]` 按值捕获 bar 变量，同时不捕获其他变量
                + `auto f5 = [a]{ return a + b; };`  // *error*，没有捕获变量b
            - `[this]` 捕获当前类中的 this 指针，让 lambda 表达式拥有和当前类成员函数同样的访问权限
                + 如果已经使用了 & 或者 =，就默认添加此选项。捕获 this 的目的是可以在 lamda 中使用当前类的成员函数和成员变量
                + `auto x4 = [this]{ return i_; };` // OK，捕获this指针(i_定义在类里面)
            - lambda 表达式的延迟调用(容易出错)
                + `int a=0;`, `auto f = [=]{ return a; };`, `a += 1;`, `cout << f() << endl;`，此时a还是0
                + 原因：在捕获的一瞬间，a 的值就已经被复制到f中了，之后 a 被修改，但此时 f 中存储的 a 仍然还是捕获时的值，因此，最终输出结果是 0
            - 定义lambda表达式定义时，此时所有被捕获的外部变量均被复制了一份存储在 lambda 表达式变量中
                + 如果希望 lambda 表达式在调用时能够即时访问外部变量，我们应当使用引用方式捕获
                + 或者使用`mutable`显式指定lambda为mutable
                    * `auto f1 = [=]{ return a++; };`            // *error*，修改按值捕获的外部变量
                    * `auto f2 = [=]() mutable { return a++; };` // OK，mutable
                        - *注意* 被 mutable 修饰的 lambda 表达式就算没有参数也要写明参数列表
    - lambda 表达式的类型
        + lambda 表达式的类型在 C++11 中被称为“闭包类型（Closure Type）”
        + 可以认为它是一个带有 `operator()` 的类，即*仿函数*
        + 对于没有捕获任何变量的 lambda 表达式，还可以被转换成一个普通的函数指针(捕获了则不行)
            * `using func_t = int(*)(int);` (入参int返回值int的函数指针)
            * `func_t f = [](int a){ return a; };`, `f(123);`
    - lambda表达式的价值在于，就地封装短小的功能闭包，可以极其方便地表达出我们希望执行的具体操作，并让上下文结合得更加紧密

## std::thread

* std::thread
    - cppreference: [std::thread](https://zh.cppreference.com/w/cpp/thread/thread)
    - `<thread>` C++11起
    - 类 thread 表示单个执行线程，线程在构造关联的线程对象时，立即开始执行
        + 若线程函数抛异常(若不能开始线程则会抛`std::system_error`异常)，则调用 std::terminate
    - 构造函数
        + 参考链接中的示例演示了几种不同的构造函数及其用法：
        + 默认构造：
            * `thread() noexcept;`
            * 使用默认构造创建的对象，并不表示线程
        + 给线程函数传参时可按值和按引用传参(若按引用传值，则需要包装传入的值，使用`std::ref()`或`std::cref()`来包装获取引用)
            * `explicit thread( Function&& f, Args&&... args );`
            * 两个包装模板，返回参数的引用或者const引用，参考：[std::ref, std::cref](https://zh.cppreference.com/w/cpp/utility/functional/ref)
        + 移动构造会转移原线程到新对象，原对象不再是线程了
            * `thread( thread&& other ) noexcept;`
        + 拷贝构造函数是被删除的，`= delete`
            * `thread(const thread&) = delete;`
            * thread 不可复制，没有两个`std::thread`对象可表示同一执行线程
        + 参考：[thread构造函数](https://zh.cppreference.com/w/cpp/thread/thread/thread)
    - `std::thread`对象可能处于不表示任何线程的状态，默认构造、被移动、 detach 或 join 后则不表示线程

## 内联

* [C++中的inline用法](https://www.cnblogs.com/fnlingnzb-learner/p/6423917.html)
* 在C/C++中，为了解决一些频繁调用的小函数大量消耗栈空间（栈内存）的问题，特别的引入了`inline`修饰符，表示为内联函数
* 在系统下，栈空间是有限的，假如频繁大量的使用就会造成因栈空间不足而导致程序出错的问题，如，函数的死循环递归调用的最终结果就是导致栈内存空间枯竭
* 如下示例
    - 在内部的工作就是在每个for循环的内部任何调用dbtest(i)的地方都换成了`(i%2>0)?”奇”:”偶”`，这样就避免了频繁调用函数对栈内存重复开辟所带来的消耗

```cpp
#include <stdio.h>
//函数定义为inline即:内联函数
inline char* dbtest(int a) {
    return (i % 2 > 0) ? "奇" : "偶";
}

int main()
{
   int i = 0;
   for (i=1; i < 100; i++) {
       printf("i:%d    奇偶性:%s /n", i, dbtest(i));
   }
}
```

* inline使用限制
    - inline只适合涵数体内代码简单的涵数使用，不能包含复杂的结构控制语句例如while、switch，并且不能内联函数本身不能是直接递归函数
    - inline函数仅仅是一个对编译器的建议，所以最后能否真正内联，看编译器的意思，它如果认为函数不复杂，能在调用点展开，就会真正内联，并不是说声明了内联就会内联，声明内联只是一个建议而已
* 类中的成员函数与`inline`
    - **定义在类中的成员函数缺省都是内联的**

```cpp
// example.h
class A
{
    public:void Foo(int x, int y) {  } // 自动地成为内联函数
}
```

* 将成员函数的定义体放在类声明之中虽然能带来书写上的方便，但不是一种良好的编程风格，上例应该改成：

```cpp
// example.h
class A
{
    public:
    void Foo(int x, int y);
}
```

```cpp
// example.cpp
inline void A::Foo(int x, int y)
{

}
```

* 关键字`inline`必须与函数定义体放在一起才能使函数成为内联
    - 仅将`inline`放在函数声明前面不起任何作用
    - 如上面例子中，需要在函数实现时用`inline`修饰
    - 对于声明时使用：`inline void Foo(int x, int y);` 并不能成为内联函数
    - 正确做法：
        + 声明：`void Foo(int x, int y);`
        + 实现：`inline void Foo(int x, int y) {}`
    - 所以说，`inline` 是一种“用于实现的关键字”，而不是一种“用于声明的关键字”
* 慎用`inline`
    - 内联是以代码膨胀（复制）为代价，仅仅省去了函数调用的开销，从而提高函数的执行效率
    - 如果执行函数体内代码的时间，相比于函数调用的开销较大，那么效率的收获会很少
    - 另一方面，每一处内联函数的调用都要复制代码，将使程序的总代码量增大，消耗更多的内存空间
    - 以下情况不宜使用内联
        + 如果函数体内的代码比较长，使用内联将导致内存消耗代价较高
        + 如果函数体内出现循环，那么执行函数体内代码的时间要比函数调用的开销大
        + 类的构造函数和析构函数容易让人误解成使用内联更有效
            * 要当心构造函数和析构函数可能会隐藏一些行为，如“偷偷地”执行了基类或成员对象的构造函数和析构函数
            * 所以*不要随便地将构造函数和析构函数的定义体放在类声明中* (如前面所说的：定义在类中的成员函数缺省都是内联的)(此处定义指的是：定义类时同时定义函数体)
            * 一个好的编译器将会根据函数的定义体，自动地取消不值得的内联（这进一步说明了 `inline` 不应该出现在函数的声明中）。

## localtime

* `struct tm *localtime(const time_t *timep);`
    - `#include <time.h>`
    - 把从1970-1-1零点零分到当前时间系统所偏移的秒数时间转换为本地时间
        + 此函数获得的tm结构体的时间是日历时间
        + `printf("%4d年%02d月%02d日 %02d:%02d:%02d\n",t->tm_year+1900,t->tm_mon+1,t->tm_mday,t->tm_hour,t->tm_min,t->tm_sec);`
    - 而`gmtime`函数转换后的时间没有经过时区变换，是UTC时间
    - 注意，`localtime`不是线程安全的，应该使用`localtime_r`(linux平台)
        + *应使用*： `struct tm *localtime_r(const time_t *timep, struct tm *result);`
        + `localtime`访问入参`timep`指向的对象指针，并且会访问和修改一个共享的内部对象，所以并发调用时可能会产生Data races，参考：[localtime](http://www.cplusplus.com/reference/ctime/localtime/)
* `struct tm *gmtime(const time_t *timep);`
    - `#include <time.h>`
    - 把日期和时间转换为格林威治(GMT)时间的函数
    - `localtime`函数获得的tm结构体的时间，是已经进行过时区转化为本地时间，而此函数功能类似获取当前系统时间，只是获取的时间未经过时区转换
    - 同`localtime`一样，*应使用*：
        + `struct tm *gmtime_r(const time_t *timep, struct tm *result);`

```cpp
// test_time.cpp 均使用线程安全的版本
#include <ctime>
#include <cstdio>

void cur_localtime(void)
{
    const char *wday[]={"Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"};
    time_t timep;
    struct tm t;
    time(&timep);
    if (NULL == localtime_r(&timep, &t)) /* 获取当前时间 */
    {
        printf("err");
        return;
    }
    printf("%d年%02d月%02d日",(1900+t.tm_year),(1+t.tm_mon),t.tm_mday);
    printf("%s %02d:%02d:%02d\n",wday[t.tm_wday],t.tm_hour,t.tm_min,t.tm_sec);
}

void cur_gmtime(void){
    const char *wday[]={"Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"};
    time_t timep;
    struct tm t;
    time(&timep);
    if (NULL == gmtime_r(&timep, &t)) /* 获取当前时间 */
    {
        printf("err");
        return;
    }
    printf("%d年%02d月%02d日",(1900+t.tm_year),(1+t.tm_mon),t.tm_mday);
    printf("%s %02d:%02d:%02d\n",wday[t.tm_wday],(t.tm_hour),t.tm_min,t.tm_sec);

    printf("\nafter hour +8:\n");
    printf("%s %02d:%02d:%02d\n",wday[t.tm_wday],(t.tm_hour+8),t.tm_min,t.tm_sec);
}

int main(int argc, const char *argv[])
{
    cur_localtime();

    printf("===============\n");
    cur_gmtime();
    return 0;
}

/*
    运行结果(当前北京时间2020.6.18 15:58:43)：

    2020年06月18日Thursday 15:58:43
    ===============
    2020年06月18日Thursday 07:58:43

    after hour +8:
    Thursday 15:58:43
*/
```

* 对于`localtime_r`比较常用，可定义一个内联函数供使用(参考spdlog内部命名空间中的实现)

```cpp
// localtime 封装线程安全的版本，定义在头文件中，内联
inline struct tm util_localtime(const time_t *timep) noexcept
{
#ifdef _WIN32
    struct tm _tm;
    localtime_s(&_tm, timep);
#else
    struct tm _tm;
    localtime_r(timep, &_tm);
#endif
    return _tm;
}
```

## 无锁编程

* [C++性能榨汁机之无锁编程](https://zhuanlan.zhihu.com/p/38664758)

使用锁：

```cpp
// test_mutex.cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>
#include <chrono>

using namespace std;
int  i = 0;
mutex mut; //互斥锁

void iplusplus() {
    int c = 10000000;  //循环次数
    while (c--) {
        mut.lock();  //互斥锁加锁
        i++;
        mut.unlock(); //互斥锁解锁
    }
}
int main()
{
    chrono::steady_clock::time_point start_time = chrono::steady_clock::now();//开始时间
    thread thread1(iplusplus);
    thread thread2(iplusplus);
    thread1.join();  // 等待线程1运行完毕
    thread2.join();  // 等待线程2运行完毕
    cout << "i = " << i << endl;
    chrono::steady_clock::time_point stop_time = chrono::steady_clock::now();//结束时间
    chrono::duration<double> time_span = chrono::duration_cast<chrono::microseconds>(stop_time - start_time);
    std::cout << "共耗时：" << time_span.count() << " ms" << endl; // 耗时
    // system("pause");
    return 0;
}

/*
    g++ test_mutex.cpp -std=c++11 -lpthread
    i = 20000000
    共耗时：0.958147 ms
*/
```

使用`atomic`：

```cpp
// test_atomic.cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>
#include <chrono>

using namespace std;
atomic<int> i = 0;

void iplusplus() {
    int c = 10000000;  //循环次数
    while (c--) {
        i++;
    }
}
int main()
{
    chrono::steady_clock::time_point start_time = chrono::steady_clock::now();//开始时间
    thread thread1(iplusplus);
    thread thread2(iplusplus);
    thread1.join();  // 等待线程1运行完毕
    thread2.join();  // 等待线程2运行完毕
    cout << "i = " << i << endl;
    chrono::steady_clock::time_point stop_time = chrono::steady_clock::now();//结束时间
    chrono::duration<double> time_span = chrono::duration_cast<chrono::microseconds>(stop_time - start_time);
    std::cout << "共耗时：" << time_span.count() << " ms" << endl; // 耗时
    // system("pause");
    return 0;
}

/*
    g++ test_atomic.cpp -std=c++11 -lpthread 编译报错：
    test_atomic.cpp:8:17: 错误：使用了被删除的函数‘std::atomic<int>::atomic(const std::atomic<int>&)’
    atomic<int> i = 0;
    In file included from test_atomic.cpp:4:0:
    /usr/include/c++/4.8.2/atomic:601:7: 错误：在此声明
       atomic(const atomic&) = delete;
*/
```

* [C++并发实战16: std::atomic原子操作](https://blog.csdn.net/liuxuejiang158/article/details/17413149)


## CPU亲和性 (绑定CPU)

* [C++性能榨汁机之CPU亲和性](https://zhuanlan.zhihu.com/p/57470627)
    - CPU亲和性就是绑定某一进程（或线程）到特定的CPU（或CPU集合），从而使得该进程（或线程）只能运行在绑定的CPU（或CPU集合）上
    - CPU亲和性利用了这样一个事实：进程上一次运行后的残余信息会保留在CPU的状态中（也就是指CPU的缓存）。如果下一次仍然将该进程调度到同一个CPU上，就能避免缓存未命中等对CPU处理性能不利的情况，从而使得进程的运行更加高效。
    - CPU亲和性分为两种：软亲和性和硬亲和性。
        + 软亲和性主要由操作系统来实现，Linux操作系统的调度器会倾向于保持一个进程不会频繁的在多个CPU之间迁移，通常情况下调度器都会根据各个CPU的负载情况合理地调度运行中的进程，以减轻繁忙CPU的压力，提高所有进程的整体性能
        + 除此以外，Linux系统还提供了硬亲和性功能，即用户可以通过调用系统API实现自定义进程运行在哪个CPU上，从而满足特定进程的特殊性能需求
    - 如何将CPU亲和性应用到程序中？
        + Linux系统中每个进程的`task_struct`结构(sched.h中定义)中有一个`cpus_allowed` 位掩码，该掩码的位数与系统CPU核数相同（若CPU启用了超线程则为核数乘以2），通过修改该位掩码可以控制进程可运行在哪些特定CPU上
        + Linux系统为我们提供了CPU亲和性相关的调用函数和一些操作的宏定义，函数主要是下面两个：
            * `sched_setaffinity()` （修改位掩码）
            * `sched_getaffinity()` （查看当前的位掩码）
        + 除此之外还提供了一些宏定义来修改掩码，如`CPU_ZERO()`(将位掩码全部设置为0)和`CPU_SET()`(设置特定掩码位为1)。
    - CPU亲和性的应用场景
        + 假如某些进程需要高密度的计算，不希望被频繁调度，则可以使用CPU亲和性将该进程绑定到一个CPU上；
        + 在股票期货高频交易场景中，交易策略线程的运行时间关系到交易延迟的大小，而交易延迟1ms的差距可能就是赚钱与亏钱的差距，所以交易策略线程的优先级非常高，这时便可以为其分配一个专门用于策略计算的CPU，以避免线程被调度产生性能损失；
        + 高性能的Nginx采用多线程模型，并且提供了worker进程绑定固定CPU的功能，降低worker进程被调度的损耗，提高了服务器工作性能；
        + 一些文献中还提到了应用CPU亲和性优化KVM虚拟化技术的性能，在不减少吞吐量的情况下，可以将KVM的网络延迟性能降低20%；
    - 一般情况下，Linux系统的进程调度器已经做得足够好，不需要我们干预进程的调度，但是系统的进程调度是面向所有应用程序的，势必会为了通用性而牺牲掉一部分性能，对于特定应用程序而言，我们可以通过CPU亲和性去优化程序的性能表现。

```cpp
// test_cpu_affinity.cpp
// g++ test_cpu_affinity.cpp -std=c++11
#include <iostream>
#include <thread>
#include<stdlib.h>
#include<stdio.h>
#include<sys/types.h>
#include<sys/sysinfo.h>
#include<unistd.h>

#define __USE_GNU
#include<sched.h>
#include<ctype.h>
#include<string.h>
#include<pthread.h>

using namespace std;

/* This method will create processes, then bind each to its own cpu. */
void do_cpu_stress(int num_of_process)
{
    int created_process = 0;
    /* We need a process for each cpu we have... */
    while ( created_process < num_of_process - 1 )
    {
        int mypid = fork();
        if (mypid == 0) /* Child process */
        {
            break;
        }
        else /* Only parent executes this */
        {
            /* Continue looping until we spawned enough processes! */ ;
            created_process++;
        }
    }
    /* NOTE: All processes execute code from here down! */
    cpu_set_t mask;
    /* CPU_ZERO initializes all the bits in the mask to zero. */
    CPU_ZERO( &mask );
    /* CPU_SET sets only the bit corresponding to cpu. */
    CPU_SET(created_process, &mask );
    /* sched_setaffinity returns 0 in success */
    if( sched_setaffinity( 0, sizeof(mask), &mask ) == -1 ){
        cout << "WARNING: Could not set CPU Affinity, continuing..." << endl;
    }
    else{
        cout << "Bind process #" << created_process << " to CPU #" << created_process << endl;
    }
    //do some cpu expensive operation
    int cnt = 100000000;
    while(cnt--){
        int cnt2 = 10000000;
        while(cnt2--){
        }
    }
}

int main(){
    int num_of_cpu = thread::hardware_concurrency();
    cout << "This PC has " << num_of_cpu << " cpu." << endl;
    do_cpu_stress(num_of_cpu);
}
```

## jsoncpp

* [C++中使用JsonCpp](https://www.jianshu.com/p/987e95cc79f4)
    - 从github下载代码，`git clone https://github.com/open-source-parsers/jsoncpp.git`
        + MIT license，宽松式许可证，对用户几乎没有限制。可以修改代码后闭源。必须保留原始的许可证声明。
    - 取需要的文件合到自己的项目里(源码方式使用)：
        + `include/json` 头文件
        + `src/lib_json` 源文件
    - 使用
        + `#include "json/json.h"`
        + 创建Json对象
        + Json对象序列化为字符串

```cpp
// main.cpp
#include <iostream>
#include <string>
#include <cstdio>
#include "json/json.h"
using namespace std;

int main(int argc, const char *argv[])
{
    Json::Value root;
    Json::Reader reader;
    string info = "{\"sign_type\":14,\"dir_type\":1}";
    if (reader.parse(info, root))
    {
        if (!root["sign_type"].isNull())
        {
            printf("sign_type:%s\n", root["sign_type"].asString().c_str());
            printf("sign_type:%d\n", root["sign_type"].asInt64());
        }
        if (!root["dir_type"].isNull())
        {
            printf("dir_type:%s\n", root["dir_type"].asString().c_str());
            printf("dir_type:%d\n", root["dir_type"].asInt64());
        }

        /*
          // json数组解析
          std::string strValue = "{\"key\":\"value1\",\
            \"array\":[{\"arraykey\":1},{\"arraykey\":2}]}";
          Json::Value arrayObj = root["array"];
          for (int i=0; i<arrayObj.size(); i++)
          {
            int iarrayValue = arrayObj[i]["arraykey"].asInt();
            std::cout << iarrayValue << std::endl;
          }
        */
    }
    return 0;
}
```

* makefile
    - 目录层次：`include(目录，里面包含头文件目录json) lib_json(目录)  main.cpp  Makefile`

```makefile
CC = g++

SRC_DIR = $(shell find . -type d -print)
$(warning SRC_DIR: [$(SRC_DIR)])

SRC_CPP_FILES = $(foreach dir,$(SRC_DIR),$(wildcard $(dir)/*.cpp))
$(warning SRC_CPP_FILES: [$(SRC_CPP_FILES)])

SRC_OBJ = $(patsubst %.cpp, %.o, $(SRC_CPP_FILES))
FLAGS = -I./include -std=c++11

jsontest:$(SRC_OBJ)
    $(CC) -o $@ $^ $(FLAGS)
%.o:%.cpp
    $(CC) -o $@ -c $< $(FLAGS)
clean:
    rm -rf *.o a.out
```

## dlopen

* dlopen实现插件式加载动态库

```cpp
// 定义函数签名的别名
typedef void  (*pfunc1)(const char* param);
typedef char*  (*pfunc2)(const char* param);

// 定义函数指针，加载动态库后可直接使用，e.g. g_func1("abc")
pfunc1 g_func1 = NULL;
pfunc2 g_func2 = NULL;

int initQuantApi()
{
    const char* szDllPathx64 = "./lib/libTestAPIx64.so";
    const char* szDllPathx86 = "./lib/libTestAPI.so";
    // 根据情况选择加载哪个动态库
    const char* szDllPath = szDllPathx86;
    if (__WORDSIZE == 64)
    {
        szDllPath = szDllPathx64;
    }

    void* pHandle = dlopen(szDllPath, RTLD_LAZY);
    void* Error = dlerror();
    if (Error)
    {
        LOGGER_ERROR("\n\nOpen Dll failed. error:[{}]",Error);
        return -1;
    }
    g_func1 = (pfunc1)dlsym(pHandle, "func1");
    g_func2 = (pfunc2)dlsym(pHandle, "func2");

    Error = dlerror();
    if (Error)
    {
        dlclose(pHandle);
        LOGGER_ERROR("\n\nDll sym failed. Error:[{}}]",Error);
        return -1;
    }

    return 0;
}
```

## 常用数学函数

* [常用数学函数](https://zh.cppreference.com/w/cpp/numeric/math)
* `std::round` 计算最接近arg的整数值(四舍五入)
    - 头文件：`<cmath>`
    - 以浮点格式表示
        + `float       round ( float arg );`  (C++11 起)
        + `double      round ( double arg );` (C++11 起)
    - 以整数格式表示
        + `long lround ( float arg );`  (C++11 起)
        + `long lround ( double arg );` (C++11 起)
    - 示例
        + `round(+2.3) = 2`  `round(+2.5) = 3`  `round(+2.7) = 3`
        + `round(-2.3) = -2`  `round(-2.5) = -3`  `round(-2.7) = -3`
* `std::floor` 计算不大于arg的最大整数值
    - 头文件：`<cmath>`
    - `float       floor ( float arg );`
    - `double      floor ( double arg );`
    - 示例
        + `floor(+2.7) = 2.000000`
        + `floor(-2.7) = -3.000000`
        + `floor(-0.0) = -0.000000`
* `std::ceil` 不小于arg的最小整数值
    - 头文件：`<cmath>`
    - `float       ceil ( float arg );`
    - `double      ceil ( double arg );`
    - 示例
        + `ceil(+2.4) = 3.000000`
        + `ceil(-2.4) = -2.000000`
        + `ceil(-0.0) = -0.000000`
* `std::abs(float)` 计算浮点值arg的绝对值
    - 头文件：`<cmath>`
    - `float       abs( float arg );`
    - `double      abs( double arg );`
    - 示例
        + `abs(+3.0) = 3`
        + `abs(-3.0) = 3`
* `std::abs(int)` 计算整数的绝对值
    - 头文件 `<cstdlib>`
        + C++17起，`<cmath>`也支持
    - `int       abs( int n );`
    - `long      abs( long n );`
    - [std::abs(int), std::labs, std::llabs, std::imaxabs](https://zh.cppreference.com/w/cpp/numeric/math/abs)

## ctest

* ctest 单元测试

## Effective C++

* 条款02：尽量以`const`，`enum`，`inline`替换`#define`
    - 对于单纯常量，最好以const对象或enums替换#defines
    - 对于形似函数的宏(macros)，最好改用inline函数替换#defines
* 条款03：尽可能使用const
    - 如果关键字const出现在星号左边，表示所指事物是常量
        + `const char* p = greeting;` 指向数据不可变，指针指向可变
            * 其中，greeting变量为`char greeting[]="hello"`
    - 如果出现在星号右边，表示指针自身是常量
        + `char* const p = greeting` 指针指向不可变，指向数据可变
    - 如果出现在星号两边，表示被指事物和指针两者都是常量
        + `const char* const p = greeting` 指针指向和指向数据都不可变

## `__attribute__`

* [C 语言中__attribute__的作用](https://winddoing.github.io/post/12087.html)
    - attribute：属性，主要是用来在函数或数据声明中设置其属性，与编译器相关
    - GNU C 的一大特色就是`__attribute__`机制
    - `__attribute__`可以设置函数属性（Function Attribute）、变量属性（Variable Attribute）和类型属性（Type Attribute）
    - 语法格式：
        + `__attribute__ ((attribute-list))`
        + 放于声明的尾部`;`之前(`struct`关键字之后 到 `;`之前)
            * e.g. redis中定义packed为16位的sds：`struct __attribute__ ((__packed__)) sdshdr16 { uint16_t len; xxx};`
    - 数据声明
        + `__attribute__ ((packed))`
            * 告诉编译器取消结构在编译过程中的优化对齐，按照实际占用字节数进行对齐，是 GCC 特有的语法
            * 该属性可以使得变量或者结构体成员使用最小的对齐方式，即对变量是一字节对齐，对域（field）是位对齐
            * e.g. `struct __attribute__ ((__packed__)) sc3 {char a; char *b;};`
                - `sc3`结构体的sizeof为9
                - 如果不加`__attribute__ ((__packed__))`，则sizeof为16
        + `__attribute__((aligned(n)))`
            * 内存对齐，指定内存对齐 n 字节
            * 该属性设定一个指定大小的对齐格式（以字节 为单位）
                - e.g. `struct S {short b[3];} __attribute__ ((aligned (8)));`
                    + 该声明将强制编译器让变量类型为`struct S`的变量在分配空间时采用8字节对齐方式，sizeof为8
                - e.g. `struct __attribute__ ((aligned(4))) sc5 {char a;char *b;};`
                    + sizeof为16 (1+8 1对齐成4，4+8，然后补齐成16)
                - e.g. `struct __attribute__ ((aligned(4))) sc6 {char a;char b[];};`
                    + sizeof为4 (1+字符数组，直接补齐为4)
            * 可以手动指定对齐的格式，也可以使用默认的对齐方式
                - 如果`aligned`后面不紧跟一个指定的数字值，那么编译器将依据你的目标机器情况使用最大最有益的对齐方式
                - e.g. `struct S {short b[3];} __attribute__ ((aligned));`
                - 如果`sizeof(short)`的大小为2（byte），那么，S的大小就为6。取一个2的次方值，使得该值大于等于6，则该值为8，所以编译器将设置S类型的对齐方式为`8`字节
            * `aligned`属性使被设置的对象占用更多的空间，相反的，使用`packed`可以减小对象占用的空间
                - 注意：`attribute`属性的效力与连接器也有关，如果连接器最大只支持16字节对齐，那么定义32字节对齐也无济于事
    - 函数声明
        + `__attribute__((noreturn))`
            * 告诉编译器这个函数不会返回给调用者，以便编译器在优化时去掉不必要的函数返回代码
            * e.g. `extern void exit(int) __attribute__((noreturn));`
        + `__attribute__((weak))`
            * 虚函数，弱符号
            * e.g. `int __attribute__((weak)) func(...){ ... return 0;}`
                - func 转成弱符号类型
                - 如果遇到强符号类型（即外部模块定义了 func, extern int func(void);），那么我们在本模块执行的 func 将会是外部模块定义的 func
                - 如果外部模块没有定义，那么将会调用这个弱符号，也就是在本地定义的 func
            * 链接器发现同时存在弱符号和强符号，就先选择强符号，如果发现不存在强符号，只存在弱符号，则选择弱符号
                - weak 属性只会在静态库 (.o .a) 中生效，动态库 (.so) 中不会生效

## std::move 和 std::forward

* 关于右值引用，前面讲移动构造时也有涉及，查看小节：`* 右值引用和移动语义，移动构造函数和移动赋值运算符的说明`

* [《C++0x漫谈》系列之：右值引用(或“move语意与完美转发”)(上)](https://blog.csdn.net/pongba/article/details/1684519)
    - 所谓“moveable”即是指（当源对象是临时对象时）在对象拷贝语法之下进行的实际动作是像auto_ptr那样的资源所有权转移：源对象被掏空，所有资源都被转移到目标对象中——好比一次搬家（move）
* [《C++0x漫谈》系列之：右值引用(或“move语意与完美转发”)(下)](https://blog.csdn.net/pongba/article/details/1697636)


* [C++11朝码夕解: move和forward](https://zhuanlan.zhihu.com/p/55856487)
    - C++11前的状况: 没法避免临时变量的copy
        + C++传值默认是copy
        + copy开销很大
    - 如下面例子, 它们都要经历至少一次复制操作
        + func("some temporary string"); //初始化string, 传入函数, 可能会导致string的复制
        + v.push_back(X()); //初始化了一个临时X, 然后被复制进了vector
        + a = b + c; //b+c是一个临时值, 然后被赋值给了a
        + x++; //x++操作也有临时变量的产生
        + a = b + c + d; //c+d一个临时变量, b+(c+d)另一个临时变量
    - 这些临时变量在C++11里被定义为`rvalue`, `右值`, 因为没有对应的变量名存它们
        + 同时有对应变量名的被称为`lvalue`, `左值`
    - C++11: 引入rvalue, lvalue和move
        + 于是就引入了rvalue和lvalue的概念, 之前说的那些临时变量就是rvalue. 上面说的避免copy的操作就是`std::move`
        + 传临时变量的时候, 可以传`T &&`, 叫`rvalue reference`(右值引用), 它能接收`rvalue`(临时变量), 之后再调用`std::move`就避免copy了，如下面的`右值引用示例`

* 示例

```cpp
class A{...};
void A::set(const string & var1, const string & var2){
  m_var1 = var1;  //copy
  m_var2 = var2;  //copy
}

A a1;
// 临时生成了2个string, 传进set函数里, 复制给成员变量, 然后这两个临时string再被回收
a1.set("temporary str1","temporary str2");

// 临时变量反正都要被回收, 如果能直接把临时变量的内容, 和成员变量内容交换一下, 就能避免复制了
    // (1)成员变量内部的指针指向"temporary str1"所在的内存
    // (2)临时变量内部的指针指向成员变量以前所指向的内存
    // (3)最后临时变量指向的那块内存再被回收
// 这就是所谓的move语义
```

* 右值引用示例

```cpp
void set(string && var1, string && var2){
  //avoid unnecessary copy!
  m_var1 = std::move(var1);
  m_var2 = std::move(var2);
}

A a1;
//temporary, move! no copy!
a1.set("temporary str1","temporary str2");
```

* 新的问题: 避免重复
    - 像上面处理临时变量用右值引用`string &&`, 处理普通变量用const引用`const string &`，有代码重复问题
    - `perfect forward` (完美转发)
        + 上面说的各种情况, 包括传`const T &`, `T &&`, 都可以由`std::forward`代替
    - forward能转发下面所有的情况(`[const] T &[&]`)，即:
        + `const T &`
        + `T &`
        + `const T &&`
        + `T &&`
    - 如果外面传来了rvalue临时变量, 它就转发rvalue并且启用move语义
    - 如果外面传来了lvalue, 它就转发lvalue并且启用复制. 然后它也还能保留const
* 有了forward为什么还要用move?
    - 首先, forward常用于template函数中, 使用的时候必须要多带一个template参数T: forward<T>, 代码略复杂
    - 明确只需要move的情况而用forward, 代码意图不清晰, 其他人看着理解起来比较费劲

```cpp
template<typename T1, typename T2>
void set(T1 && var1, T2 && var2){
  m_var1 = std::forward<T1>(var1);
  m_var2 = std::forward<T2>(var2);
}
```
