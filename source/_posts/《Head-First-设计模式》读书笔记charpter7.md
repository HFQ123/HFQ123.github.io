---
title: 《Head-First-设计模式》读书笔记charpter7
date: 2020-04-20 22:40:09
tags: [设计模式,笔记]
category: 设计模式 
---

本篇博客是《Head First 设计模式》第七章的读书笔记。

分享主题是设计模式中的 「适配器模式(Adapter Pattern)」以及 「外观模式(Facade Pattern)」。



## 适配器模式

由于这两种设计模式都比较好理解，所以先直接从定义讲起。

### 定义

{% blockquote %} 适配器模式将一个类的接口，转换成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。{% endblockquote %} 

### 日常生活的类比

我们先用日常生活中例子类比来理解适配器模式。如下图，由于接口的差异，美国笔记本电脑插头无法直接插进欧洲插座获电，而使用了中间的接口转换器后，问题就解决了。而值得注意的是，从无法使用到可以使用，我们既没有改变插座，也没有改变插头，唯一的变化就是引入了接口转换器这一**适配器**。在这里，这个适配器的作用就是将现有的不符合标准的插头接口“转化”成目标接口，这也正是适配器模式做的全部工作。

![插头接口转换器的例子](/images/image-20200420230152018.png)

### 具体应用场景

如下图，系统中有两个接口:Duck和Turkey，它们都有fly()方法，这两个接口的差异在于Duck的quack()和Turkey接口的gobble()方法。MallardDuck和WildTurkey分别实现了Duck接口和Turkey接口。

![image-20200420231854471](/images/image-20200420231854471.png)

此外，在外部有个testDuck方法，它的参数是一个Duck对象，在方法体内分别调用了这个Duck对象的quack()方法和fly()方法，我们无需关注它的目的是什么，只需要关注要想调用这个方法，需要一个Duck类型的对象作为参数。

```java
   public static void testDuck(Duck duck){
        duck.quack();
        duck.fly();
    }
```

接下来看看下面这段代码，testDuck(wildTurkey)这行代码必然会报错，因为wildTurkey 不是 Duck类型的。

```java
    public static void main(String[] args) {
        System.out.println("A duck says:");
        Duck mallardDuck =new MallardDuck();
        testDuck(mallardDuck);  //这行代码可以正常执行

        System.out.println("A turkey says:");
        Turkey wildTurkey =new WildTurkey();
        testDuck(wildTurkey);   //这行代码必然会报错，因为wildTurkey 不是 Duck类型的。
    }
```

我们用这个例子来类比上文中插头接口转换器的例子，testDuck()这个方法就相当于上例中的插座，它要求一个Duck对象作为参数才可以正常使用（换句话说，它的标准是Duck接口），就相当于欧洲插座要求一个符合欧洲标准的插头才能正常使用，不符合testDuck()标准的wildTurkey对象就相当于上例中的美国插头（类比详见下图）。而如果我们要是想在不改动WildTurkey类代码的基础上，完成testDuck(wildTurkey)这条语句的功能，就要想办法构建一个上例中的适配器。

![类比插头接口转换器的例子](/images/image-20200421001232960.png)

构建TurkeyAdapter这个适配器：
```java
/**
 * @Created by hfq on 2020/4/21
 * @used to: 为Turkey接口构造适配器，以使得它符合Duck接口这个标准
 */
public class TurkeyAdapter implements Duck{   //适配器要实现目标接口Duck
    Turkey turkey;   //组合了一个Turkey接口的对象

    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }

    @Override
    public void quack() { //这里直接调用了turkey的gobble方法，当然，你也可以做点别的
        this.turkey.gobble();
    }

    @Override
    public void fly() {
        this.turkey.fly();
    }
}
```

测试代码：

```java
System.out.println("A turkey says:");
Turkey wildTurkey =new WildTurkey();
TurkeyAdapter wildTurkeyAdapter = new TurkeyAdapter(wildTurkey);
testDuck(wildTurkeyAdapter);   //完成了接口转化
```

至此，一个应用适配器模式的demo就写完了。引入了适配器后，在既不修改WildTurkey类代码、又不修改testDuck方法的代码的基础上，我们完成了原本不兼容的工作，看上去就像实现了将一个Turkey接口的对象转化成一个Duck接口的对象。

### UML图

适配器模式的UML建模图如下，为了便于理解，我结合刚才的例子添加了红字标注出上例中的各个部分分别代表了什么。适配器Adpter实现了目标接口，且其类中组合了一个被适配器Adptee的对象。

p.s 我本篇博客介绍的例子是采用对象组合的方式实现适配器模式，书中还给出了另外一种实现适配器模式的思路——多重继承，简而言之就是Adapter既继承Target，又继承Adaptee，由于java语法不支持多重继承，在此不做过多介绍。

![适配器模式的UML图](/images/image-20200421003327658.png)

以上就是适配器模式的全部内容了，是不是很简单！接下来一起来看看更简单的外观模式吧。



## 外观模式

### 定义

{% blockquote %} 外观模式提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用。{% endblockquote %} 

### 具体应用场景
书中给出的例子是家庭影院的例子：放映电影需要执行一系列任务，这些任务涉及到了多个对象（以及它们对应的多个方法），这导致了每次需要放映电影都要写上一长串代码。
![image-20200421005039865](/images/image-20200421005039865.png)

你需要的正是一个外观：有了外观模式，通过实现一个更合理的接口的外观类，你可以将一个复杂的子系统变的容易使用。所以我们引入一个外观类HomeTheaterFacade，这个类中组合了所有与放映电影相关的对象（爆米花机、投影仪等等），它提供了一些更加友好的方法（每个方法都是一个子系统）以供调用，调用方不用知道方法内部是如何做的，以放映电影为例，调用方只需要提供需要完成放映操作的对象实例化HomeTheaterFacade，再调用它的watchMovie()方法，引入外观模式后的类图如下所示：

![家庭影院外观类图](/images/image-20200421011820380.png)

放映电影的内部方法代码和调用放映电影方法的代码见下图：

![放映电影方法、调用放映电影方法的代码实现](/images/image-20200421011152136.png)



## 适配器模式和外观模式总结

适配器模式在不修改接口的代码的基础上，提供了将一个接口转成另一个接口的方法，以达到兼容。

外观模式定义了一个高层接口,让子系统更容易使用，接口简单对调用方更友好。