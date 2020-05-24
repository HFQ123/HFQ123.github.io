(关于JVM的Cilent模式和Server模式)[https://blog.csdn.net/tang_123_/article/details/6018219]

![image-20200415234029401](../images/image-20200415234029401.png)



![image-20200415234254403](../images/image-20200415234254403.png)





For the parallel collector, Java SE provides two garbage collection tuning parameters that are based on achieving a specified behavior of the application: **maximum pause time **goal and **application throughput goal**;(These two options are not available in the other collectors.)

最大停顿时间和吞吐量目标。 

The pause time is the duration during which the garbage collector stops the application and recovers space that is no longer in use.

Maximum Pause Time GoalThe pause time is the duration during which the garbage collector stops the application and recovers space that is no longer in use. The intent of the maximum pause time goal is to limit the longest of these pauses. An average time for pauses and a variance on that average is maintained by the garbage collector. The average is taken from the start of the execution but is weighted so that more recent pauses count more heavily. If the average plus the variance of the pause times is greater than the maximum pause time goal, then the garbage collector considers that the goal is not being met.

暂停时间是垃圾收集器停止应用程序并恢复不再使用的空间的持续时间。最大暂停时间目标的目的是限制这些暂停的最长时间。暂停的平均时间和该平均值的方差由垃圾收集器维护。平均值是从执行开始时开始计算的，但要加权，以便最近的暂停计数更重。如果平均值加上暂停时间的方差大于最大暂停时间目标，则垃圾收集器认为未达到该目标。

吞吐量目标是以收集垃圾所花费的时间和垃圾收集之外所花费的时间（称为应用程序时间）来衡量的。-xx:gctimeratio=<nnn>。垃圾收集时间与应用程序时间之比为`1/(1+)`。例如，`-xx:gctimeratio=19`设置垃圾收集时间占总时间的1/20或5%的目标。The time spent in garbage collection is the total time for both the young generation and old generation collections combined. If the throughput goal is not being met, then the sizes of the generations are increased in an effort to increase the time that the application can run between collections.



如果已满足吞吐量和最大暂停时间目标，则垃圾收集器会减小堆的大小，直到无法满足其中一个目标（总是吞吐量目标）。 然后解决未达到的目标。

![image-20200416000347862](../images/image-20200416000347862.png)

是并行垃圾回收器(吞吐量优先垃圾回收器)特有的。





当垃圾收集器试图满足竞争目标时，堆的大小通常会振荡。 即使应用程序已达到稳定状态，也是如此。 实现吞吐量目标（可能需要更大的堆，【我的理解：这样的话堆能容纳的对象就更多了，垃圾回收发生的频率变低了，两次垃圾回收之间的应用程序时间自然而然就高了。】）的压力会与最大暂停时间和最小占用空间（两者都可能需要小堆，【我的理解：堆空间变小了，回收耗费时间自然而然就低了】）的目标竞争。

[
  ](http://blog.orisonchan.cc/uploads/wechat-qcode.jpg)

为什么要进行JVM调优？

Java SE平台的一个优点是它可以保护开发人员免受内存分配和垃圾回收的复杂性的影响。 但是，**当垃圾回收是主要瓶颈时**，了解此隐藏实现的某些方面很有用。 垃圾收集器对应用程序使用对象的方式做出假设，这些可以反映在可调参数中，可以调整这些参数以提高性能，而不会牺牲抽象的功能。



如果垃圾收集成为瓶颈，您很可能必须自定义总堆大小以及各代的大小。 检查详细的垃圾收集器输出，然后探索各个性能指标对垃圾收集器参数的敏感性。



While naive garbage collection examines every live object in the heap, generational collection exploits several empirically observed properties of most applications to minimize the work required to reclaim unused (garbage) objects.



https://blog.csdn.net/as02446418/article/details/47664967

[weak generational hypothesis弱年代假设](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/generations.html)

The virtual machine incorporates a number of different garbage collection algorithms that are combined using *generational collection*. While naive garbage collection examines every live object in the heap, generational collection exploits several empirically observed properties of most applications to minimize the work required to reclaim unused (garbage) objects. The most important of these observed properties is the weak *generational hypothesis*, which states that most objects survive for only a short period of time.



大部分分配的对象死的快（比如说一个for循环里new出来的对象，且没有循环外的变量引用）！而那些寿命长（有些甚至以至于从初始化开始一直存在直至到进程退出）的对象很少。

大多数对象“年轻化”



![image-20200416010033397](../images/image-20200416010033397.png)

为了优化这种情况，内存是在*代（Generations）*中管理的（内存池包含不同年龄的对象）。当某一代填满时，每一代都会发生垃圾收集。**绝大多数对象**都分配在专用于年轻对象（新生代）的池中，并且大多数对象在那里死亡。当新生代填满时，它会导致一次小的回收（*minor collection*），其中只回收新生代；其他几代的垃圾都没有回收。这些小的回收，在弱世代假设成立并且新生代中的大多数对象都是垃圾且可以被回收的情况下，是可以被优化的。这些回收的成本，首先是与回收的活动对象的数量成比例；一个充满已死对象的新生代会很快被收集。通常，来自新生代的幸存对象的一部分（！！哪一部分）在每次小的回收期间被移动到老年代。最终，老年代将填满并且必须被回收，从而产生一个收集整个堆的大回收（*major collection*）。大回收通常比小回收持续更长时间，因为涉及的对象数量明显更多。

 Major collections usually last much longer than minor collections because a significantly larger number of objects are involved.

总结：根据弱世代假设，绝大多数对象分配在新生代中，大多数对象也是在新生代死亡且被回收，而新生代的幸存对象的一部分（具体说来，就是 age到了阈值的对象）有机会晋升到老年代。如果新生代内存不足，会导致一次小回收，Minor gc。



为什么新生代采用复制？

回收的成本，首先是与被收集的活动对象的数量成比例



由人机工程学选择的堆大小参数加上自适应大小策略的功能旨在为服务器应用程序提供良好的性能。 这些选择适用于大多数情况，但不是所有情况，这引出了本文档的核心原则：

> **注意**：如果垃圾收集成为瓶颈，您很可能必须自定义总堆大小以及各代的大小。 检查详细的垃圾收集器输出，然后探索各个性能指标对垃圾收集器参数的敏感性。



这里回答了，JVM调优是调什么东西？

The total heap size as well as the sizes of the individual generations. 



![Description of Figure 3-2 follows](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/img/jsgct_dt_001_armgnt_gn.png)

At initialization, a maximum address space is virtually reserved but not allocated to physical memory unless it is needed. The complete address space reserved for object memory can be divided into the young and tenured generations.

在初始化时，最大地址空间实际上是保留的，但除非需要，否则不会分配给物理内存。为对象内存保留的完整地址空间可分为新生代和老年代。

The young generation consists of eden and two survivor spaces. Most objects are initially allocated in eden. One survivor space is empty at any time, and serves as the destination of any live objects in eden; the other survivor space is the destination during the next copying collection. Objects are copied between survivor spaces in this way until they are old enough to be tenured (copied to the tenured generation).



新生代包括Eden和两个Survivor。大多数对象最初是在Eden中分配的。两个Survivor的其中一个在任何时候都是空的，并且作为Eden中任何存活对象的目的地；另一个Survivor空间是下一次复制收集期间的目的地。对象在Survivor空间之间以这种方式被复制，直到它们足够大到可以被老年化（复制到老年代）。



Performance ConsiderationsThere are two primary measures of garbage collection performance:*Throughput* is the percentage of total time not spent in garbage collection considered over long periods of time. Throughput includes time spent in allocation (but tuning for speed of allocation is generally not needed).*Pauses* are the times when an application appears unresponsive because garbage collection is occurring.

![image-20200416011835202](../images/image-20200416011835202.png)



鱼与熊掌不可兼得

In general, choosing the size for a particular generation is a trade-off between these considerations. For example, a very large young generation may maximize throughput, but does so at the expense of footprint, promptness, and pause times. Young generation pauses can be minimized by using a small young generation at the expense of throughput. The sizing of one generation does not affect the collection frequency and pause times for another generation.



JVM调优的世界观：

There is no one right way to choose the size of a generation. **The best choice is determined by the way the application uses memory as well as user requirements.** Thus the virtual machine's choice of a garbage collector **is not always optimal** and may be overridden with command-line options described in the section [Sizing the Generations](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html#sizing_generations).



### 涉及到的 JVM 命令行参数配置：

`-verbose:gc`   //在控制台输出GC情况 

![image-20200416015459141](../images/image-20200416015459141.png)

The command-line option `-verbose:gc` causes information about the heap and garbage collection to be printed at each collection. For example, here is output from a large server application：

```
[GC 325407K->83000K(776768K), 0.2300771 secs]
[GC 325816K->83372K(776768K), 0.2454258 secs]
[Full GC 267628K->83769K(776768K), 1.8479984 secs]
```

参数说明：

```
The output shows two minor collections followed by one major collection. The numbers before and after the arrow (for example, 325407K->83000K from the first line) indicate the combined size of live objects before and after garbage collection, respectively. After minor collections, the size includes some objects that are garbage (no longer alive) but cannot be reclaimed. These objects are either contained in the tenured generation or referenced from the tenured generation.

The next number in parentheses (for example, (776768K) again from the first line) is the committed size of the heap: the amount of space usable for Java objects without requesting more memory from the operating system. Note that this number only includes one of the survivor spaces【这里是说总可用大小中只含有一个幸存区的空间，因为除了在垃圾回收期间，在任何给定时间仅将使用一个幸存空间来存储对象。】. Except during a garbage collection, only one survivor space will be used at any given time to store objects.

The last item on the line (for example, 0.2300771 secs) indicates the time taken to perform the collection, which is in this case approximately a quarter of a second.

The format for the major collection in the third line is similar.
```



`-XX:+PrintGCDetails`  // 打印更详细的GC信息

DefNew表示新生代使用Serial串行GC垃圾收集器，defNew提供新生代空间信息；

DefNewGeneration 就是 default new generation

![image-20200416015614242](../images/image-20200416015614242.png)



`-XX:+PrintGCTimeStamps`

The option `-XX:+PrintGCTimeStamps` adds a time stamp at the start of each collection. This is useful to see how frequently garbage collections occur.





在JVM的启动参数中加入-XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime，按照参数的顺序分别输出GC的简要信息，GC的详细信息、GC的时间信息及GC造成的应用暂停的时间。

————————————————
版权声明：本文为CSDN博主「iteye_7017」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/iteye_7017/article/details/82655019



**Xms**
英文解释：`Initial heap size(in bytes)`
中文释义：`堆区初始值`
使用方法：`-Xms2g` 或 `-XX:InitialHeapSize=2048m`

**Xmx**
英文解释：`Maximum heap size(in bytes)`
中文释义：`堆区最大值`
使用方法：`-Xmx2g` 或 `-XX:MaxHeapSize=2048m`



XX:NewRatio

- 新生代（Eden + 2*S）与老年代（不包括永久区）的比值
- 4 表示新生代 ：老年代 = 1：4 ，意思是老年代占 4/5



**heap动态增长和heap的默认大小不适用于parallel collector**。不过，控制heap总大小的参数和分代大小的参数是适用于parallel collector的。



影响垃圾回收性能的最重要的因素是**总可用内存**。因为gc是在分代内存用完之后发生的，吞吐量（除了垃圾回收外的时间/总时间）与可用内存数量成反比。



-XX:MinHeapFreeRatio=<minimum>和-XX:MaxHeapFreeRatio=<maximum>

当一个代的空闲空间小于min，这个代会扩展，

当一个代的空闲空间大于max，这个代会收缩。



最大heap size是JVM计算出来的。

[JVM的GC停顿时间过长该怎么处理？](https://mp.weixin.qq.com/s/tqCn1CLPdXsaqX6O2fbrnQ)





#### 总内存大小

-xms

-xmx

-XX:MinHeapFreeRatio=<minimum>和-XX:MaxHeapFreeRatio=<maximum>



####  young区比例

`NewRatio`

除了总可用内存大小，拥有第二影响力的参数是young区占总heap的比例。young区越大，minor gc发生频率越小。然而，对于一个有界的heap，更大的young区往往意味着你的tenured区很小，这会提升major gc的次数。具体如何选择，取决于你的存活对象分布图。

默认情况下，young区大小受参数`NewRatio`控制。例如，设置`-XX:NewRatio=3`意味着young:tenured=1:3，换句话说，eden+2 survivor区的大小是整个堆的4分之一。

参数`NewSize和MaxNewSize`标记了young区的上下界。



## Survivor Space Sizing 设置Survivor区大小

你可以使用`SurvivorRatio`参数来调整survivor区的大小，不过大多数这都不怎么影响性能。例如，`-XX:SurvivorRatio=6`参数表示eden:survivor=6:1。换句话说，每个survivor区的大小占1个eden区大小的6分之1，也是young区的8分之1。

如果survivor区太小，复制回收时多出来的对象会直接进tenured generation。如果survivor区太大，会造成空间浪费。

每次回收时，jvm会选择一个阈值，这个值是一个对象进入tenured区之前的复制次数。这个值最好能刚好让2个survivor区中的一个是满的。

最好是回收的时候幸存区to是满的。

幸存区太大/小的影响？



 `-XX:+PrintTenuringDistribution`

.The command line option `-XX:+PrintTenuringDistribution` (not available on all garbage collectors) can be used to show this threshold and the ages of objects in the new generation. 

![image-20200417000019538](../images/image-20200417000019538.png)

![image-20200417001057735](../images/image-20200417001057735.png)





### 可用的收集器

serial collector

parallel collector

The Mostly Concurrent Colletor



垃圾收集器的选择

-XX:+UseSerialGC

-XX+UseParallelGC和X:-UseParallelOldGC（不带并行整理(compact))

`-XX:+UseConcMarkSweepGC`和`-XX：UseG1GC`

[官网给出的选择垃圾收集器的策略](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/collectors.html)

![image-20200417004941869](../images/image-20200417004941869.png)





6.Parllel collector

The parallel collector is selected by default on server-class machines. In addition, the parallel collector uses a method of automatic tuning that allows you to specify specific behaviors instead of generation sizes and other low-level tuning details. You can specify maximum garbage collection pause time, throughput, and footprint (heap size).
当使用-XX:+UseParallelGC选择并行收集器时，它启用了一种自动调优方法，支持你指定行为，而不是指定分代的大小和其他低级调优细节信息。



如果未满足最大暂停时间目标，则一次只缩小一个分代的大小。 如果两代的暂停时间都高于目标，那么具有较大暂停时间的分代的大小先缩小。

如果吞吐量目标没有得到满足，那么这两代的大小都会增加。每一个都按其对总垃圾收集时间的贡献的比例增加。例如，如果年轻代的垃圾收集时间占总收集时间的25%，如果年轻代的完全增量为20%，那么年轻代大小将增加5%。



除非在命令行中指定了初始和最大堆大小，否则它们将根据计算机上的内存量进行计算。 默认的最大堆大小是物理内存的四分之一，而初始堆大小是物理内存的1/64。 分配给年轻代的最大空间是总堆大小的三分之一。

-XX:+PrintFlagsFinal选项，并在输出中查找MaxHeapSize。