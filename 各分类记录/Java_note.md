***# Java核心编程***

## 20160317
1. 访问修饰符 public
文件名必须与公共类名相同，并用.java作为扩展名
2. 注释方式3种：//, /* */, /** */. 第三种可用于自动生成文档*
3. 基本数据类型
    4种整型：int 4, short 2, long 8, byte 1. java7开始 0b1001表示9(二进制)
        Java没有任何无符号类型
    2种浮点型：float 4, double 8. F,D后缀,没有F后缀则默认double类型
    1种符号类型：char 1, Java中用UTF-16编码描述一个代码单元. 不建议使用char类型
    辅助字符需要一对代码单元表示。
    boolean类型: false,true两个值
4. Java中声明一个变量后，必须用赋值语句对变量进行初始化
    不区分变量的声明与定义
5. final指示常量，即只能被赋值一次
    Java中经常希望某常量在 一个类 中的 多个方法中使用，通常称类常量
    使用 static final 设置一个类常量。注意：类常量定义于main方法的外部
    const是Java保留字，但目前并没有使用
6. 运算符 >>符号位填充， >>> 0填充高位
7. println 操作一个定义在类System中的System.out对象
    sqrt操作的不是对象， 这样的方法称作 静态方法
8. int->float; long->float; long->double 可能有精度损失的转换
9. 强制类型转换注意截断。
10. String 不可变字符串， 只有字符串常量是共享的。
    空串""是一个对象，有自己的串长度0 和 内容空， 与 null 不同(无任何对象关联)
11. 创建一个数字数组时，所有元素都初始化为0，boolean数组元素被初始化为false，对象数组的元素被初始化为null
    boolean和int类型不能转换


## 20160319
1. 实例域， 访问器方法，更改器方法， 域访问器
2. 静态方法不能对对象进行操作 Math.pow(x,a),没有this参数，没有隐式参数
    静态方法不能操作对象，所以不能在静态方法中访问实例域，可以访问自身类中的静态域
3. 建议使用类名，而不是对象来调用静态方法
    使用静态方法情况： 1）方法不需要访问对象状态 2）方法只需要访问类的静态域


## 20160320
1. 工厂方法模式
    属于类创建型模式。工厂父类负责定义创建产品对象的公共接口，而工厂子类负责生成具体的产品对象，目的是将产品类的实例化操作延迟到工厂子类中完成。
2. Java中方法参数的使用情况
    1）一个方法不能修改一个基本数据类型的参数
    2）一个方法可以改变一个对象参数的状态
    3）一个方法不能让对象参数引用一个新的对象
3. 重载 overloading
    多个同名方法，参数不同
4. Java有自动的垃圾回收器，不需要人工回收内存，所以Java不支持析构器
5. 使用包 package 将类组织起来， 主要目的是确保类的唯一性。
    包作用域： public可被任意的类使用，标记为private的部分只能被定义它们的类使用；如果没有指定，则该部分可以被 同一个包中所有方法访问
    变量必须显示标记为private，否则将默认为包可见
6. 为了使类能够被多个程序共享，需要做到以下几点：
    1）把类放到一个目录中，这是包状树结构的基目录 dir1
    2）将JAR文件放在一个目录中 dir2
    3）设置类路径（class path)。类路径是所有包含类文件的路径的集合
        类路径包括：
            a. 基目录dir1
            b. 当前目录.
            c. JAR文件 dir2/xxx.jar
        e.g. /home/user/classdir:.:/home/user/archives/archive.jar
        (UNIX中用:分隔，Windows中用;)
7. 虚拟机搜寻类文件的顺序
    com.horstmann.corejava.Employee 类文件
    首先，查看jre/lib 和 jre/lib/ext 目录下的归档文件中的系统类文件(无)
    然后，查看类路径:
    /home/user/classdir/com/horstmann/corejava/Employee.class
    当前目录开始 com/horstmann/corejava/Employee.class
    /home/user/archives/archive.jar中 com/horstmann/corejava/Employee.class

    另：编译器定位文件比虚拟机复杂得多。
        对类路径的所有位置进行逐一查看，若找到一个以上的类则产生编译错误；
        检查源文件是否比类文件新，若是则会重新编译
