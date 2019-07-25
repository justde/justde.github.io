---
layout: post
title: '几个零碎的知识点'
date: 2019-07-25
author: Justd
cover: '/assets/img/2019-07/coder.jpg'
tags: 面试  java  
---
从小公司面试题抽出几个来记录下


1. 类的生命周期   
   - 创建阶段
     - 为对象分配存储空间
     - 构造对象
     - 从超类到子类对static成员进行初始化
     - 超类成员变量按顺序初始化，调用超类的构造方法  
     - 子类成员变量按顺序初始化，子类构造方法调用
   - 应用阶段
     - 对象至少被一个强引用持有
   - 不可见阶段
     - 程序的执行超出了对象的作用域
     - 对象不再被强引用持有  
   - 不可达阶段
     - 失去GCroot的连接，不可达。
     - 可以作为GC Roots的对象：
       - 栈中引用的对象
       - 本地栈引用的对象
       - 静态变量引用的对象
       - 常量引用的对象 
   - 收集阶段
   - 终结阶段
   - 对象空间重新分配阶段
2. 对jvm的理解   

   ![jdk1.8版本](/assets/img/2019-07/jvm.png)  
    - 堆：存放对象实例，几乎所有的对象实例都在这里分配内存
    - 虚拟机栈：虚拟机栈描述的是Java方法执行的内存模型：每个方法被执行的时候都会创建一个栈帧(Stack Frame)用于存储局部变量、操作栈、动态链接、方法出口等信息  
    - 本地方法栈：存放虚拟机使用到的Native方法 
    - 方法区：又叫元空间，1.8后放在物理内存中，存放被虚拟机加载的类的元数据信息，每个类都有自己的元空间，所有的元空间合并到一起就是我们说的元空间
    - 程序计数器：当前线程所执行的字节码的行号指示器

    其中堆/元空间为线程共享内存，其余每个线程都有各自的内存空间。   
3. Spring Boot相关
    > 简化了使用Spring的流程，不用关心工具是如何集成的，只需要在pom文件中引入该工具starter的jar即可使用   

    1. SpringBoot的核心配置文件有哪几种格式？有什么区别
    配置文件: application
    格式: .properties 和 .yml   
       - properties
           ```java
           app.user.name=springboot
           ```
       - yml 
           ```java
           app:
               user:
                   name: springboot
           ```
        yml不支持 `@propertySource`
    2. SpringBoot的核心注解是哪个？它主要有哪几个注解组成？  
    启动类上面的注解：`@SpringBootApplication` 主要包含一下三个注解：
        - `@SpringBootConfiguration`: 组合了`@Configuration`注解，实现配置文件的功能   
        - `@EnableAutoConfiguration`: 打开自动配置的功能。也可以关闭某个自动配置的选项，如关闭数据源自动配置功能：`@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})`。   
        - `@ComponentScan`: Spring组件扫描  
    3. 开启SpringBoot特性有哪几种方式  
       - 继承Spring-boot-starter-parent项目  
       - 导入Spring-boot-dependencies项目依赖  
    4. SpringBoot需要独立的Servlet么
    不需要，SpringBoot内置了Tomcat/Jetty等容器
    5. 运行SpringBoot的几种方式   
       - 打包命令或者放到容器中运行  
       - 用Maven/Gradle插件运行  
       - 直接执行main方法运行 
    6. 如何理解SpringBoot中的Starters  
    Starters可以理解为启动器，它包含了一系列可集成到应用中的依赖包，需要的时候只需要在pom文件中添加依赖即可
4