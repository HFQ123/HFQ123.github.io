---
title: 《Head-First-设计模式》读书笔记charpter5
date: 2020-04-18 22:19:15
tags: [设计模式,笔记]
category: 设计模式 
---

本篇博客是《Head First 设计模式》第四章的读书笔记。

分享主题是设计模式中的 「单件模式(Singleton Pattern)」。

单件模式（也常叫单例模式）可以说是最简单的设计模式了，然而具体实现的还是有些讲究的，本文梳理了四种单例模式的实现方法 。其中前三种是书上给出的，最后一种书上没有提到，以作补充。



### 单例模式的基本认识

- 定义：{% blockquote %}单件模式确保一个类只有一个实例，并提供一个全局访问点。{% endblockquote %} 

- 用来创建只能有一个实例的独一无二对象。

  比如说线程池、缓存、对话框、处理偏好设置、注册表、日志对象、打印机等设备的驱动程序的对象。

- 可以确保只有一个实例会被创建，单件模式给了我们一个全局的访问点，和全局变量一样方便，又没有全局变量的缺点。



### 单例模式的四种实现方法

#### 一、懒汉

- 实现

	- 私有的构造函数
	- 私有的静态变量表示唯一实例，不初始化
	- 公开的静态方法获取唯一实例

- 分析

	- 优：延迟实例化(lazy instantiaze)

	  类加载时不会初始化，用到了才会产生(实例化)

	- 缺：多线程可能引发创建多个实例(在不加synchronized关键字的情况下)
	- 改进：为getInstance()方法加上synchronized关键字

		- 引入了新的问题：降低了性能，为什么？

		  因为实际上只有第一次执行此方法时，才真正需要同步，一旦设置了instance变量，就不再需要同步这个方法了。所以说，之后每次调用这个方法，同步都是一种累赘。


```java
/**
 * @used to: 懒汉实现单例模式，两私有一公开
 */
public class Singleton {
    private static Singleton instance;

    private Singleton(){       

    }

    public synchronized static Singleton getInstance(){
        //如果不加synchronized关键字，多线程情况下，此处可能创建多个不同的实例对象
        if (instance ==null)
            instance =new Singleton();
        return instance;
    }
}
```

#### 二、饿汉

饿汉单例模式在类加载时就完成了初始化，所以类加载较慢，但获取对象的速度较快。

- 实现

	- 私有的构造函数
	- 私有的静态变量表示唯一实例，初始化
	- 公开的静态方法获取唯一实例

- 分析

	- 与懒汉比较：饿汉在类加载时就完成了初始化，所以类加载较慢，但获取对象的速度较快
	- 优：不会有多线程导致创建了多个实例的问题
	- 缺：即使这个单例没有用到也会被创建，而且在类加载之后就被创建，内存就被浪费了。

```java
/**
 * @used to: 饿汉实现单例模式
 */
public class Singleton2 {
    private static Singleton2 instance = new Singleton2();  //类加载时完成初始化

    private Singleton2(){

    }

    public static Singleton2 getInstance(){
        return instance;
    }
}
```



#### 三、双重校验锁DCL【推荐】 
- 目标：是对懒汉中同步性能的优化，实现有更高同步性能的延迟加载。
- 实现

	- private static volatile Singleton instance

		- 为什么要volatile？
	- if (instance == null) 校验1
	- synchronized (Singleton.class)
	- if (instance == null) 校验2
- 分析

```java
/**
 * @used to: DCL（双重校验锁）实现单例模式  核心：volatile和两次校验
 */
public class Singleton3 {
    private static volatile Singleton3 instance;

    private Singleton3(){

    }

    public Singleton3 getSingleton(){
        if (instance == null){  //校验1,仅当singleton为空会进行同步。
            synchronized (Singleton3.class){
                if(instance == null){ //校验2，防止多线程中，不同线程执行到校验1，然后依次获得多个实例。
                    instance = new Singleton3();
                }
            }
        }
        return instance;
    }
}
```



#### 四、静态内部类【推荐】

  书上没给出这种方法，是从其他博客中学习到的。

```java
/**
 * @used to:静态内部类实现单例模式
 */
public class Singleton4 {

    private static class SingletonHolder{  //静态内部类
        private static Singleton4 instance = new Singleton4();
    }

    private Singleton4(){

    }

    public static Singleton4 getInstance(){
        return SingletonHolder.instance;
    }
}
```

- 分析
  - 它与饿汉模式一样，也是利用了类加载机制，因此不存在多线程并发的问题。
  - 不一样的是，它是在内部类里面去创建对象实例。
  - 这样的话，只要应用中不使用内部类，JVM就不会去加载这个单例类，也就不会创建单例对象，从而实现懒汉式的延迟加载。也就是说这种方式可以同时保证延迟加载和线程安全。



### 总结

评价单例模式的四种实现方法主要从以下两个角度考虑：

是否延迟加载((lazy instantiaze)) ？

怎么保证线程安全，在保证线程安全的情况下，性能如何 ？