8. 编译时使用 -classpath (或-cp)选项指定类路径 (写在一行中，可使用脚本)
    e.g. java -classpath /home/user/classdir:.:/home/user/archives/archive.jar MyPro
    -classpath是首选的方法，也可以设置环境变量 CLASSPATH
9. 类设计技巧
    1）保证数据私有化
    2）对数据初始化
    3）不要在类中使用过多的基本类型（用新类替换）
    4）不是所有域都需要域访问器和修改器（一些不希望别人获得或设置的实例域）
    5）将职责过多的类进行分解
    6）类名和方法名要能够体现它们的职责（类名：名词、形容词修饰的名词、动名词修饰的名词； 方法：访问器方法用小写get开头，更改器方法用set开头）


## 20160321
1. 继承
    超类 -- 子类
    父类 -- 孩子类
    基类 -- 派生类
2. 重写 override
    super关键字指示编译器调用超类的方法
    可以通过super实现对超类构造器的调用，子类没有显式调用超类的构造器，将调用超类默认（无参数）的构造器
    super和this关键字。
        this：引用隐式参数；调用该类其他构造器。
        super：调用超类方法；调用超类构造器
3. 多态 动态绑定 继承层次 继承链
    Java不支持多继承
    动态绑定，调用对象方法的执行过程
        1）编译器查看对象的声明类型和方法名，获得所有可能被调用的候选方法
        2）查看调用方法时提供的参数类型，（若匹配则选择：重载解析）
        3）private、static、final方法或构造器，编译器准确知道该调哪个：静态绑定
        4）程序运行且采用动态绑定调用时，调用与实际类型最合适的那个类的方法
            （虚拟机预先为每个类创建了一个方法表）
4. 阻止继承 使用final修饰符，不能继承和重写
    如果将一个类声明为final，其中方法自动为final，但不包括域
5. 内联处理
    如果方法很简短、被频繁调用且没有真正地被覆盖，那么即时编译器就会将该方法进行内联处理。
6. 强制类型转换
    只能在继承层次内进行类型转换
    在将超类转换成子类之前，应该用instanceof进行检查
7. 抽象类 abstract
    抽象类不能被实例化
8. Object 所有类的超类
9. 散列码 每个对象都有一个默认的散列码，其值为对象的存储地址
    Equals和hashCode的定义必须一致
10. 只要对象与一个字符串通过操作符 "+" 连接起来，Java编译就会自动地调用toString方法。
    在调用x.toString()的地方可以用 ""+x 代替
    数组的toString()继承于Object类(旧格式)，应调用静态方法 Arrays.toString
    强烈建议为自定义的每一个类增加一个toString方法
11. 泛型数组列表
    Java中允许在运行时确定数组的大小
    若内部数组已经满了，数组列表会自动创建一个更大的数组，并将所有的对象从较小的数组中拷贝到较大的数组中。
    数组列表 与 数组 的大小有个非常重要的区别：数组固定有多少空间使用，而数组列表最初根本就不包含任何元素，但拥有保存多少个元素的潜力。
e.g.
    ArrayList<Employee> staff = new ArrayList<Employee>();
    Java 7中，可以省去右边的类型参数(但7之前会报错)：
        ArrayList<Employee> staff = new ArrayList<>();
12. 对象包装器 与 自动装箱
    所有基本类型都有一个与之对应的类，通常称这些类为包装器
    包装器中的内容不会改变，不能使用这些包装器类创建修改数值参数的方法
13. 枚举类
    e.g. public enum Size {
            SMALL(1),
            MIDDLE(2),
            LARGE(3);(简单类型;可不要，若有定制方法则需要;隔开)
         }


## 20160323
1. 反射
    能够分析类能力的程序称为反射(reflective)
    反射机制可用来：
        在运行中分析类的能力；
        运行中查看对象；
        实现通用的数组操作代码；
        利用Method对象(类似函数指针)
