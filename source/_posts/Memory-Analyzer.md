---
title: Memory Analyzer
description: Java tools
date: 2016-12-07 17:44:41
tags: [tool]
categories: Java
---
## 安装 Memory Analyzer 
1. Eclipse里help里有个install new software。
2. works里根据网站：http://www.eclipse.org/mat/downloads.php 里的update site 的地址输入。之后next即可。

## 使用 Memory Analyzer
1. ![Image](/image/1207/5.png)
2. 在Arguments里的VM arguments 里输入 ：-XX:+HeapDumpOnOutOfMemoryError
意思是：JVM遇到OutOfMemoryError时拍摄一个“堆转储快照”，并将其保存在一个文件中。
如果没有指定输出的路径（-XX:HeapDumpPath=E:\Java\dump）那么会生成到此项目的根目录下。
生成的文件后缀为 .hprof
3. Eclipse open file 打开这个文件就会显示出分析的图表 如图

![Image](/image/1207/6.png)

##  Memory Analyzer 说明

从上图可以看到它的大部分功能。
1. Histogram可以列出内存中的对象，对象的个数以及大小。
2. Dominator Tree可以列出那个线程，以及线程下面的那些对象占用的空间。
3. Top consumers通过图形列出最大的object。
4. Leak Suspects通过MA自动分析泄漏的原因。
这次重点是看Leak Suspects，点开后就能看到

![Image](/image/1207/7.png)

点 Detial就能看到实际的一些情况


## 测试代码

``` bash
List<OOMObject> list=new ArrayList<OOMObject>();
 
 while(true)
 {
 list.add(new OOMObject());
 }

 ```

内部类：
``` bash
static class OOMObject{}
```