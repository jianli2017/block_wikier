---
title: NSURL记录
date: 2019-05-10 15:35:49
tags: NSURL
categories: foundation
toc: true
---


<!--more-->



## url是什么？

url是（ Uniform Resource Locator ）统一资源定位符的缩写。

## 完整格式&语法

```
scheme:[//[user[:password]@]host[:port]][/path][?query][#fragment]
```

1. scheme: 传送协议。
2. 层级URL标记符号(为[//],固定不变)
3. user、password：访问资源需要的凭证信息（可省略）
4. host：服务器。（通常为域名，有时为IP地址）
5. port：端口号。（以数字方式表示，若为HTTP的默认值“:80”可省略）
6. path：路径。（以“/”字符区别路径中的每一个目录名称）
7. query：查询。（GET模式的窗体参数，以“?”字符为起点，每个参数以“&”隔开，再以“=”分开参数名称与数据，通常以UTF8的URL编码，避开字符冲突的问题）
8. fragment：片段。以“#”字符为起点

--------------------- 

## NSURLComponents

* Accessing Components in Native Format
   
   包括：fragment、host、path、query、queryItems、scheme等
   
* Accessing Components in URL-Encoded Format

   包括：percentEncodedFragment、percentEncodedHost等
 
* Locating Components in the URL String Representation

  包括：rangeOfFragment、rangeOfHost等










