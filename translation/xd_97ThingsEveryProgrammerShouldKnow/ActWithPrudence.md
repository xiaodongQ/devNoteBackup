## Act with Prudence

### 翻译

原文链接：  
[Act with Prudence](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_01/)

#### 行事谨慎

Act with Prudence(Prudence 英 /ˈpruːdns/ 审慎，谨慎)

>"无论你从事着什么，行事谨慎并考虑后果" Anon

"Whatever you undertake, act with prudence and consider the consequences" Anon

无论一个计划在开始迭代的前期看起来多么美好，你无法避免在后续的时间里承受着压力。如果你发现必须在“正确的做”和“快速的做”之间做一个选择，经常性的结果是你会基于后续很快会回过头来修复这些问题，然后被“快速的做”吸引。当你对你自己、你的团队、你的顾客承诺时，你是这个意思。  
但是情况往往是在下一次迭代时，又带来了新的问题，然后你开始关注于这些问题。这类拖延的工作被称作技术债，而这不应该成为你的朋友。具体的说，马丁·福勒[^Martin Fowler]在他的技术债分类中指出，这种故意的技术债不应该和无意的技术债混淆在一起。

[^Martin Fowler]: 马丁·福勒，是一个软件开发方面的著作者和国际知名演说家，专注于面向对象分析与设计，统一建模语言，领域建模，以及敏捷软件开发方法，包括极限编程，2000年3月，他成为ThoughtWorks的首席科学家。([马丁·福勒百度百科](https://baike.baidu.com/item/%E9%A9%AC%E4%B8%81%C2%B7%E7%A6%8F%E5%8B%92/3107032?fromtitle=martin%20fowler&fromid=9005728))

No matter how comfortable a schedule looks at the beginning of an iteration(迭代，循环), you can't avoid being under pressure some of the time. If you find yourself having to choose between "doing it right" and "doing it quick" it is often appealing to "do it quick" on the understanding that you'll come back and fix it later. When you make this promise to yourself, your team, and your customer, you mean it. But all too often the next iteration brings new problems and you become focused on them. This sort of deferred work is known as technical debt and it is not your friend. Specifically(特别地，具体的说), Martin Fowler(马丁·福勒，是一个软件开发方面的著作者和国际知名演说家，专注于面向对象分析与设计，统一建模语言，领域建模，以及敏捷软件开发方法，包括极限编程) calls this deliberate(英 /dɪˈlɪbərət/ 故意的，蓄意的) technical debt in his taxonomy(英 /tækˈsɒnəmi/ 分类) of technical debt, which should not be confused(英 /kənˈfjuːzd/ 混乱的，糊涂的。跟refuse：拒绝 区分开) with inadvertent(英 /ˌɪnədˈvɜːtnt/ 疏忽的，无意中做的) technical debt.

技术债像贷款一样，你短期内能从中获益，但是全部偿还的时候你必须为其偿付利息。

Technical debt is like a loan(英 /ləʊn/ 贷款): You benefit from it in the short-term, but you have to pay interest on it until it is fully paid off. Shortcuts in the code make it harder to add features or refactor your code. They are breeding grounds for defects and brittle test cases. The longer you leave it, the worse it gets. By the time you get around to undertaking the original fix there may be a whole stack of not-quite-right design choices layered on top of the original problem making the code much harder to refactor and correct. In fact, it is often only when things have got so bad that you must fix it, that you actually do go back to fix it. And by then it is often so hard to fix that you really can't afford the time or the risk.

There are times when you must incur technical debt to meet a deadline or implement a thin slice of a feature. Try not to be in this position, but if the situation absolutely demands it, then go ahead. But (and this is a big BUT) you must track technical debt and pay it back quickly or things go rapidly downhill. As soon as you make the decision to compromise, write a task card or log it in your issue tracking system to ensure that it does not get forgotten.

If you schedule repayment of the debt in the next iteration, the cost will be minimal. Leaving the debt unpaid will accrue interest and that interest should be tracked to make the cost visible. This will emphasize the effect on business value of the project's technical debt and enables appropriate prioritization of the repayment. The choice of how to calculate and track the interest will depend on the particular project, but track it you must.

Pay off technical debt as soon as possible. It would be imprudent to do otherwise.

By Seb Rose
