## 记录

[程序员练级攻略：程序员修养](https://time.geekbang.org/column/article/8700)

* [97 Things Every Programmer Should Know](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/index.html)

>其中有 97 个非常不错的编程方面的建议。这篇文章是比较经典的，别被“97”这个数字吓住，你可以快速浏览一下，会让你有不同的感觉的。另外，在工作一段时间后再来读，你会更有感觉。

* 英文能力
* 问问题的能力
    - X-Y 问题：《[X-Y 问题](https://coolshell.cn/articles/10804.html)》
        + 没有去问怎么解决问题X，而是去问解决方案Y应该怎么去实现和操作(有，改之)
        + 在一个根本错误的方向上浪费他人大量的时间和精力！
        + 这样问问题可能错过更好更合适的方案，除非你告诉大家X是什么
    - StackOverflow 上如何问问题的一些提示 [FAQ for Stack Exchange sites](https://meta.stackexchange.com/questions/7931/faq-for-stack-exchange-sites)
* 你不知道你不知道的
    - [程序员的谎谬之言还是至理名言？](https://coolshell.cn/articles/4235.html)


后续看一手的技术相关文档：

boost的TR1：
[Library Technical Report](http://open-std.org/jtc1/sc22/wg21/docs/library_technical_report.html)

## TODO List

* 脚本库，罗列编写常用sh功能脚本(Windows下可使用cmder或其他shell终端)
* 一个新的开发环境需要查看的cpu(物理cpu个数、每个cpu核心数、逻辑核数、是否支持超线程)，内存，磁盘信息

## 待看

* 编程规范, 参考：[程序员练级攻略：程序员修养](https://time.geekbang.org/column/article/8700)
    - C++, [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
    - Google 的 Shell 脚本编程规范, [Shell Style Guide](https://google.github.io/styleguide/shell.xml)
    - Effective Go, [《Effective Go》中英双语版](https://bingohuang.gitbooks.io/effective-go-zh-en/content/)
* 博客
    - Tony Bai，Go&C: [Tony Bai](https://tonybai.com/articles/)
        + [如何在Go语言中使用Websockets：最佳工具与行动指南](https://tonybai.com/2019/09/28/how-to-build-websockets-in-go/)
    - 白杨：[白杨的博客](http://baiy.cn/) 跨平台、分布式 C/C++ 开发，
        + 看了一篇《C++编码规范与技术指导-何时处理异常》，直接将自己的错误处理代码风格(do...while(0))列为了反面教材，，(实际中设置错误码break退出然后返回给客户端，自我感觉是挺清晰的，不过针对会抛异常可能导致段错误的块，确实需要额外捕获)
            * [代码风格与版式_异常](http://www.baiy.cn/doc/cpp/index.htm#%E4%BB%A3%E7%A0%81%E9%A3%8E%E6%A0%BC%E4%B8%8E%E7%89%88%E5%BC%8F_%E5%BC%82%E5%B8%B8)
    - [陶辉：聚焦分布式系统的程序员](https://blog.csdn.net/russell_tao)
        + 看了几篇 linux内核调度算法 的博客，加上查其他资料和极客时间上也有他的课程，看了下博文列表，很有学习价值
        + 性能优化，高性能编程，分布式，Nginx，MongoDB
        + 智链达CTO，《深入理解Nginx》作者+ 上面的CSDN看文章老提示登录，比较烦，不过目录导航还是比较便捷。博主的其他主页：
        + [个人站点](http://www.taohui.pub/?s=paxos)，[腾讯云](https://cloud.tencent.com/developer/article/1449436)
    - [鸟窝](https://colobu.com/)
        + Go并发
        + [基于protobuf快速生成服务治理的RPC代码](https://blog.rpcx.io/posts/generate-rpcx-code-from-protobuf-files/)
    - [yoko blog](https://pengrl.com/)
        + 一些Go的文章
        + 实现的基础库：[q191201771/naza](https://github.com/q191201771/naza)
* 官方
    - [golang.org](https://golang.org/)
        + go.dev发布之后，golang.org官网将更加聚焦go开源项目本身的开发、语言演化以及Go版本发布
    - [go.dev](https://go.dev/)
        + 该站点被Go核心团队寄望于成为全世界Gopher开发人员的中心
        + go.dev将成为gopher日常使用go语言的中心，包括go学习、go方案、go应用案例等
        + [Go官方发布的go.dev给gopher们带来了什么](https://tonybai.com/2019/11/14/what-the-godev-website-bring-to-gophers/)
* [用户态使用 glibc/backtrace 追踪函数调用堆栈定位段错误](https://blog.csdn.net/gatieme/article/details/84189280)
    - [如何调试没有core文件的coredump](https://zhuanlan.zhihu.com/p/56751496)
        + "backtrace在信号处理函数里运行是不安全的。"
* 源码
    - 网络编程 libevent(C)、Asio(C++ boost)、ACE(C++)
* C++筑基
    - [C++11 并发指南系列](https://www.cnblogs.com/haippy/p/3284540.html)