2. 运行时 类型标识
    程序运行期间，Java运行时系统始终为所有的对象维护一个被称为 运行时 的类型标识。 这个信息跟踪着每个对象所属的类。保存这些信息的类被称为Class类
    getClass()返回一个Class类型的实例, getClass().getName()返回类的名字
    静态方法 Class.forName(类名) 获得类名对应的Class对象
3. java.lang.reflect包中有三个类 Field,Method,Constructor分别描述类的域、方法和构造器。这三个类都有getName方法，返回项目的名称；还有个叫getModifiers的方法，返回一个整型数值：描述修饰符(public、private等)使用状况。
    使用Modifier类中静态方法分析getModifiers返回的整型数值。
    Class类中的getFields,getMethods,getConstructors方法，包括继承自超类的公有成员；
    Class类中的getDeclareField,getDeclareMethods,getDeclareConstructor不包括超类成员。
4. 查看对象域 使用Field类get方法。 e.g. f.get(obj), obj是某个包含f域的对象，返回一个对象，其值为f域的当前值，若为基本类型,会打包到相应对象包装器中。
    AccessibleObject.setAccessible(fields, true); 为反射对象设置可访问标志。设置域fields(e.g.)不受安全管理器约束，可以访问私有域。主要用于调试、持久存储、相似机制。
5. java.lang.reflect.Array
    static Object get(Object array, int index)
    static Object newInstance(Class componentType, int length)
    返回一个具有给定类型、给定维数的新数组
6. 可利用反射机制调用任意方法
    Method类的invoke方法签名： Object invoke(Object obj, Object... args)
    对于静态方法，第一个参数可被忽略，即可设为null
    若返回类型是基本类型，invoke会返回其包装器类
    e.g.  double y = (Double) f.invoke(null, x);
7. 继承设计的技巧
    1）将公共操作和域放在超类
    2）不要使用受保护的域
        子类可访问protected的实例域；同一个包中所有类可访问protected类
    3）使用继承实现"is-a"关系
        e.g. 雇员和钟点工，非is-a关系。 若继承加一个计时工资，薪水和计时工资两个域给打印支票或税单方法带来很大麻烦。
    4）除非所有继承的方法都有意义，否则不要使用继承
        e.g. GregorianCalender和Holiday类，add可将假日转换成非假日，若继承则对Holiday来说add含义不同
    5）在覆盖方法时，不要改变预期的行为
        e.g. 同上例，若重新定义add行为，可能产生争议(二义性)
    6）使用多态，而非类型信息
        e.g. action1,action2表示相同概念则下列应使用多态
        if(x is of type1)
            action1;
        else if(x is of type2)
            action2;
        使用多态方法或接口编写的代码比使用多种类型进行检测的代码更加易于维护和扩展。
    7）不要过多地使用反射
        反射可在运行时查看域和方法，让人们编写出更具通用性的程序，但编译器很难帮助发现程序中的错误，只有在运行时才发现并导致异常。


## 20160324
1. 相等测试与继承
    Object类中的equals方法：用于检测一个对象是否等于另外一个对象
    getClass方法返回一个对象所属的类。只有在两个对象同属于同一个类时，才有可能相等。
    在子类中定义equals方法时，首先调用超类的equals。 super.equals()
    Java语言规范要求equals方法具有一下特性：
    1）自反性: 任何非空引用x, x.equals(x)应返回true
    2）对称性: x,y y.equals(x)返回true，x.equals(y)也应true
        如果子类拥有自己的相等概念，则对称性需求将强制使用getClass进行检测；
        如果由超类决定相等的概念，可使用instanceof进行检测
    3）传递性
    4）一致性
    5）任意非空引用x, x.equals(null)应返回false
2. 接口 主要用来描述类具有什么功能
    接口不是类，而是对类的一组需求描述
    接口中的所有方法自动地属于public
    接口中绝不可能含有实例域，也不能在接口中实现方法。接口中可以定义常量
    java.lang.Comparable<T>类中 int compareTo(T other)方法
    java.util.Arrays中  static void sort(Object[] a)
    java.lang.Integer中 static int compare(int x, int y)
