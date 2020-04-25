---
title: 《Head-First-设计模式》读书笔记charpter3
date: 2020-04-18 10:12:57
tags: [设计模式,读书笔记]
category: 设计模式 
---

本篇博客是《Head First 设计模式》第三章的读书笔记。

分享主题是设计模式中的 「装饰者模式(Decorator Pattern)」。



### 应用背景——种类繁多的饮料的“类爆炸”

#### 背景
书中给出的例子是，一家饮品店的饮品种类繁多，且客户可以自行选择在饮品内加入收费的调料（调料当然也有多种）。即使是同一种的饮品，由于加入了不同的调料，价格也会有差异，如何计算饮品的价格就成了一个问题。如下图1，书中给出的第一个尝试是列举所有饮品的可能情况来构成饮品的所有子类，如HouseBlendWithSoy(表示带调料Soy的HouseBlend饮品类)，由于饮品和调料的组合方式很多，这势必会造成"类爆炸"。

![图1](/images/image-20200418103234825.png)

但凡有点开发经验的都不会像上面这样做，我们往往都能想到通过继承解决类爆炸的常规思路，而继承解决会带来一些问题，这就需要引出本文的主角装饰者模式了，一起来看看吧。
#### 通过继承解决
具体做法是：为饮品父类中添加一些布尔类型的成员变量(如milk、soy等,都是调料名)来表示是否有该调料，和它们对应的成员方法(如hasMilk())。在父类的计算饮品价格方法 cost()中，依次判断是否加入了各调料，也就是判断该调料的变量是否为true，如果是true就在饮品的基础上加上这种调料的价格。如此一来，针对一个带特定调料的饮品对象，只需要设置它的这些成员变量的状态，使用继承自父类的cost()方法就能计算价格了。
这种做法很简洁易懂，然而可拓展性不好：如果客户需要在饮品中加入双倍的某种调料，就要修改现有的代码：在饮品的父类中再加入多个成员变量(如milkNum,soyNum)来记录饮品的调料数目，同时，也会修改父类计算饮品价格的cost()方法，因为现在还要乘上调料的数量。

#### 通过装饰者模式解决

现在我们还不用着急知道什么是装饰者模式，只需要知道它的基本思路就是：用一种种调料层层“包装”饮品，然后逐级调用cost()方法来获得最终饮品的价格。

下图是用调料层层包装饮品的示例，模拟了加入了两种调料(Mocha和Whip)的DarkRoast饮品。

![图2](/images/image-20200418144735272.png)

我们姑且把调料叫做**装饰者**，把饮品叫做**被装饰者**。但是，这里有个小细节，调料既可以用来包装饮品（如图上红字标出的调料1对象包装了一个饮品对象），还可以包装其他调料（如图上红字标出的调料2包装了一个调料1对象），也就是说调料1在这里既作为最内层饮品的“装饰者”，又作为外层调料2的“被装饰者”。

为了标记每个装饰者包装的对象是谁，我们有必要在装饰者类（就是调料类）里设置个成员变量warppedObj——一个指向“被装饰者”的引用，那这个成员变量的类型应该是什么呢？这里有点绕，再次强调一下，调料2的“被装饰者”是调料1，其类型是调料，调料1的“被装饰者”是饮品，其类型是饮品，所以说这里怎么设置warppedObj的类型都不对：如果设置成饮品类，那么调料2就无法包装调料1，如果设置成调料类，那么调料1就无法包装饮品。也就是说如果要正确设定warppedObj类型以支持以上情形，就要完成调料类和饮品类的类型统一。书中的做法是让调料类继承饮品类，以达成类型的统一。

搞懂了这个，接下来的事情就好办了，如上图2，我们的目标是：计算有1份Whip、1份Mocha的DarkRoast饮品价格，做法就是先new一个Mocha对象包装DarkRoast对象记作obj1，在new一个Whip对象包装obj1。计算价格的时候调用最外层调料2对象的cost()方法，它的实现是调用其包装了的对象调料1的cost()方法获得调料1对象的价格，再加上自己的价格，而调料1的cost()实现又是调用内层饮品的cost()方法再加上自己的价格，以此类推，计算价格的过程就像上图2标出的6步流程。

如果这个饮品需要加入双倍的Whip调料，只需要再用一层Whip来包装，不用对其他代码改动。



### 装饰者模式

#### 装饰者模式思想

在给出装饰者的定义之前，先来谈谈其“主要指导思想”，装饰者模式的指导思想是组合而非继承（在策略模式中也提到过）、开放关闭原则。

+ **组合而非继承** 
    - 利用继承设计子类的行为，是在编译时静态决定的，而且所有的子类都会继承到相关同的行为。然而，如果通过组合的做法扩展对象的行为，就可以在运行时动态地进行拓展。
+ **开放关闭原则**
    - 含义：类应该对扩展开放，而对修改关闭。
    - 解读：在不修改任何底层代码的情况下，给你的对象赋予新的职责。

这两条是书中引出装饰者模式的思想，现在理解不了没关系，先通读下文，再来回顾装饰者模式是如何遵循这两条思想的。

####  装饰者模式定义

{% blockquote %}装饰者模式动态地将责任附加到对象上，若要拓展功能，装饰者模式提供了比继承更有弹性的替代方案。{% endblockquote %} 

