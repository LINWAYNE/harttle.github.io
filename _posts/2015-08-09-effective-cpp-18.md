---
layout: blog
categories: reading
title: Item 18：让接口容易被正确使用，不易被误用
subtitle: Effective C++笔记
tags: C++ 接口 STL 运算符重载
excerpt: 总之，好的接口容易被正确使用，不易被误用。可以为内置类型提供一致的接口来方便正确的使用。识别误用的手段包括：创建新的类型、限制类型的操作、限制对象的值、移除客户的资源管理责任。
---

> Item 18: Make interfaces easy to use correctly and hard to use incorrectly.

“让接口容易被正确使用，不易被误用”，这也是面向对象设计中的重要概念，好的接口在工程实践中尤其重要。
在使用优秀的第三方组件时，常常能够切身感受到好的接口原来可以这么方便，甚至不需记住它的名字和参数就能正确地调用。
反观自己写的API，常常会有人三番五次地问这个参数怎么设置，真是失败。人非圣贤孰能无过，只能在这种痛苦的驱动下努力的重构和学习！

> 虽然我已经脱离了很久的Windows开发，但想起来.NET API良好的设计，还是会五体投地。

言归正传。

在C++中，可以说到处都是接口，接口定义了客户如何与你的代码进行交互。如果用户误用了你的接口，你至少也要承担一部分的责任。
理想情况的接口是这样的：
*如果用户误用了接口，代码不会正常编译；如果代码通过了编译，那么你的接口就要完成客户想要的操作。*

<!--more-->

# 正确地构造一个Date

来个通俗的例子：`Date`对象的构造函数需要传入月、日、年。但客户在调用时常常传错顺序，这时可以将参数封装为对象来提供类型检查：

```cpp
class Date{
public:
    Date(const Month& m, const Day& d, const Year& y);
};

Date d(Day(30), Month(3), Year(1995));    // 编译错：类型不兼容！
Date d(Month(3), Day(30), Year(1995));    // OK
```

即使这样，用户的`Month`构造函数仍然会传入一个不合理的参数（例如`32`），或者搞不清楚下标从0还是1开始。
解决方案是预定义所有可用的`Month`：

```cpp
class Month{
public:
    static Month Jan(){ return Month(1); }
    static Month Feb(){ return Month(2); }
};
Date d(Month::Jan(), Day(30), Year(1995));
```

从上述`Date`的例子中可以看到，可以将运行时的数据转换为编译期的名称，可以将错误检查提前到编译期。以此解决参数顺序和范围的误用。

# 限制类型的操作

另外一个例子来自[Item 3：尽量使用常量][const]，乘法运算符返回值设为`const`，以防止误用赋值：

```cpp
if(a * b = c) ... // 用户的意图本来是判等
```

# 提供一致的接口

除此之外，提供一致的接口也很重要。例如STL容器封装了互不兼容的基本数据类型，为STL算法提供了非常一致的接口。

> 比如STL提供了`size`属性来标识容器的大小，容器可以是数组、链表、字符串、字典、集合。.NET中所有这些大小都叫Count属性。
采用哪种命名并不重要，重要的是提供一致的接口。不仅便于应用中使用，也便于库的扩展。

**好的接口不会要求用户去记住某些事情。**比如`Investment* createInvestment()`要求客户记住及时去销毁，
那么客户很可能忘记了去`delete`或`delete`了多次。解决方案便是返回一个智能指针而不是原始资源，参见：[Item 13：使用对象来管理资源][res]
尤其是当销毁操作不是简单的`delete`时，客户还需要记住如何去销毁它。而我们返回智能指针时就能指定`deleter`来自定义销毁动作：

```cpp
shared_ptr<Investment> createInvestment(){
    // 销毁一项投资时，需要做一些取消投资的业务，而不是简单地`delete`
    return shared_ptr<Investment>(new Stock, getRidOfInvestment);
}
```

> `shared_ptr`带来的好处还不仅仅是移除了客户的责任，同时还解决了跨DLL动态内存管理的问题。
> 在DLL中`new`的对象，如果在另一个DLL中`delete`往往会发生运行时错误，但`shared_ptr`进行资源销毁时，
> 总会调用创建智能指针的那个DLL中的`delete`，这意味着`shared_ptr`可以随意地在DLL间传递而不需担心跨DLL的问题。

**总之，好的接口容易被正确使用，不易被误用。可以为内置类型提供一致的接口来方便正确的使用。识别误用的手段包括：
创建新的类型、限制类型的操作、限制对象的值、移除客户的资源管理责任。**

[pointers]: {% post_url 2015-07-05-cpp-pointers-and-references %}
[args]: {% post_url 2015-07-07-cpp-functions-and-arguments %}
[const]: {% post_url 2015-07-21-effective-cpp-3 %}
[res]: {% post_url 2015-08-02-effective-cpp-13 %}