3. 接口不是类，不能用new实例化一个接口，但能声明接口的变量；
        Comparable x;
    接口变量必须引用实现了接口的类对象；
        x= new Employee(...);
    可用instanceof检查一个对象是否实现了某个特定的接口；
        if (anObject instanceof Comparable) { ... }
    接口可被扩展
        public interface Powered extends Moveable
        {
            double func();  //自动public 但不能包含实例域或静态方法
            double SPEED_DEF = 100; //自动设为常量
        }
        接口中的方法自动被设置为public，域被自动设为public static final
4. 抽象类和接口
    均可用于扩展。抽象类 每个类只能扩展于一个类；接口 每个类可以实现多个接口
        e.g. Class Employee extends Person implements Comparable
5. 对象克隆
    当拷贝一个常量时，原始变量与拷贝变量引用同一个对象。即改变一个变量引用的对象将会对另一个变量产生影响。
   默认的克隆操作是浅拷贝，并没有克隆包含在对象中的内部对象。
   为实现克隆自对象的深拷贝，必须重新定义clone方法
    如果默认的clone不满足要求，需重新定义时，类必须：
        1）实现Cloneable接口
        2）使用public访问修饰符重新定义clone方法
6. 回调 callback
   常见的一种程序设计模式，可以指出某个特定事件发生时应该采取的动作。
    e.g. 定时器 (要求传递的对象所属的类实现了java.awt.event包中的ActionListener接口)
7. 内部类 inner class
    定义在另一个类中的类。
    内部类既可以访问自身的数据域，也可以访问创建它的外围类对象的数据域。
    内部类有一个外围类的引用， OuterClass.this表示外围类引用
    构造内部类对象语法：
        outerObject.new InnerClass(construction parameters)
        e.g. ActionListener listener = this.new TimePrinter();
            其中TimePrinter是内部类
    外围类的作用域之外，引用内部类：OuterClass.InnerClass
    编译器将会把内部类翻译称$分隔外部类名与内部类名的常规类文件，
        javap -private ClassName 对类进行反射，查看运行时信息
8. 局部内部类，在一个方法中定义
    局部类不能用public或private访问说明符进行声明。作用域被限定在声明这个局部类的块中。 对外部世界完全隐藏。
    局部类不仅能访问包含它们的外部类，还可以访问局部变量，不过须声明为final
    创建后只能赋值一次，定义时没有初始化的final变量通常被称为空final变量(blank final)
9. 匿名内部类 anonymous inner class
    new SuperType(construction parameters)
        {
            inner class methos and data
        };
    SuperType可以是接口，于是内部类就要实现这个接口。
    匿名类没有类名，而构造器名必须与类名相同，因此匿名类没有构造器。将构造器参数传给超类构造器。
10. 静态内部类
    在内部类不需要访问外围类对象时，应该使用静态内部类
11. 代理
    利用代理可以在运行时创建一个实现了一组给定接口的新类。
    只有在编译时无法确定需要实现哪个接口时才有必要使用。
    代理类可以在运行时创建全新的类。具有下列方法：
        1）指定接口所需要的全部方法；
        2）Object类中的全部方法。
    调用处理器(invocation handler)
    代理类是在程序运行过程中创建的，但一旦被创建，就变成了常规类，与虚拟机中的任何其他类没有什么区别。


## 20160325
1. 图形程序设计
    AWT抽象窗口工具箱(Abstract Window Toolkit)
    Swing用户界面库
    Swing没有完全替代AWT，而是基于AWT架构之上。
    从现在开始(书中)Swing指用户界面类，AWT指像事件处理这样的窗口工具箱的底层机制。
2. Java中，顶层窗口被称为框架(frame)
    AWT库中有一个称为Frame的类，用于描述顶层窗口，Swing版本为JFrame，扩展于Frame类
    javax.swing.JFrame类 (x表示这是一个Java扩展包，而不是核心包)
    所有的Swing组件必须由时间分派线程(event dispatch thread)进行配置
    get/set，对于类型为boolean的属性，获取方法由is开头
    setIconImage(img); 将图像设置为框架的图标。
