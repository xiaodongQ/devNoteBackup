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

## 开发测试环境

Oracle VM VirtualBox搭建的虚拟机

* 系统: CentOS Linux release 7.6.1810 (Core)
* gcc 版本 4.8.5 (GCC)

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

按自己的节奏来梳理，参考已有的笔记。不完全按已有笔记顺序来，印象和理解不深。

### vector

目录: `SGI-STL\The Annotated STL Sources V3.3\container\sequence container\vector`

#### bvector.h 文件



