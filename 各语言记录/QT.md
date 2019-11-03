# QT

QWidget

熟悉使用
https://blog.csdn.net/superhcq/article/details/53506152

* QMainWindow和QWidget
    - QMainWindow中在setUi时自动为用户创建了一个菜单栏、工具栏、中心窗口和状态栏。而QWidget是没有这几点的。
    - QWidget运行后就只有一个“页面”，而QMainWindow运行后生成了一个“窗口”。
    - 因此，在继承自QWidget类的用户类中无法创建菜单栏等几种行为



demo示例：
https://blog.csdn.net/qq_36407898/article/details/78020232

在Qt Creater中提供了三种建立信号和槽的方法:

第一种方法：

1 在头文件mainwindow.h的类MainWindow的定义中声明槽函数
2 在mainwindow.cpp文件中，定义槽函数
3 建立映射，在类MainWindow的构造函数中添加如下语句，以便将信号和槽函数进行连接
    `connect(ui->pushButton,SIGNAL(clicked()),this,SLOT(on_pushButton_clicked)));`
4 编译运行即可。

```
#include < QObject>

static bool QObject::connect(const QObject *sender, const char *signal, const QObject *receiver, const char *member);
```
sender 和 receiver 分别指定了被关联的信号和槽的发送者和接受者
signal是信号，Qt要求必须使用宏SINGAL将信号函数指针转化为指定的类型。
member是槽函数，Qt要求必须使用宏SLOT转换函数指针。

宏SIGNAL和宏SLOT的参数形式如下：
    `SIGNAL(funname（param_type1,param_type2…)`
    `SLOT(funname(param_type1,param_type2)`

第二种方法：

在.ui文件上 右击“OK”按钮，选择“Go to slot…”，选择clicked()，单击OK，即完成信号和槽函数的链接。clicked是信号函数，槽函数是`on_pushButton_clicked()`;

第三种方法：

1）右击界面选择“change signals/slot”（改变信号/槽）选项，单击“+”，添加新的槽函数，如图。单击“OK”，完成槽函数的添加。
2）在窗体编辑区的下方有信号和槽的映射窗口，单击做上角的加号，出现一行新的映射，在这里编辑映射函数。
3）重复第一种方法的第1、第2步，完成槽函数的声明和定义。
4）编译运行。

## QT中PRO文件

[QT中PRO文件写法的详细介绍](https://blog.csdn.net/adriano119/article/details/5878169)

* 注释 “#”
* 模板 TEMPLATE = app
    - app -建立一个应用程序的makefile。这是默认值，所以如果模板没有被指定，这个将被使用
    - lib - 建立一个库的makefile。
    - vcapp - 建立一个应用程序的VisualStudio项目文件。
    - vclib - 建立一个库的VisualStudio项目文件。
    - subdirs -这是一个特殊的模板，它可以创建一个能够进入特定目录并且为一个项目文件生成makefile并且为它调用make的makefile。
* 指定生成的应用程序放置的目录
    - DESTDIR += ../bin
* 指定生成的应用程序名
    - TARGET = pksystem
* 配置信息 CONFIG用来告诉qmake关于应用程序的配置信息
    - CONFIG += qt warn_on release
* 头文件包含路径
    - INCLUDEPATH += .
* 工程中包含的头文件
    - HEADERS
* 工程中包含的.ui设计文件
    - FORMS += forms/painter.ui
* 工程中包含的源文件
    - SOURCES += sources/main.cpp sources/painter.cpp
* 工程中包含的资源文件
    - RESOURCES += qrc/painter.qrc
* LIBS
    - LIBS += -L folderPath  //引入的lib文件的路径  -L：引入路径
* Release:LIBS += -L folderPath // release 版引入的lib文件路径
* Debug:LIBS += -L folderPath // Debug 版引入的lib 文件路径
* `DEFINES += XX_XX_XXX`  //定义编译选项，在.h文件中可以使用：`#ifdefine xx_xx_xxx`
* RC_FILE = xxx.icns //定义资源文件

* 根据运行的平台来使用相应的作用域：

为Windows平台添加的依赖平台像这样：

win32 {
    SOURCES += hello_win.cpp
}

或者 win32: SOURCES += hello_win.cpp

* DEPENDPATH

当你已经创建好你的项目文件，生成Makefile就很容易了，你所要做的就是先到你所生成的项目文件那里然后输入：  
Makefile可以像这样由“.pro”文件生成：
    `qmake -o Makefile hello.pro` qmake在另一个安装qt时的mingw路径，两个mingw路径都要加到环境变量中

### QT文件构建和打包

左下角可以选择是debug还是release版本构建

[QT5的程序打包发布（将QT5的工程项目打包成一个exe程序）](https://blog.csdn.net/windsnow1/article/details/78004265)

QT5自带的 windeployqt
    1、windeployqt自动提取与qt相关的dll，如果你代码中引入其他第三方库，需要自己手动添加


报错 应用程序无法正常启动0x000007b

## 快捷键

后退  Alt+Left
前进  Alt+Right
跳转行(G)… Ctrl+L
历史中先前打开的文件  Ctrl+Tab
退出Qt Creator    Ctrl+Q

向下移动当前行 Ctrl+Shift+Down
向上移动当前行 Ctrl+Shift+Up
跳转至块结尾  Ctrl+]
跳转至块开始  Ctrl+[
跳转至以}结尾的块   Ctrl+}
跳转至以{开始的块   Ctrl+{

注释选中的内容 Ctrl+/
查找和替换   Ctrl+F
ctrl+shift+f 
向下查找    F3
向上查找    Shift+F3