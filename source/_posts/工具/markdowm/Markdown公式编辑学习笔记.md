---
title: Markdown公式编辑
date: 2018-06-25 12:11:12
tags: Markdown
categories: Markdown
toc: true
---
Markdown中使用LaTEX 编写含有数学公式的博客。

<!--more-->

## 如何插入公式

1. 行中公式（放在文中与其他文字编辑）：可以用如下方法表示：`$数学公式$`
2. 独立公式可以用下面的方法表示：`$$数学公式$$`
3. 自动编号的公式可以用如下的方法表示：
```
\begin{equation}
数学公式
\label{eq:当前公式名}
\end{equation}
自动编号后的公式可在全文任意处使用 \eqref{eq:公式名} 语句引用。
```

例子

```
$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，not one line } $

```

显示$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，not one line } $

例子

```
$$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，独立公式示例}$$
```

显示

$$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，独立公式示例} $$

## 下标 
`^` 表示上标， `_` 表示下标。如果上下标的内容多于一个字符，需要用`{}`将这些内容括成一个整体。上下标可以嵌套，也可以同时使用。

例子

```
$$ x^{y^z}=(1+{\rm e}^x)^{-2xy^w} $$
```

显示

$$ x^{y^z}=(1+{\rm e}^x)^{-2xy^w} $$

另外，如果要在左右两边都有上下标，可以用`\sideset` 命令

例子：

```
$$ \sideset{^1_2}{^3_4}\bigotimes $$

```

显示

$$ \sideset{^1_2}{^3_4}\bigotimes $$

## 括号和分隔符 

`()`、`[]`和`|`表示符号本身，使用 `\{\}` 来表示 `{}`。当要显示大号的括号或分隔符时，要用 \left 和 \right 命令。
一些特殊的括号：

|输入|显示|
|---|---|
|`$$\langle表达式\rangle$$`|$$\langle表达式\rangle$$|
|`$$\lceil表达式\rceil$$`|$$\lceil表达式\rceil$$|
|`$$\lfloor表达式\rfloor$$`|$$\lfloor表达式\rfloor$$|
|`$$\lbrace表达式\rbrace$$`|$$\lbrace表达式\rbrace$$|

例子

```
$$ f(x,y,z) = 3y^2z \left( 3+\frac{7x+5}{1+y^2} \right) $$
```
显示
$$ f(x,y,z) = 3y^2z \left( 3+\frac{7x+5}{1+y^2} \right) $$

## 分数

通常使用 `\frac {分子} {分母}`命令产生一个分数`\frac {分子} {分母}`，分数可嵌套。
便捷情况可直接输入 \frac ab来快速生成一个\frac ab。
如果分式很复杂，亦可使用 分子 \over 分母 命令，此时分数仅有一层。

例子：

```
$$\frac{a-1}{b-1} \quad and \quad {a+1\over b+1}$$
```

显示

$$\frac{a-1}{b-1} \quad and \quad {a+1\over b+1}$$

## 开方

使用 `\sqrt [根指数，省略时为2] {被开方数}`命令输入开方。

例子：

`$$\sqrt{2} \quad and \quad \sqrt[n]{3}$$`

显示

$$\sqrt{2} \quad and \quad \sqrt[n]{3}$$

## 省略号

数学公式中常见的省略号有两种，`\ldots` 表示与文本底线对齐的省略号，`\cdots` 表示与文本中线对齐的省略号。

例子：

```
$$f(x_1,x_2,\underbrace{\ldots}_{\rm ldots} ,x_n) = x_1^2 + x_2^2 + \underbrace{\cdots}_{\rm cdots} + x_n^2$$
```
显示：

$$f(x_1,x_2,\underbrace{\ldots}_{\rm ldots} ,x_n) = x_1^2 + x_2^2 + \underbrace{\cdots}_{\rm cdots} + x_n^2$$

## 矢量

使用 `\vec{矢量}`来自动产生一个矢量。也可以使用 `\overrightarrow`等命令自定义字母上方的符号。

例子：

```
$$\vec{a} \cdot \vec{b}=0$$
```

显示：


例子：

```
$$\overleftarrow{xy} \quad and \quad \overleftrightarrow{xy} \quad and \quad \overrightarrow{xy}$$
```

显示：
$$\overleftarrow{xy} \quad and \quad \overleftrightarrow{xy} \quad and \quad \overrightarrow{xy}$$

## 积分 

使用 \int_积分下限^积分上限 {被积表达式} 来输入一个积分。

例子：

```
$$\int_0^1 {x^2} \,{\rm d}x$$
```

显示：

$$\int_0^1 {x^2} \,{\rm d}x$$

## 字体

|输入|说明|显示|
|---|---|---|
|`\rm`|罗马体|$$\rm Sample$$|
|`\it`|意大利体|$$\it Sample $$|
|`\bf`|粗体|$$\bf Sample $$|
|`\sf`|等线体|$$\sf Sample $$|
|`\tt`|打字机体|$$\tt Sample $$|
|`\frak`|旧德式字体|$$\frak Sample $$|
|`\cal`|花体|$$\cal Sample $$|
|`\Bbb`|黑板粗体|$$\Bbb Sample $$|
|`\mit`|数学斜体|$$\mit Sample $$|
|\scr|手写体|$$\scr Sample$$|


## 积分

使用 `\int_积分下限^积分上限 {被积表达式}` 来输入一个积分。

例子：

```
$$\int_0^1 {x^2} \,{\rm d}x$$
```

显示
$$\int_0^1 {x^2} \,{\rm d}x$$

## 极限
使用\lim_{变量 \to 表达式} 表达式 来输入一个极限。如有需求，可以更改 \to 符号至任意符号。

例子：

```
$$ \lim_{n \to +\infty} \frac{1}{n(n+1)} \quad and \quad \lim_{x\leftarrow{示例}} \frac{1}{n(n+1)} $$
```
显示：

$$ \lim_{n \to +\infty} \frac{1}{n(n+1)} \quad and \quad \lim_{x\leftarrow{示例}} \frac{1}{n(n+1)} $$

## 参考
1. [Cmd Markdown 公式指导手册](https://www.zybuluo.com/codeep/note/163962)
2. [markdown中公式编辑教程](https://www.jianshu.com/p/25f0139637b7)

