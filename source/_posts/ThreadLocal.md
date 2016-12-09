---
title: ThreadLocal
description: Java API 
date: 2016-12-07 17:23:50
tags: [Java]
categories: Java
---


## Java线程
一说起Java线程一般会想到两种方法来写，一种是继承Thread，另一种是实现Runnable接口，对于第一种一般比较少用，用得最多的是第二种。原因就是Java是单继承。而对于实现接口这个方法，会很容易遇到一个问题，那么先看这段代码

``` bash
public class CountAdd implements Runnable {  
      
    private int count=0;  
  
    public void run() {  
        // TODO Auto-generated method stub  
        System.out.println("int count : "+(count++));  
  
    }  
  
}  
```
测试代码：

``` bash
CountAdd r1=new CountAdd();  
          
for(int i=0;i<10;i++)  
{  
    new Thread(r1).start();  
}  
```
我在很长一段时间内都认为他的运行结果是输出全为0，
但实际结果却是：

![Image](/image/1207/3.png)

原因是：我们只申请了一个CountAdd，他的数据是共享的，每次是不同线程对count进行操作，但是他们操作的是同一块内存，所以每次都会变化。

## ThreadLocal引入
由于上面说的，这样就得引入ThreadLocal这样一个概念，先来看一段几乎一样的代码

``` bash
package com.tjp.runnable.test;  
  
public class CountAddUseThreadLocal implements Runnable {  
      
    private static ThreadLocal<Integer> countThread=new ThreadLocal<Integer>();  
  
    public void run() {  
        // TODO Auto-generated method stub  
        Integer count=countThread.get();  
        if(count==null)  
        {  
            countThread.set(0);  
            System.out.println("ThreadLoacl count 为空");  
        }else  
        {  
            System.out.println("ThreadLoacl count : "+(count++));  
            countThread.set(count);  
        }  
  
    }  
  
}  
```
测试代码

CountAddUseThreadLocal r2=new CountAddUseThreadLocal();  
for(int i=0;i<10;i++)  
{  
    new Thread(r2).start();  
} 


他的运行结果：

![Image](/image/1207/4.png)

为什么会这样呢？我们可以看看ThreadLocal的源代码，在源代码里有这样几句话：


``` bash
Thread t = Thread.currentThread();  
ThreadLocalMap map = getMap(t);  
if (map != null)  
    map.set(this, value);  
else  
    createMap(t, value);  
```
可以看出，他是以map进行存储数据的，而map的key值是当前线程的id，每个线程的id号不一样，这样操纵的count值也就不一样，这也就解释为什么我每次set后，用另一个线程去取还是为空的原因。
