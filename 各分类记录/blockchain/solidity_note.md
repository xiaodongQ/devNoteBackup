# solidity 学习笔记

开发工具：remix
	Remix 是一个基于 Web 浏览器的 IDE，它可以让你编写 Solidity 智能合约，然后部署并运行该智能合约

[WTF Academy](https://www.wtf.academy/)

[solidity中文文档](https://learnblockchain.cn/docs/solidity/index.html)
	[learnblockchain](https://learnblockchain.cn/)该网站上还有其他区块链相关的中文文档

# solidity-101

## Solidity中的变量类型

[数值类型](https://www.wtf.academy/docs/solidity-101/ValueTypes)

数值类型(Value Type)：包括布尔型，整数型等等，这类变量赋值时候直接传递数值。

引用类型(Reference Type)：包括数组和结构体，这类变量占空间大，赋值时候直接传递地址（类似指针）。

映射类型(Mapping Type): Solidity里的哈希表。

函数类型(Function Type)：Solidity文档里把函数归到数值类型，但我觉得他跟其他类型差别很大，所以单独分一类。
	`function <function name>(<parameter types>) {internal|external|public|private} [pure|view|payable] [returns (<return types>)]`
		{internal|external|public|private}：函数可见性说明符。
			public：内部外部均可见。没有标明可见性类型的函数，默认为public
			private：只能从本合约内部访问，继承的合约也不能用
			external：只能从合约外部访问
			internal：只能从合约内部访问，继承的合约可以用
		注意:
			public|private|internal 也可用于修饰状态变量。 public变量会自动生成同名的`getter`函数，用于查询数值(remix点击变量名即可getter查看)。
			没有标明可见性类型的`状态变量`，默认为`internal`
		[pure|view|payable]
			pure跟view关键字的函数是不改写链上状态的，因此用户直接调用他们是不需要付gas的
			pure：不能读取也不能写入存储在链上的状态变量
			view：能读取但也不能写入状态变量
			**没注明，默认为payable，可读可写**

地址类型(address)存储一个 20 字节的值（以太坊地址的大小）
	地址类型也有成员变量，并作为所有合约的基础。有*普通的地址*和*可以转账ETH的地址（payable）*。
		其中，`payable`修饰的地址相对普通地址多了`transfer`和`send`两个成员。
		在`payable`修饰的地址中，send执行失败不会影响当前合约的执行（但是返回false值，需要开发人员检查send返回值）。
		`balance`和`transfer()`，可以用来查询ETH余额以及安全转账（内置执行失败的处理）

## 引用类型

包括数组（array），结构体（struct）和映射（mapping），这类变量占空间大，赋值时候直接传递地址（类似指针）

### 数据位置

solidity数据存储位置有三类：`storage`，`memory`和`calldata`。不同存储位置的gas成本不同。

storage类型的数据存在链上，类似计算机的硬盘，消耗gas多；memory和calldata类型的临时存在内存里，消耗gas少

storage上链；memory和calldata在内存中不上链；calldata相对于memory，不可更改(immutable)

### 变量的作用域

`状态变量`是数据存储在链上的变量。状态变量在合约内、函数外声明
`局部变量`的数据存储在内存里，不上链，gas低。局部变量在函数内声明
`全局变量`是全局范围工作的变量，都是solidity预留关键字。可以在函数内不声明直接使用
	msg.sender, block.number和msg.data，他们分别代表请求发起地址，当前区块高度，和请求数据

### 数组

定长 uint[10] arr1;
不定长 uint[] arr2;
	push(xxx)、pop() 成员函数
	.length成员

用方括号包着来初始化array的一种方式，并且里面每一个元素的type是以第一个元素为准的
	e.g. [uint(1),2,3]里面的元素都是uint类型，因为第一个元素指定了是uint类型了
创建数组：uint[] memory array8 = new uint[](5);

### 映射Mapping

声明映射的格式为`mapping(_KeyType => _ValueType)`
e.g.
    mapping(uint => address) public idToAddress; // id映射到地址
    mapping(address => address) public swapPair; // 币对的映射，地址到地址

* 映射规则

规则1：映射的_KeyType只能选择solidity默认的类型
规则2：映射的存储位置必须是storage
	不能用于public函数的参数或返回结果中，因为mapping记录的是一种关系 (key - value pair)
规则3：如果映射声明为public，那么solidity会自动给你创建一个getter函数，可以通过Key来查询对应的Value
规则4：给映射新增的键值对的语法为`_Var[_Key] = _Value`，其中_Var是映射变量名，`_Key`和`_Value`对应新增的键值对

## delete操作符

delete 会让变量的值变为初始值
	e.g. `delete _bool2;`

## constant和immutable

只有`数值变量`可以声明`constant`和`immutable`；
`string`和`bytes`可以声明为`constant`，但不能为`immutable`

让不应该变的变量保持不变。这样的做法能在节省gas的同时提升合约的安全性


## 构造函数 和 修饰器

构造函数（constructor）是一种特殊的函数，每个合约可以定义一个，并在**部署合约的时候**自动运行一次。
	注意：在Solidity 0.4.22之前，构造函数不使用 constructor 而是使用与合约名同名的函数作为构造函数而使用，由于这种旧写法容易使开发者在书写时发生疏漏（例如合约名叫 Parents，构造函数名写成 parents），使得构造函数变成普通函数，引发漏洞，所以0.4.22版本及之后，采用了全新的 constructor 写法。
	`constructor(){xxx}`

修饰器（modifier）是solidity特有的语法，类似于面向对象编程中的decorator，声明函数拥有的特性，并减少代码冗余。
	modifier的主要使用场景是**运行函数前的检查**，例如地址，变量，余额等。
	`modifier onlyOwer{xxx}`，`function changeOwer(address newAddr) external onlyOwer{}` 


## 事件（event）

Solidity中的事件（event）是EVM上日志的抽象。
两个特点：
	响应：应用程序（ethers.js）可以通过RPC接口订阅和监听这些事件，并在前端做响应
	经济：事件是EVM上比较经济的存储数据的方式，每个大概消耗2,000 gas；相比之下，链上存储一个新变量至少需要20,000 gas

* 声明事件：由event关键字开头，接着是事件名称，括号里面写好事件需要记录的变量类型和变量名
	e.g. 以ERC20代币合约的Transfer事件为例，`event Transfer(address indexed from, address indexed to, uint256 value);`
		from,to,value分别对应代币的转账地址，接收地址和转账数量
		其中from和to前面带有`indexed`关键字，他们会保存在以太坊虚拟机日志的topics中，方便之后检索

* 释放事件：可以在函数里释放事件
	e.g. `emit Transfer(from, to, amount);`

### EVM日志

以太坊虚拟机（EVM）用日志Log来存储Solidity事件，每条日志记录都包含主题topics和数据data两部分

* 主题(Topics)

日志的第一部分是**主题数组**，用于描述事件，长度不能超过`4`。
	它的第一个元素是事件的签名（哈希），如上述Transfer的签名：`keccak256("Transfer(addrses,address,uint256)")`
	以及至多3个indexed参数
	indexed标记的参数可以理解为检索事件的索引“键”，方便之后搜索。每个indexed参数固定256bit，参数太大则自动计算哈希

* 数据（Data）

事件中不带indexed的参数会被存储在 data 部分中，可以理解为事件的“值”。

data 部分的变量不能被直接检索，但可以存储任意大小的数据。

*很多链上分析工具包括Nansen和Dune Analysis都是基于事件工作的。*

## 继承

如果把合约看作是对象的话，solidity也是面向对象的编程，也支持继承。

virtual: 父合约中的函数，如果希望子合约重写，需要加上virtual关键字。
override：子合约重写了父合约中的函数，需要加上override关键字。

* 简单继承

```c
contract Animal {
	event Log(string msg);
	function test() public virtual{
		emit Log("Animal");
	}
}

contract Cat is Animal{
	function test() public virtual override{
		emit Log("Cat");
	}
}
```

* 多重继承

solidity的合约可以继承多个合约。

1) 继承时要按辈分最高到最低的顺序排。
	如 `contract WhiteCat is Animal, Cat{ xxx }`，而不能 `is Cat, Animal`，不然就会报错
2) 如果某一个函数在多个继承的合约里都存在，在子合约里必须重写，不然会报错。
3) 重写在多个父合约中都重名的函数时，override关键字后面要加上所有父合约名字
	如 `function test() override(Animal, Cat){ xxx }`

* Solidity中的修饰器（Modifier）同样可以继承，用法与函数继承类似，在相应的地方加virtual和override关键字即可
	可以直接在代码中使用父合约中的修饰器，也可以利用override关键字重写修饰器

```c
contract Base1 {
    modifier exactDividedBy2And3(uint _a) virtual {
        require(_a % 2 == 0 && _a % 3 == 0);
        _;
    }
}

contract Identifier is Base1 {
	modifier exactDividedBy2And3(uint _a) override {
	    _;
	    require(_a % 2 == 0 && _a % 3 == 0);
	}
}
```

* 构造函数的继承

```c
abstract contract A {
    uint public a;

    constructor(uint _a) {
        a = _a;
    }
}

contract C is A {
	// 注意调用父合约在函数体外面
    constructor(uint _c) A(_c * _c) {}
}
```

* 调用父合约的函数

1）直接调用：子合约可以直接用父合约名.函数名()的方式来调用父合约函数

```c
contract Cat is Animal{
	function callParent() public{
	    Anamal.test();
	}
}
```

2）`super`关键字：子合约可以利用`super.函数名()`来调用最近的父合约函数。

solidity继承关系按声明时从右到左的顺序是：`contract WhiteCat is Animal, Cat`，那么`Cat`是最近的父合约，`super.test()`将调用`Cat.test()`而不是`Animal.test()`：

* 钻石继承

```
/* 继承树：
  God
 /  \
Adam Eve
 \  /
people
*/

调用合约people中的super.bar()会依次调用Eve、Adam，最后是God合约
```

## 抽象合约 和 接口

* 抽象合约

如果一个智能合约里至少有一个未实现的函数，即某个函数缺少主体{}中的内容，则必须将该合约标为`abstract`，不然编译会报错；
另外，未实现的函数需要加`virtual`，以便子合约重写。

e.g. 插入排序合约，暂时没想好怎么实现，则可以先标记

```c
abstract contract InsertionSort{
    function insertionSort(uint[] memory a) public pure virtual returns(uint[] memory);
}
```

* 接口

接口类似于抽象合约，但它不实现任何功能。
接口的规则：
1. 不能包含状态变量
2. 不能包含构造函数
3. 不能继承除接口外的其他合约
4. 所有函数都必须是external且不能有函数体
5. 继承接口的合约必须实现接口定义的所有功能

**接口是智能合约的骨架，定义了合约的功能以及如何触发它们**，因为接口提供了两个重要的信息：
	如果智能合约实现了某种接口（比如ERC20或ERC721），其他Dapps和智能合约就知道如何与它交互。
	合约里每个函数的bytes4选择器，以及函数签名 和 接口id

接口与合约`ABI（Application Binary Interface）`等价，可以相互转换：
	编译接口可以得到合约的ABI，利用abi-to-sol工具也可以将ABI json文件转换为接口sol文件

以ERC721接口合约`IERC721`为例，它定义了3个event和9个function，所有ERC721标准的NFT都实现了这些函数。

```rust
interface IERC721 is IERC165 {
	// 在转账时被释放，记录代币的发出地址from，接收地址to和tokenid
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
    // 在授权时释放，记录授权地址owner，被授权地址approved和tokenid
    event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId);
    // 在批量授权时释放，记录批量授权的发出地址owner，被授权地址operator和授权与否的approved
    event ApprovalForAll(address indexed owner, address indexed operator, bool approved);
    
    // 返回某地址的NFT持有量balance
    function balanceOf(address owner) external view returns (uint256 balance);
    // 返回某tokenId的主人owner
    function ownerOf(uint256 tokenId) external view returns (address owner);
    // 普通转账，参数为转出地址from，接收地址to和tokenId
    function transferFrom(address from, address to, uint256 tokenId) external;
    // 安全转账（如果接收方是合约地址，会要求实现ERC721Receiver接口）。参数为转出地址from，接收地址to和tokenId
    function safeTransferFrom(address from, address to, uint256 tokenId) external;
    // 授权另一个地址使用你的NFT。参数为被授权地址to和tokenId
    function approve(address to, uint256 tokenId) external;
    // 查询tokenId被批准给了哪个地址
    function getApproved(uint256 tokenId) external view returns (address operator);
    // 将自己持有的该系列NFT批量授权给某个地址operator
    function setApprovalForAll(address operator, bool _approved) external;
    // 查询某地址的NFT是否批量授权给了另一个operator地址
    function isApprovedForAll(address owner, address operator) external view returns (bool);
    // 安全转账的重载函数，参数里面包含了data
    function safeTransferFrom( address from, address to, uint256 tokenId, bytes calldata data) external;
}
```

* 什么时候使用接口

无聊猿BAYC属于ERC721代币，实现了IERC721接口的功能。我们不需要知道它的源代码，只需知道它的合约地址，用IERC721接口就可以与它交互

## 异常

solidity三种抛出异常的方法：`error`，`require`和`assert`，三种方法的gas消耗不同。
	error方法gas最少，其次是assert，require方法消耗gas最多

* error

`error`是`solidity 0.8.4`版本新加的内容，方便且高效（省gas）地向用户解释操作失败的原因，同时还可以在抛出异常的同时携带参数，帮助开发者更好地调试。

在执行当中，error必须搭配`revert`（回退）命令使用

```c
// 可以在contract之外定义异常
error TransferNotOwner(); // 自定义error
error TransferNotOwner(address sender); // 自定义的带参数的error

 function transferOwner1(uint256 tokenId, address newOwner) public {
    if(_owners[tokenId] != msg.sender){
        revert TransferNotOwner();
        // revert TransferNotOwner(msg.sender);
    }
    _owners[tokenId] = newOwner;
}
```

* require

`require`命令是solidity 0.8版本之前抛出异常的常用方法，目前很多主流合约仍然还在使用它。
	它很好用，唯一的缺点就是gas随着描述异常的字符串长度增加，比error命令要高
使用方法：`require(检查条件，"异常的描述")`，当检查条件不成立的时候，就会抛出异常。

```c
function transferOwner2(uint256 tokenId, address newOwner) public {
    require(_owners[tokenId] == msg.sender, "Transfer Not Owner");
    _owners[tokenId] = newOwner;
}
```

* assert

用法：`assert(检查条件）`，当检查条件不成立的时候，就会抛出异常

```c
function transferOwner3(uint256 tokenId, address newOwner) public {
    assert(_owners[tokenId] == msg.sender);
    _owners[tokenId] = newOwner;
}
```

---

# solidity-102

## 函数重载

solidity中允许函数进行重载（overloading）
注意：solidity不允许修饰器（modifier）重载


## 库函数

库函数是一种特殊的合约，为了提升solidity代码的复用性和减少gas而存在。

和普通合约主要有以下几点不同：
	* 不能存在状态变量
	* 不能够继承或被继承
	* 不能接收以太币
	* 不可以被销毁

```c
library Strings {
    bytes16 private constant _HEX_SYMBOLS = "0123456789abcdef";
    function toString(uint256 value) public pure returns (string memory) {
    	xxx
    }

    function toHexString(uint256 value, uint256 length) public pure returns (string memory) {
    	xxx
    }
}
```

### 如何使用库合约

两种方式：
1. 利用`using for`指令
	可用于附加库函数（从库 A）到任何类型（B）。添加完指令后，库A中的函数会自动添加为B类型变量的成员，可以直接调用。

```c
	// 利用using for指令
    using Strings for uint256;
    function getString1(uint256 _number) public pure returns(string memory){
        // 库函数会自动添加为uint256型变量的成员
        return _number.toHexString();
    }
```

2. 通过库合约名称调用库函数

```c
	// 直接通过库合约名调用
    function getString2(uint256 _number) public pure returns(string memory){
        return Strings.toHexString(_number);
    }
```

### 常用的库合约有：

String：将uint256转换为String
Address：判断某个地址是否为合约地址
Create2：更安全的使用Create2 EVM opcode
Arrays：跟数组相关的库函数

## import

solidity支持利用`import`关键字导入其他源代码中的合约，让开发更加模块化

import用法：
1. 通过源文件相对位置导入
	`import './Yeye.sol';`
2. 通过源文件网址导入网上的合约
	`import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol';`
3. 通过npm的目录导入
	`import '@openzeppelin/contracts/access/Ownable.sol';`
4. 通过全局符号导入特定的合约
	`import {Yeye} from './Yeye.sol';`

## 接收ETH receive和fallback

Solidity支持两种特殊的回调函数，`receive()`和`fallback()`

它们主要在两种情况下被使用：
1. 接收ETH
2. 处理合约中不存在的函数调用（代理合约proxy contract）

### 接收ETH函数`receive()`

receive()只用于处理接收ETH。一个合约最多有一个receive()函数
声明方式与一般函数不一样，不需要function关键字：`receive() external payable { ... }`。
receive()函数不能有任何的参数，不能返回任何值，必须包含`external`和`payable`

**当合约接收ETH的时候，receive()会被触发。**

receive()最好不要执行太多的逻辑，因为如果别人用`send`和`transfer`方法发送ETH的话，gas会限制在2300，receive()太复杂可能会触发Out of Gas报错；
如果用`call`就可以自定义gas执行更复杂的逻辑

有些恶意合约，会在receive() 函数（老版本的话，就是 fallback() 函数）*嵌入恶意消耗gas的内容或者使得执行故意失败的代码，导致一些包含退款和转账逻辑的合约不能正常工作，因此写包含退款等逻辑的合约时候，一定要注意这种情况*。

### 回退函数 `fallback`

fallback()函数会在调用合约不存在的函数时被触发。可用于接收ETH，也可以用于代理合约proxy contract。
fallback()声明时不需要function关键字，必须由external修饰，一般也会用payable修饰，用于接收ETH:`fallback() external payable { ... }`。


### 区别

receive和fallback都能够用于接收ETH，他们触发的规则如下

触发fallback() 还是 receive()?
           接收ETH
              |
         msg.data是空？
            /  \
          是    否
          /      \
receive()存在?   fallback()
        / \
       是  否
      /     \
receive()   fallback()

简单来说，合约接收ETH时，msg.data为空且存在receive()时，会触发receive()；msg.data不为空或不存在receive()时，会触发fallback()，此时fallback()必须为payable。

验证：
remix里，Low level interactions的CALLDATA里，1) 不填数据触发Transact 2) 填数据触发Transact

