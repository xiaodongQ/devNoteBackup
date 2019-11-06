## Act with Prudence

### 翻译

原文链接：  
[Act with Prudence](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_01/)

#### 行事谨慎

Act with Prudence(Prudence 英 /ˈpruːdns/ 审慎，谨慎)

>"无论你做什么，都要行事谨慎并考虑后果" Anon

"Whatever you undertake, act with prudence and consider the consequences" Anon

无论一个计划在开始迭代的前期看起来多么美好，你无法避免在后续的时间里承受着压力。如果你发现必须在“正确的做”和“快速的做”之间做一个选择，经常性的结果是你会基于后续很快会回过头来修复这些问题，然后被“快速的做”吸引。当你对你自己、你的团队、你的顾客承诺时，你是这个意思。  
但是情况往往是在下一次迭代时，又带来了新的问题，然后你开始关注于这些问题。这类拖延的工作被称作技术债务，而这不应该成为你的朋友。具体的说，马丁·福勒[^Martin Fowler]在他的技术债务分类中指出，这种故意的技术债务不应该和无意的技术债务混淆在一起。

[^Martin Fowler]: 马丁·福勒，是一个软件开发方面的著作者和国际知名演说家，专注于面向对象分析与设计，统一建模语言，领域建模，以及敏捷软件开发方法，包括极限编程，2000年3月，他成为ThoughtWorks的首席科学家。([马丁·福勒百度百科](https://baike.baidu.com/item/%E9%A9%AC%E4%B8%81%C2%B7%E7%A6%8F%E5%8B%92/3107032?fromtitle=martin%20fowler&fromid=9005728))

No matter how comfortable a schedule looks at the beginning of an iteration(迭代，循环), you can't avoid being under pressure some of the time. If you find yourself having to choose between "doing it right" and "doing it quick" it is often appealing to "do it quick" on the understanding that you'll come back and fix it later. When you make this promise to yourself, your team, and your customer, you mean it. But all too often the next iteration brings new problems and you become focused on them. This sort of deferred work is known as technical debt and it is not your friend. Specifically(特别地，具体的说), Martin Fowler(马丁·福勒 ThoughtWorks的首席科学家) calls this deliberate(英 /dɪˈlɪbərət/ 故意的，蓄意的) technical debt in his taxonomy(英 /tækˈsɒnəmi/ 分类) of technical debt, which should not be confused(英 /kənˈfjuːzd/ 混乱的，糊涂的。跟refuse：拒绝 区分开) with inadvertent(英 /ˌɪnədˈvɜːtnt/ 疏忽的，无意中做的) technical debt.

技术债务像贷款一样，你短期内能从中获益，但是全部偿还的时候你必须为其偿付利息。代码中的捷径使得添加特性或者重构代码的时候更加困难。它们是不合格和脆弱的测试用例的诞生地。你离开的时间越长，带来的后果越严重。等到你开始抽出时间来开始修复最初的问题的时候，可能有一整堆基于最初问题的不完全正确的设计，使得代码变得更加难以重构和修正。事实上，很多时候是当事情变得如此糟糕以至于你必须修复它，你才会真正回头来进行修复。到那时，通常修复起来是很困难的，以至于你真的承担不起时间或者风险。

Technical debt is like a loan(英 /ləʊn/ 贷款): You benefit from it in the short-term, but you have to pay interest on it until it is fully paid off. Shortcuts(快捷键，捷径) in the code make it harder to add features or refactor(重构) your code. They are breeding(繁殖，生产) grounds for defects(缺点，瑕疵，不合格) and brittle(脆弱的) test cases. The longer you leave it, the worse it gets. By the time(等到，到...时候) you get around to(get around to, 抽出时间做，开始考虑做) undertaking the original fix there may be a whole stack(一整堆) of not-quite-right(不完全正确) design choices layered(分层的) on top of the original problem making the code much harder to refactor and correct(改正vt. 正确的adj.). In fact, it is often only when things have got so bad that you must fix it, that you actually do go back to fix it. And by then it is often so hard to fix that you really can't afford the time or the risk.

~~在你必须引发技术债来赶上截止日期或者实现特性的一小部分时，你还有时间。~~  
有时候，你必须承担技术债务来赶上截止日期或者实现功能的一小部分。尽量不要处在这种位置，~~但是形势完全要求这样~~ 但是如果情况绝对需要，那就继续吧。但是(而且这是个很大的但是)，你必须跟进技术债务并且马上偿还，要不事情会急转直下。一旦你做了妥协的决定，纪要把它写在任务卡上，或者记录在你发问题追踪系统中，以确保它不被忘记。

There are times when you must incur(/ɪn'kɜː/ 引发，蒙受) technical debt to meet a deadline or implement a thin slice of a feature. Try not to be in this position, but if the situation absolutely demands(要求) it, then go ahead. But (and this is a big BUT) you must track technical debt and pay it back quickly or things go rapidly downhill. As soon as(一...就) you make the decision to compromise(/'kɒmprəmaɪz/ 妥协), write a task card or log it in your issue(发行，问题) tracking system to ensure that it does not get forgotten.

如果你在下一次迭代中安排偿还技术债务，损失将会最小化。保留债务不偿还将会产生利息，并且应该跟踪该利息来使损失可见。需要强调项目的技术债务在商业价值上的影响，并且适当地安排偿还的优先级。如何计算和跟踪技术债务的利息依赖于具体的项目，但是跟踪是你必须要做的。

If you schedule(/'ʃedjuːl/ /'skɛdʒul/ 安排) repayment of the debt in the next iteration, the cost will be minimal. Leaving the debt unpaid will accrue(/ə'kruː/ 产生，积累) interest and that interest should be tracked to make the cost(成本，损失) visible. This will emphasize(/ˈemfəsaiz/ 强调) the effect on business value of the project's technical debt and enables appropriate(/ə'prəʊprɪət/ 适当的) prioritization(优先次序) of the repayment. The choice of how to calculate and track the interest will depend on the particular(详细的) project, but track it you must.

尽快地还清技术债务，否则将是不明智的。

Pay off(还清) technical debt as soon as possible. It would be imprudent(不明智的) to do otherwise.

Seb Rose, 顾问，教练，设计师，分析师，和超过30年的开发者。
> Consultant, coach, designer, analyst and developer for over 30 years. 《The Cucumber for Java Book》主要作者。  
> Contributing author to “97 Things Every Programmer Should Know” (O’Reilly) and lead author of “The Cucumber for Java Book” (Pragmatic Programmers)  
参考：[Seb Rose](https://leanpub.com/u/sebrose)

By Seb Rose

### 思考

有时候必须承担技术债务才能赶上截止日期，但是必须跟进它，并且尽快偿还。

项目里经常有这种妥协，有问题的功能点这一轮先不发布，留到下一轮。  
但是下一轮又关注别的可能更紧急的事情去了，导致时间越久，相关联的新功能只能基于这个问题做各种兼容性的设计。  
最后结果是积重难返。

缺乏的是一个技术债务追踪系统，和更重要的：技术债务清单的解决排期。在评估项目周期时，需要各个模块将这部分时间加到评估耗时当中。