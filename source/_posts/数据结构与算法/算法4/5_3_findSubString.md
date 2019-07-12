---
title: 子字符串查找
date: 2019-05-30 10:07:12
tags: 子字符串查找
categories: 算法4
toc: true
---

[算法4 官网地址](https://algs4.cs.princeton.edu/code/javadoc/)

## 暴力子字符串查找算法

```
public static int search(String pat,String txt)
{
	int M = pat.length();
	int N = txt.length();
	for(int i=0;i<=N-M;j++)
	{
		int j = 0;
		for(int j=0; j< M ;j++)
		{
			if(txt.charAt(i+j) != pat.charAt(j))
				break;
		}
		if(j==M) return i;
	}
	return N;
}
```

在最坏情况下，暴力子字符串查找算法在长度为N的文本中查找长度为M的模式需要NM次比较。

### KMP算法

主要思想：提前判断如何重新开始查找。

