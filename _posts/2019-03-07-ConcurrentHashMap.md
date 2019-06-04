---
layout: post
title: 'ConcurrentHashMap'
date: 2019-03-07
author: Justd
cover: ''
tags: ConcurrentHashMap 面试
---

>HashTable和synchronizedMap ：HashTable通过方法和对象上加synchronized关键字来保证线程安全，synchronizedMap也是通过家关键字来保证线程安全。这两种方式虽然说都能保证线程安全，但是锁的粒度比较大，占用资源过多，这是需要考虑到juc下的concurrentHashMap

## 一、数据结构
### 1.7与1.8的区别
1.7采用的是有Segment（锁分离）数组和多个HashEntry组成，如图
![](/assets/img/2019-03/1.7concurrentHashMap.png)

`Segment`（继承ReentrantLock）数组的意义是将大的table分割成多个小的table来进行加锁，而每个Segment元素存储的是HashEntry数组+链表，这个和HashMap数据结构相同。当一个线程占用锁访问其中一个段数据的时候（get put remove等操作），其他段的数据结构也能被其他线程访问。有些方法需要跨段，比如size()和containsValue()，他们可能需要按照顺序锁定所有段，操作完毕后，按顺序释放所有段的锁

1.8 直接用node数组+链表+红黑树,如图
![](/assets/img/2019-03/1.8concurrentHashMap.png) 


1.7与1.8相比最大的区别就是1.8的锁粒度更细，理想情况下table数组元素大小就是支持并发的最大个数，1.7中最大并发个数就是segment个数，默认值是16，可以通过构造函数改变，如果已经创建不可修改，这个值就是并发的粒度。每一个segment下面管理一个table数组，加锁的时候会锁住整个segment，这样好处是数组扩容时不会影响其他的segment，但是粒度稍粗，所以在1.8中去掉了分段锁，并将锁的级别控制在了更细的table元素级别，粒度更细，影响更小，并发效率更高。

> concurrent包实现的通用模式:    
> 首先声明共享变量为volatile   
> 然后使用CAS的原子条件更新来实现线程之间的同步  
> 同时配合volatile的读/写和CAS所具有的volatile的读/写的内存语义来实现线程之间的通信    

## 二、方法操作   
- ### Put操作
  - JDK1.7   
    通过key的第一次hash来定位segment的位置，如果该segment还没有初始化，即通过CAS操作进行赋值，然后进行第二次hash操作，找到相应的HashEntry位置，这里利用继承过来的锁的特性，在将数据插入指定的HashEntry位置时（链表的尾端），会通过继承ReentrantLock的tryLock()方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用tryLock()方法获取锁，超过指定次数就挂起，等待唤醒。
  - JDK1.8
    如果没有初始化就先调用initTable()方法来进行初始化过程；   
    如果没有hash冲突就直接CAS插入；   
    如果还再进行扩容操作就先进行扩容；    
    如果存在hash冲突，就加锁来保证线程安全，这里有两种情况，一种是链表形式就直接遍历到尾端插入，一种是链表数量大于阈值8，就先转换成红黑树结构，break再一次进入循环；   
    如果添加成功就调用addCount()方法统计size，并检查是否需要扩容。

- ### Get操作
  - JDK1.7
    ConcurrentHashMap第一次需要经过一次hash定位到Segment的位置，然后再hash定位到指定的HashEntry，遍历该HashEntry下的链表进行对比，成功则返回，不成功返回null
  - JDK1.8
    计算hash值，定位到该table索引位置，如果首节点符合就返回，如果遇到扩容的时候，会调用标志正在扩容节点ForwardingNode的find方法，查找该节点，匹配就返回不匹配返回null 

- ### Size操作
  - JDK1.7    
    1. 第一步使用不加锁的模式去尝试多次计算ConcurrentHashMap的size，最多三次，比较前后两次计算结果，结果一致就认为当前没有元素加入，计算的结果是准确的。
    2. 当第一步结果不一致，就会给每个Segment加上锁，在计算size返回。
  - JDK1.8
    1.8使用一个volatile类型的变量baseCount记录元素的个数，当插入新数据或删除数据时，会通过addCount()方法更新baseCount。主要是通过CounterCell数组记录CAS操作成功的线程数（CountCell对象），然后用baseCount+CounterCell即为需要的size。

## 三、扩容
 ### JDK1.8：   
    sizeCtl < 0 :表示有其他线程正在执行扩容。   
    采用多线程扩容，当线程1正在扩容时，其他线程进行put或者remove的操作的时候会帮其进行扩容(存在最大扩容线程数)。 而每个原始节点Node下的链表会分别存在两个链表（一个是原位置，另一个是原位置加扩容大小），当该节点扩容完毕会有一个标记，表示通知其他线程该节点惊醒过扩容操作

## 四、总结
关于ConcurrentHashMap相关的面试题：
- HashMap是线程不安全的，ConcurrentHashMap是线程安全的
- ConcurrentHashMap是如何实现线程安全的（底层实现原理是什么）
- ConcurrentHashMap因为1.7和1.8的实现细节上有所不同
- ConcurrentHashMap如何获取大小