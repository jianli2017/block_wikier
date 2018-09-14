---
title: IO库
date: 2018-08-27 12:07:12
tags: IO库
categories: C++
toc: true
---

主要内容

1. IO类
2. 文件输入输出
3. string流

<!--more-->

## IO类

### IO对象无拷贝或赋值

进行IO操作的函数通常以引用方式传递、返回流。读写一个IO对象会改变其状态，因此，传递和返回的引用不能是const的。

### 条件状态

|状态|说明|
|---|----|
|strm::iostate|strm是一种IO类型，iostate是一种机器相关的类型，提供了表达条件状态的完整功能|
|strm::badbit|指出输出流已崩溃|
|strm::faillbit|一个IO操作失败|
|strm::eofbit| 流达到了文件结尾|
|strm::goodbit|流未处于错误状态，此值保证为0|
|s.eof()|若流s的eofbit置位，则返回true|
|s.fail()|若流s的failbit置位，则返回true|
|s.bad()|若流s的badbit置位，则返回true|
|s.good()|若流处于有效状态，则返回true|
|s.clear()|将流的所有状态复位|
|s.clear(flags)|根据给定的flags标志位，将流s中对应的条件状态位复位|
|s.setstate(flags)将流s中对应条件状态位置位||
|s.rdstate()|返回流s的当前条件状态|

一个流一旦发生错误状态，其后续的IO操作都失败。只有流处于无错状态是，我们才可以从它读取数据、写入数据。

属性缓存区：

1. endl， 换行并刷新缓冲区
2. flush ，刷新缓存区，不输出任何额为的字符
3. ends，向缓冲区插入一个空字符，然后刷新缓冲区

unitbuf 操纵符：如果每次输出操作后，都属性缓存区，我们可以使用unitbuf操纵符。

如果程序崩溃，输出缓冲区不会被刷新。

tie： 如果本对象关联到一个输出流，则返回的就是这个流的指针，如果对象未关联到流，则返回空指针。tie的第二个版本接受一个指向ostream的指针，将自己关联到此ostream。

## 文件输入输出

|操作|说明|
|---|---|
|fstream fstrm| 创建一个未绑定的文件流 |
|fstream fstrm(s)| 创建一个fstream，并打开名为s的文件。 s可以是string类型，或者是一个指向C风格字符串类型|
|fstream fstrm(s,mode)| 安装mode打开文件|
|fstrm.open(s)| 打开名为s的文件，并将文件与fstrm绑定，返回void|
|fstrm.close()|关闭于fstrm绑定的文件，返回void|
|fstrm.is_open|返回一个bool，指出与fstrm关联的文件是否成功打开且尚未关闭|

创建文件流对象时，我们可以提供文件名，如果提供了一个文件名，则open自动被调用。

### 文件模式

|文件模式|说明|
|---|---|
|in| 以读方式打开 |
| out | 以写方式打开|
| app| 每次写操作前均定位到文件末尾|
| ate| 打开文件后，立即定位到文件末尾|
| trunc| 截断文件|
| binary| 以二进制方式进行IO|

## stirng流

|操作| 说明|
|---|---|
|sstream strm| strm是未绑定的stringstream对象|
|sstram strm(s)| strm 是一个sstream对象，保存string的一个拷贝，此构造函数是explicit的|
|strm.str()| 返回strm中保存的string的拷贝|
|strm.str(s)|将string s 拷贝到strm中，返回void|