3. 在组件中显示信息
    Java中，框架被设计为放置组件的容器。通常情况下应该在另一组件上绘制信息，并将这个组件添加到框架中。
    绘制一个组件，需要定义一个扩展JComponent的类，并覆盖其中的paintComponent方法；
    不要自己调用paintComponent方法，程序需要重新绘图时会自动调用。
    在Java中，所有的绘制都必须使用Graphics对象，其中包含了绘制图案、图像和文本的方法。
4. 处理2D图形、颜色、字体、显示图像


## 20160326
1. 事件处理
    Java中，将事件的相关信息封装在一个事件对象中。所有的事件对象都最终派生于java.util.EventObject类
    AWT事件处理机制概要：
    1）监听器对象是一个实现了特定监听器接口的类的实例；
    2）事件源是一个能够注册监听器对象并发送事件对象的对象；
    3）事件发生时，事件源将事件对象传递给所有注册的监听器；
    4）监听器对象将利用对象中的信息决定如何对事件作出相应。
2. 动作
    将按钮或按键映射到动作对象
3. 鼠标事件
    当用户点击鼠标按钮时，将会调用三个监听器方法：
    1）第一次被按下调用mousePressed；
    2）鼠标被释放时调用mouseReleased；
    3）最后调用mouseClicked。
    要想区分单击、双击和三击，需要使用getClickCount方法。
可以采用位掩码来测试已经设置了那个修饰符。
    getModifiersEx方法能准确地报告鼠标事件的鼠标按钮和键盘修饰符。
    e.g. if((event.getModifiersEx() & InputEvent.BUTTON3_DOWN_MASK) != 0){
         }  (其中BUTTON3_DOWN_MASK用于检测鼠标右键)
4. 语义事件和低级事件
    AWT将事件分为语义事件和低级事件。
    语义事件是表示用户动作的事件，如点击按钮；
    低级事件是形成那些事件的事件，点击按钮时，包含了按下鼠标、移动鼠标、抬起
    java.awt.event包中最常用的语义事件类：
        ActionEvent  按钮点击、菜单选择、列表项选择、文本框输ENTER
        AdjustmentEvent 调节滚动条
        ItemEvent 从复选或列表框中选择一项
    常用的5个低级事件类：
        KeyEvent  键被按下或释放
        MouseEvent  鼠标键按下、释放、移动
        MouseWheelEvent  鼠标滚轮
        FocusEvent  组件焦点
        WindowEvent 窗口状态被改变
    各自有接口监听这些事件，配有适配器类实现了相应接口中的所有方法。


## 20160327
1. Swing用户界面组件
    模型-视图-控制器设计模式
     模型(model)：存储内容。
        模型存储内容，它没有界面；模型是不可见的，显示存储的数据是视图的工作
        模型必须实现改变内容和查找内容的方法。
     视图(view)：显示内容
     控制器(controller)：处理用户输入
    布局、文本输入、选择组件、菜单、对话框

2. 部署应用程序
    JAR文件。 jar类似UNIX中tar命令
        jar cvf jarfile file1 file2 ...
    编译、创建JAR文件、执行程序：
        javac ResourceTest.java
        jar cvfm Resourcetest.jar Resourcetest.mf *.class *.gif *.txt
        java -jar Resourcetest.jar
    清单文件(manifest)：用m选项，新建清单文件，添加内容，
        如指定应用程序的主类：Main-Class: com.xd.MainAppClass (注意空格)
        也可以使用e选项指定程序的入口点，即程序调用时指定的类
        清单文件最后一行必须以换行符结束，否则无法被正确读取
    e.g.
        jar cvfe resource\ResourceTest.jar resource.ResourceTest resource\*.class resource\*.txt resource\*.gif
        jar cvfm resource\ResourceTest.jar resource\manifest.mf resource\*.class resource\*.txt resource\*.gif
        (其中manifest.mf指定了入口：
            Main-Class: resource/ResourceTest，.MF中为/
            也可使用resource.ResourceTest，.MF中为.
            resource\ResourceTest也未报错,jar文件中MANIFEST.MF中为/)
