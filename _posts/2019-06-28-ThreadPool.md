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
