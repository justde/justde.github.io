---
layout: post
title: '线程池的使用'
date: 2019-06-28
author: Justd
cover: '/assets/img/2018-10/17/404-bg.jpg'
tags: 面试  java 线程 
---
## 为什么使用线程池    
         
    
    
线程池的工作主要是控制运行的线程数量，**处理过程中将任务放入队列**，然后在线程创建后启动这些任务，如果**线程数量超过了最大数量，那么多余的线程要排队等候**，等待其他线程处理完毕，再从队列中取出任务来执行。

主要特点：`线程复用` `控制最大并发数` `管理线程`  

- 降低资源消耗。通过重复用与创建的线程降低线程创建和销毁造成的消耗   
- 提高响应速度。当任务到达时，任务可以不需要等待线程创建这一过程   
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控   

## 线程池介绍
### 架构实现 
java中的线程池是通过executor框架实现的，该框架中用到了Executor接口、Executors类、ThreadPoolExecutor类和ScheduledThreadPoolExecutor类。Executors和Executor的关系等同于从Collections与Collection和Arrays与Array，前者提供了许多辅助工具类可以很方便的使用。
![](/assets\img\2019-06\executor.png)   

### 常用方法
线程池的有5种，其中最常用的有以下三种
- Executors.newFixedThreadPool(int)  **固定线程个数：执行长期任务，性能好很多**
  具体实现：
 ```java
      public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
  ```
  主要特点：  
  1. 创建一个`定长线程池`，可控制线程最大并发数，超出的线程会在队列中等待   
  2. newFixedThreadPool创建的线程池corePoolSize和maximumPoolSize值相等，`它使用的LinkedBolickingQueue   `
- Executors.newSingleThreadExecutor() **只有一个线程 适用于单个任务现行执行的场景**
  ```java
      public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
  ```
  主要特点：
  1. 创建`一个单线程化`的线程池，他只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序执行  
  2. newSingleThreadExector将corePoolSize和maximumPoolSize都设置为1，`它使用的LinkedBlockingQueue`
- Executors.newCachedThreadPool() **动态扩大线程数，适用于执行很多短期异步的小程序或负载比较轻的服务**
  ```java
      public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
  ```
  主要特点：
  1. 创建一个`可缓存线程池`，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
  2. newCachedThreadPool将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，使用的SynchronousQueue,也就是说来了任务就创建线程运行，当线程空闲60秒，就销毁线程。

  以上三段代码可知，线程池的构造都是从一个方法而来: `ThreadPoolExecutor` 

### ThreadPoolExector
由以上三段代码可知，在Exectors内部创建线程池ide时候，实际创建的都是一个ThreadPoolExecutor对象，只是对ThreadPoolExecutor构造方法，进行了默认值的设定。其构造方法如下：
```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

```
参数含义如下：   
- corePoolSize:常驻核心线程数
- maximumPoolSize:线程池中能够容纳同时执行的最大线程数，必须大于等于1
- workQueue:任务队列，被提交到那时尚未被执行的任务
- keepAliveTime:线程空闲时间长度，即非核心线程，当队列中排队的任务过多，会创建出来小于等于最大线程数的线程作为临时线程来执行队列中的任务，如果这类临时线程空闲时间超过keepAliveTime，则会被销毁，只剩下核心线程数     
- unit 空闲时间的单位
- threadFactory 线程工厂，用于创建线程一般用默认的即可
- RejectedExecutionHandler 拒绝策略，表示当队列满了并且工作线程大于等于线程吃的最大线程数时，应对再接收到的线程的策略  

参数使用场景：
![](/assets\img\2019-06\ThreadPool.png)
1. 创建线程池后，等待提交过来的任务请求
2. 当调用execute()方法添加一个请求任务时，线程池会做如下判断：    
   1. 如果正在运行的线程数小于`corePoolSize`时，创建新的线程运行这个任务
   2. 如果正在运行的线程数大于等于`corePoolSize`,那么把任务放入`队列`中
   3. 如果这时队列满了，且正在运行的线程数小于`maximumPool`，那么还要创建`非核心线程`运行这个任务
   4. 如果线程数已经达到maximumPool，那么线程池会启动`饱和拒绝策略`来执行。
3. 当一个线程执行任务完成，他会从队列中取出下一个任务来执行
4. 当一个线程空闲超过`keepAliveTime`，线程池会判断 如果当前运行线程数大于corePoolSize，则线程被销毁
5. 线程池所有任务完成后，最终会收缩到corePoolSize（）

### 线程拒绝策略   
当队列满了而且线程数已经达到maximumPoolSize,接下来的线程会受到拒绝策略的管控
拒绝策略有四种：
- AbortPolicy(默认):直接抛出RejectedExecutionException一场阻止系统正常运行   
- CallerRunsPolicy:"调用者运行"一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者那里，从而降低新任务的流量
- DiscardOldestPolicy:抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务   
- DiscardPolicy:直接丢弃任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种方案

### 线程池配置合理线程数
#### CPU密集型    
该任务需要大量的运算，并且没有阻塞，CPU一直全速运行，CPU密集任务只有在真正的多核CPU上才可能通过多线程加速  
CPU密集型任务配置尽可能少的线程数量：CPU核数+1个线程的线程池

#### IO密集型
IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如CPU核数*2   

- 某大厂设置策略：IO密集型时，大部分线程都阻塞，故需要多配置线程数：
公式：CPU核数/1-阻塞系数      阻塞系数：0.8-0.9   
比如8核CPU： 8/1-0.9 = 80 个线程数

## 最终建议
根据阿里巴巴编码规约，不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式创建，这样让写的同学更加明确线程池的运行规则，避免资源耗尽的风险。
executors各个方法的弊端：
1. newFixedThreadPool和newSingleThreadExecutor:    
   主要问题是允许的队列长度为Integer.MAX_VALUE,堆积的请求处理队列可能会耗费非常大的内存，甚至OOM
2. newCachedThreadPool和newScheduledThreadPool:   
   主要问题是线程数最大数是Integer.MAX_VALUE,可能会创建数量非常多的线程，甚至OOM   




