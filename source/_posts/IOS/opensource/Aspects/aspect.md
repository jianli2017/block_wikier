---
title: aspect源码浅析
date: 2019-08-08 10:07:12
tags: aspect
categories: aspect
toc: true
---

图解[aspect](https://github.com/steipete/Aspects)。

<!--more-->


## 图解aspect

aspect 切方法分为两种：

1. 为实例切方法，只实例有效，类的其他实例不受影响，类似KVO，原理是交换了isa。
2. 为类切方法： 交换了类的实例方法，所有实例都生效。

![aspect主流程](aspect/aspect_1.png)

其实看过原理后，就交换了两个方法，一个是当前selector，一个是forwardInvocation。
![aspect交换方法后的结果](aspect/aspect_2.png)

一定要理解方法的中包含SEL、IMP，SEL理解为方法的名称，IMP是方法的代码实现，图中很多地方使用SEL、IMP后缀区分两着。


![aspect执行切片方法的流程](aspect/aspect_3.png)


![aspect执行切片方法的流程直观理解](aspect/aspect_4.png)
特别注意步骤9，解决的是如果向切片类发送了非切片类的方法，还走原来的消息转发流程，如不认识的方法报 not recognize selector send to instance错误。

## 参考 

无