---
title: 《Head-First-设计模式》读书笔记charpter9
date: 2020-04-26 10:22:51
tags: [设计模式,笔记]
category: 设计模式 
---

本篇博客是《Head First 设计模式》第九章的读书笔记。

分享主题是设计模式中的 「迭代器模式(Iterator Pattern)」以及 「组合模式(Composite Pattern)」。

## 迭代器模式

### 场景:菜单合并

有一家早餐馆和一家饭馆合并了，我们需要合并2张菜单，由于以前它们底层实现的数据结构不同，早餐菜单是由AarryList实现的，而正餐菜单是由数组实现的，现在要打印合并后的菜单。
注意：这里说的“合并”不要求底层实现的数据结构的统一，产生一个新的菜单，而只需要打印格式统一就行了，供服务员类菜单统一打印菜单。

#### 思路1：依次打印两张菜单

##### 实现

P.S.为了便于阅读，本文只给出了部分核心代码，完整代码见[GitHub](https://github.com/HFQ123/designpatternlearning)。

pancakeHouseMenu类和DinerMenu类分别是早餐菜单和正餐菜单，代码略。

服务员类代码：

 ```java
public class Waitress{  //服务员类，主要工作:打印菜单(包含早餐菜单和正餐菜单)
     PancakeHouseMenu pancakeHouseMenu; //早餐菜单对象，ArrayList实现
     DinerMenu dinerMenu;            //正餐菜单对象，数组实现
    .....
      public  void printMenu(){  //思路1
        System.out.println("早餐--烙饼专场:");
        ArrayList<MenuItem> arrayList = pancakeHouseMenu.getArrayList();
        for(MenuItem item : arrayList){
            System.out.println(item.toString());
        }
        
        System.out.println("正餐--套餐饭专场：");
        MenuItem[] menuItems = dinerMenu.getMenuItems();
        for(int i = 0; i<menuItems.length; i++){
            if(menuItems[i] == null)
                return;
            System.out.println(menuItems[i].toString());
        }
    }
    .....
}
 ```

如上代码，这种思路是依次打印早餐菜单和正餐菜单：

+ 用一个ArrayList对象接收早餐菜单，遍历这个ArrayList对象，打印每个早餐菜单项。
+ 用一个数组对象接收正餐菜单，遍历这个数组对象，打印每个正餐菜单项。

##### 分析

+ 这种方法需要服务员类清楚地知道各个菜单的内部实现是怎么样的，这样才能分别使用正确的数据类型接收各菜单。
+ 如果有新的菜单需要合并（比如这家餐厅又并入了一个咖啡馆），毫无疑问，我们要修改服务员类的printMenu()方法：根据新的菜单的数据结构接收菜单，再遍历这个集合。
+ 总结前面两点，也就是说，如果有第三张菜单的出现，那么就会需要三个循环，服务员类需要清楚地知道这三张菜单各自的底层数据结构，数组？ArrayList ？HashTable？......



#### 思路2：使用迭代器模式

经过对思路1的分析，我们发现，对于服务员类来说，其实没有必要关注菜单实现的细节：不需要知道每张菜单的各个菜单项是存放在什么类型的集合中的，它只需要关注如何依次获得每个菜单项对象。为此，改进思路是设计原则中的**封装变化的部分**：在这里，需要封装由不同集合类型所造成的遍历。

##### 使用迭代器之后

以正餐菜单为例，设计迭代器：

```java
public class DinerMenu implements IMenu {  //正餐菜单
    static int MAX_ITEMS = 10 ;
    int numberOfItems = 0 ;
    MenuItem [] menuItems;
	..........................//省略了初始化代码
    @Override
    public Iterator createIterator() {
        return new DinerMenuIterator(this.getMenuItems());  //获得迭代器
    }
}


public class DinerMenuIterator implements Iterator { //正餐菜单的迭代器
    MenuItem [] items;  //正餐菜单的数据结构是数组
    int postion = 0;

    public DinerMenuIterator(MenuItem[] items) {
        this.items = items;
    }

    @Override
    public boolean hasNext() {
        if(postion>=items.length || items[postion]==null)
            return false;
        return true;
    }

    @Override
    public MenuItem next() {
        return items[postion++];
    }

}
```

服务员类中打印整张菜单的方法：

```java
    public  void printMenuWithIterator(){  //使用迭代器后，打印整张合并后的菜单
        System.out.println("早餐--烙饼专场:");
       // printMenu(pancakeHouseMenu.createIterator());
        printMenu(pancakeHouseMenu.getArrayList().iterator());  //pancakeHouseMenu的菜单项组织的数据结构是ArrayList，所以可以直接获得ArrayList内置的迭代器
        System.out.println();
        System.out.println("正餐--套餐饭专场：");
        printMenu(dinerMenu.createIterator());
    }

    public  void printMenu(Iterator iterator){  //遍历某个菜单迭代器可访问的全部菜单项
        while (iterator.hasNext()){
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.toString());
        }
    }
```

可以看到，服务员类中，只需要调用各个菜单迭代器的hasNext()和next()来获得每个菜单项，无需关注其数据结构，只有在迭代器内部才需要关注具体的数据结构。

##### 进一步改进：

如上服务员类的代码，服务员类中分别组合了早餐菜单和午餐菜单两个对象，为了提高可拓展性，可以用一个ArrayList保存所有菜单，便于今后加入新的菜单或者删除现有菜单。

改进后更加优雅的服务员类代码:

```java
/**
 * @Created by hfq on 2020/4/26
 * @used to: 更加优雅的服务员
 */
public class WaitressImproved {
    ArrayList menuList;  //保存各个菜单

    public WaitressImproved(ArrayList menuList) {
        this.menuList = menuList;
    }

    public void printMenu(){
        Iterator menuIterator = menuList.iterator();
        while (menuIterator.hasNext()){ 
            IMenu menu = (IMenu) menuIterator.next();  //遍历各个菜单 
            printMenu(menu.createIterator());  //调用printMenu方法依次遍历并打印该菜单的每个菜单项
            System.out.println();
        }
    }

    public  void printMenu(Iterator iterator){   //遍历某个菜单迭代器可访问的全部菜单项
        while (iterator.hasNext()){
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.toString());
        }

    }
}
```

可以看到，这样做的好处在于，如果有新的菜单（需要实现createIterator()方法，获得这个菜单的迭代器）加入，不需要修改服务员类的代码，就能直接调用打印整个菜单的功能。

### 定义

接下来我们来看看迭代器模式的定义：

{% blockquote %}迭代器模式提供一种顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。{% endblockquote %} 

结合上例解读迭代器模式：服务员类不需要知道各个菜单内部究竟是用数组表示的还是ArrayList表示的，它只需要获得菜单的迭代器，由迭代器告诉服务员类**还有没有没有遍历到的对象( hasNext() )**、**下一个遍历的对象是谁)( next() )**，来遍历其中的每个对象。