3. 包密封 seal
    以保证其他类加入到其中
    默认情况下，JAR文件中的包是没有密封的。
    可在清单文件中加入：Sealed:true。
    想要密封一个包，需要创建一个包含清单指令的文本文件，然后用常规的方式运行jar命令： jar cvfm MyArchive.jar manifest.mf files


## 20160328
1. 异常处理
    Java中，异常对象都是派生于Throwable类的一个实例。
    Error类和Exception类派生于Throwable。
        Error类层次结构描述了Java运行时系统的内部错误和资源耗尽程度，应用程序不应该抛出这种类型的对象；
        设计Java程序时，需关注Exception层次结构。两个分支：由程序错误导致的异常属于RuntimeException；而程序本身没问题，但由于像I/O错误这类问题导致的异常属于其他异常。
    Java语言规范将派生于Error类或RuntimeException类所有异常称为未检查异常，所有其他的异常称为已检查异常(checked)
2. 如果一个方法可能抛出多个已检查异常，那么就必须在方法的首部列出所有的异常类。每个异常类之间用逗号隔开。
    e.g. public Image loadImage(String s) throws FileNotFoundException, EOFException
        {
            ...
        }
    不需要声明Java的内部错误，即从Error继承的错误；也不应该声明从RuntimeException继承的那些未检查异常。
    一个方法必须声明所有可能抛出的已检查异常，而未检查异常要么不可控制(Error)，要么就应该避免发生(RuntimeException)。
    如果超类方法没有抛出任何已检查异常，子类也不能抛出任何已检查异常。
3. 创建异常类
    定义一个派生于Exception的类或派生于Exception子类的类
4. 捕获异常
    要想捕获一个异常，必须设置try/catch语句块。
    如果在try语句块中任何代码抛出了一个在catch子句中说明的异常类那么
    1）程序将跳过try语句块的其余代码；
    2）程序将执行catch子句中的处理器代码
    如果try语句块中没有抛出任何异常，那么程序将跳过catch子句；
    如果方法中抛出了一个catch子句中没有声明的异常类型，方法会立刻退出。
    通常，将异常传递给调用者。如果采用这种处理方式，就必须声明这个方法可能会抛出一个异常，使用throws
    编译器严格地执行throws说明符，如果调用了一个抛出已检查异常的方法，就必须对它进行处理，或者将它继续进行传递。
5. 捕获多个异常
    try{
    }
    catch (){
    }
    catch (){
    }
6. 再次抛出异常与异常链
    catch (SQLException)
    {
        throw new ServletException();
    }
更好的一种处理方法，包装技术。将原始异常设置为新异常的原因：
    catch (SQLException)
    {
        Throwable se = new Servletexception("database error");
        se.initCause(e);
        throw se;
    }
    捕获到异常时，使用 Throwable e = se.getCause(); 重新得到原始异常
7. finally子句
8. 带资源的try语句 Java SE 7中
    try (Scanner in = new Scanner(new FileInputStream("path")), .....)
    {
        ...
    }
    try块退出时，会自动调用close


## 20160329
1. 泛型类，泛型方法
    类型参数(type parameters)使得程序具有更好的可读性和安全性
泛型类e.g.
    public class Pair<T>
    {
        public T first;
    }
泛型方法e.g.
    class ArrayAlg
    {
        public static <T> T getMiddle(T... a)
        {
            return a[a.length/2];
        }
    }
    类型变量放在修饰符(此处public static)后面，返回类型前面。
调用泛型方法e.g.
    String middle = ArrayAlg.<String>getMiddle("hel","llo","her");
    大多数情况可省略类型参数，编译器有足够信息推断出所调用的方法：
    String middle = ArrayAlg.getMiddle("hel","llo","her");
2. 类型变量的限定
    保证T所属的类具有compareTo方法:
    public static <T extends Comparable> T min(T[] a) ...
    一个类型变量或通配符可以有多个限定，限定类型用&分隔
    T extends Comparable & Serializable
