---
title: NSScanner
date: 2019-05-13 09:35:49
tags: NSScanner
categories: foundation
toc: true
---


<!--more-->


A string parser that scans for substrings or characters in a character set, and for numeric values from decimal, hexadecimal, and floating-point representations.

NSScanner是一个string的解析器。


An NSScanner object interprets and converts the characters of an NSString object into number and string values. 

NSScanner对象将 NSString对象的字符 解析转化为 数字或string。

NSScanner是个类族。

初始化方法：

```
+ scannerWithString:
```

扫描字符或string

```
- scanCharactersFromSet:intoString:
```

扫描一个数字

```
- scanDecimal:
```



