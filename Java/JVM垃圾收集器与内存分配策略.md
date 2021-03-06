##判断对象是否需要回收

###引用计数算法
给对象添加一个引用计数器，被一个地方引用就+1，当引用失效时，就-1。当计数器值为0时，表面该对象需要被回收。

<font color=red>这种算法有严重的缺陷，例如，如果两个对象A、B互相引用，此外没有别的地方对它俩有引用，那么意味着它们俩永远不会被回收。</font>

###可达性分析算法(主流)
通过一系列被称为“GC Roots”的对象作为起始点，从这些点开始向下搜索，如果一个对象无法从GC Roots到达，那么这个对象就是不可达的，即需要回收的。

可以作为GC Roots的对象：

1. 虚拟机栈中引用的对象
2. 方法区中类静态属性引用的对象
3. 方法区中常量引用的对象
4. Native方法中引用的对象

###究竟是生是死
一个对象的回收会经历两步：

1. 可达性分析判断不可达
2. 是否有必要执行finalize()方法，（未覆盖和已经调用过一次均视作不必要），如果不必要执行，直接回收掉，有必要执行，可以在finalize()中拯救自己。


##回收方法区
对方法区的回收主要是常量和无用的类。
无用类这里比较复杂，满足以下三个条件的才是无用类：

1. 所有实例被回收
2. ClassLoader被回收
3. 对应的class对象在任何地方无引用，无法通过反射访问该类。

##垃圾收集算法
###标记-清除算法
顾名思义，算法分为两步，先标记除需收集的对象，完毕后再回收被标记对象。

<font color=red>有两点不足:

1. 效率低
2. 会产生大量的不连续内存碎片 

</font>

###复制算法
为了解决效率的问题，将内存空间分为两部分，这里举例A、B说明，每次只在A或B上进行分配，假定现在在A上分配空间，发现空间不足，那么将A中存活的对象复制到B上，然后将A的内存空间一次性清除，这样可以避免内存碎片的产生。<font color=red>代价就是牺牲了一半的内存空间</font>

**<font color=bray>现代的商业虚拟机都采用这种方法来回收新生代，实际上，98%的对象都是朝生夕死的，所以不需要按照1:1来进行分配，于是就有了优化的复制算法</font>**

将新生代内存一次性分为一块大的Eden空间A，两块小的Survivor空间BC，比例为8:1:1，每次使用A和BC中的一个进行分配，也就是90%的空间进行分配内存，内存不够时，采取复制算法，将存活对象复制到未使用的一块Survivor上，然后一次性清理掉Eden和一块Survivor，但是这里也会出现一块Survivor不够存放存活对象，这个时候，就会依赖老年代内存的支持。

###标记-整理算法
与标记-清除算法类似，标记之后，不直接清理被标记对象，而是将他们一起移动到一端，然后一次性清理掉，这样就避免了内存碎片的问题。

###分代收集算法
这里只是说，将Java堆分为新生代和老年代，新生代采用复制算法，老年代可以采用标记-清除或标记-整理算法。

##垃圾收集器
- 新生代：Serial、ParNew、Parallel Scavenge
- 老年代：CMS、Serial Old、Parallel Old
- G1

###Serial
单线程收集器(开启一个线程进行垃圾收集)，运行时会暂停用户的全部线程，在限定单个cpu的环境中，无线程交互开销。比如运行在Client模式下的虚拟机是一个很好地选择。

###ParNew
可以看作是Serial的多线程版本，在多CPU环境下性能高于Serial，其余区别不大，一样会在收集时暂停所有用户线程。

<font color=red>但它是目前Server模式下首选的新生代收集器，目前只有它能与老年代的CMS收集器一起工作，CMS是第一款实现垃圾收集线程与用户线程并发的收集器。当然Serial也可以与CMS一起搭配使用，而Parallel Scanvenge不行。</font>

###Parallel
多线程，吞吐量可控，即控制垃圾收集的时间，并带有自适应策略。

###Serial Old
Serial的老年代版，单线程，采用标记-整理算法，同样会在运行时暂停所有用户线程。适用于Client模式下虚拟机，Server模式下两大用途：

1. JDK1.5及以前，搭配Parallel Scavenge使用。
2. 在CMS发生并发模式错误时，作为备用。


###Parallel Old
Parallel收集器的老年版本，多线程，采用标记-整理算法，同样会暂停用户线程，搭配Parallel收集器，在吞吐量与CPU资源敏感的场景下使用。

###CMS收集器
并发清理，并发重置，会占用一定CPU资源，基于标记-清除，会有内存碎片，但可以开启碎片整理开关，也可以控制定期碎片整理。

###其他分配策略
1. 长期存活对象直接分配至老年代，每熬过一次GC，年龄+1，达到max时，转入老年代。
2. 当某一年龄的对象数量占整体对象的一半时，大于此年龄的直接转入老年代。


###<font color=green>一些参数设置</font>
1. -Xms:初始堆大小
2. -Xmx:最大堆大小
3. -XX:NewSize=n:设置年轻代大小
4. -XX:NewRatio=n:设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
6. -XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
7. -XX:MaxPermSize=n:设置持久代大小


