---
title: Java基础查漏补缺之反射
date: 2020-05-12 16:17:45
tags: Java基础
---

## 反射的原理

反射是什么?

> JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

为什么要学习反射？

>  “反射是框架设计的灵魂。”

大三上初学Spring框架时，就听过这句话，看了介绍反射的知识点云里雾里，完全是硬着头皮跟着做，现在学了JVM类加载过程之后，这块知识还是比较好理解的。



首先回顾一下类加载过程：加载->连接->初始化

其中加载这一步骤要做三件事：

+ 通过类的全限定名获取此类的二进制字节流

+ 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
+ 在堆内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的访问入口。

我们需要重点关注，**生成的Class对象描述了这个对象的所有信息，比如都有哪些构造方法，都有哪些成员方法，都有哪些字段等**，在反射中就要利用到Class对象。



## 反射的使用

使用反射要关注以下2个重点问题。

### 如何获取类的Class对象?

有以下3种获取类的Class对象的方法:

```java
        //方法1：调用类的一个实例的getClass方法
        Dog dog = new Dog();
        Class<Dog> dogClass = dog.getClass();

        //方法2：类的字面量
        Class<Dog> dogClass1 = Dog.class;
        
        //方法3：推荐！利用Class类的静态方法forName()，其参数是需要获取的类的全限定类名，需处理ClassNotFoundException异常
        try {
            Class<Dog> dogClass2 = (Class<Dog>) Class.forName("reflect.Dog");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
```

对这三种获取Class对象的方式的理解：每个对象的**对象头**里存了该对象所在类的Class对象的引用（这块知识点在JVM中有详细介绍），所以从原理上看，可以直接根据对象获取Class对象，如方法1，实际上getClass()是Object类的方法,所以自然所有对象都会有这个方法了。



### 获得Class对象之后有什么用？

当我们获得了想要操作的类的 Class 对象后，可以通过 Class 类中的方法获取该类中的构造方法、成员变量、方法等信息，类的这一个个组成部分会被映射成一个个对象,比如说利用反射获得类的构造器对象，然后就可以利用这个类的构造器对象构造一个这个类的实例，总的来说,利用反射能做的事情有：

+ 获取构造方法，用此构造方法创建对象
  + aClass.getConstructor();
  + newInstance() 
+ 获取成员变量
  + getFiled(“filedName”)
  + getFileds()
  + getDeclaredFileds()
+ 获取成员方法，并执行此成员方法
  + getMethod()
  + getDeclaredMethod()
  + invoke()



## 反射的应用

+ 修改私有成员变量的值
+ 执行私有成员方法
+ 反射妙用：通过反射运行配置文件内容

我利用反射+代理模式写了个demo，用来计算任意类的无参方法的执行时间。

```java
package reflect;

import java.lang.reflect.Method;

/**
 * @Created by hfq on 2020/5/12
 * @used to:  用于计算任意类的任意无参方法的执行时间
 */
public class TimeProxy {
    String className;
    String methodName;

    public TimeProxy(){

    }

    public TimeProxy(String className, String methodName) {
        this.className = className;
        this.methodName = methodName;
    }

    public Long exeucte() throws Exception {
        Class<?> aClass = Class.forName(className);
        Long start = System.currentTimeMillis();
        Method method = aClass.getDeclaredMethod(methodName,null);
        method.setAccessible(true);
        method.invoke(aClass.newInstance(),null);
        return System.currentTimeMillis()-start;
    }

    public static void main(String[] args) throws Exception {


        TimeProxy timeProxy = new TimeProxy("reflect.Person","eat");
        System.out.println("人吃饭，吃了:"+timeProxy.exeucte()+" ms");
        TimeProxy timeProxy2 = new TimeProxy("reflect.Brid","fly");
        System.out.println("鸟飞行，飞了:"+timeProxy2.exeucte()+" ms");


//        测试系统Math类的random()方法，报java.lang.IllegalAccessException
//        TimeProxy  println = new TimeProxy("java.lang.Math", "random");   
//        System.out.println("Math执行random方法，执行了"+println.exeucte()+" ms");
    }
}
class Person{
    private void eat() throws InterruptedException {
        Thread.sleep(1000);  //人吃饭三秒
    }
}

class Brid{
    private void fly() throws InterruptedException {
        Thread.sleep(3000);  //鸟飞翔三秒
    }
}

/*
打印结果如下：
人吃饭，吃了:1001 ms
鸟飞行，飞了:3001 ms

Process finished with exit code 0
*/
```

更多内容我就不在此展开了，参考如下文章，案例比较全面：

原文链接：https://blog.csdn.net/sinat_38259539/article/details/71799078