### 设计原则

- 一个类应该只有一个引起变化的原因

### java的Iterator接口

迭代器模式在平时开发中经常用到。

- hasNext()
- next()


## 组合模式

### 场景

引入组合模式的场景仍然是上例中菜单的例子，现在餐厅菜单引入了新的需求：上例的正餐菜单里需要嵌套一个甜品菜单，支持嵌套菜单后，整张菜单应该长这样：

![支持嵌套子菜单的菜单结构](/images/image-20200430062020664.png)

重点关注上图的“正餐菜单”，在这个菜单集合中，其集合项既可以是它的子菜单（图上的甜品菜单），又可以是普通菜品。

在代码中，我们也需要某种树形结构，可以在一个菜单下同时容纳子菜单和菜品。

### 引入组合模式

没错，我们要引入组合模式来解决这个问题，先来看看组合模式是什么吧：

<%blockquote%>组合模式允许你将对象组合成树形结构来表现“**整体/部分**”层次结构，组合能让客户以一致的方式处理**个别对象**以及**对象组合**。<%endblockquote%>

有了场景的铺垫，很容易配合其定义解读，解读之前，再来规定一下案例中涉及到的几个业务概念，避免混淆。

+ 菜单：作为一个整体菜单独立存在，比如上图的正餐菜单，在代码中独立以一个集合的形式存在。

+ 菜品：菜单下具体的一道菜，比如，上图鸡腿饭是正餐菜单的一道菜品，是形成树形结构的“叶子节点”。

+ 子菜单：某张菜单下嵌套的下级菜单，比如，上图甜品菜单是正餐菜单的一个子菜单，是形成树形结构的“子树”。

+ 菜单项：在菜单概念中，提到了菜单是以一个集合的形式独立存在的，菜单项则就是菜单集合的集合项了，本案例中，菜品和子菜单都可以作为菜单项。

绕了这么多，让我们结合组合模式的定义分析：在本例中，**“整体”**对应**菜单**，**“部分”**对应**菜单项**，**“个别对象”**对应**菜品**，**“对象组合”**对应**子菜单**。因为菜单项既可以是菜品，也可以是子菜单，就构成了有层级关系的树形结构，而我们引入组合模式的目标就是希望能够以统一的方式地处理菜单下级的菜品和子菜单这两种不同结构，而不用关注它到底有没有、有几个子菜单，甚至在子菜单中也能再嵌套子菜单。写出简单的代码，就能够对整个菜单结构应用相同的操作，可以忽略对象组合（子菜单）和个别对象（菜品）的差别。

