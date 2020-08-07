---
layout: 	post
title: 		"Java知识合集"
subtitle:	" \"Java知识整理\""
date:		2020-08-07 23:40:00
author:		"duwanjiang"
header-img:	"img/about-bg-walle.jpg"
catalog: true
categories: Java
tags:
    - Java
---

> “从现在开始实现你的梦想，就不晚 ”

# 《Java知识合集》

这篇文章是我准备找工作时，自己花了2个月时间复习并整理的Java相关的知识合集，可能知识涵盖面不全，主要是整理了我平时容易忘记的内容，同时也希望可以帮助你进行知识的回顾，这里有两个博主的文章需要推荐给大家，他们这里的知识面很全，我也是借鉴了他们很多内容，望收藏：
* https://github.com/Snailclimb/JavaGuide
* https://github.com/CyC2018/CS-Notes

同时也欢迎来我的Blog >>[传送门](http://www.duwanjiang.com)<<

# 1、基础知识

## 1.1、线程安全

### 1.1.1、线程有哪些基本状态?


![]({{"\img\posts_img\technology\java\knowledgeCollection\6bf58fc1-230b-4901-a94f-2e809738440e.jpg"|prepend:site.url}})

### 1.1.2、Synchronized的原理

每个对象都有一个监视器锁（monitor），jvm中会使用monitorenter和monitorexit来获取或放弃监视器对象的所有权：

1. 线程执行monitorenter方法来尝试获取monitor的所有权，当monitor的进入数为0时，获取到monitor的所有权，且将monitor进入数加1，线程重入时再加1。

2. 当线程执行monitorexit方法时，monitor进入数减1，直到为0时，释放对monitor的所有权，此时其他被阻塞的线程就可以获取monitor所有权。



### 1.1.3、各种锁的定义

（参考：<https://www.cnblogs.com/linghu-java/p/8944784.html>）

1. 锁的状态有4种：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态。

2. jdk1.6对锁做了哪些优化：偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术减少锁的开销。

	1. 偏向锁：当没有多线程竞争的情况下，虚拟机会使用CAS操作将第一个获取到锁的线程的ID记录到该对象的Mark Word之中，之后该线程执行同步代码块时，虚拟机不在进行同步操作，如果偏向锁失败，会升级为轻量级锁。

	2. 轻量级锁：轻量级锁不是来替代重量级锁，它减少了使用系统互斥量产生的性能开销，而是由偏向锁升级而来的，虚拟机使用CAS操作将锁对象的Mark Word交换为线程栈中的Lock Record的指针，如果锁竞争激烈，那么轻量级锁很快将膨胀为重量级锁。

	3. 自旋锁：轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段，也就是让后面来的线程尝试等待一会，而不直接进入挂起状态，看看持有锁的线程会不会很快释放锁，所以这里的等待实现就是执行一个忙循环，称之为自旋，默认执行次数为10次。

	4. 适应性自旋锁：是对自旋锁的改进，及自旋的时间不固定，而是和前一次同一个锁上的自旋时间以及锁的拥有者状态来决定的，如果上一次成功了，那虚拟机会认为这一次待定相比更长的时间也很有可能成功，虚拟机会越变越聪明。

	5. 锁消除：虚拟机及时编译器在运行时，如果检测到共享数据不可能存在竞争，就会消除该锁的同步，如：在使用StringBuffer拼接字符串时。

	6. 锁粗化：通常情况下锁的作用范围越小，对性能的开销越小，但是如果一系列的连续操作都对同一对象反复加锁和解锁时，就可以提升锁的粒度，达到减少锁的开销。

3. volatile原理：<https://www.jianshu.com/p/157279e6efdb>



### 1.1.4、公平锁与非公平锁区别

1. 概念：
公平锁：加锁时先看看队列中是否有等待的线程，如果有先处理队列中等待的线程，先到先得。
非公平锁：加锁时直接尝试获取锁，获取不到再到队列末尾等待。 

2. 优缺点：
非公平锁性能高于公平锁5-10倍，减少了线程切换的时间，能更充分利用cpu的时间片。

3. 使用场景：
平时更多的是直接使用非公平锁，因为公平锁需要在多核情况下维护一个队列，如果当前线程不是队列的第一个就无法获取锁，增加了线程切换次数；但是如果线程业务执行时间远大于线程等待时间，那用非公平锁效果就不太明显了，但是用公平锁能更好的控制业务。


### 1.1.5、乐观锁与悲观锁

1. 概念
悲观锁：假设一定会发生并发冲突，通过阻塞其他线程来保证数据的完整性。
乐观锁：假设不会发生并发冲突，直接不加锁完成某项更新，如果冲突就返回失败。 
2. 例子
悲观锁：Synchronized多线程同步，具有排他性，容易产生死锁。
乐观锁：CAS机制，简单来说有3个操作数，当前内存变量值V，变量预期值A，即将更新值B，当需要更新变量的时候，判断V与A，如果相等，则更新B，否则将当前V值刷新到预期值A当中，然后再重新尝试比较更新。
3. 适用场景
乐观锁：适用于数据争用不严重/重试代价不大/需要响应速度快的场景
悲观锁：适用于数据争用严重/重试代价大的场景



### 1.1.6、线程池

#### 1.1.6.1、如何创建线程池

1. 通过ThreadPoolExecutor创建：通过该类创建的线程池，能更清楚线程池的运行规则，避免资源的浪费
2. 通过Executors创建，其中FixedThreadPool、SingleThreadExecutor、CachedThreadPool的内部都是通过ThreadPoolExecutor实现的，具体为：
	1. newFixedThreadPool：该方法返回一个固定线程数的线程池。当有新的任务提交时，如果有空的线程则立即执行，如果没有空闲则加入到一个队列中，待线程空闲时再按先进先出顺序执行。
	2. newSingleThreadExecutor：该方法返回只有一个线程的线程池。当有新的任务提交时，如果有空的线程则立即执行，如果没有空闲则加入到一个队列中，待线程空闲时再按先进先出顺序执行。
	3. newCachedThreadPool：该方法返回一个根据可根据实际线程数调整的线程池。当有新的任务提交时，如果有空闲则复用执行，如果没有则新创建一个线程来执行。
	4. newScheduledThreadPoolExecutor：该方法返回一个定时任务线程池，用于周期性的执行任务，可以定时执行或延时执行。

#### 1.1.6.2、ThreadPoolExecutor的饱和策略

1. ThreadPoolExecutor.AbortPolicy：抛出RejectExecutionException来拒绝执行
2. ThreadPoolExecutor.CallerRunsPolicy：使用启动线程池的主线程去执行被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。
3. ThreadPoolExecutor.DiscardPolicy：抛弃掉当前最新的任务。
4. ThreadPoolExecutor.DiscardOldestPolicy：抛弃掉最老的未处理任务。


#### 1.1.6.3、ThreadPoolExecutor的execute的执行流程

![]({{"\img\posts_img\technology\java\knowledgeCollection\ada9ee02-a27a-4087-bf37-d804335ed2d4.png"|prepend:site.url}})


### 1.1.7、AQS同步器

#### 1.1.7.1、AQS分析

1. 定义：AQS（AbstractQueuedSynchronizer）是一个在concurrent包下的，用来构建锁和同步器的框架，采用CLH队列锁来控制线程的阻塞和唤醒机制。处理模型如下图：

![]({{"\img\posts_img\technology\java\knowledgeCollection\cf7e2e4e-da50-480e-bb23-fe716c6030e4.png"|prepend:site.url}})


2. 资源共享方式：
	1. 独占锁（Exclusive）：一次只能支持一个线程访问资源，可以分为“公平锁和非公平锁”，如：Reentrantlock。
	2. 共享锁（Share）：多个线程同时访问资源，如：CountDownLatch、Semaphore。

3. 实现方式：AQS使用了模板方法模式，实现自定义同步器时，只需要将继承了AQS的使用者组合到其内部，并让使用者实现AQS对资源state的获取和释放，其中线程队列的等待维护已由AQS顶层实现好了，然后自定义同步器再调用使用者重写的AQS模板方法即可。

## 1.2 对象创建

### 1.2.1、单例模式

(参考：<https://blog.csdn.net/yeyazhishang/article/details/90445330>)

### 1.2.2、创建对象的方法

（参考：<https://blog.csdn.net/qq_33314107/article/details/80271963>）

1. 通过new方法
2. 通过反射方法
3. 通过Object的clone方法
4. 通过序列化方法

## 1.3、注解

（参考：<http://www.itsoku.com/article/274#menu_18>）

## 1.4、反射

### 1.4.1、反射机制

定义：是指在运行状态下，能够获取任意类的属性方法，且可以调用任意对象的属性方法的功能叫做java的反射机制。

### 1.4.2、动态代理与静态代理

1. 静态代理：是在运行前就确定了代理类与被代理类之间的关系，类似于装饰者模式，需要代理类和被代理类都实现相同的接口。

2. 动态代理：是在运行时创建代理对象，一般有2中方式实现JDK动态代理和Cglib动态代理。
	1. JDK动态代理：只能代理接口，使用InvocationHandler接口和Proxy类实现
	2. Cglib动态代理：是类的代理，通过继承被代理类来实现对象的增强，使用Enhancer类和MethodInterceptor接口实现

(参考：<https://blog.csdn.net/flyfeifei66/article/details/81481222>)

### 1.4.3、ASM和Javassist区别

（参考：<https://www.cnblogs.com/xd502djj/p/13062165.html>）

1. Javassist是可以源码级别操作，比ASM的字节码操作更加容易使用。

2. Javassist使用反射机制，在执行性能上比ASM慢很多。



## 1.5、集合

![]({{"\img\posts_img\technology\java\knowledgeCollection\0.19550936268073071.png"|prepend:site.url}})

### 1.5.1、ArrayList、LinkedList、Vector的区别

1. 线程安全性：ArrayList和LinkedList是非线程安全的、Vector是线程安全的。

2. 底层数据结构：ArrayList是由Object[]构成，Linkedlist是由双向链表，Vector底层就是Object[]构成。

3. 是否支持随机访问：ArrayList和Vector支持高效随机访问，但是Vector所有方法都加了同步锁，所有性能较差，LinkedList不支持高效的随机访问。

4. 插入删除事件复杂度：ArrayList和Vector默认插入到数组末尾，时间复杂度O(1)，但是插入或删除指定位置i的时间复杂度为O(n-i)（即：i后面的元素需要位移）；LinkedList插入和删除指定节点对象时，时间复杂度是O(1)，但是插入和删除指定位置的元素时时间复杂度近似为O(n/2)（即：需要先判断i是否大于n/2来判断是前序遍历还是后续遍历半个链表来定位i，才能进行插入和删除操作）

### 1.5.2、List、Set、Map的区别

1. List：是可以重复的，有序集合，继承Collection接口。

2. Set：不允许重复的集合，继承Collection接口，TreeSet有序，HashSet无序。

3. Map：存储键值对，Key不允许重复，但是Value允许重复，不继承Collection接口。

### 1.5.3、HashMap、LinkedHashMap、Hashtable、ConcurrentHashMap的区别

1. 线程安全性：HashMap、LinkedHashMap是非线程安全的，Hashtable和ConcurrentHashMap是线程安全的，其中Hashtable是使用Synchronized的整体对象锁，效率较低目前已舍弃使用；

ConcurrentHashMap在jdk1.7的时候使用分段锁，在jdk1.8之后使用CAS和Synchronized锁直接对链表和红黑树进行同步操作，当插入对象的hash对应的数组槽位为null时，使用CAS进行操作，否则使用Synchronized对当前的链表或红黑树进行加锁（jdk1.6之后对Synchronized进行优化，性能提升了很多），获取数据时使用getObjectVolatile保证数据一致性。

2. 底层数据结构：HashMap和HashTable在jdk1.8之前使用数组+链表的结构，在jdk1.8之后HashMap在链表长度大于8时转为红黑树，Hashtable没有变；ConcurrentHashMap在jdk1.7使用分段数组+链表，在jdk1.8之后使用和HashMap一样的结构，LinkedHashMap是在HashMap基础上套了层链表，使其有序。

3. 对Null Key和Null Value的支持：HashMap和LinkedHashMap的Key和Value都支持Null，但是只能一个Key为Null，Hashtable和ConcurrentHashMap的Key和Value都不支持Null。

4. 扩容机制：HashMap、LinkedHashMap和ConcurrentHashMap默认容量为16，每次扩容为2n倍，指定容量时，实际容量是大于等于指定容量2的幂次方；Hashtable默认容量是11，每次扩容为2n+1倍，指定容量时实际容量就是指定容量。

### 1.5.4、HashMap源码解析

https://www.cnblogs.com/ZhaoxiCheung/p/Java-HashMap-Source-Analysis.html

### 1.5.5、ConcurrentSkipListMap与TreeMap区别

（参考：<https://www.cnblogs.com/java-zzl/p/9767255.html>）

1. ConcurrentSkipListMap：线程安全，适合大并发情况下使用，数据结构为跳表，key是有序的且不能为空，value不能为null。

![]({{"\img\posts_img\technology\java\knowledgeCollection\0.8062861681475306.png"|prepend:site.url}})

2. TreeMap：非线程安全，在结合Collections.synchronizedSortedMap方法下，适合小并发，数据结构为红黑树，key是有序的且不能为空，value允许为null。

(参考：<https://cloud.tencent.com/developer/article/1522774>)

### 1.5.6、Queue

![]({{"\img\posts_img\technology\java\knowledgeCollection\0.34375776373510214.png"|prepend:site.url}})

#### 1.5.6.1、LinkedBlockingQueue

是一个采用Reentrantlock（2把锁）保证线程安全的阻塞队列，它实现了BlockingQueue接口，采用**链表**实现的，FIFO原则排序，队列默认大小为Integer最大值，可以指定大小。

1. put(E)方法：在队尾添加元素，如果队满就一直等到，直到有空位为止。（阻塞队列特有）

2. offer(E)方法：在队尾添加元素，如果队满就返回false，否则返回true。

3. offer(E,long,TimeUnit)方法：在队尾添加元素，当队满就等待指定时间，如果超时且失败就返回false，否则返回true。

4. take()方法：获取队首元素并删除队首，如果队列为空，则一直等待。（阻塞队列特有）

5. poll()方法：获取队首元素并删除队首，如果队列为空，则返回null。

6. poll(long,TimeUnit)方法：获取队首元素并删除队首，当队列为空则等待指定时间，如果超时且依旧为空就返回null。

7. peek()方法：获取队首元素但不删除队首，如果为空队列，则返回null。

8. remove(E)方法：移除指定值的节点，移除成功返回true，否则返回false。

#### 1.5.6.2、ArrayBlockingQueue

是一个采用Reentrantlock（一把锁）保证线程安全的阻塞队列，它实现了BlockingQueue接口，采用**数组**实现，FIFO原则排序，队列必须指定大小。

调用方法同LinkedBlockingQueue
1. removeAt(int)方法：删除指定下标的元素，如果下标元素不是队首，那下标之后的元素都要整体移动。

#### 1.5.6.3、SynchronousQueue

（参考：<https://www.jianshu.com/p/376d368cb44f>）

是一个容量为0且使用LockSupport.park保证线程安全的阻塞队列，每次删除都需要等待插入，每次插入都需要等待删除。
1. remove(Object)方法：直接返回false。

#### 1.5.6.4、ConcurrentLinkedQueue

是一个采用CAS操作保证线程安全的非阻塞队列，它实现了Queue接口，采用FIFO原则排序，数据结构采用**链表**，队列可以无限大，不能指定大小，队列大小是通过遍历链表来获得效率较差。

1. add(E)和offer(E)方法：在队尾添加元素，如果插入成功则返回true，否则一直等待。

2. poll()方法：获取队首元素并删除队首，如果队列为空，则返回null。

3. peek()方法：获取队首元素但不删除队首，如果为空队列，则返回null。



#### 1.5.6.5、PriorityBlockingQueue

（参考：<https://www.jianshu.com/p/fd26c91cd2a0>）

是一个采用Reentrantlock保证线程安全的阻塞队列，它实现了BlockingQueue，数据结构为由数组实现的**无界二叉堆优先级队列**，队列可以无限扩容，可以指定大小，默认11，扩容时：容量n<64则2n+2，>=64后则1.5n。

##### 1.5.6.5.1 二叉堆

二叉堆是完全二叉树或近似完全二叉树，其父节点的键值总是和子节点的键值保持一定的序关系，且每个节点的子节点也是一个二叉堆。

![]({{"\img\posts_img\technology\java\knowledgeCollection\0.9260143840909643.png"|prepend:site.url}})

#### 1.5.6.6、PriorityQueue

是一个非线程安全的队列，继承了AbstractQueue，其他实现和PriorityBlockingQueue一样，只是操作不加锁。

## 1.6、数据类型

### 1.6.1、BigDecimal

是一个java.math包中用于高精度计算超过16位有效位的类，BigInteger类是针对大整数。

（参考：<https://juejin.im/post/5d81e96b51882507ba2269d8>）

## 1.7 IO流分类

1. 按处理方式分类

![]({{"\img\posts_img\technology\java\knowledgeCollection\4dd3c6ea-5b4d-4102-96b9-361a69164cdd.jpg"|prepend:site.url}})

2. 按对象分类

![]({{"\img\posts_img\technology\java\knowledgeCollection\d8c04b27-0142-425e-9485-4c833d534fe9.jpg"|prepend:site.url}})


# 2、JVM

## 2.1、字节码(.class)文件加载过程

（参考：https://www.cnblogs.com/linlf03/p/10795563.html）

## 2.2、JVM内存划分

![]({{"\img\posts_img\technology\java\knowledgeCollection\76bc0116-8ee5-4bf1-9401-8c64cc9da76c.png"|prepend:site.url}})


![]({{"\img\posts_img\technology\java\knowledgeCollection\13884d30-cb5d-483a-84a2-69e635df6be3.png"|prepend:site.url}})

## 2.3、GC

### 2.3.1、GC算法有哪些

1. **标记-清除**：首先标记需要回收的对象，然后统一清除被标记的对象，它是最基础的收集算法。缺点：1、效率较低。 2、导致内存不连续。

2. **复制算法**：它为了解决效率问题，将内存划分为大小相同的2块，每次只使用其中一块，当这一块内存使用完之后，就将存活的对象复制到另一块，最后清理这一块的空间。

3. **标记-整理**：根据老年代特点特出的一种算法，标记过程还是和“标记-清除”一样，但后续是将存活的对象向一端移动，然后直接清理掉端边界以外的内存。

4. **分代收集**：当前虚拟机都采用了该算法，即根据对象的GC年龄来划分内存，一般分为老年代和新生代，新生代一般采用“复制算法”，老年代使用“标记-复制”或“标记-清除”算法。

### 2.3.2、GC收集器有哪些

![]({{"\img\posts_img\technology\java\knowledgeCollection\0.5097002651463372.png"|prepend:site.url}})

（参考:<https://www.cnblogs.com/super-jing/p/10795099.html>)

1. **Serial收集器**：它是一个单线程的新生代收集器，不仅在垃圾收集时单线程处理，还会暂停其他所有工作线程（Stop-The-World），采用复制算法，它是JVM在Client模式下的新生代默认收集器。

2. **ParNew收集器**：它是Serial收集器的多线程版本，也会Stop-The-World，也是新生代收集器，采用复制算法，当JVM设置为CMS时，它是默认的新生代收集器。

	1. **并行**：多条回收线程同时并行执行，但是用户线程处于等待状态。

	2. **并发**：用户和回收线程同时执行（并不一定是并行，可能是交替执行），用户线程继续工作，回收线程运行在另一个CPU上。

3. **Parallel Scavenge收集器**：它是并行的新生代收集器，采用复制算法，它可以控制JVM的吞吐量，可以配置“停顿时间”和“吞吐量大小”。

4. **Serial Old收集器**：他是Serial的老年代版本，同样是单线程，使用“标记-整理”算法。主要用于JVM的Client模式下，在Server模式下（1）在JDK1.5之前与Parallel Scavenge搭配使用 （2）作为CMS收集失败后的备选方案

5. **Parallel Old收集器**：他是Parallel Scavenge的老年代版本，使用“标记-整理”算法。在JDK1.6中才出现。

6. **CMS（Concurrent Mark Sweep）收集器**：（JDK1.5）它是并发执行的老年代收集器，采用“标记-清除”算法，目标是减少用户线程的停顿，主要采用4个步骤：

	1. 初始标记：stop-the-world，标记GCRoots（被GCRoots引用的对象不被GC回收）能直接关联的对象，时间很短。

	2. 并发标记：恢复用户线程，标记GCRoots可达性分析的过程。

	3. 重新标记：stop-the-world，修正并发标记期间用户线程使标记发生变化的对象，时间介于1-2步骤之间。

	4. 并发清除：恢复用户线程，同时清理死亡对象。

优点：并发收集，停顿少。

缺点：（1）对CPU资源敏感，在并发阶段可能会导致用户线程变慢，导致吞吐量降低（2）无法处理悬浮垃圾，导致“Concurrent Mode Failure”而进行Full GC（3）由于采用“标记-清除”算法，导致大量碎片。

7. **G1收集器**：（JDK1.7）他是唯一一个同时适用于新生代和老年代的收集器，有如下特点：

	1. **并发与并行**：G1能够很好的利用多CPU、多核的优势，缩短stop-the-world的停顿时间。

	2. **分代收集**：保留了年代划分的概念，也分有年轻代和老年代，但是它们不是物理隔离的了，而是由若干个（5%-60%）可以不连续的Region的集合组成。

	3. **空间整合**：从整体来看是“标记-整理”算法，从2个region间来看是“复制算法”。

	4. **可预测停顿**：他和CMS都是以降低停顿时间为目的，还建立了可预测的停顿时间模型，因为他不像以往的收集器都是对整个年轻代或老年代内存进行回收，而是将内存划分为若干个region，并且跟踪Region中的垃圾堆积价值的大小，维护一个优先列表，最后根据允许的收集时间来回收最大价值的Region。

(参考：<https://blog.csdn.net/coderlius/article/details/79272773>)

8. **ZGC**：（JDK11）可拓展的低延时垃圾收集器，支持4T最大堆、最大GC停顿10ms、吞吐量影响不超过15%

（参考：<https://www.jianshu.com/p/4e4fd0dd5d25>）

## 2.4、类加载器有哪些

1. BootstrapClassLoader：启动类加载器，由C++实现，是最顶层的加载器，主要加载%JAVA_HOME%lib或者-XBootClassPath参数指定目录下的jar包和类。

2. ExtClassLoader：拓展类加载器，继承自ClassLoader，主要加载%JAR_HOME%libext或-Djava.ext.dirs参数指定目录下的jar包和类。

3. AppClassLoader：应用程序类加载器，继承自ClassLoader，主要加载classpath目录下的jar包和类。

## 2.5、双亲委派模式

每个类都有一个对应的类加载器，默认情况下ClassLoader都采用双亲委派模式，即：加载类的时候，先判断当前类是否已经被加载过，如果已经加载，则直接返回，否则依次委派父类加载器直到顶层的启动类加载器来处理，当父类加载器无法处理时，才会使用自定义类加载器处理。

* 优点：（1）防止类重复加载（2）防止核心类被篡改

## 2.6、自定义类加载器的目的

（参考：<https://blog.csdn.net/weixin_39746740/article/details/85236373>）

## 2.7、JVM分析工具

（参考：<https://www.cnblogs.com/zxf330301/articles/5256669.html>）



# 3、数据库

## 3.1、MySQL

（参考：<https://zhuanlan.zhihu.com/p/77383599>）

### 3.1.1、聚集索引、非聚集索引、B、B+树的关系

1. 聚集索引：数据行的物理顺序与列值（一般是主键的那一列）的逻辑顺序相同，一个表中只能拥有一个聚集索引。以mysql为例，Innodb引擎中聚集索引就是主键索引，在实现方式中B+Tree的叶子节点上的data就是数据本身，key为主键，如果是一般索引的话，data便会指向对应的主索引。

2. 非聚集索引：该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同，一个表中可以拥有多个非聚集索引，在实现方式中B+Tree的叶子节点上的data并不是数据本身，而是数据存放的地址。在MySIAM引擎中主索引和辅助索引没啥区别，只是主索引中的key一定得是唯一的。

### 3.1.2、MySIAM与InnoDB的区别

1. 是否支持行级锁：MySIAM只有表级锁，而InnoDB支持行级锁。

2. 是否支持事务与崩溃后的安全恢复：MySIAM不支持事务但查询速度快，也不支持恢复，而InnoDB支持事务且支持崩溃后恢复

3. 是否支持外键：MySIAM不支持，而InnoDB支持

4. 是否支持MVCC：只有InnoDB支持，对于高并发事务，MVCC比单纯的事务更高效，MVCC只在**已提交读**和**可重复读**2个隔离级别时工作。不同数据库的MVCC实现并不统一，可以使用**悲观锁**和**乐观锁**实现。

### 3.1.3、MVCC

（参考：<https://www.jianshu.com/p/8845ddca3b23>）

MVCC（Muti-Version Concurrency Control）多版本并发控制，在MySQL的InnoDB引擎中，是只在**已提交读RC**和**可重复读RR**2个隔离级别的事务下对于select操作会访问版本链中记录的过程。用于在不加锁的情况下，提高数据库的并发能力。

1. 版本链：是在每张表中有3个隐藏列**事务id**、**回滚指针**和**隐藏自增ID**。

	1. 事务Id（db_trx_id）：最近修改（修改/插入）事务ID，记录创建这条记录或最后一次修改该记录的事务ID。

	2. 回滚指针(db_roll_pointer)：指向这条记录的上一个版本地址，存储于undo日志中。

	3. 隐藏自增ID(db_row_id):是隐藏的主键，如果数据表没有主键，则InnoDB会自动以该字段创建一个聚簇索引。

2. ReadView：是事务进行快照读时产生的一个读视图，并记录了当前快照时活跃的事务ID列表，通过这个列表来判断当前查询是否可见，原则是用trx_id<up_limit_id 或 trx_id < low_limit_id && 活动列表不包含trx_id，则表示该trx_id的记录可见；**RC**是每次快照读都会创建ReadView，**RR**是事务开始的第一个快照读创建一个ReadView，该事务之后的快照读不在创建新的。

### 3.1.4、隔离级别

（参考：<https://www.cnblogs.com/balfish/p/8298296.html>）


![]({{"\img\posts_img\technology\java\knowledgeCollection\0.3092687476451086.png"|prepend:site.url}})

### 3.1.5、锁分类

1. 按照粒度分类

	1. 表级锁：是锁定整张表，粒度较大，并发低，消耗资源少，不会死锁，MySIAM和InnoDB都支持。InnoDB还支持2个表级锁：

		1. 意向共享锁（IS）：在插入共享锁之前需要先取得该表的IS锁，表示准备加入共享锁。
		2. 意向排它锁（IX）：在插入排它锁之前需要先取得该表的IX锁，表示准备加入排它锁。

	2. 行级锁：是锁定需要操作的一行记录，粒度小，并发高，消耗资源多，加锁慢，会产生死锁，只有InnoDB支持

		1. Record lock：对索引项加锁，锁定符合条件的行，其他事务不能再修改、删除改行。
		2. Gap lock：称为**“间隙锁”**，对索引行的前后间隙加锁，锁定一定范围的数据，其他事务无法在该范围内插入数据，防止增加幻影行。

	3. Next-key lock：是Gap lock + Record lock的结合，锁定一个范围，并锁定记录本身，对于查询都是采用该方法，主要解决幻影读。
（参考：<https://www.cnblogs.com/zhoujinyi/p/3435982.html>）

2. 按照是否可写分类

	1. 共享锁（S）：又被称为读锁，添加了读锁后可以再次加读锁，但不能添加排它锁，添加了该锁只能查询数据。
	2. 排它锁（X）：又被称为写锁，添加了写锁后，不能再添加其他锁，但是添加了该锁可以查询、插入、删除、更新数据。

### 3.1.6、快照读和当前读

1. 快照读：不加锁的select就是快照读，即不加锁的非阻塞读。所以快照读可能读到的并非当前最新的数据，是基于MVCC实现的。

2. 当前读：是通过select lock in share model（共享锁）；select for update、update、insert、delete（排它锁）这些操作时都是当前读，会对读的记录加锁，可以获取到当前最新的记录。

（参考：<https://www.cnblogs.com/cat-and-water/p/6427612.html>）

### 3.1.7、大表优化方案

（参考：<https://segmentfault.com/a/1190000006158186>）

1. 单表优化：

	1. 字段优化：尽量使用适当长的字段，如用SMALLINT能满足就不要用INT，VARCHAR的长度只分布实际使用长度，使用TIMESTAMP代替DATETIME等

	2. 索引优化：索引不是建的越多越好，可以根据where和order by字段来建立，并通过explain进行分析是否使用了索引，值很稀少的字段不适合建索引，如：性别；最好不要使用外键；

	3. sql优化：开启慢查询日志slow_query_log=1来分析查询较慢的sql；不做列运算；不用select *；or写成in；少用join；使用同类型比较，如：'123'='123'；避免使用!=和<>；连续值使用between替代in；不要全表查询使用limit。

	4. 引擎选择：MySIAM适合Select密集型的表，InnoDB适合insert和update密集型的表。

	5. 参数调优：提高数据缓存innodb_buffer_pool_size、索引缓存key_buffer_size、查询缓存query_cache_size等。

	6. 提升硬件：增强服务器的cpu和存储读写速度都对数据库有提升。

	7. 读写分离：通过主库写入数据，从库读取数据，主库一般为1台，减少复杂度

	8. 缓存：缓存可以发生在

		1. 数据库层：通过系统参数调节
		2. 数据访问层：使用Mybatis对sql的缓存，主要缓存持久化对象

		3. 应用服务层：使用编程的手段对缓存做更精细的控制，主要缓存数据传输对象

		4. web层：主要做页面缓存如CDN等

		5. 浏览器客户单端：主要是浏览器的缓存

	9. 垂直拆分：就是将一张表的多个字段拆分到不同的表中，通过主键来关联，可以减少单行数据，减少IO，充分利用缓存，但是依然存在单表数据量过大问题

	10. 水平拆分：

		* 通过表分区方式，减少单个表的压力，有四种分区list、range、key、hash等方式，但是单表最多支持1024个分区表

		* 通过分布式数据分片方式：通过某种策略将数据分片存储，可以用库内分表和分库等方式，能支持非常大数据量

		1. 库内分表：只能解决单表数据量过大问题，但是数据还在单台服务器上，性能提升不大，前面的分区表也是库内分表。

		2. 分库：可以将数据放在不同的数据库中，进行分布式存储，但是需要保证事务一致性问题、夸库join、夸库合并分页等问题，主要的解决方案分：

			1. 客户端框架：主要分片逻辑在应用层，通过修改或封装jdbc层实现，如：当当网的shardingJDBC、阿里的TDDL等

			2. 中间件代理框架：在应用和数据库之间加了一层代理框架，分片逻辑在中间件中，如：MyCat、360的Atlas、网页的DDB等方案。



### 3.1.8、连接池

（参考：<https://www.jianshu.com/p/b9b98ac3e010>）

![]({{"\img\posts_img\technology\java\knowledgeCollection\0.14999437025521178.png"|prepend:site.url}})

### 3.1.9、分库分表的Id策略

1. UUID：太长不适合主键

2. 数据库自增ID：不同库使用不同的步长和初始值来实现唯一

3. Redis生成ID：性能好，比较灵活，但是引入了新的系统，增加了系统复杂度

4. Twitter的snowflake算法：是一种划分命名空间来生成ID的算法，将64位划分为不同的段来表示不同机器的时间（参考：<https://www.cnblogs.com/relucent/p/4955340.html>）

5. 美团的leaf分布式ID生产系统：能保证全局性，唯一性、趋势递增、单调递增、信息安全，单需要以来关系型数据库和zookeeper等中间件（参考：<https://tech.meituan.com/2017/04/21/mt-leaf.html>）

### 3.1.10、SQL语句的执行过程

1. mysql主要分为server层和引擎层：server层主要包括连接器、查询器、分析器、优化器、执行器，同时还有一个日志模块（binlog），这个日志模块所有引擎公用，redolog只有InnoDB引擎独有；引擎主要包括：InnoDB、MySIAM、Memory等。

2. SQL语句的执行过程：

	1. 查询语句：连接器权限校验->查询缓存->分析器->优化器->权限校验->执行器->引擎

	2. 更新语句：分析器->权限校验->执行器->引擎->redolog prepare -> binlog -> redolog commit
（参考：<https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485097&idx=1&sn=84c89da477b1338bdf3e9fcd65514ac1&chksm=cea24962f9d5c074d8d3ff1ab04ee8f0d6486e3d015cfd783503685986485c11738ccb542ba7&token=79317275&lang=zh_CN%23rd>）

### 3.1.11、一条SQL执行很慢的原因

1. 偶尔慢：说明这条SQL书写没有问题，可能有如下原因

	1. 数据库在刷新脏页：（1）当redolog满了，需要暂停其他操作将log中的内容写入磁盘（2）当内存不足时，查询需要申请更多的内存，就需要淘汰更多的内存数据页，刚好在淘汰脏页时，就需要将数据输入磁盘中（3）当系统认为当前是空闲时，就会刷新脏页。（4）数据库在关闭时，需要刷新所有脏页。

	2. 拿不到锁：我们需要查询的表或某行数据被加了锁，所以需要等待锁释放。

2. 一直慢：可能SQL语句有问题

	1. 没使用索引：（1）可能是没创建索引（2）有索引但是没用到，如：字段进行了运算或使用了函数操作，导致无法使用索引

	2. 数据库选错了索引：由于统计的失误，导致系统没有走索引，而是全表扫描，即当语句中有多个索引的时候，系统可能选错索引



（参考：<https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485185&idx=1&sn=66ef08b4ab6af5757792223a83fc0d45&chksm=cea248caf9d5c1dc72ec8a281ec16aa3ec3e8066dbb252e27362438a26c33fbe842b0e0adf47&token=79317275&lang=zh_CN%23rd>）

### 3.1.12、书写高质量SQL的30条建议

(参考：<https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486461&idx=1&sn=60a22279196d084cc398936fe3b37772&chksm=cea24436f9d5cd20a4fa0e907590f3e700d7378b3f608d7b33bb52cfb96f503b7ccb65a1deed&token=1987003517&lang=zh_CN%23rd>)

### 3.1.13、redo log、undo log、binlog区别

（参考：<https://www.jianshu.com/p/57c510f4ec28>）

### 3.1.14、事务的特性

ACID：原子性（Atomic）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）

（参考：<https://blog.csdn.net/chosen0ne/article/details/10036775>）

### 3.1.15、读写分离

（参考：<https://zhuanlan.zhihu.com/p/68035302>）

### 3.1.16、分布式事务一致性问题解决方案

（参考：<https://www.cnblogs.com/dyg0826/p/11127992.html>  或 <https://blog.csdn.net/CharlesYooSky/article/details/102773548>）





## 3.2、Redis

### 3.2.1、基本命令

（参考：<http://doc.redisfans.com/>）

### 3.2.2、redis的线程模型

redis内部使用了文件事件处理器（File Event Handler），由于该事件处理器是单线程的，且采用IO多路复用机制同时监听多个Socket，所以Redis采用的是单线程IO复用模型。

（参考：<https://www.javazhiyin.com/22943.html>）

### 3.2.3、缓存穿透、雪崩、热点key

（参考：<https://blog.csdn.net/u012240455/article/details/81843870>）

1. 缓存穿透：大量请求的key在缓存中不存在，导致请求直接查询DB，从而失去了缓存的意义。

	**解决方法**：
	1. 缓存无效key：将缓存和DB中都没有的key进行缓存，并设置过期时间，缺点是浪费空间，不能根本解决问题。
	
	2. 布隆过滤器：把所有请求可能存在的key都放在布隆过滤器中，当请求的key不在过滤器中时，则直接返回错误。Bloom过滤器是由位数组和hash函数组成的数据结构，它只能判断数据一定不存在，不能判断数据一定存在。
（参考：<https://github.com/Snailclimb/JavaGuide/blob/master/docs/dataStructures-algorithms/data-structure/bloom-filter.md> 或 <https://www.cnblogs.com/clarencezzh/p/11439193.html>）

2. 缓存雪崩：同一时间大量缓存失效，导致请求落在DB上，造成数据库短时间承受大量请求而崩溃。
	**解决方法**：

	1. 在缓存失效后，使用锁或队列限制读取数据库的线程数。
	
	2. 如果能预知并发的情况下，可以通过reload机制手动预先更新缓存。
	
	3. 不同的key设置不同的过期时间，尽量让缓存失效时间均匀。
	
	4. 做二级缓存或者双缓存策略，A1为原始缓存，A2为拷贝缓存，A1的失效时间比A2短，当A1失效时，可以访问A2。
	
	5. 尽量保证Redis集群的高可用，发现机器宕机能尽快补上，并设置合理的内存淘汰机制。

3. 热点key：一个热点信息，在大量并发请求时，缓存过期，刚好构建该缓存需要复杂计算或多个依赖，导致系统崩溃。

	**解决方法**：

	1. 使用互斥锁：一次只能一个线程来构建缓存，其他线程重新从缓存获取数据
	
	2. 提前使用互斥锁：给key设置2个过期时间timeout1和timeout2且timeout1<timeout2，timeout2为Redis实际过期时间，当到达timeout1时，重构缓存
	
	3. 永不过期：设置Redis过期时间，通过异步线程来设置逻辑过期，并更新缓存
	
	4. 资源保护：使用hystrix进行限流

### 3.2.4、内存淘汰机制

当内存不足以容纳新写入的数据时，有以下策略：

1. volatile-lru：在设置了过期时间的key空间中，移除最近最少使用的数据（一般不用）

2. volatile-ttl：在设置了过期时间的key空间中，移除将要过期的数据

3. volatile-random：在设置了过期时间的key空间中，随机移除数据

4. allkeys-lru：在所有key空间中，移除最近最少使用的数据（最常用）

5. allkeys-random：在所有key空间中，随机移除数据（一般不用）

6. no-eviction：禁止驱逐数据，插入新数据报错



4.0版本增加了以下2个策略：

1. volatile-lfu：在设置了过期时间的key空间中，移除最不经常使用的数据

2. allkeys-lfu：在所有key空间中，移除最不经常使用的数据



### 3.2.5、Redis的持久化

（参考：<https://www.cnblogs.com/itdragon/p/7906481.html>）

Redis的持久化分为 快照（RDB）和文件追加（AOF）

1. 快照持久化（RDB）：Redis默认开启方式，在通过指定时间内，发生指定次数的数据变更，来将内存中的数据写入到磁盘中。适合大规模的数据恢复，但一致性和完整性较差。

2. 文件追加持久化（AOF）：可以每秒或Always将操作日志追加到AOF文件中。它比RDB完整性高，但是记录较多，会影响数据恢复的效率。

	* AOF重写机制：当AOF大小达到阈值时（AOF大小超过上次rewrite后文件大小的设定百分比且大小超过设定值），Redis会新创建一个线程直接从内存中读取数据写入到新的AOF临时文件中，写完后替换原有AOF文件，达到瘦身的目的。

### 3.2.6、Redis数据结构及应用场景

（参考：<https://www.cnblogs.com/jasonZh/p/9513948.html>）

### 3.2.7、Redis底层数据结构

（参考：<https://juejin.im/post/5d71d3bee51d453b5f1a04f1>）

### 3.2.8、Redis集群方式

（参考：<https://www.cnblogs.com/L-Test/p/11626124.html>）

### 3.2.9、保证并发时数据的正确性

（参考：<https://blog.csdn.net/qiyangjlu/article/details/100996832> 或 <https://blog.csdn.net/u011832039/article/details/78924418>）



# 4、网络

（面试看这个就够了：<https://juejin.im/post/5b7be0b2e51d4538db34a51e>）

## 4.1、tcp建立和关闭连接过程



![]({{"\img\posts_img\technology\java\knowledgeCollection\0.16593814931876794.png"|prepend:site.url}})

## 4.2、网络分层

1. OSI体系结构：物理层、链路层、网络层、传输层、会话层、表示层、应用层

2. TCP/IP体系结构：网络接口层、网际IP层、传输层、应用层

3. 五层协议体系结构：物理层、链路层、网络层、传输层、应用层

	1. 应用层：直接为应用进程之间的通信和交互提供服务，对于不同的应用需要不同的应用层协议，如：DNS、HTTP、FTP、SMTP等。

	2. 传输层：为2台计算机中进程提供通用的数据传输服务，由于一台计算机有多个进程，所以传输层一般具有重用和分用功能。**“重用”**即应用层多个进程可以同时使用传输层协议，**“分用”**即传输层在接收到数据之后可以分发给相应的应用层进程。

	3. 网络层：保证2台计算机在路由和交互节点之间数据的及时送达，对传输层的数据包或报文段封装成分组（即：包）进行传送。

	4. 链路层：2台主机间有很多一段一段的链路，该协议保证2个相邻节点之间的帧传输，他有对帧的查错和纠错的功能。

	5. 物理层：实现2台计算机之间比特流的透明传送，尽可能屏蔽物理传输介质与物理设备间的差异。



## 4.3、TCP与UDP的差别

![]({{"\img\posts_img\technology\java\knowledgeCollection\d8caf893-ec36-4d37-9e53-23566c074e89.jpg"|prepend:site.url}})

1. 头部区别：<https://blog.csdn.net/dangzhangjing97/article/details/80991965>

## 4.4、TCP协议如何保证可靠传输

通过数据分块、序列号、校验和、确认应答、重发控制、连接管理、窗口控制、流量控制、拥塞控制实现可靠。

（参考：<https://www.jianshu.com/p/6aac4b2a9fd7>）

## 4.5、浏览器输入URL到返回页面的过程

![]({{"\img\posts_img\technology\java\knowledgeCollection\c4e107ef-0643-4d3b-960c-35bb53fecd7d.jpg"|prepend:site.url}})

**注意**：

1. OPSF写错了，应该是OSPF（Open Short Path First）开放式最短路径优先协议。

2. ARP（Address Resolution Protocol）地址解析协议，将IP解析为MAC地址。

## 4.6、ARQ协议

（Auto Repeat Request）自动重复请求协议，主要通过确认、超时机制实现数据的重发。

1. 停止等待ARQ协议：每发完一个分组就会停止等待对方确认（回复ACK），如果超时未收到确认则重复发送分组，直到收到确认才继续发送下一分组。接收方收到重复分组直接丢弃。

2. 连续ARQ协议：发送方维持一个发送窗口，连续发送分组，不需要等待对方确认，接收方一般累计确认，对按顺序到达的最后一个分组发送确认，表明到这个分组为止的所有分组已全部收到。

## 4.7、TCP的粘包/拆包原因及其解决方法是什么

1. 原因：TCP是字节流，会把应用发送的数据拆分为大小不能的无结构字节流分组，没有边界，而且TCP头部也没有记录分组大小，所以就会导致

	1. 要发送的数据大于缓冲区剩余空间，被拆包

	2. 要发送数据大于缓冲区最大空间长度，被拆包

	3. 要发送数据小于缓冲区大小，TCP会将多次写入缓冲区的数据一次发送出去，产生粘包

	4. 接收方应用层没有及时读取缓冲区数据，导致粘包

2. 解决方法：关键在于如何给每个数据包添加边界

	1. 发送端给每个数据包添加首部，首部中至少包含数据包的长度

	2. 发送端给每个数据包之间添加特殊字符进行分隔

	3. 发送端给每个数据包封装为固定长度，不足可以用0填充



## 4.8、IO流

### 4.8.1 IO多路复用中select、poll、epoll的区别

（参考：<https://www.cnblogs.com/aspirant/p/9166944.html> 或 https://blog.csdn.net/lsgqjh/article/details/65629609 或 <https://www.jianshu.com/p/af202026ffc5> 或 <https://blog.csdn.net/sehanlingfeng/article/details/78920423>）

1. select：
缺点：两次拷贝耗时、轮询所有fd耗时，支持的文件描述符太小
优点：跨平台支持

2. poll
优点：连接数（也就是文件描述符）没有限制（链表存储）
缺点：大量拷贝，水平触发（当报告了fd没有被处理，会重复报告，很耗性能）

3. epoll
LT：延迟处理，当检测到描述符事件通知应用程序，应用程序不立即处理该事件。那么下次会再次通知应用程序此事件。
ET：立即处理，当检测到描述符事件通知应用程序，应用程序会立即处理。
ET模式减少了epoll被重复触发的次数，效率比LT高。我们在使用ET的时候，必须采用非阻塞套接口，避免某文件句柄在阻塞读或阻塞写的时候将其他文件描述符的任务饿死
	* 优点：
		1. 没有最大并发连接的限制
		2. 只有活跃可用的fd才会调用callback函数
		3. 内存拷贝是利用mmap()文件映射内存的方式加速与内核空间的消息传递，减少复制开销。（内核与用户空间共享一块内存）

### 4.8.2 BIO,NIO,AIO 有什么区别?

1. BIO（Blocking I/O）：同步阻塞I/O模型，数据读写必须阻塞在一个线程内等待其完成。试用于单机1000以内的低并发，开发模型简单。

2. NIO（Non-Blocking I/O）：同步非阻塞I/O模型，它支持面向缓存的，基于管道的I/O操作方式，在java.nio包中提供了Channel、Selector、Buffer抽象类，在网络层提供了SocketChannel和ServerSocketChannel两中不同的套接字实现，并且支持阻塞和非阻塞2种方式。

3. AIO（Asynchronous I/O）：异步非阻塞I/O模型，也是NIO2，他是基于事件和回调机制实现，也就是应用操作之后会直接返回，不会阻塞在哪里，当后台处理完成后，操作系统会通知相应线程进行后续操作。

（详细解答：https://www.cnblogs.com/crazymakercircle/p/10225159.html）



# 5、算法

**五大特征**：

1. 有穷性：算法必须在执行有限次数之后停止

2. 确切性：算法的每一步必须有确切的意义

3. 输入项：0项或多项

4. 输出项：至少一项，没有输出的算法是没有意义的

5. 可行性

## 5.1、散列表--线性探测法

平均查找长度=总的查找次数/元素数

参见（https://www.cnblogs.com/ricklz/p/9006424.html）

## 5.2、哈夫曼编码

每次取最小的2个节点，相加之和替换原来的2个数到原有数组，再继续取最小的2个数相加，重复循环这个方法，最后构成的树左0右1。

**权值计算**：依次将 同一层级节点相加 * 深度 累加起来就是权值

（参考：<https://baijiahao.baidu.com/s?id=1651316704300729180&wfr=spider&for=pc> 或 <https://blog.csdn.net/yushupan/article/details/82735773>）

## 5.3、分布式事务算法

### 5.3.1、paxos算法

它是一个解决分布式一致性问题的选举算法。分为2阶段：

1. 第一阶段：

	1. Proposor提出预案编号N，并向一半以上的Acceptor发送**预案**编号N的prepare请求

	2. 如果一个Acceptor接收到**预案**编号N，且N大于该Acceptor已经响应所有prepare请求的编码，那么它会将已经响应的最大编号的**提案**返回给Proposor，同时该Acceptor也不再接收编号小于N的**提案**。

2. 第二阶段：

	1. Proposor对超过一半接受的**预案**返回值[N,V]再发送**提案**给Acceptor，注意：V为第一步收到响应中最大编号的**提案**的值，如果第一步响应中不包含**提案**，则Proposor可以定义任意值为V。

	2. 如果Acceptor收到一个编号为N的accept请求，只要该Acceptor没有对编号大于N的prepare请求做出过响应，它就接受该**提案**。

![Paxos算法流程]({{"\img\posts_img\technology\java\knowledgeCollection\0.9201595354970573.png"|prepend:site.url}})

（参考：<https://zhuanlan.zhihu.com/p/31780743> 或者 <https://www.cnblogs.com/linbingdong/p/6253479.html>）

#### 5.3.1.1、ZAB与Paxos区别

（参考：<https://www.jianshu.com/p/85709eaf25fc>）

### 5.3.2、2阶段、3阶段区别

（参考：<https://mp.weixin.qq.com/s?__biz=MzI5ODQ2MzI3NQ==&mid=2247487531&idx=1&sn=b3fbc4dee7cea4a78db062a4a656afdf&chksm=eca4296fdbd3a079a8e328ec7946ced7d1f94c0f105463743a8bee569bae6da00bf2133c3e1a&mpshare=1&scene=1&srcid=&sharer_sharetime=1564287693704&sharer_shareid=136c9cdacd0d016d5845301b60ff010d&pass_ticket=RmWTghz3ejQjrtJBFtb9Nmlh2VmwJLW4ao5zaOnxURyfffs0yKiYzoIh2Uywfebh#rd>）

## 5.4、KMP算法

（参考：<https://www.cnblogs.com/xiaokang01/p/11593513.html> 或 <https://blog.csdn.net/dark_cy/article/details/88698736> 或 <http://baijiahao.baidu.com/s?id=1659735837100760934&wfr=spider&for=pc>）

## 5.5、高斯求和公式

高斯求和公式：1+2+3+...+n = n(n+1)/2

# 6、数据结构

## 6.1、树

1. 满二叉树：深度为k，则节点数量为2^k-1

2. 完全二叉树：只有最后一层和次层可以为叶子节点，且最后层的叶子节点必须从做到右依次排列。

3. 二叉堆：它本质是完全二叉树，且每个节点的值大于等于左右子节点的值，称为**最大堆**；每个节点的值小于等于左右节点的值，称为**最小堆**。

4. 二叉排序（查找、搜索）树：左节点小于根节点，右节点大于跟节点，且每个节点的左右节点都是二叉排序树，且没有相同值的节点。

5. 平衡二叉树：是基于二叉排序树，只是其左右子树的深度不能大于1。

（参考：<https://baijiahao.baidu.com/s?id=1646617486319372351&wfr=spider&for=pc>）

6. 红黑树：基于二叉排序树，

	1. 根节点颜色为黑

	2. 红节点的左右子节点必须为黑色

	3. 所有叶子节点必须为黑色的空节点

	4. 根节点到所有叶子节点的黑节点数相同

7. B树（B-树）：如果该B树有M阶，他有如下特点

	1. 根节点的孩子数[2,M]

	2. 除根节点其余非叶子节点包含k-1个关键字，k个孩子，其中M/2 <= k <=M，向上取整

	3. 每个子叶节点包含k-1个元素，其中M/2 <= k <=M，向上取整

	4. 所有子叶节点都在同一层

	5. 每个节点有k-1个关键字时，会将节点拆为k段，分别指向k个儿子，同时满足查找树的大小关系。

（参考：<https://blog.csdn.net/login_sonata/article/details/75268075> 或 <https://mp.weixin.qq.com/s?__biz=MzI2NjA3NTc4Ng==&mid=2652079363&idx=1&sn=7c2209e6b84f344b60ef4a056e5867b4&chksm=f1748ee6c60307f084fe9eeff012a27b5b43855f48ef09542fe6e56aab6f0fc5378c290fc4fc&scene=0&pass_ticket=75GZ52L7yYmRgfY0HdRdwlWLLEqo5BQSwUcvb44a7dDJRHFf49nJeGcJmFnj0cWg#rd>）



8. B+树：它是B树的变种，因为比B树更矮胖IO次数更少，所以查询性能更好，与B树的区别有：



	1. 非子叶节点不保存数据，只用来索引，所有叶子节点存储指向数据的指针（B树所有节点都可以指向数据），且非叶子节点只存储孩子中最大（小）关键字
	
	2. 所有叶子节点在同一层且构成一个链表，可以按照关键字大小排序，所以通过链表进行范围查询（B树需要重复的中序遍历进行范围查询）

## 6.2、胜者数、败者树

（参考：<https://blog.csdn.net/whz_zb/article/details/7425152>）



# 7、操作系统

## 7.1、进程间通信

（参考：<https://www.jianshu.com/p/c1015f5ffa74>）

1. **匿名管道**：

	1. 只能用于父子、兄弟进程间通信

	2. 先进先出的无格式字节流，不能进行定位读写

	3. 只支持单向数据流(读/写)，没有名字

	4. 管道的缓冲区是有限的

2. **有名管道**

	1. 有名管道已磁盘文件的方式存在，可以实现本机任意两个进程通信

	2. 无名管道阻塞：无名管道无需打开，创建时直接返回文件描述符，在读写时确定对方的存在，否则退出。当缓冲区满时，写进程阻塞，当缓冲区空时，读进程阻塞。

	3. 有名管道阻塞：打开时需要确定对方的存在，否则将阻塞。即：以读方式打开某管道，在此之前必须有进程以写方式打开此管道，否则阻塞。但也可以一个进程以读写模式打开，则不会阻塞。

3. **信号(Signal)**：是Linux系统中用于进程间相互通信或操作的一种机制，如：软件异常、用户注销、Ctrl+C 退出等

4. **消息队列**：

	1. 是具有特定格式的消息链表，存在内存中并有消息队列标识

	2. 消息队列允许一个或多个进程向它写入和读取消息

	3. 管道和消息队列的通信数据都是先进先出的原则，但是消息队列可以实现消息的随机查询，也可以按照类型读取，比FIFO更有优势

	4. 目前主要有2种类型的消息队列：POSIX消息队列和System V消息队列，SystemV是内核持续的，只有内核重启或者人工删除队列才会删除。

5. **共享内存**：是一种效率高的通信方式，进程直接读写内存，不需要任何数据拷贝。对于管道和消息队列等通信方式，需要在内核与用户空间进行4次拷贝，而共享内存只拷贝两次：一次从输入文件到共享内存，另一次从共享内存输出到文件。实现主要有：mmap、POXIS共享内存、SystemV共享内存

6. **信号量**：信号量是一个非负整数的计数器，用于多进程对共享数据的同步访问。

7. **套接字**：是一种通信机制，像常见的：流套接字（TCP）、数据报套接字（UDP）和原始套接字（IP、ICMP）等





# 8、框架

## 8.1、Spring

（参考：<https://blog.csdn.net/a745233700/article/details/80959716> 或 <http://www.itsoku.com/archives>）

### 8.1.1、Spring解耦

SpringIOC使得获取依赖对象的过程由自身管理变为由IOC容器主动注入，利用依赖关系注入的方式降低了对象间的耦合性。

（参考：<https://blog.csdn.net/weixin_41866960/article/details/83994137>）

### 8.1.2、Spring特性

1. 控制反转（IOC）：这是一种设计理念，将对象的创建和组装的主动控制权交给Spring容器来进行管理，控制权被反转了，降低了系统耦合度，利于拓展和维护。

2. 依赖注入（DI）：表示Spring容器创建对象时给其设置依赖对象的方式。

3. 面向切面（AOP）：

### 8.1.3、Spring容器

![img]({{"\img\posts_img\technology\java\knowledgeCollection\befce336-f50e-452a-8ca8-9ac05c3dd4c1.png"|prepend:site.url}})


Spring容器负责对象实例化、对象初始化、对象依赖关系组装、对象销毁、对外提供对象查找等功能，对象的生命周期都由容器来控制，可以通过XML和Java注解的形式配置管理对象，常见的容器接口和对象有：BeanFactory接口、ApplicationContext接口、ClassPathXmlApplicationContext类、AnnotationConfigApplicationContext类等。

**创建bean的方式**：（1）通过反射调用构造方法 （2）通过静态工厂 （3）通过实例工厂 （4）通过FactoryBean接口

**bean的作用域**：（1）singleton：单例模式，整个IOC容器 （2）prototype：多例模式 （3）request：一次请求 （4）session：一次回话 （5）application：单例模式，整个web容器

（参考：<http://www.itsoku.com/article/260>）

### 8.1.4、Bean的生命周期

（参考：<http://www.itsoku.com/article/281#menu_9>）

![img]({{"\img\posts_img\technology\java\knowledgeCollection\0.23779089074195214.png"|prepend:site.url}})

1. **阶段1-Bean元信息配置阶段**：bean的元信息定义阶段，可以采用XML、注解、API、Properties等方式，bean注册者只识别BeanDefinition对象，不管哪种方式最终都会将bean转化为BeanDefinition对象，然后注册到Spring容器中。

2. **阶段2-Bean元信息解析阶段**：通过将XML、Properties、注解等方式的元信息解释转化为BeanDefinition对象

3. **阶段3-Bean注册阶段**：注册阶段需要用到一个重要接口BeanDefinitionRegistry，他的重要实现就是DefaultListableBeanFactory类。

4. **阶段4-Bean合并阶段**：将那些存在多层级的父子级关系的bean进行递归合并，最终得到一个RootBeanDefinition对象，通过getMergedBeanDefinition方法获取。

5. **阶段5-Bean Class加载阶段**：这个阶段就是将bean的class名称转换为Class对象的阶段。

6. **阶段6-Bean实例化阶段**：

	1. 实例化前阶段：容器在创建对象时AbstractAutowireCapableBeanFactory会遍历beanPostProcessors集合，如果类型是**InstantiationAwareBeanPostProcessor**接口，则调用postProcessBeforeInstantiation方法获取bean，如果获取到对象存在则返回bean，跳过spring自带的实例化。

	2. 实例化阶段：容器在创建对象时，AbstractAutowireCapableBeanFactory会调用SmartInstantiationAwareBeanPostProcessor接口的determineCandidateConstructors方法，该方法会返回候选的构造器列表，也可返回为空。

7. **阶段7-合并后的BeanDefinition处理**：spring会轮询BeanPostProcessors，依次调用MergedBeanDefinitionPostProcessor接口的postProcessMergedBeanDefinition方法，该接口有2个实现：

	1. AutowiredAnnotationBeanPostProcessor：对@AutoWired和@Value修饰的方法、字段进行缓存

	2. CommonAnnotationBeanPostProcessor：对@Resource修饰的字段、方法，@PostConstruct修饰的字段，@PreDestroy修饰的方法进行缓存

8. **阶段8-Bean属性设置阶段**：

	1. 实例化后阶段：InstantiationAwareBeanPostProcessor接口
postProcessAfterInstantiation方法返回false的时候，后续的Bean属性赋值前处理、Bean属性赋值都会被跳过了。

	2. Bean属性赋值前处理：如果InstantiationAwareBeanPostProcessor接口中的postProcessProperties和postProcessPropertyValues都返回空的时候，表示这个bean不需要设置属性，直接返回了，直接进入下一个阶段。

	3. Bean属性赋值处理：循环处理PropertyValues中的属性值信息，通过反射调用set方法将属性的值设置到bean实例中。

9. **Bean初始化阶段**：

	1. Bean Aware接口回调：会调用BeanNameAware, BeanClassLoaderAware, BeanFactoryAware接口的set方法

	2. Bean初始化前操作：会调用BeanPostProcessor的postProcessBeforeInitialization方法，若返回null，当前方法将结束。

	3. Bean初始化操作：会调用InitializingBean接口的afterPropertiesSet方法，然后在调用自定义的初始化方法

	4. Bean初始化后操作：会调用BeanPostProcessor接口的postProcessAfterInitialization方法，返回null的时候，会中断上面的操作。

10. **阶段10-所有单例bean初始化完成后阶段**：所有单例bean实例化完成之后，会调用SmartInitializingSingleton接口的afterSingletonsInstantiated方法

11. **阶段11-bean使用阶段**：调用getBean方法得到了bean之后进行使用操作。

12. **阶段12-bean销毁阶段**：

	1. 触发bean销毁的方式：调用AbstractAutowireCapableBeanFactory#destroyBean，ConfigurableBeanFactory#destroySingletons，ApplicationContext#close等方法。

	2. Bean销毁会依次执行：（1）DestructionAwareBeanPostProcessor#postProcessBeforeDestruction方法。（2）DisposableBean#destroy方法。（3）bean自定义销毁方法。



![img]({{"\img\posts_img\technology\java\knowledgeCollection\21907c71-0137-43dc-93ff-e8b545d02763.png"|prepend:site.url}})

### 8.1.5、Spring父子容器

（参考：<http://www.itsoku.com/article/284>）

1. **特点**：（1）父子容器相互隔离，可以存在相同beanName（2）父容器不能访问子容器，子容器可以访问父容器（3）子容器调用getBean方法会沿着容器向上查找，直到找到bean为止（4）子容器可以通过任何方式注入父容器中的bean，父容器不行

2. BeanFactory接口支持层次查找，ListableBeanFactory接口不支持层次查找，BeanFactoryUtils工具类中提供了大量使用方法，如：支持层次查找。

### 8.1.6、Spring事务传播性

1. PROPAGATION_REQUIRED：支持当前事务，如果当前无事务，则新建一个事务

2. PROPAGATION_SUPPORTS：支持当前事务，如果当前无事务，则无事务执行

3. PROPAGATION_MANDATORY：支持当前事务，如果当前无事务，则抛异常

4. PROPAGATION_REQUIRTED_NEW：新建一个事务，如果当前有事务，则把当前事务挂起

5. PROPAGATION_NOT_SUPPORTED：以非事务方式执行，如果当前有事务，则把当前事务挂起

6. PROPAGATION_NESTED：支持嵌套事务，如果当前无事务，则与REQUIRED类似操作

7. PROPAGATION_NEVER：不支持事务，如果当前有事务，则抛异常

### 8.1.7、面试题

（参考：<https://blog.csdn.net/ThinkWon/article/details/104397516> 或 <https://juejin.im/post/5e5cfa4be51d4526f23a2660>）



## 8.2、SpringBoot

SpringBoot是一个不仅继承了Spring框架的优秀特征，还通过简化配置来进一步简化Spring应用的搭建和开发过程，它还包含1个重要策略：**约定大于配置**，指在开发过程中，通过starter方式简化了集成第三方框架的依赖和版本冲突问题，去掉了XML配置，使用注解来管理对象生命周期，使开发人员只需要关注业务逻辑开发。





### 8.2.1、SpringBoot启动流程

（参考：<https://blog.csdn.net/weixin_43570367/article/details/104960677> 或 <https://blog.csdn.net/mnicsm/article/details/93893669> 或 <https://www.jianshu.com/p/b1b195f21dd4>）

### 8.2.2、SpringBoot自动配置原理

SpringBoot启动时，会通过@EnableAutoConfiguration注解来找到META-INF/spring.factories文件中的自动配置类，并对其进行加载，这些自动配置类都是以AutoConfiguration结尾来命名的，它其实就是JavaConfig形式的Spring容器配置类，它能通过以Properties结尾命名的类中取得在全局配置文件中的属性，如：server.port，而XxxxProperties类是通过@ConfigurationProperties注解与全局配置文件中对应的属性进行绑定的

（参考：<https://blog.csdn.net/u014745069/article/details/83820511>）

### 8.2.2、面试问题

（参考：<https://blog.csdn.net/ThinkWon/article/details/104397299>）



## 8.3、Hibernate

（参考：<https://cloud.tencent.com/developer/article/1062118>）



## 7.4、Mybatis

（参考：<https://blog.csdn.net/a745233700/article/details/80977133>）

### 8.4.1、自定义插件

Mybatis仅实现了Executor、ParameterHandler、StatementHandler、ResultSetHandler这4个接口的插件，使用JDK动态代理为接口生成代理对象，实现接口方法的拦截，要开发我们的插件，只需要实现Interceptor接口并重写intercept方法，方法最后记得调用invocation.proceed方法，防止断链。

（参考：<https://www.jianshu.com/p/96a417f8cf5f>）

## 8.5、SpringMVC

（参考：<https://blog.csdn.net/a745233700/article/details/80963758>）

### 8.5.1、拦截器（Interceptor）与过滤器（Filter）

（参考：<https://www.cnblogs.com/junzi2099/p/8022058.html>）

1. 过滤器（filter）：是Servlet规范中规定的，通过实现javax.servlet.Filter接口的init,destroy,doFilter方法，并在web.xml文件中配置使用，可以在HttpServletRequest达到Servlet之前过滤并修改其头和数据内容，也可以在HttpSerletResponse到达客户端之前过滤，修改其头和数据内容。不能使用Spring容器资源，是被Tomcat等Server调用。

2. 拦截器（interceptor）：是在面向切面编程中应用的，通过实现Spring中的HandlerInterceptor或WebRequestInterceptor接口，还可以继承以上2个接口的子类，并在SpringMVC-dispatcher配置文件中配置使用，主要拦截Action请求，实现的方法有：

	1. preHandle：这里有请求链，返回false断链，是Controller前的处理操作

	2. postHandle：当preHandler返回true才执行，是在controller执行之后，但在DispatcherServlet进行视图返回渲染之前被调用

	3. afterCompletion：也是preHandler返回true才执行，是在DispatcherServlet渲染了视图之后执行

3. 拦截器（Interceptor）声明方式：1、直接定义Interceptor实现类的bean对象，这样是对所有请求进行拦截  2、使用mvc:interceptor标签进行声明，通过mvc:mapping子标签来定义拦截路径。

4. 过滤器与拦截器的执行顺序：过滤前-拦截前-Action处理-拦截后-过滤后



## 8.6、SpringSecurity

### 8.6.1、CSRF跨站请求伪造

（参考：<https://www.cnblogs.com/yangzhaon/p/11048079.html>）

## 8.7、SpringCloud

### 8.7.1、Eureka

#### 8.7.1.1、@LoadBalanced

（参考：<https://www.cnblogs.com/yourbatman/p/11532729.html>）

#### 8.7.1.2、自我保护模式

建议开启，用于防止因网络分区故障时，实例间服务不能调用的情况，如：实例C1和C2相互之间可以调用，但是网络分区故障导致EurekaServer无法访问C2，所以如果不开启保护模式，C2将被剔除，导致服务不可用。

（参考：<https://www.cnblogs.com/xishuai/p/spring-cloud-eureka-safe.html>）

## 8.8、Reactor

（参考：<https://www.cnblogs.com/huaweicloud/p/11861317.html>）

## 8.9、消息中间件

### 8.9.1、ActiveMQ

ActiveMQ是一个异步详细中间件，支持点对点（queue）和一对多（topic）方式，

（参考：<https://blog.csdn.net/qq_33404395/article/details/80590113>）

### 8.9.2、ActiveMQ、RabbitMQ、Kafka区别

（参考：<https://blog.csdn.net/lifaming15/article/details/79942793>）

## 8.10、Zookeeper

Zookeeper是典型的分布式数据一致性解决方案，分布式应用程序可以基于Zookeeper实现诸如：数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁和分布式队列等功能。

（参考：<https://blog.csdn.net/jiahao1186/article/details/82633588>）

## 8.11、Netty

（参考：<https://www.cnblogs.com/xiaoyangjia/p/11526197.html>）

### 8.11.1、Netty的核心组件

Netty是一个NIO的网络编程框架，

（参考：<https://www.cnblogs.com/leesf456/p/6831661.html>）

1. Channel：他是网络操作的抽象，包含了基础的IO操作，如：绑定、链接、读写等

2. EventLoop：主要配合Channel进行IO操作，用于处理链接生命周期中发生的事件

3. ChannelFuture：Netty中的所有IO操作都是异步的，所以需要ChannelFuture的addListener添加一个ChannelFutureListener来监听所有操作的处理结果。

4. ChannelHandler：包含了所有处理入站和出站数据的用户逻辑，可以处理各种事件，如：链接、数据转换、数据接收、异常等。

5. ChannelPipeline：为ChannelHandler链提供了容器，当Channel创建时，会自动创建一个专属的ChannelPipeline，这个关联是永久的。

6. Boostrap和ServerBootstrap：Netty的引导类，为应用程序网络层配置提供容器。主要用于绑定进程端口或链接远程服务端。

### 8.11.2、Netty的执行顺序

1. 创建ServerBootstrap实例

2. 设置并绑定EventLoopGroup

3. 设置并绑定Channel

4. 创建处理网络事件的ChannelPipeline和ChannelHandler

5. 绑定并启动监听端口

6. 当轮训到准备就绪的Channel后，NioEventLoop执行pipeline中的方法，最终调度并执行ChannelHandler。

### 8.11.3、Netty的内存零拷贝

1. Netty的接收和发送的buffer都是使用了堆外内存，减少了JVM堆到直接内存的拷贝

2. Netty提供了组合buffer，减少了将多个小的buffer拷贝合并为一个大buffer的过程

3. Netty的文件传输使用了transferTo方法，它直接将文件缓冲区的数据发送到目标channel，避免了传统的while循环write方式导致了内核拷贝。

### 8.11.4、Netty的线程模型

（参考：<https://segmentfault.com/a/1190000015484187#articleHeader2>）

其实就是Reactor模型，分为：单线程、多线程、主从多线程



## 8.12、Nginx

（参考：<https://zhuanlan.zhihu.com/p/97710579> 或 <https://juejin.im/post/5e941ec4e51d45471263ef32>）

## 8.13、ProtoBuf

（参考：<https://www.jianshu.com/p/293c1e7c0a99>）

## 8.14、KeepAlive

（参考：<https://www.jianshu.com/p/db4b1d645c4b>）