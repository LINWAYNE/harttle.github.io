---
layout: blog
categories: reading
title: Item 45：使用成员函数模板来接受所有兼容的类型
subtitle: Effective C++笔记
tags: C++ 智能指针 指针 模板 成员函数 类型转换
excerpt: 成员函数模板可以使得函数可以接受所有兼容的类型。如果你用成员函数模板声明了拷贝构造函数和赋值运算符，仍然需要手动编写普通拷贝构造函数和拷贝运算符。
---

> Item 45: Use member function templates to accept "all compatible types".

[Item 13][item13]提到智能指针可用来自动释放堆中的内存，STL中的迭代器也是一种智能指针，它甚至支持链表元素指针的`++`操作。
这些高级特性是普通指针所没有的。本文以智能指针为例，介绍成员函数模板的使用：

* 成员函数模板可以使得函数可以接受所有兼容的类型。
* 如果你用成员函数模板声明了拷贝构造函数和赋值运算符，仍然需要手动编写普通拷贝构造函数和拷贝运算符。

<!--more-->

# 隐式类型转换

智能指针虽然比普通指针提供了更多有用的特性，但也存在一些问题，比如我们有一个类的层级：

```cpp
class Top{};
class Middle: public Top{};
class Bottom: public Middle{};
```

普通指针可以做到派生类指针隐式转换为基类指针：

```cpp
Top *p1 = new Bottom;
const Top *p2 = p1;
```

但如果是智能指针，比如我们实现了`SmartPtr`，我们则需要让下面代码经过编译：

```cpp
SmartPtr<Top> p1 = SmartPtr<Bottom>(new Bottom);
SmartPtr<const Top> p2 = p1;
```

同一模板的不同实例之间是没有继承关系的，在编译器看来`AutoPtr<Top>`和`AutoPtr<Bottom>`是完全不同的两个类。
所以上述代码直接编译是有问题的。

# 重载构造函数

为了支持用`SmartPtr<Bottom>`初始化`SmartPtr<Top>`，我们需要重载`SmartPtr`的构造函数。
原则上讲，有多少类的层级我们就需要写多少个重载函数。因为类的层级是会扩展的，因此需要重载的函数数目是无穷的。
这时便可以引入成员函数模板了：

```cpp
template<typename T>
class SmartPtr{
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other);
};
```

> 注意该构造函数没有声明为`explicit`，是为了与普通指针拥有同样的使用风格。子类的普通指针可以通过隐式类型转换变成基类指针。

接受同一模板的其他实例的构造函数被称为**通用构造函数**（generalized copy constructor）。

# 兼容类型检查

事实上，*通用构造函数*提供了更多的功能。他可以把一个`SmartPtr<Top>`隐式转换为`SmartPtr<Bottom>`，把一个`SmartPtr<int>`转换为`SmartPtr<double>`。
但普通指针是不允许这些隐式转换的。因此我们需要把它们禁用掉。注意一下通用构造函数的实现方式便可以解决这个问题：

```cpp
template<typename T>
class SmartPtr{
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other): ptr(other.get()){};
    T* get() const{ return ptr; }
private:
    T *ptr;
};
```

在`ptr(other.get())`时编译器会进行类型的兼容性检查，只有当`U`可以隐式转换为`T`时，`SmartPtr<U>`才可以隐式转换为`SmartPtr<T>`。
这样就避免了不兼容指针的隐式转换。

# 其他使用方式

除了隐式类型转换，*成员函数模板*还有别的用途，例如赋值运算符。。下面是`shared_ptr`的部分源码：

```cpp
template<class T> 
class shared_ptr{
public:
    template<class Y>
        explicit shared_ptr(Y *p);
    template<class Y>
        shared_ptr<shared_ptr<Y> const& r>;
    template<class Y>
        shared_ptr& operator=(shared_ptr<Y> const& r);
};
```

可以看到普通指针`Y*`到`shared_ptr<Y>`声明了`explicit`，需要显式的类型转换；而`shared_ptr<Y>`之间只需要隐式转换。
使用拷贝构造函数模板存在一个问题：编译器是会生成默认的拷贝构造函数？还是会从你的模板实例化一个拷贝构造函数？
即`Y == T`场景下的编译器行为。

**事实上，成员函数模板不会改变C++的规则**。C++规则讲：如果你没有声明拷贝构造函数，那么编译器应该生成一个。
所以`Y == T`时拷贝构造函数不会从成员函数模板实例化，而是会自己生成一个。

所以`shared_ptr`模板中还是手动声明了拷贝构造函数：

```cpp
template<class T>
class shared_ptr{
public:
    shared_ptr(shared_ptr const& r);
    template<class Y>
        shared_ptr(shared_ptr<Y> const& r);

    shared_ptr& operator=(shared_ptr const& r);
    template<class Y>
        shared_ptr& operator=(shared_ptr<Y> const& r);
};
```

[pointers]: {% post_url 2015-07-05-cpp-pointers-and-references %}
[args]: {% post_url 2015-07-07-cpp-functions-and-arguments %}
[item2]: {% post_url 2015-07-20-effective-cpp-2 %}
[item3]: {% post_url 2015-07-21-effective-cpp-3 %}
[item4]: {% post_url 2015-07-22-effective-cpp-4 %}
[item6]: {% post_url 2015-07-23-effective-cpp-6 %}
[item7]: {% post_url 2015-07-24-effective-cpp-7 %}
[item11]: {% post_url 2015-07-30-effective-cpp-11 %}
[item12]: {% post_url 2015-08-01-effective-cpp-12 %}
[item13]: {% post_url 2015-08-02-effective-cpp-13 %}
[item15]: {% post_url 2015-08-05-effective-cpp-15 %}
[item18]: {% post_url 2015-08-09-effective-cpp-18 %}
[item20]: {% post_url 2015-08-13-effective-cpp-20 %}
[item22]: {% post_url 2015-08-19-effective-cpp-22 %}
[item25]: /2015/08/23/effective-cpp-25.html
[item30]: /2015/08/28/effective-cpp-30.html
[item31]: /2015/08/29/effective-cpp-31.html
[item32]: /2015/08/30/effective-cpp-32.html
[item33]: /2015/08/31/effective-cpp-33.html
[item35]: /2015/09/02/effective-cpp-35.html
[item36]: /2015/09/03/effective-cpp-36.html
[item38]: /2015/09/05/effective-cpp-38.html
[item39]: /2015/09/06/effective-cpp-39.html
[item43]: /2015/09/10/effective-cpp-43.html