```c
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract ReceiveFallback{
    // 定义事件
    event Received(address Sender, uint Value);
    // 接收ETH时释放Received事件
    receive() external payable {
        emit Received(msg.sender, msg.value);
    }

    event fallbackCalled(address Sender, uint Value, bytes);
    fallback() external payable {
        emit fallbackCalled(msg.sender, msg.value, msg.data);
    }
}
```

calldata为空，触发后可看到logs里触发了Received事件：

```
[
	{
		"from": "0xb27A31f1b0AF2946B7F582768f03239b1eC07c2c",
		"topic": "0x88a5966d370b9919b20f3e2c13ff65706f196a4e32cc2c12bf57088f88525874",
		"event": "Received",
		"args": {
			"0": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4",
			"1": "0",
			"Sender": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4",
			"Value": "0"
		}
	}
]
```

calldata不为空，如填一个0x的数据0xb27A31f1b0AF2946B7F582768f03239b1eC07c2c，触发后可看到logs里触发了fallbackCalled事件：

```
[
	{
		"from": "0xb27A31f1b0AF2946B7F582768f03239b1eC07c2c",
		"topic": "0x7bf8121d238f1338d4842c396807cbe93f5a2396e2b3cad1639735997867b1f5",
		"event": "fallbackCalled",
		"args": {
			"0": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4",
			"1": "0",
			"2": "0xb27a31f1b0af2946b7f582768f03239b1ec07c2c",
			"Sender": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4",
			"Value": "0"
		}
	}
]
```

