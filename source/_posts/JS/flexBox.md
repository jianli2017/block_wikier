---
title: flexBox 伸缩盒子模型
date: 2016-12-2 16:18:26
categories: weex
tags: flexBox
toc: true
---

本文指在理解flexBox的各个效果，学习React Native的时候总结。


<!--more-->

##  盒子模型

### 标准盒子模型

标准的盒子模型如下图所示：

![盒子模型原理](flexBox/17.png)

### 伸缩盒子模型

任何一个元素都可以指定为flexbox 布局，设置为display:flex或display:inline-flex的元素称为伸缩容器，伸缩容器的子元素称为伸缩项目，下面是伸缩的模型：

![伸缩盒子模型](flexBox/1.1.png)

## 二、React Native中使用flexBox

1. flexDirection（伸缩容器）
2. alignItems（伸缩容器）
3. flexWrap（伸缩容器）
4. justifyContent（伸缩容器）
5. alignSelf（伸缩项目）
6. flex （伸缩项目）


### flexDirection 指定主轴方向

```
flexDirection:row|column
```

row、column的效果图如下：

![row的效果](flexBox/1.png)

![column的效果](flexBox/2.png)


### alignItems 

该属性用来定义伸缩项目在伸缩容器的交叉轴上的对齐方式

```
alignSelf:auto|flex-start|flex-end|center|stretch
```

flex-start、flex-end、center的效果图如下：

![flex-start](flexBox/3.png)

![flex-end](flexBox/4.png)

![center的效果](flexBox/5.png)



### flexWrap 

指定伸缩容器的主轴方向空间不足的情况下，是否换行以及如何换行

```
flexWrap:wrap|nowrap
```

wrap、nowrap的效果图如下：

![wrap](flexBox/6.png)

![不折叠的效果](flexBox/7.png)

### justifyContent

指定伸缩项目沿主轴线的对齐方式

```
justifyContent:flex-start|flex-end|center|space-between|space-around
```

flex-start、flex-end、center、space-between、space-around的效果图如下：

![flex-start](flexBox/8.png)

![flex-end](flexBox/9.png)

![center](flexBox/10.png)

![space-between](flexBox/11.png)

![space-around](flexBox/12.png)

### alignSelf 

设置单独的伸缩项目在交叉轴上的对齐方式，会覆写默认的对齐方式

```
alignSelf:auto|flex-start|flex-end|center|stretch
```

lex-start、flex-end、center 的效果图如下：

![lex-start](flexBox/13.png)

![flex-end](flexBox/14.png)

![center](flexBox/15.png)

### flex

```
flex:number
```

分别设置四个伸缩项目的 flex为3、2、1、4，效果图如下：

![flex](flexBox/16.png)
