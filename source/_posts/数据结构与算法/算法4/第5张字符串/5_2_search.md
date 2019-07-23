---
title: 单词查找数
date: 2019-05-30 10:07:12
tags: 单词查找数
categories: 算法4
toc: true
---

[算法4 官网地址](https://algs4.cs.princeton.edu/code/javadoc/)

## 单词查找树

每个节点都有R条连接，其中R为字母表的大小

每个键所关联的值保存在该键的最后一个字母所对应的节点中。值为空的节点在符号表中没有对应的键，他们的存在是为了简化单词查找树中的查找操作。

### 查找

以被查找的键中的字符为导向，单词查找树中的每个节点都包含了下一个可能出现的所有字符的链接。

查找结果：

1. 尾字符对应的节点中的值非空，查找一次命中，键所对应的值就是尾字符所对应的节点中保存的值。
2. 键的尾字符所对应的节点中值为空，未命中，符号表中不存在被查找的键
3. 查找结束于一条空链接，这也是一次未命中的查找

查找过程就是在单词查找树中从根节点开始检查某条路径上所有节点。

### 插入

1. 在到达键尾部前就遇到了一个空链接，需要为键中还未被检查的每个字符创建一个对应的节点，并将键对应的值保存在最后一个节点中。
2. 在遇到空链接前就到达了健的尾部，将该节点中值设为键对应的值。

### 节点的表示

每个节点都含有一个值和26个链接

在单词查找树中，键是由从根节点到含有非空值的节点的路径隐式表示的。数据节点并不会保存字符串或字符，它保存了链接数组和值。基于含有R个字符的字母表的单词查找数称为**R向单词查找树**。

### 大小

size的延时实现：

```
private int size()
{	
	return size(root);
}
private int size(Node x)
{
	if(null == x) return 0;
	
	int cnt = 0;
	if(x.val) ++cnt;
	for(int i =0 ;i<R, ++i)
	{
		cnt += size(next[i]);
	}
	return cnt;
}
```

### get

```
private Node get(Node x,String key, int d){
	
	if(x == null) return null;
	if(d== key.length()) return x;
	
	//开始递归
	char c = key.charAt(d++);
	Node next = x.next[c];
	return get(next, key,d)
}

public Value get(String key)
{
	Node node = get(root,key,0);
	if(node == null) return null;
	return node.val;
}
```

### put

```
private Node put(Node x, String key, Value val, int d) {
	if(x == null) x= new Node();
	if(d == key.length()) {x.val = val, return x;}
	char c = key.charAt(d);
	x.next[c] = put(x.next[c],key,val,d+1);
	return x;
}

public void put(String key, Value val) {
	root = put(root,key,val,0);
}
```

### 查找所有键

```
private void collect(Node x, String pre, Queue<String>) {
	if(x== null) return;
	if(x.val != null) q.enqueue(pre);
	for(char c=0; c<R; c++) {
		collect(x.next[c],pre+c,q);
	}
}
public Iterable<String> keysWithPrefix(String pre) {
	Queue<String>  q = new Queue<Sting>();
	collect(get(root,pre,0),pre,q);
	return q;
}
```

### 通配符匹配