## 发送ETH

Solidity有三种方法向其他合约发送ETH，他们是：`transfer()`，`send()`和`call()`，其中`call()`是被鼓励的用法。

在发送ETH合约SendETH中实现`payable`的`构造函数`和`receive()`，让我们能够在部署时和部署后向合约转账

* `transfer`
	用法是`接收方地址.transfer(发送ETH数额)`
	测试部署时，选择对应的value余额进行deploy；(支持转账的合约，需要构造时指定构造为payable?)
* `send`
	用法是`接收方地址.send(发送ETH数额)`
* `call`
	用法是`接收方地址.call{value: 发送ETH数额}("")`
	call()的返回值是(bool, data)，其中bool代表着转账成功或失败
	e.g. `(bool success,) = _to.call{value: amountxxx}("");`

call没有gas限制，最为灵活，是最提倡的方法；
transfer有`2300` gas限制，但是发送失败会自动revert交易，是次优选择；
send有2300 gas限制，而且发送失败不会自动revert交易，几乎没有人用它。

示例：SendETH.sol

```c
// SPDX-License-Identifier:MIT
pragma solidity ^0.8.4;

contract SendETH{
    // 需要该构造才能部署payable函数
    constructor()payable{}
    
    receive() external payable {}

    function transferETH(address payable _to, uint256 amount)external {
        _to.transfer(amount);
    }

    // send()发送ETH
    error SendFailed();
    function sendETH(address payable _to, uint256 amount) external payable{
        // 处理下send的返回值，如果失败，revert交易并发送error
        bool success = _to.send(amount);
        if(!success){
            revert SendFailed();
        }
    }

    error CallFailed();
    // call()发送ETH
    function callETH(address payable _to, uint256 amount) external payable{
        // 处理下call的返回值，如果失败，revert交易并发送error
        (bool success,) = _to.call{value: amount}("");
        if(!success){
            revert CallFailed();
        }
    }
}
```

