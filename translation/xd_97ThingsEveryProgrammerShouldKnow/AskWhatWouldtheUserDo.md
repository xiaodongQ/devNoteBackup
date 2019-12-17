## Ask "What Would the User Do?" (You Are not the User)

### 翻译

原文链接：  
[Ask "What Would the User Do?" (You Are not the User)](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_03/)

#### 问“用户会怎么做？”(你不是用户)

我们总是倾向于别人和我们想得一样，但是并不是这样。心理学家把这个叫做"虚假同感偏差"。当人们想得或者做得和我们不一样时，我们很可能以某些方式(下意识地)给他们贴上某种缺陷的标签。

We all tend to(倾向于；常常) assume that other people think like us. But they don't. Psychologists(英 /sai'kɔledʒist/,心理学家) call this the false consensus(英 /kənˈsensəs/, 共识；一致) bias(英 /ˈbaɪəs/,偏见；斜纹。 幸存者偏差：survivorship bias,英 /sə'vaɪvəʃɪp/). When people think or act differently to us, we're quite likely to label them (subconsciously(英 /ˌsʌbˈkɒnʃəsli 下意识地)) as defective(英 /dɪˈfektɪv/ 有缺陷的) in some way.

这种偏见解释了为什么程序员很难把自己放在用户的位置上。用户不像程序员那样思考。首先，他们就用很少的时间使用计算机。他们不知道也不关心计算机是如何工作的。这意味着他们不能使用程序员所熟悉的解决问题的技术。他们不能认出程序员通过用户界面提供的模式和线索。

This bias explains why programmers have such a hard time putting themselves in the users' position. Users don't think like programmers. For a start(首先), they spend much less time using computers. They neither know nor care how a computer works. This means they can't draw on(利用，动用) any of the battery of problem-solving techniques so familiar to programmers. They don't recognize the patterns and cues(英 /kjuːz/  线索；提示) programmers use to work with, through, and around an interface.

了解用户想法的最好方式就是观察一个用户场景。请用户使用与你正开发软件类似的软件来完成一项任务。确保任务是真实的："一列数字相加"不错，"计算你上个月的花费"更好。避免任务过于具体，像"你能选中这些电子表格单元并在下面输入一个求和公式吗?"--在这个问题中有一个很大的提示。让用户谈论他或者她的进展。不要打断，不要试着帮忙，不断问自己"为什么他要做这个?"和"为什么她不做这个?"。

The best way to find out how users think is to watch one. Ask a user to complete a task using a similar piece of software to what you're developing. Make sure the task is a real one: "Add up a column of numbers" is OK; "Calculate your expenses for the last month" is better. Avoid tasks that are too specific(英 /spəˈsɪfɪk 特定的；详细的；明确的), such as "Can you select these spreadsheet(英 /ˈspredʃiːt/,电子表格) cells and enter a SUM formula(英 /ˈfɔːmjələ/,公式) below?" — there's a big clue in that question. Get the user to talk through(谈论，讨论) his or her progress. Don't interrupt. Don't try to help. Keep asking yourself "Why is he doing that?" and "Why is she not doing that?"

你首先会注意到用户做事的核心是类似的。他们试图用同样的顺序完成任务--并且他们在相同的地方犯同样的错误。你应该围绕着那个核心行为进行设计。这和设计会议不同，设计会议上常常会听到说"如果用户想做...怎么办?"，这样导致了复杂的特性和用户需求的混乱。观察用户来消除这种混乱。

The first thing you'll notice is that users do a core of things similarly. They try to complete tasks in the same order — and they make the same mistakes in the same places. **You should design around that core behavior**. This is different from design meetings, where people tend to be listened to for saying "What if the user wants to...?" This leads to elaborate(英 /ɪ'læb(ə)rət/, 详尽的；复杂的) features and confusion over what users want. Watching users eliminates(英 /ɪˈlɪmɪneɪt/ 消除) this confusion.

你将会看到用户陷入困境。当你陷入困境时，你会环顾四周。当用户陷入困境时，他们会缩小他们的关注范围，这让他们很难在屏幕上的其他地方看到解决方案。这也是为什么帮助文档对于一个糟糕的用户界面设计来说是一个糟糕的解决方案。如果一定要有说明或者帮助文档，确保它正确地放在靠近相应问题的区域中。用户注意力关注的焦点很窄，这也是为什么工具提示比帮助菜单更有用。

You'll see users getting stuck(英 /stʌk/, 卡住的，卡壳). When you get stuck, you look around. When users get stuck, they narrow their focus. It becomes harder for them to see solutions elsewhere on the screen. It's one reason why help text is a poor solution to poor user interface design. If you must have instructions or help text, make sure to locate it right next to your problem areas. A user's narrow focus of attention is why tool tips are more useful than help menus.

用户往往敷衍了事，他们会找到一个可行的方法，并且不管有多复杂他们都会坚持使用它。提供一种显而易见的做事方式比提供两三种捷径更好。你还会发现用户说他们想要什么和他们实际做的之间存在差距。这很令人担忧，因为正常的收集用户需求的方式就是询问他们。这就是为什么观察用户是抓住需求的最佳方式。花一个小时观察用户比花一天时间猜用户想要什么更能提供有用的信息。

Giles Colborne

(《简约至上》的作者。物理学出身，参与过航空航天项目，干过出版，现在开办了自己的设计公司cxpartners。
参考：[简约至上》作者Giles Colborne：让用户变成设计的一部分](http://www.woshipm.com/pd/130807.html))

Users tend to muddle(英 /ˈmʌdl/, 困惑；混乱。muddle through 混过去；走过场；敷衍了事) through. They'll find a way that works and stick with it no matter how convoluted(英 /ˈkɒnvəluːtɪd/, 复杂的). It's better to provide one really obvious way of doing things than two or three shortcuts. You'll also find that there's a gap(英 /ɡæp/, 差距；分歧) between what users say they want and what they actually do. That's worrying as the normal way of gathering(英 /ˈɡæðərɪŋ/,收集，聚集) user requirements is to ask them. It's why the best way to capture requirements is to watch users. Spending an hour watching users is more informative than spending a day guessing what they want.
by Giles Colborne

### 思考

用户想要的和实际做的往往存在差距。观察他们使用类似工具解决实际问题的步骤比只是问用户想要什么更能提供有用信息，更能抓住核心需求。

* 为什么要观察：
    - 用户(其实不单是用户)做事的核心是类似的
    - 大家(倾向于)试图用同样的顺序完成任务，并且很容易在相同的地方犯同样的错误
    - 多观察有助于找到做事的核心模型(方法论)，然后围绕模型进行改善