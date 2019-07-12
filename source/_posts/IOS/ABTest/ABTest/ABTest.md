---
title: ABTest IOS SDK设计
date: 2019-03-22 14:18:26
categories: ABTest
tags: ABTest 
toc: true
---

阅读目录：

1. [ABTest使用场景分析](#design)
2. [SDK设计图](#sdk)
3. [附录：XX App AB效果](#jd)

## <div id= design>ABTest使用场景分析</div>

下图说明ABTest使用场景的分析。

![流程解析](ABTest/ABL1.png)

依据上图的分析，设计出下面的SDK流程。

## <div id= sdk>SDK设计图</div>


下图说明SDK的设计图。

![SDK设计图](ABTest/ABL2.png)

## <div id= jd>附录：XX App AB效果</div>

经过反编译XX APP，分析XX客户端的AB实现原理，然后修改XX的代码，对比底部Bar的AB效果。


![A](ABTest/JDA.PNG)
![B](ABTest/JDB.jpeg)