## 调用其他合约

调用已部署的合约

1. 传入合约地址
	可以利用合约的地址和合约代码（或接口）来创建合约的引用：`_Name(_Address)`，其中`_Name`是合约名，`_Address`是合约地址。然后

```c
// 其中OtherContract为已部署的合约。调用时传入已部署合约的地址给_Address
function callSetX(address _Address, uint256 x) external{
    OtherContract(_Address).setX(x);
}
```

2. 传入合约变量
	参数的类型改为目标合约名

```c
// 调用时传入已部署合约的地址给_Address
function callGetX(OtherContract _Address) external view returns(uint x){
    x = _Address.getX();
}
```

3. 创建合约变量
	创建合约变量，然后通过它来调用目标函数

```c
	OtherContract oc = OtherContract(_Address);
	oc.getX();
```

4. 调用合约并发送ETH
	如果目标合约的函数是payable的，那么我们可以通过调用它来给合约转账：
	`_Name(_Address).f{value: _Value}()` (`_Value`是要转的ETH数额)

```c
function setXTransferETH(address otherContract, uint256 x) payable external{
    OtherContract(otherContract).setX{value: msg.value}(x);
}
```

## call

前面也提到利用call来发送ETH

call是solidity官方**推荐**的通过触发fallback或receive函数发送ETH的方法。
**不推荐用call来调用另一个合约**，因为当你调用不安全合约的函数时，你就把主动权交给了它。推荐的方法仍是声明合约变量后调用函数。
当我们不知道对方合约的源代码或ABI，就没法生成合约变量；这时，我们仍可以通过call调用对方合约的函数。

