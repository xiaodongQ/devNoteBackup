## Automate Your Coding Standard

### 翻译

原文链接：  
[Automate Your Coding Standard](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_04/)

#### 自动化你的编码规范

你可能也经历过这样的情况：在项目开始时，每个人都有很多的美好的意愿--称这些为"新项目的决议"。很多时候，这些决议中很多都写进了文档里。关于代码的那些决议最终成为了项目的编码规范。在项目启动会期间，主要开发人员会仔细检查这些文档，然后，最好的情况下每个人都同意他们会遵循这些。然而，一旦项目开始，这些好的意愿一个个都被遗弃一旁。当项目最终交付的时候，代码看起来一团糟，而且似乎没有人知道项目是怎么变成这样的。

You've probably been there too. At the beginning of a project, everybody has lots of good intentions(目的，意向；打算) — call them "new project's resolutions(美 /,rɛzə'lʊʃən/, 决议；决心；分辨率)." Quite often(Quite often 经常；很多时候), many of these resolutions are written down in documents. The ones about code end up in the project's coding standard. During the kick-off(项目启动) meeting, the lead developer goes through(通过；仔细检查) the document and, in the best case, everybody agrees that they will try to follow them. Once the project gets underway(get underway 开始进行), though, these good intentions are abandoned, one at a time. When the project is finally delivered(deliver,英 /dɪˈlɪvə(r)/,交付；实现；投递) the code looks like a mess, and nobody seems to know how it came to be this way.

是什么时候出问题了呢？可能在项目启动会的时候就已经出了问题，而一些项目成员是没有注意，其他一些是并没有理解这些规范。更坏的情况是一些成员并不认同，而且已经在准备不遵循这些编码规范了。最后，一些理解这些编码规范并且遵循的项目成员，当项目压力变得太大时，他们也不得不放弃一些东西。格式良好的代码并不会为你赢得想要更多功能的客户的好感。此外，如果没有自动化，遵循一个编码规范将会是一件很枯燥的任务。你可以尝试手工缩进一个混乱的类来看是不是如此。

When did things go wrong? Probably already at the kick-off meeting. Some of the project members didn't pay attention. Others didn't understand the point. Worse, some disagreed and were already planning their coding standard rebellion(英 /rɪˈbeljən/,反抗；不服从). Finally, some got the point and agreed but, when the pressure in the project got too high, they had to let something go. Well-formatted code doesn't earn you points with a customer that wants more functionality. Furthermore, following a coding standard can be quite a boring task if it isn't automated. Just try to indent a messy(凌乱；乱七八糟) class by hand to find out for yourself.

但是如果还是这样的问题(有了规范还是一团糟)，那为什么一开始我们还是要有一个编码规范呢？以统一格式来格式化代码的一个原因是防止任何人都可以用他/她私有的方式来格式化一小块代码。为了避免一些常见的错误，我们可能会希望阻止开发者使用某些反模式。总之，一个编码标准应该使项目工作更容易，并且从始至终保持维护和开发的速度。因此，每个人也应该就编码规范达成一致--如果一个开发者用三个空格缩进代码而另一个用四个空格，这对项目并没有什么帮助。

But if it's such a problem, why is that we want to have a coding standard in the first place? One reason to format the code in a uniform way is so that nobody can "own" a piece of code just by formatting it in his or her private way. We may want to prevent developers using certain(必然的；某些) anti-patterns(anti,反对的), in order to avoid some common bugs. In all, a coding standard should make it easier to work in the project, and maintain development speed from the beginning to the end. It follows then that everybody should agree on the coding standard too — it does not help if one developer uses three spaces to indent code, and another one four.

有许多可以用于生成代码质量报告、记录和维护编码规范的工具，但是这不是全部的解决方案。编码规范应该自动化，并在任何可能的地方强制执行。这里有些例子：

There exists a wealth of tools that can be used to produce code quality reports and to document and maintain the coding standard, but that isn't the whole solution. It should be automated and enforced where possible. Here are a few examples:

* 确保代码格式化是编译过程的一部分，这样每个人都能在每次编译代码的时候自动格式化代码。
* 使用静态代码分析工具来扫描代码中不需要的反模式。如果扫描到则退出构建。
* 学习配置那些工具，这样你就可以扫描你自己的特定项目中的反模式。
* 不要只度量测试覆盖率，对测试结果也进行自动化检测。另外，如果测试覆盖率太低则退出构建。

* Make sure code formatting is part of the build process, so that everybody runs it automatically every time they compile the code.
* Use static code analysis tools to scan the code for unwanted anti-patterns. If any are found, break the build.
* Learn to configure those tools so that you can scan for your own, project-specific anti-patterns.
* Do not only measure test coverage, but automatically check the results too. Again, break the build if test coverage is too low.

尽可能对每件你认为重要的事情采用上述方法。你无法自动化所有你真正关心的事情。至于那些你没法自动标记或者修正的东西，把它们当做是一组自动化编码规范外的补充指南，但是要接受这个事实，你和你的同事们可能并不会努力地遵守这些。

Try to(试图；尽力) do this for everything that you consider important. You won't be able to automate everything you really care about. As for the things(至于那些东西) that you can't automatically flag or fix, consider them to be a set of guidelines(指南；指导) supplementary(补充的；追加的) to the coding standard that is automated, but accept that you and your colleagues may not follow them as diligently(英 /ˈdɪlɪdʒəntli/ 勤奋地).

最后，编码规范应该是动态而不是静态的。随着项目的发展，项目的需求会发生变化，开始阶段看似聪明的想法，几个月后就不一定是这样了。

Finally, the coding standard should be dynamic rather than static. As the project evolves, the needs of the project change, and what may have seemed smart in the beginning, isn't necessarily(不一定) smart a few months later.

* By Filip van Laenen
    - Computas公司的首席工程师，拥有鲁汶大学计算机科学硕士，电子与电子工程硕士学位，超过20年软件工程经验
    - 参考：[Filip Van Laenen](https://www.linkedin.com/in/filipvanlaenen)

By Filip van Laenen

### 思考

* 自动化检查编码规范，有利于消除代码的坏味道。
* 当前的编辑器已经做好了部分这样的事情。
    - Go的语法和格式不满足标准时会进行提示(命名)或者直接报错(缩进)，还会自动化格式一些代码。
    - VS Code中各种语言的lint插件，用于检查代码格式
* 不将编码规范(代码规范)自动化，或者说不进行强制执行，很容易导致的结果就是： 各人按各人的习惯拼凑成格式风格混乱的项目代码。
    - 在不好的风格上面进行后续的开发维护，很容易积重难返(一直没花精力进行重构导致技术债累积)。
    - 看过的博客中有类似例子："但是由于前期的经验不足，在内部代码结构上留下了比较恶劣的smell，后来人也“效仿”了我的风格，以致现在我再看B-2代码，只能用“惨不忍睹”来形容了，Dreamhead的fanfou中曾经提到过*“给予程序员最佳的惩罚便是让他维护自己一年前编写的代码”*。" 详见：[Tony Bai个人博客文章](https://tonybai.com/2008/12/23/the-dawn/)

