---
title: Objective-C 中的元类（meta class）是什么？
date: 2016-11-12 19:56:17
tags: runtime
categories: runtime
toc: true
---

在本文我们会看到一个在Objective-C中很陌生的概念——元类。Objective-C中的每个类都有和自己相关联的元类，但我们几乎从来不直接使用它，它们依然是那么神秘。我们将开始学习怎样在运行时创建一个类。通过创建的“class pair”，我会解释什么是元类，然后探讨它对于Objective-C中对象和类的意义。


<!--more-->

## 在运行时创建一个类

下面的代码在运行时创建了一个NSError的子类，并且添加了一个方法：

```
Class newClass =
objc_allocateClassPair([NSError class], "RuntimeErrorSubclass", 0);
class_addMethod(newClass, @selector(report), (IMP)ReportFunction, "v@:");
objc_registerClassPair(newClass);
```

ReportFunction函数就是添加的实例方法，具体实现如下：

```
void ReportFunction(id self, SEL _cmd)
{
    NSLog(@"This object is %p.", self);
    NSLog(@"Class is %@, and super is %@.", [self class], [self superclass]);
 
    Class currentClass = [self class];
    for (int i = 1; i < 5; i++)
    {
        NSLog(@"Following the isa pointer %d times gives %p", i, currentClass);
        currentClass = object_getClass(currentClass);
    }
 
    NSLog(@"NSObject's class is %p", [NSObject class]);
    NSLog(@"NSObject's meta class is %p", object_getClass([NSObject class]));
}
```

表面上看来，这相当简单。在运行时创建一个类只需要3个步骤:

1. 为”class pair”分配内存 (使用objc_allocateClassPair).
2. 添加方法或成员变量到有需要的类里 (我已经使用class_addMethod添加了一个方法).
3. 注册类以便它能使用 (使用objc_registerClassPair).


然而，有一个很迫切的问题：什么是“class pair”？objc_allocateClassPair函数仅返回了一个值：the class。那另一半pair在哪？

我相信你已经猜到了，另一半pair就是元类（这篇文章的主题）。为了解释它是什么和我们为什么需要它，还需要交代下Objective-C的对象和类的相关背景。

## 什么数据结构才能称之为对象？

每个对象都有类。这是面向对象的基本概念，但是在Objective-C中，它对数据结构也一样。含有一个指针且该指针可以正确指向类的数据结构，都可以被视作为对象。

在Objective-C中，对象的类是isa指针决定的。isa指针指向对象所属的类。

实际上，Objective-C中对象最基本的定义是这样的：

```
typedef struct objc_object {
    Class isa;
} *id;
```

这说的是：任何带有以指针开始并指向类结构的结构都可以被视作objc_object。

Objective-C中对象最重要的特点是你可以发送消息给它们：

```
[@"stringValue"
    writeToFile:@"/file.txt" atomically:YES encoding:NSUTF8StringEncoding error:NULL];
```

这能工作是因为Objective-C对象（这儿是NSCFString）在发送消息时，运行时库会追寻着对象的isa指针得到了对象所属的类（这儿是NSCFString类）。这个类包含了能应用于这个类的所有实例方法和指向超类的指针以便可以找到父类的实例方法。运行时库检查这个类和其超类的方法列表，找到一个匹配这条消息的方法（在上面的代码里，是NSString类的writeToFile:atomically:encoding:error方法）。运行时库基于那个方法调用函数（IMP）。

重点就是类要定义这个你发送给对象的消息。

## 什么是元类

现在，可能你已经知道了，Objective-C的一个类也是一个对象。这意味着你可以发送消息给一个类。

```
NSStringEncoding defaultStringEncoding = [NSString defaultStringEncoding];
```

因为Objective-C中每个类本身也是一个对象。如上面所展示的，这意味着类结构必须以一个isa指针开始，从而可以和objc_object在二进制层面兼容，然后这个结构的下一字段必须是一个指向超类的指针（对于基类则为nil）。