* call的使用规则：`目标合约地址.call(二进制编码);`
	其中二进制编码利用结构化编码函数`abi.encodeWithSignature`获得：`abi.encodeWithSignature("函数签名", 逗号分隔的具体参数)`
	如：`abi.encodeWithSignature("f(uint256,address)", _x, _addr)`

* 另外call在调用合约时可以指定交易发送的ETH数额和gas
	`目标合约地址.call{value:发送数额, gas:gas数额}(二进制编码);`

```c
// 定义Response事件，输出call返回的结果success和data
event Response(bool success, bytes data);

function callSetX(address payable _addr, uint256 x) public payable {
    // call setX()，同时可以发送ETH
    (bool success, bytes memory data) = _addr.call{value: msg.value}(
    	// 函数签名里面必须指定具体位数?uint256？
        abi.encodeWithSignature("setX(uint256)", x)
    );

    emit Response(success, data); //释放事件
}
```

* 如果我们给call输入的函数不存在于目标合约，那么目标合约的fallback函数会被触发

## delegateCall

delegatecall与call类似，是solidity中地址类型的低级成员函数

调用结构：你（A）通过合约B调用目标合约C。
	合约B必须和目标合约C的变量存储布局必须相同，并且顺序相同

delegatecall语法和call类似，也是：
	`目标合约地址.delegatecall(二进制编码)`

