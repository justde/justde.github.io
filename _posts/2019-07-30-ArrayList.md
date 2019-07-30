---
layout: post
title: 'ArrayList'
date: 2019-07-30
author: Justd
cover: '/assets/img/2019-07/reflect.jpg'
tags: 面试  java  集合
---

##  List接口框架
   - Collection接口：单列集合，用来存储一个一个的对象
     - List接口：存储有序的、可重复的数据
       - ArrayList:作为List接口的主要实现类，线程不安全，效率高；底层使用Object[] elementData存储  
       - LinkedList:对于频繁的中间插入、删除操作，使用LinkedList效率更高；底层使用双向链表存储   
       - vector：作为List接口的古老实现类；线程安全，效率低；底层使用Object[] elementData存储 
     - Set接口：存储无序、不可重复的数据
       - HashSet：作为Set接口的主要实现类，线程不安全，可以存储null值  
         - LinkedHashSet：HashSet的子类，遍历内部数据时，可以按照添加的顺序遍历  
       - TreeSet：可以按照添加对象得指定属性，进行排序  

1. ArrayList
   1. ArrayList 1.7 

    ```java
    ArrayList list = new ArrayList();     
    ```
    底层创建了长度是10的Object[]数组elementData       
   ```java
   list.add("test"); 
   ``` 
   如果此次添加导致底层elementData数组容量不够，则扩容。默认情况下，扩容为原来的容量的1.5倍(capacity + capacity>>1)，同时需要将原有数组中的数据复制到新的数组中。   
   建议：开发中使用带参构造器，指定容量大小，避免扩容过程   
   2. jdk1.8
   ```java
   ArrayList list = new ArrayList();    
   ```
   底层Object[] elementData初始化为{}，并没有创建长度为10的数组    
   ```java
   list.add("test"); 
   ```    
   第一次调用add()时底层才创建了长度为10的数组，并将数据test添加到elementDate[0];
   后续操作与jdk1.7中相同   
   3. 总结：  
   jdk1.7中ArrayList的对象创建类似于单例的饿汉式，而jdk1.8中的ArrayList的对象创建类似于单例的懒汉式，延迟数组的创建，节省内存。

2. LinkedList  
    ```java
    LinkedList list = new LinkedList();  
    ```
    内部声明了Node类型的first和last属性，默认值为null   
   ```java
   list.add("test"); 
   ```   
   将test封装到Node中，创建Node对象，结构如下
    ```java
     private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    ```   
## Set  
Set：存储无序的、不重复的数据   
以HashSet为例：   
1.  无序性：不等于随机性，存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的hashCode决定   
2.  不可重复性：保证添加的元素按照equals()判断时，不能返回true,即：相同元素只能添加一个。
Set接口没有额外定义的方法，都是Collection中声明过的方法  
