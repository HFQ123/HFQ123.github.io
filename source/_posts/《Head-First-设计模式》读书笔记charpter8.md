---
title: 《Head-First-设计模式》读书笔记charpter8
date: 2020-04-21 14:52:00
tags: [设计模式,笔记]
category: 设计模式 
---

本篇博客是《Head First 设计模式》第八章的读书笔记。

分享主题是设计模式中的 「模板方法模式(Template Method Pattern)」。

## 定义

{% blockquote %} 模板方法模式在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。{% endblockquote %} 

## 具体应用场景

从饮品店制作咖啡和茶的制作方法讲起，咖啡和茶的制作流程如下图所示。

![咖啡和茶的制作流程](/images/image-20200421145905200.png)

接下来用代码表示咖啡的制作过程：

```java
/**
 * @Created by hfq on 2020/4/21
 * @used to: 咖啡的制作流程
 */
public class Coffee {
    public void prepareRecipe(){  //冲泡咖啡
        boilWater();
        brewCoffeeGrinds();
        pourInCup();
        addSugarAndMilk();
    }
    public void boilWater(){
        System.out.println("Bolling water");
    }
    public void brewCoffeeGrinds(){
        System.out.println("Dripping Coffee through filter");
    }
    public void pourInCup(){
        System.out.println("Pouring into cup");
    }
    public void addSugarAndMilk(){
        System.out.println("Adding sugar and milk");
    }

}
```

同样的，茶的制作过程代码如下：

```java
/**
 * @Created by hfq on 2020/4/21
 * @used to: 茶的制作过程
 */
public class Tea {
    public void prepareRecipe(){  //冲泡茶
        boilWater();
        steepTeaBag();
        pourInCup();
        addLemon();
    }
    public void boilWater(){
        System.out.println("Bolling water");
    }
    public void steepTeaBag(){
        System.out.println("Steeping the tea");
    }
    public void pourInCup(){
        System.out.println("Pouring into cup");
    }
    public void addLemon(){
        System.out.println("Adding lemon");
    }
}
```

可以发现，boilWater()和pourInCup()方法的代码重复了。究其原因，这是因为咖啡和茶的制作流程大致上一样，它们的制作过程都可以用下图的算法表示，1、3步骤是一样的，2、4步骤有差异，所以我们想到用继承的方法重用1、3步骤的代码，将实现1、3步骤的方法抽取到一个父类中，而2、4步骤这两个有差异的步骤在咖啡子类和茶子类中实现。
![制作咖啡和制作茶的相同和不同的步骤](/images/image-20200421150454878.png)

基于此想法，我们设置一个咖啡因饮料的父类：

```java
/**
 * @Created by hfq on 2020/4/21
 * @used to: 咖啡因饮料的父类，是一个抽象类
 */
abstract class CaffeineBeverage{
    final void prepareRecipe(){ //为了防止子类修改制作流程，使用final关键字修饰
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }
    void boilWater(){   //步骤1，共同的操作，由父类实现
        System.out.println("Bolling water");
    }

    abstract void brew();  //步骤2，冲泡，由于不同饮料实现有差异，由子类实现
    
    void pourInCup(){  //步骤3，共同的操作，由父类实现
        System.out.println("Pouring into cup");
    }
   
    abstract void addCondiments(); //步骤4，添加调料，由于不同饮料实现有差异，由子类实现
}
```

现在咖啡子类和茶子类的代码就变成了：

```java
/**
 * @Created by hfq on 2020/4/21
 * @used to: 咖啡的冲泡流程
 */
public class Coffee extends CaffeineBeverage{
    @Override
    void brew() {
        System.out.println("Dripping Coffee through filter");
    }

    @Override
    void addCondiments() {
        System.out.println("Adding sugar and milk");
    }
}

/**
 * @Created by hfq on 2020/4/21
 * @used to: 茶的制作过程
 */
public class Tea extends CaffeineBeverage{
    @Override
    void brew() {
        System.out.println("Steeping the tea");
    }

    @Override
    void addCondiments() {
        System.out.println("Adding Lemon");
    }
}
```

