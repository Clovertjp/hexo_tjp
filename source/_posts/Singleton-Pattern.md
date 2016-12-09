---
title: Singleton Pattern
date: 2016-12-07 17:10:27
description: Singleton Pattern
tags: [Pattern]
categories: Java
---
单例分为懒汉模式和饿汉模式：

## 饿汉模式

``` bash
package com.tjp.test;  
  
public class SingletonClass {  
      
     private static final SingletonClass instance = new SingletonClass();   
      
      public static SingletonClass getInstance() {   
        return instance;   
      }   
        
      private SingletonClass() {   
           
      }   
}  
```

## 懒汉模式

本文主要分析懒汉模式相关。
先说下以前面试时让我写单例模式我的写法：


``` bash
package com.tjp.test;  
  
public class SingletonClass {  
      
     private static SingletonClass instance = null;   
      
      public static synchronized SingletonClass getInstance() {   
          if(instance==null)  
              instance=new SingletonClass();  
        return instance;   
      }   
        
      private SingletonClass() {   
           
      }   
}  
```
先看下上面写得代码，这个代码基本没有问题，并且还考虑到了线程的问题，但是如果要我打分的话这段代码顶多60分刚及格。那就先分析一下这段代码，这段代码很简单，而且也能在多线程的情况下保证其正确性，但有个致命缺点就是性能，多个线程同时调用这个函数，那这个的性能就可想而知了，每个线程必须等待一个线程执行完才可以进入，这样性能肯定不能满足需求。
再来看下下面的代码：


``` bash
package com.tjp.test;  
  
public class SingletonClass {  
      
     private static SingletonClass instance = null;   
       
     private int testNum;  
      
      public static SingletonClass getInstance() {   
        if(instance==null)  
        {  
            synchronized (SingletonClass.class) {  
                if(instance==null)  
                {  
                    instance=new SingletonClass();  
                }  
            }  
        }  
        return instance;   
      }   
        
      private SingletonClass() {   
          testNum=100;  
      }   
        
      public int getTestNum() {  
        return testNum;  
    }  
        
}  
```
这段代码将同步块加到函数里，并且在进入同步块之前有个判断，这样当instance不为空的时候，无论有多少线程调用，效率都会非常高。但是这段代码是有问题的代码。
为了了解这段代码的问题所在，那就先分析下Java创建对象时候的内存情况。




instance=new SingletonClass(); 


这句话在这看似是一句，其实在内存中可以分为一下几步：

1.在堆中创建一个SingletonClass的类对象。

2.调用SingletonClass()函数，来初始化。

3.将类对象地址传递给对象的引用。

还是没有懂得可看这：