*不同点*
	在于运行的语境，B call C，语境为C(即get和set的是C)；而B delegatecall C，语境为B(即get和set的还是B)。
	和call不一样，delegatecall在调用合约时可以指定交易发送的gas，但不能指定发送的ETH数额

注意：delegatecall有安全隐患，使用时要保证当前合约和目标合约的状态变量存储结构相同，并且目标合约安全，不然会造成资产损失。

目前delegatecall主要有两个应用场景：
1. 代理合约（Proxy Contract）：将智能合约的存储合约和逻辑合约分开
	代理合约（Proxy Contract）存储所有相关的变量，并且保存逻辑合约的地址；
	所有函数存在逻辑合约（Logic Contract）里，通过delegatecall执行。当升级时，只需要将代理合约指向新的逻辑合约即可。
	例如：A通过合约B调用合约C，B是代理合约，C是逻辑合约
2. EIP-2535 Diamonds（钻石）：钻石是一个支持构建可在生产中扩展的模块化智能合约系统的标准。钻石是具有多个实施合同的代理合同。

## 在合约中创建新合约：create操作码 (关键字还是new)

在以太坊链上，用户（外部账户，EOA）可以创建智能合约，智能合约同样也可以创建新的智能合约。
有两种方法可以在合约中创建新合约，create和create2，这里我们讲create，下一讲会介绍create2。