#### 装饰者模式UML图

本案例中，使用装饰者模式后，uml图建模如下：


![image-20200418112721899](/images/image-20200418112721899.png)

饮品是**被装饰者**（书中也叫做组件Component），可以被多种调料“包装”，它始终在最内层。

调料是**装饰者**，用来”包装“饮品，但是需要注意的是，调料同时也可能作为**被装饰者**，因为它外层可能还有别的调料来包装它。

在uml图中，我们特别需要关注的是饮品类和调料类的关系：

+ 首先，调料子类里**组合**了饮品的对象。这个其实很好理解，就是记录调料这一装饰者包装了什么饮品（或者是包装了”包装了饮品“的调料）。
+ 其次，调料的父类**继承**了饮品父类，也就是说调料也是饮品的子类，就像上文说的，它的目的是完成“调料类和饮品类的类型统一”。以我的理解，这里的继承的具体作用有两个:一是让调料类获得饮料类的方法，如cost()方法，二是以便于其它的调料类包装”包装了饮品的调料类“。

####  实际中的装饰者模式——java.io

java.io包提供的众多输入输出流相关的类，就是应用了装饰者模式。

输入流层层包装，外层流为内层流提供了新的服务。如下图，InputStream是被装饰者类父类,FilterInputStream是一个很多装饰者类的父类，其中有一个InputStream类型的名为in的成员变量。

![image-20200418111628569](/images/image-20200418111628569.png)

下面我给出了一个自定义InputStream的装饰类Demo,作用是将输入流字母变成小写。

```java
/**
 * LowerCaseInputStream.class 
 * @Created by hfq on 2020/4/18
 * @used to: 编写自己的Java IO装饰者类,负责将输入流字母变成小写
 * @return:
 */
public class LowerCaseInputStream extends FilterInputStream {
    /**
     * Creates a <code>FilterInputStream</code>
     * by assigning the  argument <code>in</code>
     * to the field <code>this.in</code> so as
     * to remember it for later use.
     *
     * @param in the underlying input stream, or <code>null</code> if
     *           this instance is to be created without an underlying stream.
     */
    protected LowerCaseInputStream(InputStream in) {
        super(in);
    }

    public int read() throws IOException {
        int c = super.read();
        return c==-1?c:Character.toLowerCase((char)c);
    }
}
```

测试代码如下：

```java
/**
 * JavaIoTest.class 
 * @Created by hfq on 2020/4/18
 * @used to: 测试代码
 * @return:
 */
public class JavaIoTest {
    public static void main(String[] args) throws Exception {
        InputStream inputStream = null;
        try {
            inputStream = new FileInputStream("designpatternlearning.iml");
        } catch (FileNotFoundException e) {
            System.out.println("文件不存在！");
        }
        System.out.println("读到的第一个字符是:"+(char)inputStream.read());
        DataInputStream dataInputStream = new DataInputStream(inputStream);  //使用DataInputStream包装InputStream
        System.out.println("读到的一行字符是:"+dataInputStream.readLine());    //使用自定义的DataInputStream包装InputStream
        LowerCaseInputStream lowerCaseInputStream = new LowerCaseInputStream(dataInputStream);
        int c = 0;
        System.out.println("剩下的字符【经过自定义IO流过滤大写字母为小写字母处理】：");
        while((c = lowerCaseInputStream.read()) > 0){
            System.out.printf(String.valueOf( (char)c) );
        }
        inputStream.close();

        //长得更像装饰者模式的写法：
        InputStream in = new LowerCaseInputStream(new DataInputStream(new FileInputStream("designpatternlearning.iml")));
        while((c = in.read()) > 0){
            System.out.printf(String.valueOf( (char)c) );
        }
        in.close();

    }

}

```



### 总结

![装饰者模式的UML建模图](/images/image-20200418153331522.png)



如上图所见，装饰者模式的核心就是：装饰者既**是一个**(is-a)被装饰者，又**有一个**(has-a)被装饰者。

- is-a 装饰者和被装饰者是相同的类型，这样做是为了利用继承达到“类型匹配”而不是为了利用继承“获得行为”。**“类型匹配是说”一个装饰类的接口必须与被装饰类的接口保持相同。**
- has-a 指的是在装饰者的类中组合了一个被装饰者对象。



### 感悟

+ 首先我觉得，书中这个这个例子其实并不是很好，调料类继承了饮品类，虽然知道它这样做是为了统一类型（使得装饰类既可以包装饮品，也可以包装其他的装饰类），但将调料作为饮品的子类的子类总归很别扭。

+ 另外我觉得理解装饰者模式的核心就是理解"装饰类的接口必须与被装饰类的接口保持相同"，其根本原因是装饰者既要包装被装饰者，也要包装和它同类型的其他装饰者。

+ 装饰者模式通过“层层包装”来装饰内层对象，为最内层的原对象锦上添花，添加新功能，扩展性好。

最后，以书中关于开放关闭原则的描述结尾：
{% blockquote %}代码应该如同晚霞中的莲花一样关闭（免于改变），如同晨曦中的莲花一样开发（能够扩展）。{% endblockquote %} 

这也就是引入装饰者模式的目标吧！