这个讲的[this逃逸](http://coolxing.iteye.com/blog/1464501)(我感觉和我说的这个问题属于一个问题，嘻嘻)

还有这里所讲的[创建对象及初始化过程](http://zhangjunhd.blog.51cto.com/113473/17124/)

通过这几个步骤，很容易就可以分析出上面的问题所在。

假如第一个线程进来，此时还未创建对象，instance的值为空，进入同步块，此时执行new对象。而执行顺序是1->3->2。在执行完3后，另一个线程进来，此时instance的值已经不为空了，直接返回，之后这个线程来获取testNum，那么获取的值为0.原因是上一个线程还没有执行初始化，这样获取的值就会错误，程序就会出现一些意想不到的bug。

为了解决上面所说的这些问题，比较完美的单例模式(我自己认为的，其他更好的还没看到。。我见识比较少。。要是有更好的欢迎提出)：


``` bash
package com.tjp.test;  
  
public class SingletonClass {  
      
     private static volatile SingletonClass instance = null;   
       
     private int testNum;  
      
      public static SingletonClass getInstance() {   
        if(instance==null)  
        {  
            synchronized (SingletonClass.class) {  
                if(instance==null)  
                {  
                    instance=new SingletonClass();  
                }  
            }  
        }  
        return instance;   
      }   
        
      private SingletonClass() {   
          testNum=100;  
      }   
        
      public int getTestNum() {  
        return testNum;  
    }  
        
}  
```
不仔细的人肯定认为这段代码和上面那段一样，注意看变量声明那块，这段代码在声明变量的时候加了个关键字volatile。那么现在就来说说这个关键字。
在说这个关键字之前得先说下Java的重排序相关(好吧~~这篇说的知识点有点多。。而且很枯燥。。)。

简单来说，重排序的意思就是你在执行Java代码的时候，程序的运行顺序可能和你所写顺序不同。举个例子：

``` bash
double pi = 3.14; //A

double r = 1.0; //B

double s=pi * r * r; //C
```

一般认为，我书写的是A->B->C，那么执行的顺序也应该是A->B->C.可是由于重排序的关系执行顺序可能会发生改变，变为B->A->C。因为C和A，B有数据依赖关系，所以C的结果是不会变的，而A B之间不存在数据依赖，他们的执行顺便有可能会发生重排序。对于这个例子来说，无论怎么变，都不会影响数据结果。那么来看看这个例子：


``` bash
package com.tjp.test;  
  
public class ReorderExample {  
    int a=0;  
    boolean flag=false;  
      
    public void write()  
    {  
        a=2;    //1  
        flag=true;    //2  
    }  
      
    public void read()  
    {  
        if(flag)    //3  
        {  
            int i=a*a;  //4  
        }  
    }  
  
}  
```
如果有两个线程，一个执行write，一个执行read。问i的值会是多少？
首先可以肯定的是值为4.那大家想想有没有可能值为0呢？

根据前面重排序的理论，a和flag没有数据依赖，那么他们在运行时可能会发生重排序，那么就会变为一个线程运行2->1 ，在这个线程运行2的时候另一个运行 3->4 ，也就是运行顺序是2->3->4->1.

解决方法如下：


``` bash
package com.tjp.test;  
  
public class ReorderExample {  
    int a=0;  
    volatile boolean flag=false;  
      
    public void write()  
    {  
        a=2;    //1  
        flag=true;    //2  
    }  
      
    public void read()  
    {  
        if(flag)    //3  
        {  
            int i=a*a;  //4  
        }  
    }  
  
}  
```
只要在flag申明的时候加个volatile就可解决。
那么现在就先分析这段代码。

根据happens-before原则，

根据程序次序：1 happens-before 2, 3 happens-before 4

根据volatile规则：2 happens-befoe 3

根据传递原则：1 happens-before 4

这样就不会出现上面说的：2->3->4->1这样的结果了。

这次再来分析一下上面所说的单例模式。

给instance加上volatile后，会阻止instance重排序，根据规则（对于volatile的读，总是能看到对volatile的最后写入），这样我们new instance的时候，如果instance未写入，其他线程都会取得instance为空，当instance写入后，由于volatile阻止了重排序，这样写入后初始化也完成了，再次获取初始化的值也就是正确的了，也就不会有上面那种错误了。

按道理文章应该到此结束，单例的相关已经分析完成，但是对于volatile的理解可能还是模棱两可，而且在程序中很容易理解错误。

很多人认为我把一个变量加上volatile后就能保证多线程同步，该变量的操作就变成了原子操作，这个想法绝对不对！！！

首先，volatile不能保证同步，他可以阻止重排序，每次读的值是上次赋的值，比如：

a初始值为0,100个线程去操作。

a=a+1;

问a的值是多少？

答案是1到100都可能。有人会说，不是说volatile读的是上次的值嘛，为什么还会有这种情况。对volatile是读取的是上次的值，可是我们要得分析一下a=a+1的操作。从上层来看这就是一步加法操作，可是电脑不认，我们得从汇编角度来看这段代码，汇编寄存器记号什么的我还给老师了，那就用口述吧。

1，获取a的值，将a放到寄存器中；

2，将a自加一；

3，将结果放回变量地址；

这个三步，假如A线程获得a的值，++后，还未放回去，B线程再来获取a的值时，获取的是以前的值，这时a的值就会发生错误。当写入后下一个线程过来读取的就是新的a的值。如此继续下去，a的值肯定会和所想值不一样。

我想现在应该比以前更了解volatile的作用了。

好吧，稍微总结下：

程序中使用volatile，可以防止重排序，这样可以免去重排序所带来的未知错误。

volatile没办法保证同步问题，要解决同步相关还得加synchronized或者lock。

以上文章是自己的认识，有什么不对请指出，谢谢！

更多关于多线程的知识请参考：

“深入理解java内存模型”和“JSR133”这两本书