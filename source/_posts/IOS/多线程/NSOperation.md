---
title: NSOperation
date: 2019-09-17 10:35:49
tags: NSOperation
categories: 多线程
toc: true
---

NSOperation类文档学习记录


<!--more-->



## 操作依赖

依赖可以控制操作执行的顺序，相关函数：

1. addDependency
2. removeDependency

当依赖的operation全部为完成状态时，operation才能为ready的状态。当最后一个依赖完成后，operation的状态变为ready状态。

依赖不区分operation是完成了还是取消了。

## KVO属性 

KVO属性包括：

1. isCancelled
2. isAsynchronous
3. isExecuting
4. isFinished
5. isReady
6. dependencies
7. queuePriority
8. completionBlock

## 多线程安全

多线程调用NSOperation的方法时安全的，不需要加锁。
子类的自定义方法需要考虑多线程问题。


## 异步VS同步

操作可以手动执行、也可以添加到queue中执行。

手动执行（直接调用start方法），分为同步operation和异步operation ，同步在当前线程立即执行。异步在新的线程执行任务。

如果操作在队列中执行，一般定义为同步的，队列不关注asynchronous属性，总是在一个单独的线程中调用start方法。所有没有理由设计为异步的。

## 子类化

非并发队列：重写 main方法。
并发队列：需要重写start、 asynchronous、executing、finished方法。

**注意是否重写main决定了是否是并发操作。**

下面是子类化的要点：

1. 在并发队列中，start方法负责异步开始操作。
2. start方法中需要通过KVO更新operation的executing状态为YES。
3. operation完成或取消后，并发队列必须通过KVO更新isExecuting为NO 和 isFinished为YES。如果是取消，也需要更新isFinished状态为YES。
4. operation 只有完成了，才能从queue中移除。
5. 同时需要重写 executing、finished属性（KVC）。
6. start方法需要检查是否operation被取消了。
7. 如果定制了依赖，需要KVO isReady属性。


**状态管理**：

1. isReady，一般不用管理，依赖的时候处理。
2. isExecuting，替换了start方法，一定要替换isExecuting方法，并在start开始的时候发出KVO
3. isFinished，替换了start方法，一定要替换isFinished方法。operation完成或取消，发出KVO
4. isCancelled，不需要发出KVO

**响应取消**

一旦将operation添加到queue中，queue就掌管了operation。你可以通过调用operation的cancell方法取消，或者通过queue的cancelAllOperations取消。

执行中的任务并不会立马取消， 你必须显式的检测状态，需要的时候取消。