这样，我们就完成了共同代码（prepareRecipe()、boilWater()、pourInCup()三个方法）的重用。其中，prepareRecipe()方法就叫做**模板方法**。它作为制作咖啡因饮料算法的模板，在这个模板中，每个步骤都被一个方法代表了，其中，制作不同的咖啡因饮料有**共同的步骤**：就是上文说的步骤1和步骤3，它们可以直接在父类中实现(分别对应于boilWater()和pourInCup()方法)。制作不同的咖啡因饮料也有**不同的步骤**：步骤2和步骤4，由于子类有不同的实现，所以父类中不实现，设置为抽象方法，由子类实现，这就是模板方法模式中说的“将一些步骤延迟到子类中”。

如果有新的制作流程一致的咖啡因饮料加入，只需要继承父类，并负责实现父类的抽象方法（表示制作步骤中不同的部分）。

## UML图

模板方法模式的UML建模图如下：

![模板方法模式的UML建模图](/images/image-20200421155001199.png)

## 使用钩子(Hook)

在上例中引入一个新的需求：客户可以自行选择是否需要在饮料中加入调料（也就是选择是否执行制作饮料中的步骤4），这时模板方法的钩子就起作用了。

做法很简单：在模板方法中，加入判断，如果判断为真，则执行步骤4，修改后的模板方法类代码如下：

```java
abstract class CaffeineBeverage{
    final void prepareRecipe(){  //为了防止子类修改制作流程，使用final关键字修饰
        boilWater();
        brew();
        pourInCup();
        if(customerWantsCondimentsHook()){
            addCondiments();
        }
    }
	...省略了相同的代码.... 
    boolean customerWantsCondimentsHook(){  //这是一个钩子，子类可以选择是否覆盖
        return true;
    }
}
```

这样一来，在子类中可以覆盖customerWantsCondimentsHook()方法来自行选择是否需要执行步骤4，它还可以更灵活地实现钩子：比如接受用户的输入，据此判断是否需要执行该步骤。

在这里，钩子作为条件控制，影响了抽象类的算法流程。

## 好莱坞原则

书中本章引入了一个新的设计原则，称为好莱坞原则：

{% blockquote %} **好莱坞原则（Hollywood Principle）：别打电话给我们，有事我会打电话给你。**{% endblockquote %} 

好莱坞原则和模板方法之间的连接比较明显，当我们设计模板方法模式时，我们告诉子类，不要调用我们，我们会调用你。让我们再看一下的咖啡因饮料的设计类图：

![好莱坞原则和模板方法](/images/image-20200421163742368.png)

## Java API中的模板方法

书中指出，模板方法模式很常见，因为对于创建框架来说，这个模式简直棒极了。由框架控制如何做事情，而由使用框架的人指定框架算法中的每个步骤的细节。另外，模板方法有多种实现，不一定看起来和上例讲述的设计一致。

书中给出三个java api中的模板方法示例：

+ 类实现comparable接口后可以用Collections.sort 

+ Swing
+ Applet

第一个示例归结于模板方法我觉得似乎有些牵强，后两个示例平时开发没使用过，不熟悉。所以讲讲另外一个比较熟悉的示例吧：Servlet类的service()方法就是一个模板方法，定义了算法骨架，doGet()、doPost()方法是这个算法骨架中的具体步骤，子类可以实现自己的doGet()方法和doPost()方法，这样做子类自己的doGet()、doPost()方法就”挂进"了父类service()方法这一模板方法。 与我们上文咖啡因饮料示例不同的是，子类具有不同实现的方法在父类是抽象的，子类必须实现，而Servlet中的doGet()和doPost()有默认实现。

![Servlet中的模板方法](/images/image-20200421175736238.png)



Servlet中的模板方法具体实现参考[这里](https://blog.csdn.net/y_love_f/article/details/38302413)。

## 总结

+ 模板方法模式实际上是封装了一个固定流程，该流程由几个步骤组成，具体步骤可以由子类进行不同实现，从而让固定的流程产生不同的结果。
+ 模板方法封装了有固定结构的算法块，模板类中可以有各个子类具有相同实现的方法，也可以有各个子类具有不同实现的方法，具有相同实现的方法可以直接在父类（模板方法所在的类）中实现，而各个子类具有不同实现的方法由每个子类自行实现，这样做，子类实现的方法就“挂接”进模板方法里。