去中心化交易所uniswap就是利用工厂合约（Factory）创建了无数个币对合约（Pair）

create的用法很简单，就是new一个合约，并传入新合约构造函数所需的参数：
	`Contract x = new Contract{value: _value}(params)`
	其中Contract是要创建的合约名，x是合约对象（地址），如果构造函数是payable，可以创建时转入_value数量的ETH，params是新合约构造函数的参数。

### 极简Uniswap

Uniswap V2核心合约中包含两个合约：
	UniswapV2Pair: 币对合约，用于管理币对地址、流动性、买卖。
	UniswapV2Factory: 工厂合约，用于创建新的币对，并管理币对地址。

```c
// 币对合约负责管理币对地址
contract Pair{
    address public factory; // 工厂合约地址
    address public token0; // 代币1
    address public token1; // 代币2

    constructor() payable {
        factory = msg.sender;
    }

    // called once by the factory at time of deployment
    function initialize(address _token0, address _token1) external {
        require(msg.sender == factory, 'UniswapV2: FORBIDDEN'); // sufficient check
        token0 = _token0;
        token1 = _token1;
    }
}

// PairFactory工厂合约用于创建新的币对，并管理币对地址
contract PairFactory{
    mapping(address => mapping(address => address)) public getPair; // 通过两个代币地址查Pair地址
    address[] public allPairs; // 保存所有Pair地址

    // remix测试：
    // 	WBNB地址: 0x2c44b726ADF1963cA47Af88B284C06f30380fC78
	// 	BSC链上的PEOPLE地址:0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c
    function createPair(address tokenA, address tokenB) external returns (address pairAddr) {
        // 创建新合约
        Pair pair = new Pair(); 
        // 调用新合约的initialize方法
        pair.initialize(tokenA, tokenB);
        // 更新地址map
        pairAddr = address(pair);
        allPairs.push(pairAddr);
        getPair[tokenA][tokenB] = pairAddr;
        getPair[tokenB][tokenA] = pairAddr;
    }
}
```

## 在合约中创建新合约：CREATE2

`CREATE2` 操作码使我们在智能合约部署在以太坊网络之前就能预测合约的地址。
Uniswap创建Pair合约用的就是`CREATE2`，而不是`CREATE`

* `CREATE`如何计算地址
	智能合约由其他合约和普通账户利用CREATE操作码创建
	`新地址 = hash(创建者地址, nonce)`
	即：创建者的地址(通常为部署的钱包地址或者合约地址) 和 nonce(该地址发送交易的总数,对于合约账户是创建的合约总数,每创建一个合约nonce+1))的哈希。
	创建者地址不会变，*但nonce可能会随时间而改变，因此用CREATE创建的合约地址不好预测*。
* `CREATE2`如何计算地址
	`CREATE2`的目的是为了让合约地址独立于未来的事件。不管未来区块链上发生了什么，你都可以把合约部署在事先计算好的地址上。
	`新地址 = hash("0xFF",创建者地址, salt, bytecode)`
	用CREATE2创建的合约地址由4个部分决定：
		0xFF：一个常数，避免和CREATE冲突
		创建者地址
		salt（盐）：一个创建者给定的数值
		待部署合约的字节码（bytecode）
	CREATE2 确保，如果创建者使用 CREATE2 和提供的 salt 部署给定的合约bytecode，它将存储在 新地址 中