组合模式的UML图如下，接下来我们就要根据这个类图来编写本案例引入组合模式的代码，我已在图上红字标出本例中涉及到的类分别对应的组合模式角色。

![组合模式的UML图](/images/image-20200430073434886.png)

### 代码

#### 第一步，编写菜单项这一抽象父类代码

再次强调，菜品可以作为菜单项，子菜单也可以作为菜单项，所以作为菜品和子菜单的共同接口，菜单项既要规定菜品有的方法，也要规定子菜单有的方法。

对于子菜单，它需要的方法有：添加菜单项、删除菜单项等。

对于菜品，它需要的方法有：获取菜品名、获取价格等。

子菜单需要的全体方法+菜品需要的全体方法就构成了菜单项抽象类的方法了。

```java
package com.hfq.headfirstjava.charpter9.composite;

/**
 * @Created by hfq on 2020/4/26
 * @used to: 菜单项（子菜单+菜品）的共同接口，抽象类实现
 * 其中，有部分方法只针对子菜单有意义，有部分方法只针对菜品有意义，有部分方法对于它们都有意义，提供统一操作
 */
public abstract class MenuComponent {
    public void add(MenuComponent menuComponent){
        throw new UnsupportedOperationException();
    }
    public void remove(MenuComponent menuComponent){
        throw new UnsupportedOperationException();
    }
    public MenuComponent getChild(int i){
        throw new UnsupportedOperationException();
    }

    public String getName(){
        throw new UnsupportedOperationException();
    }
    public String getDescription(){
        throw new UnsupportedOperationException();
    }
    public double getPrice(){
        throw new UnsupportedOperationException();
    }
    public boolean isVergetarian(){	//是否是蔬菜
        throw new UnsupportedOperationException();
    }
    
    public void print(){
        throw new UnsupportedOperationException();
    }

}
```

在这个抽象类接口中，我们要注意，有些方法，比如说add()、remove()，只针对子菜单这一菜单项有意义而菜品这一菜单项不需要，而有些方法，比如说getPrice()，只针对菜品这一菜单项有意义而子菜单这一菜单项不需要，

正是因为子类菜品不需要其中的某些方法，而子菜单不需要另外一些方法，所以在抽象类的默认实现中，都抛出了异常。

而print()方法是一个”操作方法“，则可看作菜品和菜单项两者共同的操作。

至此，就完成了就是**菜单项**父类代码的编写（对应上面UML建模图中的Component角色），接下来就来完成其子类的代码，其子类包括**菜品**（对应Leaf角色）和**子菜单**（对应Composite角色）。

#### 第二步，编写菜单项的两个子类代码

菜品类和子菜单类只要挑选抽象父类中自己感兴趣的、有意义的方法实现即可。

菜品类：

```java
package com.hfq.headfirstjava.charpter9.composite;

/**
 * @Created by hfq on 2020/4/30
 * @used to: 菜单项的子类之一--菜品类
 */
public class MenuItem extends MenuComponent {

    String name;    //名称
    String desc;    //描述
    boolean vegetarian;     //是否为蔬菜
    double price;           //价格

    public MenuItem(String name, String desc, boolean vegetarian, double price) {
        this.name = name;
        this.desc = desc;
        this.vegetarian = vegetarian;
        this.price = price;
    }

    public String getName(){
        return name;
    }

    public String getDescription() {
        return desc;
    }

    public double getPrice() {
        return price;
    }

    public boolean isVergetarian() {
        return vegetarian;
    }

    public void print() {
        String item =  "MenuItem{" +
                "name='" + name + '\'' +
                ", desc='" + desc + '\'' +
                ", vegetarian=" + vegetarian +
                ", price=" + price +
                '}';
        System.out.println(item);
    }
}
```

子菜单类：

