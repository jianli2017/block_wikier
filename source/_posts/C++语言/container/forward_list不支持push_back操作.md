---
title: 为什么`forward_list`不支持`push_back`操作？
date: 2018-06-27 10:07:12
tags: forward_list
categories: container
toc: true
---

## 为什么forward\_list不支持push\_back操作？

由于forward_list是单向链表，所以我们如果想要访问尾元素，都要从首元素开始跌代，算法复杂度为O(n)。而对于list为双向链表，直接通过尾指针可以访问尾元素，实现在尾元素添加元素，函数复杂度为O(1)，而vector，string,deque,也可以通过尾指针来添加元素，函数复杂度为O(1)。

同样的原因也有forward\_list不支持pop\_back。 
由于类似原因(算法的复杂度)，有vector，string，不支持push\_front，pop\_front，但是通过insert，和erase操作仍然可以完成添加/删除首元素。