* create2的实际应用场景
	1. 交易所为新用户预留创建钱包合约地址
	2. 由 CREATE2 驱动的 factory 合约，在uniswapV2中交易对的创建是在 Factory中调用create2完成。
		这样做的好处是: 它可以得到一个确定的pair地址, 使得 Router中就可以通过 (tokenA, tokenB) 计算出pair地址, 不再需要执行一次 Factory.getPair(tokenA, tokenB) 的跨合约调用

	CREATE2让我们可以在部署合约前确定它的合约地址，这也是 一些layer2项目的基础。

CREATE2的用法和之前讲的Create类似，同样是new一个合约，并传入新合约构造函数所需的参数，**只不过要多传一个salt参数**：
	`Contract x = new Contract{salt: _salt, value: _value}(params)`

```c
contract PairFactory2{
    mapping(address => mapping(address => address)) public getPair; // 通过两个代币地址查Pair地址
    address[] public allPairs; // 保存所有Pair地址

    function createPair2(address tokenA, address tokenB) external returns (address pairAddr) {
        require(tokenA != tokenB, 'IDENTICAL_ADDRESSES'); //避免tokenA和tokenB相同产生的冲突
        // 计算用tokenA和tokenB地址计算salt
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA); //将tokenA和tokenB按大小排序
        // Keccak算法是一种安全性高、性能优良、可扩展性好的加密函数，它的主要结构是海绵结构，内部使用了创新的Keccak-f密码置换。并于2012年10月在SHA-3竞赛中获胜，成为了新的Hash函数标准。
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        // 用create2部署新合约
        Pair pair = new Pair{salt: salt}(); 
        // 调用新合约的initialize方法
        pair.initialize(tokenA, tokenB);
        // 更新地址map
        pairAddr = address(pair);
        allPairs.push(pairAddr);
        getPair[tokenA][tokenB] = pairAddr;
        getPair[tokenB][tokenA] = pairAddr;
	}

	// 提前计算pair合约地址
    function calculateAddr(address tokenA, address tokenB) public view returns(address predictedAddress){
        require(tokenA != tokenB, 'IDENTICAL_ADDRESSES'); //避免tokenA和tokenB相同产生的冲突
        // 计算用tokenA和tokenB地址计算salt
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA); //将tokenA和tokenB按大小排序
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        // 计算合约地址方法 hash()
        // 新地址 = hash("0xFF",创建者地址, salt, bytecode)
        predictedAddress = address(
        	uint160(
        		uint(
    				keccak256(
    					abi.encodePacked(
				            bytes1(0xff),
				            address(this),
				            salt,
				            keccak256(type(Pair).creationCode)
        				)
    				)
        		)
        	)
        );
    }
```

## 删除合约

`selfdestruct`命令可以用来删除智能合约，并将该合约剩余ETH转到指定地址。selfdestruct是为了应对合约出错的极端情况而设计的。

用法：`selfdestruct(_addr);`
	其中_addr是接收合约中剩余ETH的地址

注意事项：
1. 对外提供合约销毁接口时，最好设置为只有合约所有者可以调用，可以使用函数修饰符`onlyOwner`进行函数声明
2. 当合约被销毁后与智能合约的交互也能成功，并且返回0
3. 当合约中有selfdestruct功能时常常会带来安全问题和信任问题，合约中的Selfdestruct功能会为攻击者打开攻击向量(例如使用selfdestruct向一个合约频繁转入token进行攻击，这将大大节省了GAS的费用，虽然很少人这么做)。
	此外，此功能还会降低用户对合约的信心。

合约销毁后的ETH返回给了指定的地址，并且在合约销毁后依然可以请求交互，所以我们不能根据这个来判断合约是否已经销毁

## ABI编码解码

`ABI` (`Application Binary Interface`，应用二进制接口)是与以太坊智能合约交互的标准。

Solidity中，ABI编码有4个函数：`abi.encode`, `abi.encodePacked`, `abi.encodeWithSignature`, `abi.encodeWithSelector`。
而ABI解码有1个函数：`abi.decode`，用于解码abi.encode的数据。

