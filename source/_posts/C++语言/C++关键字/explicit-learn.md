---
title: explicit实例浅析(转载)
date: 2018-07-16 09:07:12
tags: explicit
categories: C++关键字
toc: true
---

在C++程序中很少有人去使用explicit关键字，不可否认，在平时的实践中确实很少能用的上。再说C++的功能强大，往往一个问题可以利用好几种C++特性去解决。但稍微留心一下就会发现现有的MFC库或者C++标准库中的相关类声明中explicit出现的频率是很高的。了解explicit关键字的功能及其使用对于我们阅读使用库是很有帮助的，而且在编写自己的代码时也可以尝试使用。既然C++语言提供这种特性，我想在有些时候这种特性将会非常有用。

<!--more-->

## 初识

按默认规定，只用传一个参数的构造函数也定义了一个隐式转换。举个例子：
（下面这个CExample没有什么实际的意义，主要是用来说明问题）

```
//Example.h
#pragma once
class CExample
{
public:
CExample(void);
public:
~CExample(void);
public:
int m_iFirst;
int m_iSecond;
public:
CExample(int iFirst, int iSecond = 4);
};
//Example.cpp
#include "StdAfx.h"
#include "Example.h"
CExample::CExample(void)
: m_iFirst(0)
{
}
CExample::~CExample(void)
{
}
CExample::CExample(int iFirst, int iSecond):m_iFirst(iFirst), m_iSecond(iSecond)
{
}
//TestExplicitKey.cpp
...//其它头文件
#include "Example.h"
int _tmain(int argc, _TCHAR* argv[])
{
CExample objOne; //调用没有参数的构造函数
CExample objTwo(12, 12); //调用有两个参数的构造函数
CExample objThree(12); //同上，可以传一个参数是因为该构造函数的第二个参数有默认值
CExample objFour = 12; //执行了隐式转换,等价于CExample temp(12);objFour(temp);注意这个地方调用了
//编译器为我们提供的默认复制构造函数
return 0;
}
```

如果在构造函数声明中加入关键字explicit，如下

```
explicit CExample(int iFirst, int iSecond = 4);
```

那么CExample objFour = 12; 这条语句将不能通过编译。在vs05下的编译错误提示如下

```
error C2440: 'initializing' : cannot convert from 'int' to 'CExample'
    Constructor for class 'CExample' is declared 'explicit'
```

## explicit意义

对于某些类型，这一情况非常理想。<font color=blue>但在大部分情况中，隐式转换却容易导致错误（不是语法错误，编译器不会报错）。隐式转换总是在我们没有察觉的情况下悄悄发生，除非有心所为，隐式转换常常是我们所不希望发生的。通过将构造函数声明为explicit（显式）的方式可以抑制隐式转换。也就是说，explicit构造函数必须显式调用。</font>
引用一下Bjarne Stroustrup的例子:

```
class String{
   explicit String(int n);
   String(const char *p);
};
String s1 = 'a'; //错误：不能做隐式char->String转换
String s2(10);  //可以：调用explicit String(int n);
String s3 = String(10);//可以：调用explicit String(int n);再调用默认的复制构造函数
String s4 = "Brian"; //可以：隐式转换调用String(const char *p);再调用默认的复制构造函数
String s5("Fawlty"); //可以：正常调用String(const char *p);
void f(String);
String g()
{
  f(10); //错误：不能做隐式int->String转换
  f("Arthur"); //可以：隐式转换，等价于f(String("Arthur"));
  return 10; //同上
}
```

在实际代码中的东西可不像这种故意造出的例子。
发生隐式转换，除非有心利用，隐式转换常常带来程序逻辑的错误，而且这种错误一旦发生是很难察觉的。
原则上应该在所有的构造函数前加explicit关键字，当你有心利用隐式转换的时候再去解除explicit，这样可以大大减少错误的发生。

原文链接：http://blog.csdn.net/chollima/article/details/3486230