3. 无论何时定义一个泛型类型，都自动提供了一个相应的原始类型(raw type)。
    擦除类型变量，并替换为限定类型(无限定类型的变量用Object)
    (C++中每个模板的实例化产生不同的类型，这一现象称"模板代码膨胀"，Java不存在这个问题)
4. 希望对泛型方法的调用具有多态性，并调用最合适的方法。但类型擦除与多态发生了冲突。 使用：桥方法
    在虚拟机中，用参数类型和返回类型确定一个方法。编译器可能产生两个仅返回类型不同的方法字节码，虚拟机能够正确地处理这一情况。
    Java泛型转换的事实：
    1）虚拟机中没有泛型，只有普通的类和方法；
    2）所有类型参数都用它们的限定类型替换；
    3）桥方法被合成来保持多态；
    4）为保持类型安全性，必要时插入强制类型转换。
4. 泛型的约束与局限性
大多数限制都是由类型擦除引起的。
    1）不能用基本类型实例化类型参数
        没有Pair<double>，只有Pair<Double>
    2）运行时类型查询只使用于原始类型
        instanceof,getClass或涉及泛型类型的强制转换都会有编译器警告。
    3）不能创建参数化类型的数组
        擦除后转换成Object[]，存储其他类型的元素会报异常
    4）Varargs警告
        向参数个数可变的方法传递一个泛型类型的实例，得到警告而不是错误
        可抑制这个警告：(添加标注)
            @SuppressWarnings("unchecked")
            @SafeVarargs
    5）不能实例化类型变量
         new T(...), new T[...], T.class都是非法的
    6）泛型类的静态上下文中的类型变量无效
        不能在静态域或静态方法中引用类型变量。
    7）不能抛出或捕获泛型类的实例
        try{} catch(T e){} 非法。catch子句中不能是使用类型变量。
    8）注意擦除后的冲突

5. 线程状态
    1）New (新创建线程)
        新创建。线程还没有运行
    2）Runnable (可运行线程)
        一旦调用start方法，线程处于runnable状态。可能在运行也可能没有运行，一个正在运行中的线程仍然处于可运行状态。
    3）Blocked (被阻塞线程)
        请求锁，而该锁被其他线程持有，进入阻塞状态
    4）Waiting (等待线程)
        等待另一个线程通知调度器一个条件时，进入等待状态
    5）Timed waiting
        计时等待，等待超时或收到适当的通知
    6）Terminated (被终止的线程)
        因两个原因之一被终止：
            run方法正常退出而自然死亡；
            因一个没有捕获的异常终止了run方法而意外死亡。


## 20160330
1. 线程属性
    包含：线程优先级、守护线程、线程组以及处理未捕获异常的处理器。

20160402
1. 反射
    Class类
    1）在面向对象的世界里，万事万物皆对象。
    类是对象，类是java.lang.Class类的实例对象。
    There is a class named Class
    2）//Foo的实例对象如何表示
    Foo foo1=new Foo();//foo1就表示出来了
    //Foo这个类也是一个实例对象，Class类的实例对象，如何表示呢？
    //任何一个类都是Class的实例对象，这个实例对象有三种表示方式
    //第一种表示方式--->实际在告诉我们任何一个类都有一个隐含的静态成员变量class
    Class c1=Foo.class;
    //第二种表达方式--->已经知道该类的对象通过getClassF方法
    Class c2=foo1.getClass();
    //官网c1,c2表示了Foo类的类类型（class type），万事万物皆对象，类也是对象，是Class类的实例对象
    //这个对象我们称为该类的类类型
    //不管c1 or c2都代表了Foo类的类类型，一个类只可能是Class类的一个实例对象
    //第三种表达方式
    Class c3=null;
    c3=Class.forName("com.imooc.reflect.Foo");
    //我们完全可以通过类的类类型创建类的对象实例--->通过c1 or c2 or c3创建Foo的实例对象
    Foo foo=(Foo)c1.newInstance();//需要有无参数的构造方法
2. 反射的操作都是编译之后的操作。
    Java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译就无效了。（使用反射就无效）


