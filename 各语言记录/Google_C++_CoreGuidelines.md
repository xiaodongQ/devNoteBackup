# Google C++ Core Guidelines

[TOC]

[Google C++ Core Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-org)

[中文版本](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/formatting/)

## 编码风格

示例里的编码风格:

* 定义类

```cpp
class X {      // { 紧接类名，类名和 { 间有空格
public:
    // interface
protected:
    // unchecked function for use by derived class implementations
private:
    // implementation details (types, functions, and data)
};
```

* 函数

```

```


## [NL: Naming and layout rules](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#nl-naming-and-layout-rules)

背景：

Consistent naming and layout are helpful. If for no other reason because it minimizes “my style is better than your style” arguments.However, there are many, many, different styles around and people are passionate about them (pro and con). Also, most real-world projects includes code from many sources, so standardizing on a single style for all code is often impossible. After many requests for guidance from users, we present a set of rules that you might use if you have no better ideas, but the real aim is consistency, rather than any particular rule set. IDEs and tools can help (as well as hinder).

一致的命名和布局是有帮助的。不是其他原因，而是它最小化了“我的风格比你的风格好”的争论。然而，有许多许多不同的风格，人们对于它们是很热情的(赞成和反对)。而且，大多数实际项目都包含来自许多来源的代码，因此将所有代码标准化为单一样式通常是不可能的。在用户多次请求指导之后，我们提供了一组规则，如果您没有更好的想法，您可以使用这些规则，但是真正的目标是一致性，而不是任何特定的规则集。IDE和工具可以提供帮助(也可能成为阻碍)。

### NL.16: Use a conventional class member declaration order

使用常规的类成员声明顺序，各成员声明顺序参考以下惯例：

A conventional order of members improves readability.
When declaring a class use the following order

* types: classes, enums, and aliases (using) 类型,别名
* constructors, assignments, destructor 构造,赋值,析构
* functions 函数
* data

Use the `public` before `protected` before `private` order.

示例：

```cpp
class X {
public:
    // interface
protected:
    // unchecked function for use by derived class implementations
private:
    // implementation details
};
```

一些时候成员和上面顺序冲突，可以把私有类型和函数放到private里

避免声明多个相同的访问域，多个public、private等

**坏**的示例：

```cpp
class X {   // bad
public:
    void f();
public:
    int g();
    // ...
};
```

### NL.18: Use C++-style declarator layout

使用C++风格的声明符布局

C风格的布局强调在`表达式`和`语法`中使用，而C++风格强调`类型`

```cpp
T& operator[](size_t);   // OK 应该使用这种形式
T &operator[](size_t);   // just strange 比较奇怪
T & operator[](size_t);   // undecided 有误，未确定的
```

