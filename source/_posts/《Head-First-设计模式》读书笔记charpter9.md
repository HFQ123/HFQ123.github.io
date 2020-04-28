---
title: 《Head-First-设计模式》读书笔记charpter9
date: 2020-04-26 10:22:51
tags: [设计模式,读书笔记]
category: 设计模式 
---

本篇博客是《Head First 设计模式》第九章的读书笔记。

分享主题是设计模式中的 「迭代器模式(Iterator Pattern)」以及 「组合模式(Composite Pattern)」。

## 9-1 迭代器模式

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

###### 使用迭代器之后

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

###### 进一步改进：

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