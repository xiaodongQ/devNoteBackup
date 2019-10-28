
### C#

[C# 教程](https://www.runoob.com/csharp/csharp-tutorial.html)

C# 是一个简单的、现代的、通用的、面向对象的编程语言，它是由微软（Microsoft）开发的。
C# 编程是基于 C 和 C++ 编程语言的

 C# 是 .Net 框架的一部分，且用于编写 .Net 应用程序。

.cs CS文件扩展名是一个C＃源代码文件类型


当一个值类型转换为对象类型时，则被称为 装箱；另一方面，当一个对象类型转换为值类型时，则被称为 拆箱。

#### 类型

动态（Dynamic）类型
dynamic d = 20;

字符串（String）类型

* C# 封装
internal：同一个程序集的对象可以访问；
protected internal：访问限于当前程序集或派生自包含类的类型。

* C# 可空类型（Nullable）
可空类型可以表示其基础值类型正常范围内的值，再加上一个 null 值。(取值范围之外，加上null)

int? num1 = null;
int? num2 = 45;

打印为空 (Console.WriteLine)

* 空合并运算符(??)：用于定义可空类型和引用类型的默认值。
a??b,当a为null时则返回b,a不为空时返回a本身。

  空合并运算符为右结合运算符，即操作时从右向左进行组合的。如，“a??b??c”的形式按“a??(b??c)”计算。

* 数组
声明数组使用下面的语法
double[] balance;

* 字符串
string 关键字是 System.String 类的别名。 (调用方法还是String)

string fname, lname;
fname = "Rowan";

string[] sarray = { "Hello", "From", "Tutorials", "Point" };
string message = String.Join(" ", sarray);

string chat = String.Format("Message sent at {0:t} on {0:D}", waiting);

#### 循环

C# 也支持 foreach 循环，使用foreach可以迭代数组或者一个集合对象。

```csharp
 int[] fibarray = new int[] { 0, 1, 1, 2, 3, 5, 8, 13 };
  foreach (int element in fibarray)
  {
      System.Console.WriteLine(element);
  }
```

#### 方法

* 方法的参数传递
  - 值参数
    + 当形参的值发生改变时，不会影响实参的值
  - 引用参数
    + 这种方式复制参数的内存位置的引用给形式参数。
    + 当形参的值发生改变时，同时也改变实参的值
    + 在 C# 中，使用 ref 关键字声明引用参数
        public void swap(ref int x, ref int y)
        swap 函数内的值改变了
  - 输出参数
    + 这种方式可以返回多个值
    + 输出参数会把方法输出的数据赋给自己，其他方面与引用参数相似。
       public void getValue(out int x )
       {
         int temp = 5;
         x = temp;
       }
        提供给输出参数的变量不需要赋值

#### 多态

C# 不支持多重继承, 可通过使用接口来实现多重继承

* C# 允许您使用关键字 abstract 创建抽象类
通过在类定义前面放置关键字 sealed，可以将类声明为密封类。当一个类被声明为 sealed 时，它不能被继承。抽象类不能被声明为 sealed。

* 重写：(需要override关键字??)
public override int area ()
{
 Console.WriteLine("Rectangle 类的面积：");
 return (width * length);
}

* 虚方法是使用关键字 virtual 声明的。
虚方法可以在不同的继承类中有不同的实现。
对虚方法的调用是在运行时发生的。
动态多态性是通过 抽象类 和 虚方法 实现的。

* 运算符重载
通过关键字 operator 后跟运算符的符号来定义的。

```cpp
public static Box operator+ (Box b, Box c)
{
   Box box = new Box();
   box.length = b.length + c.length;
   box.breadth = b.breadth + c.breadth;
   box.height = b.height + c.height;
   return box;
}
```

可重载和不可重载运算符

#### C# 接口（Interface）

接口使用 interface 关键字声明，它与类的声明类似。接口声明默认是 public 的。

```cpp
using System;

interface IMyInterface
{
        // 接口成员
    void MethodToImplement();
}

class InterfaceImplementer : IMyInterface
{
    static void Main()
    {
        InterfaceImplementer iImp = new InterfaceImplementer();
        iImp.MethodToImplement();
    }

    public void MethodToImplement()
    {
        Console.WriteLine("MethodToImplement() called.");
    }
}
```

如果一个接口继承其他接口，那么实现类或结构就需要实现所有接口的成员。

#### 命名空间

命名空间的设计目的是提供一种让一组名称与其他名称分隔开的方式
命名空间的定义是以关键字 namespace 开始，后跟命名空间的名称，

using 关键字表明程序使用的是给定命名空间中的名称。

using System;
using first_space;
using second_space;

命名空间可以被嵌套

#### C# 预处理器指令

在 C# 中，预处理器指令用于在条件编译中起作用。
与 C 和 C++ 不同的是，它们不是用来创建宏。
一个预处理器指令必须是该行上的唯一指令。

```cpp
#line    它可以让您修改编译器的行数以及（可选地）输出错误和警告的文件名。
#error   它允许从代码的指定位置生成一个错误。
#warning 它允许从代码的指定位置生成一级警告。
#region  它可以让您在使用 Visual Studio Code Editor 的大纲特性时，指定一个可展开或折叠的代码块。
#endregion 它标识着 #region 块的结束。
```

#### C# 正则表达式
Regex 类

#### 异常处理

try...catch...finally...throw

C# 中的异常类，C# 中的异常类主要是直接或间接地派生于 System.Exception 类

System.ApplicationException 类支持由应用程序生成的异常。所以程序员定义的异常都应派生自该类。
System.SystemException 类是所有预定义的系统异常的基类。

使用 throw 语句来抛出当前的对象

#### 文件输入输出

C# I/O 类
System.IO 命名空间有各种不同的类，用于执行各种文件操作，如创建和删除文件、读取或写入文件，关闭文件等。
    System.IO 命名空间中的 FileStream 类有助于文件的读写与关闭。该类派生自抽象类 Stream。

```csharp
例如，创建一个 FileStream 对象 F 来读取名为 sample.txt 的文件：

FileStream F = new FileStream("sample.txt", FileMode.Open, FileAccess.Read, FileShare.Read);
```

#### C# 特性（Attribute）
特性（Attribute）是用于在运行时传递程序中各种元素（比如类、方法、结构、枚举、组件等）的行为信息的**声明性**标签。

一个声明性标签是通过放置在它所应用的元素前面的方括号（[ ]）来描述的。

特性（Attribute）的名称和值是在方括号内规定的，放置在它所应用的元素之前。

.Net 框架提供了两种类型的特性：预定义特性和自定义特性。
    预定义特性（Attribute）.Net 框架提供了三种预定义特性：
        AttributeUsage Conditional Obsolete

#### C# 反射（Reflection）
反射指 程序可以访问、检测和修改它本身状态或行为的一种能力。

System.Reflection.MemberInfo info = typeof(MyClass);

#### C# 属性（Property）
属性（Property） 是类（class）、结构（structure）和接口（interface）的命名（named）成员

类或结构中的成员变量或方法称为 域（Field）。

属性（Property）是域（Field）的扩展，且可使用相同的语法来访问。它们使用 访问器（accessors） 让私有域的值可被读写或操作。

* 访问器（Accessors）

属性（Property）的访问器（accessor）包含有助于获取（读取或计算）或设置（写入）属性的可执行语句。
    访问器（accessor）声明可包含一个 get 访问器、一个 set 访问器，或者同时包含二者。

```csharp
// 声明类型为 int 的 Age 属性
public int Age
{
   get
   {
      return age;
   }
   set
   {
      age = value;
   }
}
```


```csharp
using System;
namespace runoob
{
   class Student
   {
      private int age = 0;

      // 声明类型为 int 的 Age 属性
      public int Age
      {
         get
         {
            return age;
         }
         set
         {
            age = value;
         }
      }
      public override string ToString()
      {
         //xdcomment 会用到属性的get访问器
         return "Code = " + Code +", Name = " + Name + ", Age = " + Age;
      }
    }
    class ExampleDemo
    {
      public static void Main()
      {
         // 创建一个新的 Student 对象
         Student s = new Student();

         // 设置 student 的 code、name 和 age
         //xdcomment 会用到属性的set访问器
         s.Age = 9;
         Console.WriteLine("Student Info: {0}", s);
         // 增加年龄
         s.Age += 1;
         Console.WriteLine("Student Info: {0}", s);
         Console.ReadKey();
       }
   }
}
```

抽象属性（Abstract Properties）

抽象类可拥有抽象属性，这些属性应在派生类中被实现。

```csharp
public abstract class Person
{
  public abstract string Name
  {
     get;
     set;
  }
  public abstract int Age
  {
     get;
     set;
  }
}

 class Student : Person
{
  private string code = "N.A";
  private string name = "N.A";
   // 声明类型为 string 的 Name 属性
  public override string Name
  {
     get
     {
        return name;
     }
     set
     {
        name = value;
     }
  }
}
```

#### C# 索引器（Indexer）

索引器（Indexer） 允许一个对象可以像数组一样被索引。
索引器的行为的声明在某种程度上类似于属性（property）。可使用 get 和 set 访问器来定义索引器。

索引器定义的时候不带有名称，但带有 this 关键字，它指向对象实例。

```csharp
class IndexNames
{
    private string[] namelist = new string[size];
    public string this[int index]
    {
        get
        {
           string tmp;
           if( index >= 0 && index <= size-1 )
           {
               tmp = namelist[index];
           }
           else
           {
               tmp = "";
           }
           return ( tmp );
        }
        set
        {
            if( index >= 0 && index <= size-1 )
            {
                namelist[index] = value;
            }
        }
    }
}
```

#### C# 委托（Delegate）
C# 中的委托（Delegate）类似于 C 或 C++ 中函数的指针。
委托（Delegate） 是存有对某个 *方法* 的引用的一种引用类型变量。引用可在运行时被改变。

委托（Delegate）特别用于实现事件和回调方法。所有的委托（Delegate）都派生自 System.Delegate 类。

```csharp
public delegate int MyDelegate (string s);
```

上面的委托可被用于引用任何一个带有一个单一的 string 参数的方法，并返回一个 int 类型变量。

* 实例化委托
一旦声明了委托类型，委托对象必须使用 new 关键字来创建，且与一个特定的方法有关。

当创建委托时，传递到 new 语句的参数就像方法调用一样书写，但是不带有参数。

```csharp
static int num = 10;
public static int AddNum(int p)
{
   num += p;
   return num;
}

delegate int NumberChanger(int n);

// 创建委托实例
NumberChanger nc1 = new NumberChanger(AddNum);
// 使用委托对象调用方法
nc1(25);
```

#### C# 事件（Event）
C# 中使用事件机制实现线程间的通信。
事件使用 发布-订阅（publisher-subscriber） 模型。

发布器（publisher） 类
    事件在类中声明且生成，且通过使用同一个类或其他类中的委托与事件处理程序关联。包含事件的类用于发布事件。
    发布器（publisher） 是一个包含事件和委托定义的对象。
    事件和委托之间的联系也定义在这个对象中。
    发布器（publisher）类的对象调用这个事件，并通知其他的对象。

订阅器（subscriber）
    其他接受该事件的类被称为 订阅器（subscriber） 类
    订阅器（subscriber） 是一个接受事件并提供事件处理程序的对象。
    在发布器（publisher）类中的委托调用订阅器（subscriber）类中的方法（事件处理程序）。

```csharp
 // 事件发布器
   class DelegateBoilerEvent
   {
      //在类的内部声明事件，首先必须声明该事件的委托类型。
      public delegate void BoilerLogHandler(string status);

      // 基于上面的委托定义事件
      //代码定义了一个名为 BoilerLogHandler 的委托和一个名为 BoilerEventLog 的事件，该事件在生成的时候会调用委托。
      public event BoilerLogHandler BoilerEventLog;

      public void LogProcess()
      {
         OnBoilerEventLog("Logging Info:\n");
         OnBoilerEventLog("\nMessage: ");
      }

      protected void OnBoilerEventLog(string message)
      {
         if (BoilerEventLog != null)
         {
            BoilerEventLog(message);
         }
      }
   }

   // 事件订阅器
   public class RecordBoilerInfo
   {
      static void Logger(string info)
      {
         Console.WriteLine(info);
      }//end of Logger

      static void Main(string[] args)
      {
        /* 实例化对象,第一次没有触发事件 */
         DelegateBoilerEvent boilerEvent = new DelegateBoilerEvent();
         /* 注册, 创建Logger的委托，事件触发时，执行委托的方法*/
         boilerEvent.BoilerEventLog += new DelegateBoilerEvent.BoilerLogHandler(Logger);
         /* 触发事件 */
         boilerEvent.LogProcess();
      }//end of main
    }
```

#### C# 集合（Collection）

集合（Collection）类是专门用于数据存储和检索的类
这些类提供了对栈（stack）、队列（queue）、列表（list）和哈希表（hash table）的支持。大多数集合类实现了相同的接口。

动态数组（ArrayList）
    它基本上可以替代一个数组。
    与数组不同的是，您可以使用索引在指定的位置添加和移除项目，动态数组会自动重新调整它的大小。
哈希表（Hashtable）
    当您使用键访问元素时，则使用哈希表
    哈希表中的每一项都有一个键/值对
排序列表（SortedList）
    排序列表是数组和哈希表的组合。它包含一个可使用键或索引访问各项的列表。
    集合中的各项总是按键值排序。
堆栈（Stack）
    代表了一个后进先出的对象集合。 LIFO
队列（Queue）
    代表了一个先进先出的对象集合。 FIFO
点阵列（BitArray）
    代表了一个使用值 1 和 0 来表示的二进制数组。
    当您需要存储位，但是事先不知道位数时，则使用点阵列。

#### C# 泛型（Generic）
允许您延迟编写类或方法中的编程元素的数据类型的规范，直到实际在程序中使用它的时候。

```csharp
public class MyGenericArray<T>
{
    private T[] array;
    public MyGenericArray(int size)
    {
        array = new T[size + 1];
    }
    public T getItem(int index)
    {
        return array[index];
    }
    public void setItem(int index, T value)
    {
        array[index] = value;
    }
}

/*使用 */
// 声明一个整型数组
MyGenericArray<int> intArray = new MyGenericArray<int>(5);
intArray.setItem(1, 15);
intArray.getItem(1);
```

* 泛型（Generic）委托

```csharp
delegate T NumberChanger<T>(T n);

static int num = 10;
public static int AddNum(int p)
{
    num += p;
    return num;
}

// 创建委托实例
NumberChanger<int> nc1 = new NumberChanger<int>(AddNum);
// 使用委托对象调用方法
nc1(25);
```

#### C# 匿名方法

匿名方法（Anonymous methods） 提供了一种传递代码块作为委托参数的技术。
匿名方法是没有名称只有主体的方法。
在匿名方法中您不需要指定返回类型，它是从方法主体内的 return 语句推断的。

```csharp
// 使用匿名方法创建委托实例
NumberChanger nc = delegate(int x)
{
   Console.WriteLine("Anonymous Method: {0}", x);
};

// 使用匿名方法调用委托
nc(10);
```

#### C# 不安全代码

当一个代码块使用 unsafe 修饰符标记时，C# 允许在函数中使用指针变量。

不安全代码或非托管代码是指使用了指针变量的代码块。

在同一个声明中声明多个指针时，星号 * 仅与基础类型一起写入；而不是用作每个指针名称的前缀。

```cpp
int* p1, p2, p3;     // 正确
int *p1, *p2, *p3;   // 错误

注意! 与C/C++中有区别
```

```cpp
static unsafe void Main(string[] args)
{
    int var = 20;
    int* p = &var;
    Console.WriteLine("Data is: {0} ",  var);
    Console.WriteLine("Address is: {0}",  (int)p);
    Console.ReadKey();
}
```

为了编译不安全代码，您必须切换到命令行编译器指定 /unsafe 命令行。
`csc /unsafe prog1.cs`

#### C# 多线程

在 C# 中，System.Threading.Thread 类用于线程的工作

```csharp
using System;
using System.Threading;

namespace MultithreadingApplication
{
    class MainThreadProgram
    {
        static void Main(string[] args)
        {
            Thread th = Thread.CurrentThread;
            th.Name = "MainThread";
            Console.WriteLine("This is {0}", th.Name);
            Console.ReadKey();
        }
    }
}
```

线程是通过扩展 Thread 类创建的。扩展的 Thread 类调用 Start() 方法来开始子线程的执行。
    ThreadStart childref = new ThreadStart(CallToChildThread);
    Thread childThread = new Thread(childref);
    childThread.Start();

Thread.Sleep(sleepfor);在一个特定的时间暂停线程

Abort() 方法用于销毁线程。

#### C# Lambda表达式

[C# 知识回顾 - Lambda](https://www.cnblogs.com/liqingwen/p/6216582.html)

Lambda 表达式，是一种简化的匿名函数，可用于创建委托或表达式目录树。

也可以将 Lambda 表达式作为参数进行传递，或者将它作用于函数调用值调用后返回的一个函数来使用。

创建 Lambda 表达式的简单语法形式：
  输入参数 => 表达式或语句块

  => 为 Lambda 运算符，可读作“goes to”，右结合 (与赋值运算符=优先级相同)

```c
delegate int MyDel(int x);
static void Main(string[] args)
{
    MyDel myDel = x => x++;
    var j = myDel(5);
    //传入5, x++, x变成6, 返回5??
}
```


创建表达式树:
Expression<MyDel> myDel = x => x++;

  表达式在 => 运算符右侧，称“lambda 表达式”。

### C#中decimal的用法
decimal拥有比float更高的精度，最高能处理到小数点后面的28位。适合用在财务类等对数字精确度要求比较高的场合。

```csharp
decimal price;
decimal discount;
decimal discount_price;

//注意：必须要带“m”,否则将和标准的浮点类型一样。而我们要求的却是
//用来计算货币类的浮点数，但是可以给其赋整数值。
price = 19.95m;
discount_price = price - (price * discount);
```