## 20160403
1. Collection接口扩展了Iterator接口。因此对于标准库中的任何集合都可以使用"for each"循环
2. Iteraotr接口的remove方法删除上次调用next方法返回的元素。
    调用remove前须先调用next方法。
3. Java中，所有链表实际上都是双向链接的（doubly linked）

## 20160405
1. 文本输入与输出
e.g.
整数1234：
    二进制格式：00 00 04 D2
    文本格式："1234"
尽管二进制I/O高速且高效，但不宜人们阅读。
2. 存储文本字符串时，需考虑字符编码方式。

## 20160406
1. XML (Extensible Markup Language)可扩展标记语言


***
***# Java编程思想***

## 20160413
1. 在类的内部，即时变量定义散布于方法之间，仍会在任何方法(包括构造函数)
之前得到初始化。

> 静态初始化动作只进行一次

## 20160417
1. 枚举类型enum
enum实际上是类，具有自己的方法 ordinal()声明顺序,values()实例的名字

## 20160425
1. final, static final

## 20160427
1. 多态
前期绑定、后期绑定
运行时根据对象的类型进行绑定，也叫动态绑定或运行时绑定。
**Java中除了static和final(private属于final方法)方法外，其他方法都是后期绑定**
2. 只有普通的方法调用可以是多态的，域和静态方法不是多态的：
  子类向上转型时，由编译器解析，因此不是多态的；
  静态方法是与类，而非单个的对象相关联的；
3. 清理：先对导出类进行清理，然后基类。(导出类的清理可能会调用基类中的方法，应仍起作用而非过早销毁)
4. 垃圾回收，finalize()可添加终结条件 第五章
5. 在构造器中调用其他方法，可能达不到预期效果；
**构造器内唯一能够安全调用的是基类中的final方法(包含private)**

# 20160429
1. 协变返回类型
导出类中的被覆盖方法可返回基类返回类型的导出类类型
2. 接口
接口可以包含域，但这些域隐式地是static和final的。
3. 多重继承(C++)
Java中可继承多个接口，所有接口名都置于implements关键字后，并用逗号隔开
通过继承来扩展接口，且可继承多个接口: interface RedBook extends Book，Book2 {}

# 20160506
1. 内部类

# 20160510
1. Queue
offer()将一个元素插到队尾
peek(),elecment()返回对头，空时分别为返回null、抛出NoSuchElementException异常
poll(),remove()方法移除并返回对头

# 20160513
1. format("%b", )  
对于boolean和Boolean类型，结果是对应的true,false
**但对于其他类型那个的参数，只要该参数不为null，结果永远是ture**
2. 正则表达式
. 任意字符
^ 一行的起始
$ 结束
Java对反斜杠 '\' 的处理不同
\\ 表示插入一个正则表达式的\ 所以后面的字符有特俗的意义。
e.g. 若想表示一位数字: \\d  
     若想插入一个\: \\\\
 \d数字    \D非数字
 \w 单词字符                               \W 非单词字符
 \s 空白符(包含空格、tab、换行、换页和回车)    \S 非空白符
 \b 词的边界                               \B 非词的边界
 \G 前一个匹配的结束
3. 量词
  量词描述了一个模式吸收文本的方式：
  * 贪婪型 尽可能多的匹配。                        X*   零个或多个
  * 勉强型 用问号来指定，匹配满足模式所需最少字符。    X*?
  * 占有型 仅在Java中                            X*+
  X? 一个或0个     X+  一个或多个   X{n}  n个
  X??             X+?            X{n}?
  X?+             X++            X{n}+

表达式X通常必须用()括起来。 abc+  (abc)+
4. 模式标志
compile(正则表达式, 模式标志)
(?m) (?i) (?x)

# 20160519
1. 泛型
基本指导原则：无论何时，只要你能做到，就应该尽量使用泛型方法。
```
 要定义泛型方法，只需将泛型参数列表置于返回值之前
 public <T> void fun(T x) {

 }
```

# 20160520
1. 匿名内部类中使用其外部对象，该外部对象须为final限定

# 20160521
1.

# 20160525
1. Java IO 输入和输出

# 20160614
1. Spring
