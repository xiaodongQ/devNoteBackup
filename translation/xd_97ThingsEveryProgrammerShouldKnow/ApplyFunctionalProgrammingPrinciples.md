## Apply Functional Programming Principles

### 翻译

原文链接：  
[Apply Functional Programming Principles](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_02/)

#### 应用函数式编程原理

函数式编程最近受到了主流编程社区的重新关注。部分原因是因为函数式范式的天然属性可以很好地解决我们的行业向多核转型所带来的挑战。然而，尽管那当然是一个重要的应用，但不是本文建议您了解函数式编程的原因。

Functional programming has recently enjoyed renewed interest from the mainstream programming community. Part of the reason is because emergent(意外的,自然发生的) properties of the functional paradigm(/'pærədaɪm/ 范例) are well positioned to(定位于) address the challenges(应对挑战) posed by(由...构成) our industry's(工业、产业) shift toward multi-core. However, while that is certainly an important application, it is not the reason this piece(件;篇;凑合) admonishes(/əd'mɒnɪʃ/ 劝告、训诫) you to know thy(/ðaɪ/ 你的) functional programming.


精通函数式编程范式能够极大提升你在其他上下文里编写的代码的质量。如果你深入理解并应用函数式范式，你的设计将会表现出更高的引用透明性

Mastery(/'mɑːst(ə)rɪ/ 精通;优势) of the functional programming paradigm can greatly improve the quality of the code you write in other contexts(环境;上下文). If you deeply understand and apply the functional paradigm, your designs will exhibit(/ɪg'zɪbɪt/ 显示、展出) a much higher degree of referential(/ˌrefə'renʃ(ə)l/ 用作参考的) transparency(/træn'spær(ə)nsɪ/ 透明;透明性).

引用透明性是非常可取的属性：这意味着在给定相同输入的情况下，函数始终产生相同的结果，而不管它们在何时何地被调用。也就是说，函数计算对可变状态的副作用的依赖程度较小（理想情况下完全不依赖）。

Referential transparency is a very desirable(值得拥有的) property(财产;性质): It implies that functions consistently(/kənˈsɪstəntlɪ/ 一贯地;坚持地) yield(/jiːld/ 出产;屈服) the same results given the same input, irrespective(不考虑的,不顾的) of where and when they are invoked(调用). That is, function evaluation(估算;值的计算) depends less — ideally, not at all — on the side effects(on the side effects 副作用) of mutable state.

在命令式代码中导致缺陷的一个主要原因是因为可变变量。每个阅读此处的人应该都排查过为什么在特定情况下某些值和预期不一致。可见性语义可以帮助减轻这些隐患，或者至少可以大大缩小其位置，但其真正的罪魁祸首实际上可能是采用了过多的可变性的设计。

A leading(主要的;领导的;行距) cause of defects(缺点;瑕疵) in imperative(必要的;极重要的;命令的) code is attributable(/ə'trɪbjətəbl/ 可归因于的;由于) to mutable variables. Everyone reading this will have investigated(调查) why some value is not as expected in a particular situation. Visibility semantics(/sɪ'mæntɪks/ 语义学) can help to mitigate(减轻) these insidious(/ɪn'sɪdɪəs/ 潜在的) defects, or at least to drastically(大大地) narrow(狭窄;变窄) down their location, but their true culprit(/'kʌlprɪt/ 犯人,罪犯) may in fact be the providence(/'prɒvɪd(ə)ns/ 远见;天意) of designs that employ(使用;雇佣) inordinate(/ɪ'nɔːdɪnət/ 过度的,过量的) mutability(易变性).

在这方面，我们当然不会从行业中获得太多帮助。对面向对象的采用默认地促进了这种设计，因为它们经常有由生存周期相对较长的对象组成的、对象间互相愉快地调用设值方法的示例，这可能很危险。然而，使用敏捷测试驱动的设计，特别是对确切的模拟角色，而不是对象时，可以设计消除掉不必要的可变性。

And we certainly don't get much help from industry in this regard(问候;尊重;注意)(in this regard在这方面). Introductions(介绍;引进;采用) to object orientation tacitly(/'tæsitli/ 沉默地) promote(提升;推动) such design, because they often show examples composed of graphs(图表) of relatively long-lived objects that happily call mutator(['mju:teitə] 改变对象属性的方法,设值方法) methods on each other(互相), which can be dangerous. However, with astute(/ə'stjuːt/ 机敏的,精明的) test-driven design, particularly when being sure to "Mock Roles, not Objects", unnecessary mutability can be designed away.

最终的结果是，这种设计通常具有更好的责任分配，并使用更多更小的函数来处理传入的参数，而不是引用可变的成员变量。这样缺陷将会更少，而且它们通常更容易调试，因为在这些设计中找到引入非法值的位置，要比推断引起错误赋值的特定上下文更容易。这增加了更高程度的引用透明性，而且没有什么比学习函数式编程语言更能深入您的骨髓了，在函数式编程语言中，这种计算模型是标准的。

The net(网;得到;净余的) result is a design that typically(典型地) has better responsibility allocation with more numerous(很多的), smaller functions that act on arguments passed into them, rather than referencing mutable member variables. There will be fewer defects, and furthermore they will often be simpler to debug, because it is easier to locate where a rogue(/rəʊg/ 流氓;凶猛的) value is introduced(提出;引进) in these designs than to otherwise deduce(推断) the particular context that results in an erroneous(/ɪ'rəʊnɪəs/ 错误的) assignment. This adds up to a much higher degree of referential transparency, and positively nothing will get these ideas as deeply into your bones as learning a functional programming language, where this model of computation is the norm.

当然，这种方法并非在所有情况下都是最优的。例如，在面向对象的系统中，与用户界面开发相比，这种形式通常在领域模型开发(即协作可打破业务规则的复杂性）方面产生更好的结果。

Of course, this approach(/ə'prəʊtʃ/ 方法;入口) is not optimal(/'ɒptɪm(ə)l/ 最佳的) in all situations. For example, in object-oriented systems this style often yields better results with domain model development (i.e., where collaborations(/kə'læbəreɪt/ 合作) serve to break down the complexity of business rules) than with user-interface development.

掌握函数式编程范式，这样您就能够明智地将学到的经验应用于其他领域。您的目标系统(对于一个)将同时拥有引用透明性的优点，并且比你预期想要的更接近对应的功能。实际上，甚至有人断言，函数式编程的顶点和面向对象只是彼此的反映，是一种计算阴和阳的形式。

Master the functional programming paradigm so you are able to judiciously(/dʒu:'diʃəsli/ 明断地;明智地) apply the lessons learned to other domains. Your object systems (for one) will resonate(/'rez(ə)neɪt/ 共鸣;共振) with referential transparency goodness(精华;良好;上帝) and be much closer to their functional counterparts(/'kaʊntəpɑːt/ 与对方地位相当的人) than many would have you believe. In fact, some would even assert that the apex(/'eɪpeks/ 定点;顶端) of functional programming and object orientation are merely(仅仅) a reflection of each other, a form of computational(/ˌkɑmpju'teʃənl/ 计算的) yin and yang.

爱德华·加森

>一名经验丰富的软件开发者、作家和投资者，拥有20多年与世界上最热门的品牌合作的经验，包括沃达丰、英国航空、欧佩克基金和食品网络。兴趣包括技术、创造性的问题解决、软件架构设计、自动化测试和敏捷方法。包括函数式编程、通用语言设计和计算、跨平台移动开发工具、emacs和GNU Linux和数据科学/数学。
参考[领英介绍](https://www.linkedin.com/in/egarson/)

By Edward Garson

### 思考

对于函数式编程不大了解，看得还是有点懵。。

只是大概知道C++中的lambda和Golang里的匿名函数可以往这方面靠，目前还没适应这种用法。

* 引用透明性
	- 参考：[函数的引用透明性（referential transparency）](https://blog.csdn.net/lanchunhui/article/details/52473003)
	- 函数的返回值只依赖于其输入值，这种特性就称为引用透明性（referential transparency）(不受全局变量等其他因素的影响)

* 使用函数式编程范式来编程，有助于提高函数的引用透明性。
* 在非函数式编程中，设计上也应该是用更多更小的功能函数来进行明确的功能责任划分。
* 减少函数间的耦合关系，设计时各函数尽量只依赖输入值