[正如我上周展示的](http://www.cocoawithlove.com/2010/01/getting-subclasses-of-objective-c-class.html)，类被定义的方式有点不同，依赖于你的运行时库版本，但是，它们都以isa字段开始，随后是superclass字段。

```
typedef struct objc_class *Class;
struct objc_class {
    Class isa;
    Class super_class;
    /* followed by runtime specific details... */
};
```

为了调用类里的方法，类的isa指针必须指向包含这些类方法的类结构体。

这就引出了元类的定义：元类是类对象的类。
简单说就是：

1. 当你给对象发送消息时，消息是在寻找这个对象的类的方法列表。
2. 当你给类发消息时，消息是在寻找这个类的元类的方法列表。


元类是必不可少的，因为它存储了类的类方法。每个类都必须有独一无二的元类，因为每个类都有独一无二的类方法。

## 元类的类是什么？

元类，就像之前的类一样，它也是一个对象。你也可以调用它的方法。自然的，这就意味着他必须也有一个类。

所有的元类都使用根元类（继承体系中处于顶端的类的元类）作为他们的类。这就意味着所有NSObject的子类（大多数类）的元类都会以NSObject的元类作为他们的类

根据这个规则，所有的元类使用根元类作为他们的类，根元类的元类则就是它自己。也就是说基类的元类的isa指针指向他自己。

## 类和元类的继承

类用 super_class指针指向了超类，同样的，元类用super_class指向类的super_class的元类。

说的更拗口一点就是，根元类把它自己的基类设置成了super_class。

在这样的继承体系下，所有实例、类以及元类（meta class）都继承自一个基类。

这意味着对于继承于NSObject的所有实例、类和元类，他们可以使用NSObject的所有实例方法，类和元类可以使用NSObject的所有类方法

这些文字看起来莫名其妙难以理解。Greg Parker给出了一份精彩的图谱来展示这些关系：

## 实验证明

为了验证，让我们看看我在文章开始写的ReportFunction 函数的输出。这个函数的目的是跟随isa指针并打印出它的路途。

为了运行ReportFunction，我们需要创建一个动态实例来创建类调用report方法。

```
id instanceOfNewClass =
    [[newClass alloc] initWithDomain:@"someDomain" code:0 userInfo:nil];
[instanceOfNewClass performSelector:@selector(report)];
[instanceOfNewClass release];
```

这里没有声明report方法，但我使用performSelector:调用它，所以编译器不会给出警告。
然后ReportFunction函数会沿着isa进行检索，来告诉我们class，meta-class以及meta-class的class是什么样的情况：

得到对象的类：ReportFunction 函数使用object_getClass跟踪isa指针，因为isa指针是类的保护成员（你不能直接接收其他对象的isa指针）。ReportFunction不使用类方法，因为在类对象里调用类方法不能返回元类，它会再次返回这个类（因此[NSString class]会返回NSString 类而不是NSString元类）

This is the output (minus NSLog prefixes) when the program runs:
这是程序运行时的输出（省略了NSlog前缀）：

```
This object is 0x10010c810.
Class is RuntimeErrorSubclass, and super is NSError.
Following the isa pointer 1 times gives 0x10010c600
Following the isa pointer 2 times gives 0x10010c630
Following the isa pointer 3 times gives 0x7fff71038480
Following the isa pointer 4 times gives 0x7fff71038480
NSObject's class is 0x7fff710384a8
NSObject's meta class is 0x7fff71038480
```

观察isa到达过的地址的值：

1. 对象的地址是 0x10010c810.
2. 类的地址是 0x10010c600.
3. 元类的地址是 0x10010c630.
4. 根元类（NSObject的元类）的地址是 0x7fff71038480.
5. NSObject元类的类是它本身.


这些地址的值并不重要，重要的是它们说明了文中讨论的从类到meta-class到NSObject的meta-class的整个流程。

## 最后

元类是 Class 对象的类。每个类（Class）都有自己独一无二的元类（每个类都有自己第一无二的方法列表）。这意味着所有的类对象都不同。

元类总是会确保类对象和基类的所有实例和类方法。对于从NSObject继承下来的类，这意味着所有的NSObject实例和protocol方法在所有的类（和meta-class）中都可以使用。

所有的meta-class使用基类的meta-class作为自己的基类，对于顶层基类的meta-class也是一样，只是它指向自己而已。