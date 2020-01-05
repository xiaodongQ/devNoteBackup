[TOC]

<!--
 * @Description: 记录该fork学习过程
 * @Author: xd
 * @Date: 2019-10-12 08:24:44
 * @LastEditTime: 2019-10-23 08:41:22
 * @LastEditors: xd
 * @Note:
 -->
# SGI-STL学习笔记

* 《STL源码剖析》 --侯捷，根据本书流程学习梳理
    - STL由 Alexander Stepanov 创造于1979年前后
* STL GitHub源码和笔记链接：[xiaodongQ/SGI-STL](https://github.com/xiaodongQ/SGI-STL)
    - 原项目fork自[steveLauwh/SGI-STL](https://github.com/steveLauwh/SGI-STL)
* SGI版本STL由STL之父 Alexander Stepanov、经典书籍《Generic Programming and the STL》作者 Matthew H. Austern、STL巨匠 David Musser等人实现。
    - 这份产品被纳为GNU C++标准程序库。
    - SGI指的是：
        + Silicon Graphics, Inc 硅谷图形公司(后来更名为SGI，历史上被称为Silicon Graphics Computer Systems, Inc 或SGCS) (硅谷图形计算机系统公司)
        + 2006年3月8日，SGI申请破产保护
        + 2016年8月11日，慧与科技（HPE, Hewlett Packard Enterprise）宣布以每股美金7.75, 合共2亿7仟5百万美金, 收购SGI所有股权
        + [硅谷图形公司](https://zh.wikipedia.org/zh-hans/%E7%A1%85%E8%B0%B7%E5%9B%BE%E5%BD%A2%E5%85%AC%E5%8F%B8)

## 开发测试环境

Oracle VM VirtualBox搭建的虚拟机

* 系统: CentOS Linux release 7.6.1810 (Core)
* gcc 版本 4.8.5 (GCC)

## 《STL源码剖析》 --侯捷

> 追踪一流作品并与其中吸取养分，远比自己关起门来写个三流作品，价值高得多。我的确认为99.99%的程序员所写的程序，在SGI STL面前都是三流水准。

### STL概论

* STL概论
    - 复用性(reusability)的提升
    - STL所实现的，是根据泛型思维(Generic Paradigm)架设起来的一个概念结构。
* STL六大组件
    - 容器(`containers`)
        + 各种数据结构，如vector、list、queue、set、map，用来存放数据
        + 从实现角度看，STL容器是一种 class template
    - 算法(`algorithms`)
        + 各种常见算法，如sort、search、copy、erase...
        + 从实现角度看，STL算法是一种 function template
    - 迭代器(`iterators`)
        + 扮演容器和算法之间的胶合剂，即"泛型指针"，共五种类型
        + 从实现的角度看，迭代器是一种将`operator*`,`operator->`,`operator++`,`operator--`等相关操作予以重载的 class template
    - 仿函数(`functors`)
        + 和函数类似，可作为算法的某种策略(policy)
        + 从实现的角度看，仿函数是一种重载了operator()的class或class template
    - 配接器(`adapters`)
        + 用来修饰 容器 或 仿函数 或 迭代器接口。e.g. queue和stack虽然看似容器，实际是一种容器配接器，底部完全借助deque
        + 改变functor接口，称为functor adapter；改变container接口，称为container adapter；改变iterator称iterator adapter
    - 配置器(`allocators`)
        + 负责空间配置与管理
        + 从实现角度看，配置器是一个实现了动态空间配置、空间管理、空间释放的 class template
* 六大组件交互关系
    - `container`通过`allocator`取得数据存储空间，`algorithm`通过`iterator`存取`container`内容，`functor`可以协助`algorithm`完成不同的策略变化，`adapter`可以修饰或套接`functor`
* `open source`
    - 开放源代码的关联源自美国人Richard Stallman(理查德·斯托曼)
    - Stallman于1984年离开麻省理工学院，创立自由软件基金会(Free Software Foundation)，写下著名的`GNU`宣言
    - GUN代表 GUN is Not Unix 的递归缩写(计算机族的幽默展现)
    - GNU以`GPL`(General Public License 广泛开放授权)来保护其成员：使用者可以自由阅读与修改GPL软件的源代码，但如果使用者要传播借助GPL软件而完成的软件，他们必须也同意GPL规范。
    - GPL对于版权(`copyright`)观念带来巨大挑战，甚至被称为"反版权"(`copyleft`，又一个属于计算机族群的幽默)
* STL版本
    - `HP`(惠普)实现版本 是所有STL实现版本的始祖。每个HP STL头文件都有对应的一份声明
        + `Copyright (c) 1994`, `Hewlett-Packard Company`
        + 本版本的授权并不属于GNU GPL范畴，但属于open source范畴
    - `P.J.Plauger`实现版本(PJ STL)，PJ版本继承HP版本，所以它的每个头文件都有HP的版本声明
        + P.J.Plauger版本被Visual C++采用
        + 本版本不属于open source范畴，更不是GNU GPL(因为HP的版本声明并非GPL，并没有强迫其衍生产品必须开放源代码)
    - `Rouge Wave`实现版本，继承自HP版本
        + 本版本不属于open source范畴，更不是GNU GPL(因为HP的版本声明并非GPL，并没有强迫其衍生产品必须开放源代码)
    - `STLport`实现版本
        + 提供一个以SGI STL为蓝本的高度可移植性实现版本
    - `SGI STL`实现版本
        + 由 Silicon Graphics Computer Systems, Inc 公司发展，继承自HP版本。
        + 所以它的每个头文件也都有HP的版本声明，此外还加上SGI公司的版权声明。
        + 属于open source的一员，但不属于GNU GPL
            * 可以看到本份STL源码中除了上面的HP声明外，还有以下声明：
            * `Copyright (c) 1996`, `Silicon Graphics Computer Systems, Inc.`
        + SGI版本被`GCC`采用。
* `stl_config.h`
    - 不同的编译器对C++语言的支持程度不尽相同。
    - SGI STL准备了一个 环境组态文件<stl_config.h>，其中定义了许多常量，标识某些组态是否支持(通过宏控制代码块或功能的开关)
    - 预处理器(pre-processor)根据各个常量决定取舍哪一段程序代码

### 空间配置器 (allocator)







## 容器库概述

参考：
cppreference.com:
[容器库](https://zh.cppreference.com/w/cpp/container)

github repository website:
[容器(container)](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container)

容器库是类模板与算法的汇集，允许程序员简单地访问常见数据结构，例如队列、链表和栈。

有三类容器——顺序容器(序列式容器)、关联容器和无序关联容器

### 顺序容器(序列式容器 sequence container)

顺序容器实现能按顺序访问的数据结构。

* array         (C++11 起)静态的连续数组 (固定大小类似T[N] 但不会退化为`T*`;各类容器接口)
* vector        动态的连续数组 (支持随机访问)
* deque         双端队列 (双向进出的连续空间， 支持随机访问)
* forward_list  (C++11 起)单链表
* list          双链表 (不支持随机访问，插入和删除元素: O(1)，不连续)

---
* heap (内含一个 vector) (完全二叉树，最大堆或最小堆)
* priority-queue (内含一个 heap)
* slist (非标准)
* stack (内含一个 deque) (adapter 配接器)
* queue (内含一个 deque) (adapter 配接器)

---

### 关联容器

关联容器实现能快速查找（ O(log n) 复杂度）的数据结构。

* set           唯一键的集合，按照键排序  (内含一个 RB-tree)
* map           键值对的集合，按照键排序，键是唯一的 (内含一个 RB-tree)
* multiset      键的集合，按照键排序 (内含一个 RB-tree)
* multimap      键值对的集合，按照键排序 (内含一个 RB-tree)

---
* RB-tree (非公开)
* hashtable (非标准)
* hash_set (内含一个 hashtable) (非标准)
* hash_map (内含一个 hashtable) (非标准)
* hash_multiset (内含一个 hashtable) (非标准)
* hash_multimap (内含一个 hashtable) (非标准)

---

### 无序关联容器

无序关联容器提供能快速查找（均摊 O(1) ，最坏情况 O(n) 的复杂度）的无序（哈希）数据结构。

* unordered_set         (C++11 起)唯一键的集合，按照键生成散列
* unordered_map         (C++11 起)键值对的集合，按照键生成散列，键是唯一的
* unordered_multiset    (C++11 起)键的集合，按照键生成散列
* unordered_multimap    (C++11 起)键值对的集合，按照键生成散列

## 学习目录结构

* 按此目录梳理学习：The Annotated STL Sources V3.3(学习源代码的注释)，各子目录内容如下：
    - adapter   配接器
    - algorithm 算法
    - allocator 配置器
    - container 容器
    - functor-function object 仿函数或函数对象
    - iterator  迭代器
    - Other     部分演示示例图和思维导图

* STL 六大组件的交互关系
    - Container 通过 Allocator 取得数据储存空间
    - Algorithm 通过 Iterator 存取 Container 内容
    - Functor 可以协助 Algorithm 完成不同的策略变化
    - Adapter 可以修饰或套接 Functor、Iterator。

## 顺序容器 /container/sequence container/

参考：

* [顺序容器README.md](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container)
    - [vector](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container/vector)
    - [list](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container/list)
    - [deque](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container/deque)
    - [stack](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container/stack)
    - [queue](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container/queue)
    - [heap](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container/heap)
    - [priority_queue](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container/queue)
    - [slist](https://github.com/steveLauwh/SGI-STL/tree/master/The%20Annotated%20STL%20Sources%20V3.3/container/sequence%20container/slist)

* vector和array
    - vector 与 array 唯一区别是空间的运用的灵活性
    - 增加新元素，如果超过当时的容量，则容量会扩充至两倍。

按自己的节奏来梳理，参考已有的笔记。思路不完全照着已有笔记的顺序，否则印象和理解不深。

### vector

目录: `SGI-STL\The Annotated STL Sources V3.3\container\sequence container\vector`

* bvector.h 文件







