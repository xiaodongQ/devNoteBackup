## 设置

* workspace vs user

VS Code提供了两种设置方式：

  - 用户设置： 这种方式进行的设置，会应用于该用户打开的所有工程；
  - 工作空间设置：工作空间是指使用VS Code打开的某个文件夹，在该文件夹下会创建一个名为.vscode的隐藏文件夹，里面包含着**仅适用于当前目录的**VS Code的设置。工作空间的设置会覆盖用户的设置。

  **使用通用的配置则统一加到user用户模式下的配置中。**

* 显示空格和tab

file->perferences->settings->搜 renderWhitespace->选all

或：
    1.打开vscode的用户设置
    2.在右边添加新设置
    {
        "editor.renderControlCharacters": true,
        "editor.renderWhitespace": "all"
    }

* 缩进设置为tab (makefile空格和tab有区别)

在vscode下边栏点击 "space" 在上面选项里设置 使用 indent using spaces 缩进


### 实用快捷键修改：

```
vim相关键冲突：
将vim中ctrl相关的键取消，ctrl+c,v,x,a,b,f,g,b等

代码补全提示冲突：
ctrl+shift+space 改成别的，ctrl+space(代码补全提示)改成ctrl+shift+space

VS Code默认无切换大小写，映射快捷键：
    转换为大写: Ctrl+Shift+u
    转换为小写: Ctrl+Shift+l
    (keyboards shortcuts搜upper,lower进行映射 之前已存在的快捷键remove)
```

### 自定义代码片段snippets设置

很方便的功能，推荐使用