```java
package com.hfq.headfirstjava.charpter9.composite;

import java.util.ArrayList;

/**
 * @Created by hfq on 2020/4/30
 * @used to: 子菜单类
 */
public class Menu extends MenuComponent{
    ArrayList menuComponents = new ArrayList();     //组合了一个ArrayList对象来保存子菜单的所有菜单项
    String name;
    String description;

    public Menu(String name, String description) {
        this.name = name;
        this.description = description;
    }
    public void add(MenuComponent menuComponent){
        menuComponents.add(menuComponent);
    }
    public void remove(MenuComponent menuComponent){
        menuComponent.remove(menuComponent);
    }
    public MenuComponent getChild(int i){
        return (MenuComponent) menuComponents.get(i);
    }
    public String getName(){
        return name;
    }
    public String getDescription(){
        return description;
    }
    public void print(){ //难点
        System.out.println(name+"("+description+")");
        System.out.println("--------------------------------");
        Iterator iterator = menuComponents.iterator();
        while (iterator.hasNext()){
            MenuComponent menuComponent = (MenuComponent) iterator.next();
            menuComponent.print();  //递归调用子菜单的所有菜单项print方法
        }
    }
}
```
#### 第三步，编写服务员类的代码

服务员类对应上面UML建模图中的Client客户角色。

```java
package com.hfq.headfirstjava.charpter9.composite;

/**
 * @Created by hfq on 2020/4/30
 * @used to: 引入组合模式后的服务员类，是菜单的客户(UML类图中的Client)
 */
public class Waitress {
    MenuComponent allMenus;

    public Waitress(MenuComponent allMenus) {
        this.allMenus = allMenus;
    }
    public void printMenu(){
        allMenus.print();
    }

    //测试代码
    public static void main(String[] args) {
        MenuComponent totalMenu = new Menu("总菜单","存放所有分菜单");
        MenuComponent breakfastMenu = new Menu("早餐菜单","含有各种早餐");
        breakfastMenu.add(new MenuItem("肉包子","有猪肉的包子",false,2.0));
        breakfastMenu.add(new MenuItem("菜包子","有香菇的包子",true,1.0));
        breakfastMenu.add(new MenuItem("烙饼","荤素搭配的烙饼",false,3.0));
        MenuComponent dinnerMenu = new Menu("正餐菜单","含有各种饭,还包含了一个甜品菜单");
        dinnerMenu.add(new MenuItem("鸡腿饭","有红烧鸡腿",false,10.0));
        dinnerMenu.add(new MenuItem("牛肉饭","有香辣牛肉",false,15.0));
        MenuComponent dessertMenu = new Menu("甜品菜单","包含多种甜品");
        dessertMenu.add(new MenuItem("烧仙草","美味的烧仙草",false,3.0));
        dessertMenu.add(new MenuItem("绿豆汤","清爽的绿豆汤",true,1.0));
        dinnerMenu.add(dessertMenu);
        totalMenu.add(breakfastMenu);
        totalMenu.add(dinnerMenu);

        Waitress waitress = new Waitress(totalMenu);
        waitress.printMenu();
    }
}

```

上述测试代码打印菜单的结果如下所示，成功达到我们的目标：

![测试打印整张菜单的结果](/images/image-20200430082134855.png)

为了让显示效果好一点，看得出菜单嵌套的层级效果，我又对上面的代码做了略微的改动，print()方法加了一个String类型的前缀，具体实现请移步[Github](https://github.com/HFQ123/designpatternlearning)。如下图，改动后，能在打印结果看出来甜品菜单是正餐菜单的子菜单。

![改动后的显示效果具有层次](/images/image-20200430084047221.png)



### 总结

组合模式中让组件接口（对应于本例中的菜单项的抽象类）同时包含一些管理子节点（子菜单）和叶子节点（菜单项）的操作，**客户(Client)就可以将组合和叶节点一视同仁**，也就是说，一个元素究竟是组合还是叶节点，对用户是透明的，在案例中，即使是用户不知道一个菜单项究竟是子菜单还是普通菜品的情况下，也能调用统一的print方法打印。

然而，组合模式这种做法失去了一些“安全性”，这是因为客户有机会对一个元素做不恰当或者是没有意义的操作(列入本例中试图调用一个菜品对象的add方法，菜品类继承了菜单项抽象类，所以它有add方法，但是这个add方法实际上是为子菜单类服务的，而对菜品类无意义)。假如我们将菜品和子菜单各自需要的方法区分开来放到不同的接口里，客户代码就必须用条件语句和instanceof操作符来区别对待不同类型的节点(菜品和子菜单)。

所以说，使用组合模式是便利性和违背涉及原则折衷的一个选择。



## 组合迭代器

分别学完了迭代器模式和组合模式之后，来看看它们共同作用会什么样的火花吧，亲密无间的配合也是它们放在同一个章节的原因。

我们可以这样理解，在上文介绍迭代器模式时的场景中，菜单集合是一维的，因为其集合元素都是菜品，而组合模式中的案例场景中，菜单结构是有嵌套层次的。针对一个元素全都是菜品对象的普通的菜单集合，构建它的迭代器很简单，而对于构建像这一树形层次结构的集合的迭代器又该如何做呢？

