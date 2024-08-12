**ZK learn**

记录零知识证明（Zero-Knowledge Proof，ZKP）技术学习。

## 1. 初步了解ZK概念

1、视频：Amit Sahai教授针对不同年龄和知识水平的人讲解ZKP的几个示例

Amit Sahai是UCLA的计算机方面的一位杰出教授，研究兴趣集中在密码学和安全等领域。

油管视频：[初步理解 ZK 是什么](https://www.youtube.com/watch?v=fOGdb1CTu5c)

2、中文播客：[零知识证明](https://www.xiaoyuzhoufm.com/episode/6672a76bb6a8412729e0b103)

还是先看文章吧，，只是听吸收不到多少内容

3、[社区](https://learn.z2o-k7e.world/)整理的学习ZKP资料，可作为参考学习

[ZKP 新手村入门攻略](https://learn.z2o-k7e.world/beginner.html)

番外：  
找郭宇老师推特（[1dot2](https://twitter.com/1dot2)）过程中，看到已退推的[kurtpan.eth](https://twitter.com/kurtpan666)（复旦密码学博士，也是上面z2o-k7e项目的主要贡献者之一）博客上截取自《过河卒》的这篇文章：[过河卒·日出江花红胜火](https://zkfold.ing/guohezu-wd)。  
周六去杭州的web3 summer线下，正好嘉宾也谈到比特币白皮书和国人的渊源，引用文献的第一篇就是一个华人的文章。也就是上面文章中的[Wei Dai](https://en.wikipedia.org/wiki/Wei_Dai)。

先学习下面的3篇文章，建立对 zkp 的直观理解：

* [1. 初识「零知识」与「证明」](https://learn.z2o-k7e.world/zkp-intro/1/zkp-back.html)
* [2. 理解「模拟」](https://learn.z2o-k7e.world/zkp-intro/2/zkp-simu.html)
* [3. 寻找「知识」](https://learn.z2o-k7e.world/zkp-intro/3/zkp-pok.html)

### 1.1. [1. 初识「零知识」与「证明」](https://learn.z2o-k7e.world/zkp-intro/1/zkp-back.html)

三个概念：`证明`、`知识`、`零知识`

不同年代对`证明`的诠释：证明 == 洞见、证明 == 符号推理、证明 == 程序、证明 == 交互

> 交互证明的表现形式是两个（或者多个图灵机）的「对话脚本」，或者称为 Transcript。而这个对话过程，其中有一个显式的`「证明者」`角色，还有一个显式的`「验证者」`。其中证明者向验证者证明一个命题成立，同时还 **「不泄露其他任何知识」**。这种就被称为`「零知识证明」`。
> 
> 证明凝结了`「知识」`，但是证明过程却可以不泄露`「知识」`

零知识证明的一些用处：

* 数据的隐私保护。只需提供有限的信息
* 计算压缩与区块链扩容。有了计算的证明，就不用重复计算多次了，计算过程可压缩
* 端到端的通讯加密。不用担心服务器拿到所有的消息记录
* 身份认证。用户可以向网站证明，他拥有私钥
* 去中心化存储。向用户证明他们的数据被妥善保存并且不泄漏
* 信用记录。选择性出示满足对方要求的信用记录
* 构造完全公平的线上数字化商品的交易协议

**地图三染色问题：**

问题：如何用三种颜色染色一个地图，保证任意两个相邻的地区都是不同的颜色。

`「证明者」`Alice、`「验证者」` Bob。现在 Alice 想证明给 Bob 她有答案，但是又不想让 Bob 知道这个答案。

`信息（Information）` vs `知识（Knowledge）`

我们可以尝试定义一下，如果 Bob 在交互过程中获得的`「信息」`，可以帮助提升 Bob 直接破解 Alice 秘密的能力，那么我们说 Bob 「获得了知识」。
由此可见，知识这个词的定义与 Bob 的计算能力相关，如果信息并不能增加 Bob 的计算能力，那么信息不能被称为`「知识」`。

可是，Bob 每次看到的局部染色情况都是 Alice 变换过后的结果，无论 Bob 看多少次，都不能拼出一个完整的三染色答案出来。实际上，Bob 在这个过程中，虽然获得了很多`「信息」`，但是却没有获得真正的`「知识」`。

**可验证计算**、**电路可满足性问题**

先来了解 **`NP问题（Non-deterministic Polynomial time，非确定性多项式时间）`**：

NP问题（Nondeterministic Polynomial time问题）是计算复杂性理论中的一个重要概念。简单来说，NP问题是指那些`可以在多项式时间内验证答案`的决策问题。  
`多项式时间`是指算法的运行时间随着输入大小的增长而增长，但增长速度是缓慢的，运行时间可以用一个多项式函数来表示，比如线性搜索、二分搜索、快速排序。

在NP问题中，多项式时间是指`验证答案`的时间，而不是`求解问题`的时间。换句话说，NP问题的多项式时间是指检查答案是否正确的时间，而不是找到答案的时间。

在NP问题中，有一个特别重要的子集，称为`NP完全问题（NP-complete problems）`。NP完全问题是指那些在多项式时间内不能被转化为其他NP问题的NP问题。换句话说，NP完全问题是NP问题中的最难问题，比如旅行商问题、背包问题、图着色问题。

对于`NP-Complete`问题，求解过程是多项式时间内难以完成的，即`「求解困难」`，但是验证解的过程是多项式时间可以完成的，即`「验证简单」`

**算术电路：**

场景：Bob 交给 Alice 一段代码 P，和一个输入 x，让 Alice 来运行一遍，然后把运行结果（假设为y）告诉 Bob。

问题：如何让 Bob 在不运行代码的前提下，相信代码 P 运行的结果一定是 y 呢？

答案：Bob 把程序 P 转换成一个完全等价的算术电路，然后把电路交给 Alice。Alice 只要计算这个电路并把过程记录下来，并提供给Bob，Bob 就能确信 Alice 确实进行了计算。

所谓的 **`电路可满足性`**就是指：存在满足电路的一个解。如果这个解的输出值等于一个确定值，那么这个解就能「表示」电路的计算过程。

但是上述方法有弊端：1）如果电路比较大，那么证明就很大，Bob 检查证明的工作量也很大；2）Bob 在验证过程中，知道了所有的电路运算细节，包括输入

解决方式：Alice 以一种零知识的方式，向 Bob 证明她计算过了电路，并且使用了她的秘密输入

`「零知识的电路可满足性证明协议」`提供了一种最直接的保护隐私/敏感数据的技术

### 1.2. [2. 理解「模拟」](https://learn.z2o-k7e.world/zkp-intro/2/zkp-simu.html)

**`模拟`**是理解零知识的关键。任何一个零知识的协议，都可以通过构造一个`「理想世界」`来理解。

**问题**：如何证明一个交互式系统是「零知识」呢？

如果我们深入地分析每一个我们感觉安全的系统，都存在大量的似乎不那么稳固的`安全假设`。

所谓`安全假设`，比如我们说一个系统的权限隔离做得无比精确，每一个用户只能看到被授权的信息，但是这基于一个安全假设：管理员账号没有被破解。又比如在手机银行软件里，只能通过短信认证码，才能完成转账功能，这也基于一个安全假设：你的手机 SIM 卡没有被克隆。

比特币私钥安全吗？比特币账户的安全假设也不少：首先你的助记词不能让别人知道，手机钱包里私钥保存加密算法足够强，密钥派生算法正规，你不能忘记助记词，等等等。

`完美安全`（香农）：假设你是一个攻击者，你通过密文获取不到任何有价值的信息，破解的唯一手段就是靠瞎蒙。

`语义安全`（概率加密论文，Goldwasser 与 Micali 等）：假设你是一个攻击者，你通过密文在多项式时间内计算不出来任何有价值的信息。

**例子：**

1、两个世界：真实世界 vs 理想世界

真实世界`证明者`个体知道答案，即有`知识`；理想世界没有`知识`。

* 不可区分性，针对理想世界中的个体认知
* 可区分性，针对世界外部的“神”：模拟器（Simulator）

> 证明零知识的过程，就是要寻找一个算法，或者更通俗点说，写出一段代码，它运行在外部计算机系统中，但是实现了虚拟机的功能。而且在虚拟机中，需要有一个不带有「知识」作为输入的 Zlice，可以骗过放入虚拟机运行的 Bob。

2、阿里巴巴、洞穴与芝麻开门

3、柏拉图的洞穴寓言

### 1.3. [3. 寻找「知识」](https://learn.z2o-k7e.world/zkp-intro/3/zkp-pok.html)

本文探讨`「可靠性」`和`「To Know」`

「可靠性」(`Soundness`)保证了知识的「存在性」：Alice 在没有知识的情况下不能通过 Bob 的验证

「零知识」保证了 验证者 Bob 没有（计算）能力来把和「知识」有关的信息「抽取」出来。不能抽取的「知识」不代表不存在。「可靠性」保证了知识的「存在性」。

为了进一步分析「知识」，接下来首先介绍一个非常简洁，用途广泛的零知识证明系统 —— `Schnorr 协议`。

**`Schnorr 协议`举例：**

Alice 拥有一个秘密数字`a`，我们可以把这个数字想象成`「私钥」`，然后把它`「映射」`到椭圆曲线群上的一个点 `a*G`，简写为 `aG`。这个点我们把它当做`「公钥」`。

`sk = a`  (a当作私钥)
`PK = aG` (映射后当作公钥)

椭圆曲线群有限域之间存在着一种`同态映射`关系。

* 给任意一个有限域上的整数 r，我们就可以在循环群中找到一个对应的点 rG，或者用一个标量乘法来表示 r*G。
* 但是反过来计算是很`「困难」`的，这是一个「密码学难题」—— 被称为`离散对数难题`
    * 也就是说，如果任意给一个椭圆曲线循环群上的点 R，那么到底是有限域中的哪一个整数对应 R，这个计算是很难的，如果有限域足够大，比如说 256bit 这么大，我们姑且可以认为这个反向计算是不可能做到的。

个人简单理解：数字映射到椭圆曲线上的点容易，但反过来难度很大，基本当作不可能。

Schnorr 协议充分利用了有限域和循环群之间 **`单向映射`**，实现了最简单的零知识证明安全协议：Alice 向 Bob 证明她拥有 PK 对应的私钥 sk。

**证明零知识:**

弱一些的「零知识」性质：`SHVZK`(Special Honest Verifier Zero-Knowledge)

模拟器、抽取器

并不是所有的可靠性都必须要求存在抽取器算法。采用抽取器来证明可靠性的证明系统被称为`「Proof of Knowledge」`。

`ECDSA（Elliptic Curve Digital Signature Algorithm，椭圆曲线数字签名算法）`：

中本聪在构思比特币时，选择了 ECDSA 作为签名算法，但是曲线并没有选择 NIST（美国国家标准局） 标准推荐的椭圆曲线 —— `secp256-r1`，而是 `secp256-k1`。

几个脑洞：
黑客帝国、庄子、缸中之脑、计算机模拟的世界

数学知识来描述和证明加密算法，有点抽象，先放一放。

## 2. zk-SNARK

`zk-SNARK (Zero-Knowledge Succinct Non-Interactive Argument of Knowledge)`(简明扼要的非交互式知识论证)是一种特殊的零知识证明协议，它允许一方（证明者）向另一方（验证者）证明某个陈述是真的，而无需透露任何有关该陈述的信息，除了其真实性之外。

Succinct /sək'sɪŋ(k)t/ 简洁的；简明的；紧身的  
Argument 参数；论证

特点：

* 简洁性：证明非常短，验证这些证明的时间是常数级别的，这意味着验证时间与实际执行计算所需的时间无关。
* 非交互性：证明者可以独立创建证明，并将其发送给验证者，验证者可以在任何时间验证这个证明，无需进一步的交互。
* 零知识性：验证者无法从证明中获取除证明本身的真实性以外的任何信息。

构造：

* 可信设置：zk-SNARKs通常需要一个可信设置阶段，在此阶段会生成一些`公共参数`。这些参数是通过多方计算等方式生成的，目的是为了防止恶意参与者影响系统的安全性。
* 证明生成：*证明者*使用这些公共参数来生成一个证明，这个过程涉及到将计算`转换成算术电路`的形式，并在电路的基础上构建证明。
* 证明验证：*验证者*使用相同的公共参数来验证证明的有效性。验证算法检查证明是否正确无误地对应于原始计算。

应用：

* 隐私保护：在区块链应用中，如`Zcash`加密货币，zk-SNARKs用于实现匿名交易。
* 高效验证：在区块链和其他分布式系统中，zk-SNARKs能够减少验证复杂计算的成本，提高系统的效率和吞吐量。
* 数据验证：用于验证数据的完整性和一致性，而不泄露数据的具体内容。

技术细节：

* 椭圆曲线密码学：zk-SNARKs通常依赖于椭圆曲线对数问题作为其安全性基础。
* 多项式承诺：通过多项式承诺方案来确保计算的正确性。
* 算术电路：复杂的计算被转化为`算术电路`，使得验证者可以通过简单的代数运算验证计算结果。

SNARK相关数学知识：

* [1-Polynomial-Interaction-and-Proof](https://learn.z2o-k7e.world/zk-snarks/1-Polynomial-Interaction-and-Proof.html)
* [2-Non-interactivity&Distributed-Setup](https://learn.z2o-k7e.world/zk-snarks/2-Non-interactivity&Distributed-Setup.html)
* [3-General-Purpose-Computation](https://learn.z2o-k7e.world/zk-snarks/3-General-Purpose-Computation.html)
* [4-Construction-Properties.md](https://learn.z2o-k7e.world/zk-snarks/4-Construction-Properties.html)
* [5-Pinocchio-Protocol](https://learn.z2o-k7e.world/zk-snarks/5-Pinocchio-Protocol.html)

第一篇还能大概跟上，剩下的大概浏览了下，理解还是比较费劲。

### 2.1. [1-Polynomial-Interaction-and-Proof](https://learn.z2o-k7e.world/zk-snarks/1-Polynomial-Interaction-and-Proof.html)

Polynomial-Interaction-and-Proof：多项式交互和证明

Polynomial /ˌpɒlɪ'nəʊmɪəl/ 多项式；多项式的

**对于一个阶数为 d 的多项式最多有 d 个解，`至多有 d 个共同点`。**

举例1：假设有两个多项式

* f(x) = x³ – 6x² +11x– 6
* f(x) = x³ – 6x² + 10x– 5

它们的交点：x³ – 6x² + 11x – 6 = x³ – 6x² + 10x – 5，得到 x=1，即这两个多项式有一个交点。

任意一个由阶数为 d 的多项式组成的等式，最后都会被化简为另外一个阶数至多为 d 的多项式，这是因为等式中没有能够用来构造更高阶数的乘法。

例如，举例2：5x³ + 7x² – x + 2 = 3x³ – x² + 2x– 5，简化为 2x³ + 8x² – 3x + 7 = 0。 阶数最多就是 3 (次方)

我们可以得到结论：

**任何多项式在任意点的计算结果，都可以看做是其唯一身份的表示。**

如果一个 prover 声称他知道一些 verifier 也知道的多项式（无论多项式的阶数有多大）时，他们就可以按照一个简单的协议去验证：

* verifier 选择一个随机值 x 并在本地计算多项式结果
* verifier 将 x 值丢给 prover，让他计算该多项式的结果
* prover 代入 x 到多项式计算并将结果给到 verifier
* verifier 检查本地的计算结果和 prover 的计算结果是否相等，如果相等那就说明 prover 的陈述具有较高的可信度

与低效的位检查协议相比，新的协议只需要一轮验证就可以让声明具有非常高的可信度（前提是假设 d 远小于 x 取值范围的上限 (是低阶多项式)，可信度几乎是 100%）

#### 2.1.1. 同态加密

它允许加密一个值并在密文上进行算术运算。获取加密的同态性质的方法有多种，我们来介绍一个简单的方法。

总体思路就是我们选择一个基础的（基数需要具有某些特定的属性）的自然数 g（如 5），然后我们以要加密的值为指数对 g 进行求幂。例如，如果我们要对 3 进行加密： 5^3=125，这里 125 就是 3 对应的密文。

模运算：

模运算的思路如下：除了我们所选择的组成有限集合的前 n 个自然数（即，0，1，…，n-1）以外，任何超出此范围的给定整数，我们就将它“缠绕”起来。

那么模运算到底有什么用呢？就是如果我们使用模运算，从运算结果再回到原始值并不容易，因为不同的组合会产生一个同样的运算结果

设想一下 , 如果没有模运算的话，计算结果的大小会给找出原始值提供一些线索。

## 3. 基础电路

[ZK Shanghai 基础电路教学](https://www.youtube.com/watch?v=CTJ1JkYLiyw)

编辑器：[zkREPL](https://zkrepl.dev/)

### 3.1. 视频中的第一个示例：

```c
pragma circom 2.1.4;

template Main () {
    signal input x1;
    signal input x2;
    signal input x3;
    signal input x4;

    signal y1;
    signal y2;
    signal output out;

    // f(x) = (x1+x2)/x3 - x4
    y1 <==  x1 + x2;
    y2 <== y1 / x3;
    out <== y2 - x4;

}

component main = Main();

/* INPUT = {
    "x1": "4",
    "x2": "6",
    "x3": "2",
    "x4": "1"
} */
```

保存报错（提示github访问，可点取消）：

quadratic /kwɒ'drætɪk/ 二次的；二次方程式  
constraints /kən'streint/ 约束条件

```sh
stderr: 
error[T3001]: Non quadratic constraints are not allowed!
   ┌─ "main.circom":15:5
   │
15 │     y2 <== y1 / x3;
   │     ^^^^^^^^^^^^^^ found here
   │
   = call trace:
     ->Main

previous errors were found
stdout: 
Compiled in 4.87s
fail: 
ENOENT: no such file or directory, open 'main_js/main.wasm'
```

改成：

```c
pragma circom 2.1.4;

template Main () {
    signal input x1;
    signal input x2;
    signal input x3;
    signal input x4;

    signal y1;
    signal y2;
    signal output out;

    // f(x) = (x1+x2)/x3 - x4
    y1 <==  x1 + x2;

    // y2 <== y1 / x3;
    // 改成如下
    y2 <-- y1 / x3;
    // === 是约束
    y1 === y2 * x3;

    out <== y2 - x4;
}

component main = Main();

/* INPUT = {
    "x1": "4",
    "x2": "6",
    "x3": "2",
    "x4": "1"
} */
```

### 3.2. 算术电路概念

算术电路：

* 是`计算复杂性理论`中的概念，和电子电路无关
* 有向无环图
* 输入节点：x1, ..., xn
* 内部节点：+, -, *
* 每个内部节点也成为`门(gate)`

算术电路（Arithmetic Circuit）是一种计算模型，用于研究算术运算的复杂性和效率。它是一种抽象的电路模型，用于模拟算术运算的计算过程。

Arithmetic /ə'rɪθmətɪk/ 计算；算术  
Circuit /'sɜːkɪt/ 电路；环行

计算复杂性理论（Computational Complexity Theory）是一门研究计算机算法的效率和限制的理论。它关注的是计算机能够解决的问题的难易程度，以及解决这些问题所需的计算资源（如时间和空间）的数量。

Computational /ˌkɑmpju'teʃənl/  
Complexity /kəm'pleksətɪ/

### 3.3. 零知识证明电路常见设计方法

* 提示
    * 由于算术电路的约束性，每个门电路的计算都会转换为`约束`，进而增加证明和验证的工作量
    * 我们可以将复杂的计算过程转换为预先计算的`提示值`，在电路中`对提示值进行验证`，从而降低证明和验证的工作量。
* 判零函数
    * 示例：`let y = input > 0 ? 0 : 1;`
* 选择
    * 示例：`let y = s ? (a+b) : (a*b);`
* 二进制化
    * 示例：`5 -> 101`
* 比较
    * `let y = s1 > s2 ? 1 : 0;`
    * 转换关系：y = s1 - s2 + 2^n，其中2^n是数的最大值，然后取y二进制化的最高位，即符号位
* 循环
    * 注意：由于算术电路的固定性，电路只能设计成支持最大数量
    * `for(let i = 0; i < N; i++)`，N是某个最大值
* 交换
* 逻辑
    * &、|、!

理解各方法背后对应的数学运算

* 排序：实现冒泡排序
    1) 在算术电路上做排序，可借助`排序网络`的概念
    2) 利用多个比较器形成排序网络进行排序

### 3.4. Circom

可以参考：

* [官网文档](https://docs.circom.io/circom-language/signals/)
* 语法：[circom语言中文教程](https://suyanlong.github.io/circom-language-ch/) （章节内容不全）
* 安装和示例：[Circom 语言教程与 circomlib 演示](https://learnblockchain.cn/article/6811)

Circom 是学习 zk-snarks 的绝佳工具。

Circom是一种新颖的领域特定语言，用于定义可用于生成零知识证明的算术电路。Circom compiler是一个用 Rust 编写的 circom 语言编译器，可用于生成带有一组关联约束的 `R1CS` 文件和一个程序（用 C++ 或 WebAssembly 编写），以有效计算对电路所有线路的有效分配。

* 它的主要特点之一circom是它的模块化，它允许程序员定义称为模板的可参数化电路，可以将其实例化以形成更大的电路。
* 利用小型独立组件构建电路的想法使得测试、审查、审计或正式验证大型复杂circom电路变得更加容易
* Circcom 旨在为开发人员提供一个整体框架，通过易于使用的接口构建算术电路，并抽象化证明机制的复杂性。

`circomLib`是一个公开可用的库，带有数百个电路，例如比较器、哈希函数、数字签名、二进制和十进制转换器等等。

`R1CS（Rank-1 Constraint System）`是一种用于构建零知识证明的数学模型，广泛应用于 zk-SNARKs（零知识简洁非交互式论证系统）。它为证明计算的正确性提供了一个有效的方法，并且有助于实现高效的零知识证明。

#### 3.4.1. 相关语法：

* `信号`：信号可以定义为输入或输出，否则被视为中间信号。

```rust
// 声明了一个带有标识符的输入信号in
signal input in;
// 一个带有标识符的输出信号的N维数组out
signal output out[N];
// 一个带有标识符的中间信号inter
signal inter;
```

* `===`：指定一个约束，对给定的等式创建一个简化形式的约束
    * 示例：`a*(a-1) === 0;` 表示约束a取值0或者1
    * **相当于加了一个`assert`断言**
* `<--`、`-->`：用于给`signal`变量赋值
    * 给`signal`赋值一般需要用`===`添加约束，描述赋了什么值
    * 示例：`out <-- 1 - a*b;`
* `<==` 等价于`<--`后再`===`进行约束判断
    * `out <== 1 - a*b;`，相当于
        * `out <-- 1 - a*b`
        * `out === 1 - a*b`

在构造阶段，变量可以包含使用`乘法`、`加法`和其他变量或信号和字段值构造的算术表达式

约束中只允许包含`二次表达式`。比如 `out === 1 – a*b;`，而 ~~`out <== in*in2*in3;`~~ 是不允许的

开始用circom写程序时，容易误用`<--`，它会分配一个二次表达式**并且不会添加约束**，一般情况需要使用`<===`。

template 和 component：

* template
    * 详情参考[circom docs-templates](https://docs.circom.io/circom-language/templates-and-components/#templates)
    * 在 Circom 创建通用电路的机制是所谓的`模板`(template)
    * 模板实例化为一个新的电路对象，可以用来组合成更大的电路。每个模板有它自己的信号，比如input、outpput
    * 模板里不能包含其他模板或者函数的定义
* component
    * 组件(component)定义了一个算术电路，接收`N`个input信号，并产生`M`个output信号和`K`个中间信号，另外还能产生一系列约束。
    * 可使用`.`访问组件里的input和output信号，其他信号则在组件外不可见
    * 只有所有的input信号都分配了具体值，component才会触发实例化。因此component的创建指令并不意味着执行对象实例化，可能会延迟实例化。
    * ouput信号也需要在所有input都设置值之后才能使用，否则编译器会报错
    * component是不可变的（immutable）
    * 若多个component是独立的，那么各部分可以并发执行，添加`parallel`标签：`template parallel NameTemplate(...){...}`

下面是一个示例说明：

```c
pragma circom 2.0.0;

template Internal() {
   signal input in[2];
   signal output out;
   out <== in[0]*in[1];
}

template Main() {
   signal input in[2];
   signal output out;
   // 定义了一个component组件，可以基于template创建
   component c = Internal ();
   c.in[0] <== in[0];
   // 这里编译器会报错，因为component c里面的input信号还没全部设置具体值
   c.out ==> out;  // c.in[1] is not assigned yet
   c.in[1] <== in[1];  // this line should be placed before calling c.out
}

component main = Main();
```

### 3.5. 基础电路练习

[基础电路练习](https://github.com/wenjin1997/zkshanghai-workshop/blob/main/lecture2-homework.md) 

练习环境：[zkREPL](https://zkrepl.dev/)

注意：  
使用 zkREPL 的输入注释功能测试电路，需要将数字括在引号中，这是由于 JSON 无法解析大整数。  
例如，测试值为-1的输入信号x应如下所示：
`"x": "21888242871839275222246405745257275088548364400416034343698204186575808495616"`

忘记打引号是测试失败的常见原因。

#### 3.5.1. 算术电路：转换为bit位 Num2Bits

```c
pragma circom 2.1.4;

template Num2Bites (nBits) {
    signal input in;
    
    signal output b[nBits];

    // 先计算 in 的二进制表达
    for (var i = 0; i < nBits; i++) {
        // `\` 是整数除法，此处为 in \ (2^i)，然后再 mod 2，判断是取1还是取0
        b[i] <-- (in \ 2 ** i) % 2;
    }

    // 对信号赋值后，检查约束
    // 约束：确保 b 是 in 的二进制表达
    var accm = 0; // 可变变量，累加
    for (var i = 0; i < nBits; i++) {
        accm += b[i] * 2**i;
    }
    accm === in; // 约束

    // 每位也得满足约束
    // 约束：b的每一位是二进制，也就是 0 / 1
    for (var i = 0; i < nBits; i++) {
        0 === b[i] * (1 - b[i]);
    }
}

component main = Num2Bites(5);

/* INPUT = {
    "in": "10"
} */
```

跟着上述代码去查看文档，熟悉语法

[约束](https://docs.circom.io/circom-language/constraint-generation/)

* `===`：指定一个约束，对给定的等式创建一个简化形式的约束
    * 示例：`a*(a-1) === 0;` 表示约束a取值0或者1
    * **相当于加了一个`assert`断言**
* `<--`、`-->`：用于给`signal`变量赋值
    * 给`signal`赋值一般需要用`===`添加约束，描述赋了什么值
    * 示例：`out <-- 1 - a*b;`
* `<==` 等价于`<--`后再`===`进行约束判断
    * `out <== 1 - a*b;`，相当于
        * `out <-- 1 - a*b`
        * `out === 1 - a*b`

在构造阶段，变量可以包含使用`乘法`、`加法`和其他变量或信号和字段值构造的算术表达式

约束中只允许包含`二次表达式`。比如 `out === 1 – a*b;`，而 ~~`out <== in*in2*in3;`~~ 是不允许的

开始用circom写程序时，容易误用`<--`，它会分配一个二次表达式**并且不会添加约束**，一般情况需要使用`<===`。

#### 3.5.2. 算术电路：判零 IsZero

要求：如果in为零，out应为1。 如果in不为零，out应为0。

初步看起来很简单，if else分支处理（**有坑**）

```c
pragma circom 2.0.0;

template IsZero() {
    signal input in;
    signal inv;

    if (in == 0) {
        inv <-- 0;
    } else {
        inv <-- 1 / in;
    }
}

component main = IsZero();

/* INPUT = {
    "in": "10"
} */
```

放到 [zkREPL](https://zkrepl.dev/) 里运行测试（刷新即会重新编译运行），**报错：**

```sh
stderr: 
error[T3001]: Exception caused by invalid assignment: signal already assigned
   ┌─ "main.circom":10:9
   │
10 │         inv <-- 1 / in;
   │         ^^^^^^^^^^^^^^ found here
   │
   = call trace:
     ->IsZero

previous errors were found
```

原因：所有signal都是Unknown的，包括in，编译时会看到inv有两个赋值语句，而**signal只能赋值一次**（无论分支是否只走一次），因此会出现编译错误。

详情可参考circom文档：[signals](https://docs.circom.io/circom-language/signals/#public-and-private-signals)，里面有示例，并且有其他几种限制情况说明。

> Signals are immutable, which means that once they have a value assigned, this value cannot be changed any more. Hence, if a signal is assigned twice, a compilation error is generateds

修改：inv直接用`条件表达式`进行赋值，如下

```c
pragma circom 2.0.0;

template IsZero() {
    signal input in;
    signal output out;

    signal inv;

    inv <-- in!=0 ? 1/in : 0;

    out <== -in*inv +1;
    // 加了一个约束（测试时不加也可通过）
    in*out === 0;
}

component main = IsZero();

// 若这里缺少，测试也会报错
/* INPUT = {
    "in": "10"
} */
```

结果：

```sh
stdout: 
    template instances: 1
    non-linear constraints: 2
    linear constraints: 0
    public inputs: 0
    public outputs: 1
    private inputs: 1
    private outputs: 0
    wires: 4
    labels: 4
    Written successfully: ./main.r1cs
    Written successfully: ./main.sym
    Written successfully: ./main_js/main.wasm
    Everything went okay, circom safe
    Compiled in 1.96s
Output: 
    out = 0
Artifacts: 
    Finished in 2.45s
    main.wasm (34.52KB)
    main.js (9.18KB)
    main.wtns (0.20KB)
    main.r1cs (0.38KB)
    main.sym (0.04KB)
```

#### 3.5.3. 算术电路：相等 IsEqual

要求：如果 in[0] 等于 in[1]，则 out 应为 1。 否则，out 应该是 0。

```c
template IsEqual() {
    signal input in[2];
    signal output out;

    // 通过template创建一个component
    component isz = IsZero();

    in[1] - in[0] ==> isz.in;

    isz.out ==> out;
}
```

`==>` 的作用和用法，具体见：[signals](https://docs.circom.io/circom-language/signals/#types-of-signal-assignments)

* 信号（signal）变量只能由`<--`、`<==`和`-->`、`==>`赋值，这些操作都会被编译器转换成见证生成代码中的赋值操作
    * 双箭头(`==`)和单箭头(`--`)的关键差别是，双箭头会在`R1CS`系统添加一个约束：信号和赋值表达式的值是相等的

在 Circom 语言中，这些符号是用来表示约束（constraints）和连接（wires）的

* `-->`：表示一个信号或变量的输出到另一个信号或变量的输入。它可以用来定义一个 wire 的方向。
* `<--`：类似于 `-->`，但表示信号或变量的输入到输出的方向，往往用在更复杂的信号流动中。

#### 3.5.4. 选择器 Selector

要求：输出out应该等于in[index]。 如果 index 越界（不在 [0, nChoices) 中），out 应该是 0。

比较复杂：思路是循环遍历下标，用i和index是否相等作选择器，如果i和index相等，选择器置1，不等置0。

```c
// 求和
template CalculateTotal(n) {
    signal input in[n];
    signal output out;

    signal sums[n];

    sums[0] <== in[0];

    for (var i = 1; i < n; i++) {
        sums[i] <== sums[i-1] + in[i]
    }

    out <== sums[n-1];
}

// 实现的功能即 根据位数组各位0/1进行过滤，并求和
template QuinSelector(choices) {
    signal input in[choices];
    signal input index;
    signal output out;
    
    // Ensure that index < choices
    // 校验小于关系
    component lessThan = LessThan(4);
    lessThan.in[0] <== index;
    // 两个input变量都设置后，component才真正完成了实例化
    lessThan.in[1] <== choices;
    lessThan.out === 1;

    // 求和组件
    component calcTotal = CalculateTotal(choices);
    component eqs[choices];

    // For each item, check whether its index equals the input index.
    for (var i = 0; i < choices; i ++) {
        eqs[i] = IsEqual();
        eqs[i].in[0] <== i;
        // 这里也校验约束
        eqs[i].in[1] <== index;

        // eqs[i].out is 1 if the index matches. As such, at most one input to
        // calcTotal is not 0.
        // 初始化上面 calcTotal 组件里的input变量（是个数组）
        calcTotal.in[i] <== eqs[i].out * in[i];
    }

    // Returns 0 + 0 + 0 + item
    out <== calcTotal.out;
}
```

#### 3.5.5. 判负 IsNegative

要求：如果根据我们的约定，in 为负数，则 out 应为 1。 否则，out 应该是 0。

示例有点复杂：
[lecture2/IsNegative.circom](https://github.com/wenjin1997/zkshanghai-workshop/blob/main/lecture2/IsNegative.circom)

```c
pragma circom 2.1.4;

// 整数转换成二进制形式，并输出一个二进制位数组
template Num2Bits(n) {
    signal input in;
    signal output out[n];
    var lc1=0;

    var e2=1;
    for (var i = 0; i<n; i++) {
        // 将输入 in 的第 i 位提取出来
        out[i] <-- (in >> i) & 1;
        // 约束：确保 out[i] 只能是 0 或 1
        out[i] * (out[i] -1 ) === 0;
        lc1 += out[i] * e2;
        // *2，作为二进制每位的值
        e2 = e2+e2;
    }

    // 确保转换后的二进制数与原始输入相等
    lc1 === in;
}


// Returns 1 if in (in binary) > ct
// 比较输入 in 和常数 ct 的大小，它计算出 in 是否大于 ct
// CompConstant 模板通过逐位比较的方式，确定输入 in 是否大于常数 ct。每个位的比较结果都会根据 ct 的值进行不同的计算，然后累加起来形成最终的结果。最终的布尔值由 sout 的最高位决定，如果 sout 大于等于 0，则输出 1 表示 in 大于 ct；否则输出 0 表示 in 不大于 ct。
template CompConstant(ct) {
    // 输入是一个包含 254 个元素的二进制位数组，表示要比较的数字。
    signal input in[254];
    // 返回一个布尔值，表示输入是否大于常数 ct
    signal output out;

    // 用于存储每个位比较的结果
    signal parts[127];
    // 最终的累加比较结果
    signal sout;

    // 分别表示常数 ct 的低位和高位以及输入 in 的低位和高位
    var clsb;
    var cmsb;
    var slsb;
    var smsb;

    var sum=0;

    // 初始化一个大的负数，用于后续的比较
    var b = (1 << 128) -1;
    // 用于计算正数的比较
    var a = 1;
    // 用于更新 b 和 a 的值
    var e = 1;
    var i;

    for (i=0;i<127; i++) {
        clsb = (ct >> (i*2)) & 1;
        cmsb = (ct >> (i*2+1)) & 1;
        slsb = in[i*2];
        smsb = in[i*2+1];

        if ((cmsb==0)&&(clsb==0)) {
            parts[i] <== -b*smsb*slsb + b*smsb + b*slsb;
        } else if ((cmsb==0)&&(clsb==1)) {
            parts[i] <== a*smsb*slsb - a*slsb + b*smsb - a*smsb + a;
        } else if ((cmsb==1)&&(clsb==0)) {
            parts[i] <== b*smsb*slsb - a*smsb + a;
        } else {
            parts[i] <== -a*smsb*slsb + a;
        }

        sum = sum + parts[i];

        b = b -e;
        a = a +e;
        e = e*2;
    }

    sout <== sum;

    // 调用 Num2Bits 模板将比较结果转换成二进制形式
    component num2bits = Num2Bits(135);

    num2bits.in <== sout;

    // 输出最左边的位作为结果
    out <== num2bits.out[127];
}


template IsNegative() {
    signal input in;
    signal output out;

    // 这个值在零知识证明和相关的密码学应用中非常重要，因为它是一个大素数，通常被用作椭圆曲线加密（ECC）或其它密码学算法中的模数。
    // 在 Circom 中，这个数字经常出现，因为它与用于构建零知识证明的安全参数有关。
    // 在 Circom 的文档中，这个数值通常被称为 p，并且它代表了一个特定的素数，所有的信号计算都在模 p 的有限域中进行。
    var p = 21888242871839275222246405745257275088548364400416034343698204186575808495617;
    // (p - 1) \ 2 = 10944121435919637611123202872628637544274182200208017171849102093287904247808
    component num2bits = Num2Bits(254);
    component compconstant = CompConstant((p - 1) \ 2);
    num2bits.in <== in;
    compconstant.in <== num2bits.out; 
    out <== compconstant.out;

}

component main = IsNegative(); 

/* INPUT = {
    "in": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
} */
```

#### 少于 LessThan

```c
pragma circom 2.1.4;

// 将 in 转换为 n 位二进制表达
template Num2Bits(n) {
    signal input in;
    signal output out[n];

    for (var i = 0; i < n; i++) {
        out[i] <-- (in \ 2**i) % 2;
    }

    var accm = 0;
    for (var i = 0; i < n; i++) {
        accm += out[i] * 2**i;
    }
    accm === in;

    // 约束输出的每一位是 0 / 1
    for (var i = 0; i < n; i++) {
        out[i] * (1 - out[i]) === 0;
    }
}


// in[0] < in[1] 返回 1，否则返回0
template LessThan (n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);
    n2b.in <== in[1] - in[0] + (2**n); // n+1位二进制，最大数是 2**n
    out <== n2b.out[n];
}

component main = LessThan(3);

/* INPUT = {
    "in": ["3", "4"]
} */
```