参考：
[vs code设置自定义代码块的方法](https://blog.csdn.net/qq_504762354/article/details/81437118)

其他配置最好是参考官网说明：
[Snippets in Visual Studio Code](https://code.visualstudio.com/docs/editor/userdefinedsnippets)

1. 首先打开“首选项”，选择“用户代码片段”
2. 然后再选择你需要将此代码块作用在哪种类型的文件，会生成一个json文件
    - 也可以选全局，生成的json文件中会多一个key，用于设置代码生效范围
3. 然后配置一个json文件

File->Preferences->User Snippets->选择某个语言或者global->进行配置

配置方法说明(选的是全局为例)：

```json
{
    // 生成的文件里面的内容是json形式的，文件中给了一个例子
    // Example:
    "Print to console": {             // 自定义，功能描述
     "scope": "javascript,typescript",//"scope" 语言范围，试了下C++并不生效，可能关键字没输对
     "prefix": "log",          // "prefix" 前缀，输入前缀后会提示该语句片段，tab或回车则会生成
     // $1,$2,$3...的作用是按tab会依次跳转到下一个标号(或者shift tab跳到上一个)，
     // $0是最后的位置，再按tab就不跳转了
     // 若没有设置$标号，按tab则跳出自定义语句块结束
     "body": [                 // "body" 自定义的代码片段内容
         "console.log('$1');", // 可以包含\n,\t，每个","间会换行
         "$2"
     ],
     "description": "Log output to console", // 自定义，描述信息
    }
}
```

以自己在C/C++中常用的片段示例：

```json
{
    "cpp function begin block": {
        //尝试过C++,CPP,Cpp 输入前缀时都没有出提示，建议选择指定语言的配置或者字段置""
        "scope": "",
        "prefix": "cb",
        "body": [
            // 可以使用\n,\t,"    "，来调整缩进
            "int iRetVal = UTIL_ERROR;\n",
            "do",
            "{",
            "    $1",
            "    iRetVal = UTIL_OK;",
            "} while (0);\n",
            "return iRetVal;"
        ],
        "description": "c/c++ function block"
    },
    // 允许定义多个不同功能的代码片段
    "cpp do while(0) block": {
        "scope": "",
        "prefix": "dw",
        "body": [
            "do",
            "{",
            "    $1",
            "} while (0);\n",
        ],
        "description": "c/c++ function block"
    }
}
```

## 插件

vim
Code Runner               代码运行
Bracket Pair Colorizer    让每个括号都有自己的颜色
vscode-icon               给项目文件夹前边加上icon
Import Cost：             显示导入的包的大小(前端，go不生效)
highlight-words:          全局高亮字符串
    Highlight Current
    Highlight Remove

koroFileHeader 注释插件，关闭自动添加注释(配置文件放在最后章节)

    快捷键： ctrl+alt+i 文件头部注释  (mac: ctrl+cmd+i)
            ctrl+alt+t函数注释       (mac: ctrl+cmd+t)

### 配置云端同步
使用 Settings Sync 插件，同步到云端提供给其他机器(vscode中搜索安装即可)
(当前版本是3.4.3, 并不需要在github上手动生成gist，选择upload会自动生成，生成后从配置页中复制出来保存即可。)
(网上博客介绍的可能是比较早的版本，需要手动生成gist且下载的操作也略有不同)

Gist ID
3c9a1a862a60453426f214b94414fe81
令牌token (token每次可能会变，提示token过期的话，通过重置配置->"login with github"->执行相应命令即可)
b48fa742e2d6036c5ffca098c926942016bf3c02

ctrl+shift+p, 输入sync，
  之前有残留配置则reset extension settings;
  update/upload settings 出来的界面配置gist id和token，再执行一次，左下角状态栏会显示操作(字体比较小)
  download settings      出来的界面配置gist id和token

## 快捷键

主命令框
F1 或 Ctrl+Shift+P: 打开命令面板。在打开的输入框内，可以输入任何命令，例如：

按一下 Backspace 会进入到 Ctrl+P 模式
在 Ctrl+P 下输入 > 可以进入 Ctrl+Shift+P 模式
在 Ctrl+P 窗口下还可以:

直接输入文件名，跳转到文件

? 列出当前可执行的动作
! 显示 Errors或 Warnings，也可以 Ctrl+Shift+M
: 跳转到行数，也可以 Ctrl+G 直接进入

@ 跳转到 symbol（搜索变量或者函数），也可以 Ctrl+Shift+O 直接进入
@ 根据分类跳转 symbol，查找属性或函数，也可以 Ctrl+Shift+O 后输入:进入


 常用快捷键
编辑器与窗口管理
打开一个新窗口： Ctrl+Shift+N
关闭窗口： Ctrl+Shift+W
同时打开多个编辑器（查看多个文件）
新建文件 Ctrl+N
文件之间切换 Ctrl+Tab
切出一个新的编辑器（最多 3 个） Ctrl+\，也可以按住 Ctrl 鼠标点击 Explorer 里的文件名
左中右 3 个编辑器的快捷键 Ctrl+1 Ctrl+2 Ctrl+3
3 个编辑器之间循环切换 Ctrl+
编辑器换位置， Ctrl+k然后按 Left或 Right
代码编辑
格式调整
代码行缩进 Ctrl+[ 、 Ctrl+]
Ctrl+C 、 Ctrl+V 复制或剪切当前行/当前选中内容
代码格式化： Shift+Alt+F，或 Ctrl+Shift+P 后输入 format code
上下移动一行： Alt+Up 或 Alt+Down
向上向下复制一行： Shift+Alt+Up 或 Shift+Alt+Down
在当前行下边插入一行 Ctrl+Enter
在当前行上方插入一行 Ctrl+Shift+Enter
光标相关
移动到行首： Home
移动到行尾： End
移动到文件结尾： Ctrl+End
移动到文件开头： Ctrl+Home
移动到定义处： F12
定义处缩略图：只看一眼而不跳转过去 Alt+F12
移动到后半个括号： Ctrl+Shift+]
选择从光标到行尾： Shift+End
选择从行首到光标处： Shift+Home
删除光标右侧的所有字： Ctrl+Delete
扩展/缩小选取范围： Shift+Alt+Left 和 Shift+Alt+Right
多行编辑(列编辑)：Alt+Shift+鼠标左键，Ctrl+Alt+Down/Up
同时选中所有匹配： Ctrl+Shift+L
Ctrl+D 下一个匹配的也被选中 (在 sublime 中是删除当前行，后面自定义快键键中，设置与 Ctrl+Shift+K 互换了)
回退上一个光标操作： Ctrl+U
重构代码
找到所有的引用： Shift+F12
同时修改本文件中所有匹配的： Ctrl+F12
重命名：比如要修改一个方法名，可以选中后按 F2，输入新的名字，回车，会发现所有的文件都修改了
跳转到下一个 Error 或 Warning：当有多个错误时可以按 F8 逐个跳转
查看 diff： 在 explorer 里选择文件右键 Set file to compare，然后需要对比的文件上右键选择 Compare with file_name_you_chose

查找替换

查找 Ctrl+F
    在查找框中，alt+w 切换是否全词匹配

查找替换 Ctrl+H
整个文件夹中查找 Ctrl+Shift+F
显示相关
全屏：F11


显示 Git Ctrl+Shift+G
显示 Debug Ctrl+Shift+D
显示 Output Ctrl+Shift+U


括号展开/收回 zoomIn/zoomOut：Ctrl +/-

侧边栏显/隐：Ctrl+B
显示资源管理器 Ctrl+Shift+E
显示搜索 Ctrl+Shift+F

ctrl+c 不选中内容时，复制当前行
ctrl+x 不选中内容时，删除当前行

返回上一个位置 alt+左箭头
下一个位置 alt+右箭头

## C++开发环境

参考VS Code官方文档：
[Using Mingw-w64 in VS Code](https://code.visualstudio.com/docs/cpp/config-mingw)

安装mingw (Minimalist GNU for Windows)，(当前自己用的是安装QT时一并安装的mingw730_64)

1. 安装C/C++插件
2. `c_cpp_properties.json`配置文件，ctrl+shift+p 输入C/C++ edit... ，会自动生成`c_cpp_properties.json`配置文件，可视化的配置界面可以进行配置(不用手动创建文件，若手动创建空文件，则输入命令跳转后是空的。删除文件重新输入命令自动生成)
    - compilerPath 编译器路径(vscode自动识别当前系统可用的编译器，gcc/vs的cl/g++等)
    - IntelliSense 智能提示模式
    - includePath，把boost头文件路径添加到这里，左键可以跳转
3. `tasks.json` 告诉VS Code怎么编译程序
    - 步骤：View > Command Palette and then type "task" and choose Tasks: Configure Default Build Task. In the dropdown, select Create tasks.json file from template, then choose Others

```sh
# tasks.json
{
    "version": "2.0.0",
    "tasks": [
      {
        "label": "build firstTask",
        "type": "shell",
        "command": "g++",
        "args": ["-g", "-o", "helloworld", "test_auto.cpp"],
        "group": {
          "kind": "build",
          "isDefault": true   # 用于ctrl+shift+b启动，只是为了方便，置为false则每次手动输入执行命令"Run Build Tasks"
        }
      }
    ]
}
```

4. `launch.json` gdb调试，按F5进入配置
    - "program" 输入要调试的程序
    - "miDebuggerPath" 设置为gdb程序的完整路径
    - "stopAtEntry" 设置为true时，程序在目标的入口点停止(功能是：即使没有设置断点，也会停留在main函数开始处，等待继续)
    - "externalConsole" 是否在外部控制台运行(输出在终端上的结果可以用外部窗口显示，vs自身的debug控制台用于跟进代码执行)
    - 在DEBUG CONSOLE控制台，如果要输入一些命令：`-exec xxx`形式，e.g. `-exec info threads` (尝试调用不了stl成员函数:像map.size())


## Go插件安装

由于wall导致gocode等一些扩展自动安装不了。手动下载安装。如果已经手动下载了包到go_path对应src下，则go install进行安装(go get=clone+install)

```cpp
// gocode
https://github.com/mdempsky/gocode

// 下载下来的部分插件的作者可能不同，find模糊查对应插件名即可
go install github.com/nsf/gocode
go install github.com/rogpeppe/godef
go install github.com/zmb3/gogetdoc //暂时没装
go install github.com/golang/lint/golint
go install github.com/lukehoban/go-outline
go install sourcegraph.com/sqs/goreturns
go install golang.org/x/tools/cmd/gorename
go install github.com/uudashr/gopkgs
go install github.com/newhook/go-symbols
go install github.com/cweill/gotests/...
go install golang.org/x/tools/cmd/guru
go install github.com/go-delve/delve/cmd/dlv
```

## koroFileHeader的user配置备份

```json
"fileheader.configObj": {
        "createFileTime": true,
        "language": {
          "languagetest": {
            "head": "/$$",
            "middle": " $ @",
            "end": " $/"

          }
        },
        "autoAdd": false, //检测文件没有头部注释，自动添加文件头部注释
        "autoAlready": false, //只添加插件支持的语言以及用户通过`language`选项自定义的注释
        "annotationStr": {
          "head": "/*",
          "middle": " * @",
          "end": " */",
          "use": false
        },
        "headInsertLine": {
          "php": 2
        },
        "beforeAnnotation": {},
        "afterAnnotation": {},
        "specialOptions": {},
        "switch": {
          "newlineAddAnnotation": true
        },
        "prohibitAutoAdd": [
          "json"
        ],
        "moveCursor": true,
        "dateFormat": "YYYY-MM-DD HH:mm:ss",
        "atSymbol": "@",
        "atSymbolObj": {},
        "colon": ": ",
        "colonObj": {},
        "commitHooks": {
          "allowHooks": false,
          "noHooks": [
            "项目名"
          ],
          "showLog": true
        }
    },
      // 头部注释
    "fileheader.customMade": {
        // 头部注释默认字段
        "Description": "",
        "Author": "",
        "Date": "Do not edit", // 设置后默认设置文件生成时间
        "LastEditTime": "Do not edit", // 设置后，保存文件更改默认更新最后编辑时间
        "LastEditors": "xd", // 设置后，保存文件更改默认更新最后编辑人
        "Note": "",
    },
    // 函数注释
    "fileheader.cursorMode": {
        // 默认字段
        "description": "",
        "param": "",
        "return": "",
        "note": "",
    },
```

