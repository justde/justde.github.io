---
layout: post
title: 'MySQL系列之一条更新SQL的生命历程'
date: 2018-11-20
author: Justd
cover: '/assets/img/2018-11/21/timg.jpg'
tags:  MySQL  
---
# MySQL系列之一条更新SQL的生命历程
      

>听说MySQL能恢复到半个月内任意一秒的状态    

要从一条更新语句说起，如果将ID=2这一行的值+1，SQL语句可以这样写：
```sql
mysql> update T set c=c+1 where ID=2;
```
执行一条更新语句同样会走一遍[查询语句](https://justed.github.io/2018/11/14/MySQL-select.html)的流程:
- **连接**数据库
- 清空该表涉及到的**缓存**
- **分析器**通过词法和语法解析出这是一条更新语句，并确定涉及到表与字段
- **优化器**决定使用`"ID"`这个索引
- **执行器**找到这一行数据，然后进行更新操作   
与查询过程不同的是，更新涉及到两个日志模块：**redo log（重做日志）**和**bin log（归档日志）**

## 临时记录：redo log    
酒店掌柜有一个账本和一个小黑板，来做赊账的记录。有以下两种方案：   
1. 每一笔账都打开账本做记录，当有人还账时，找到对应的赊账记录，修改记录的状态
2. 先在黑板上记录本次要做的操作，打烊后按照黑板上的记录向账本上进行核算    

当生意红火，顾客络绎不绝时，第一种方案效率实在是低下，掌柜的一定按照第二种方案来记账。    
同样的，MySQL如果每次更新操作都要写入磁盘，在磁盘中找到对应记录，然后更新，这个过程的IO成本、查找成本都太高了。   
为了解决和这个问题，MySQL就使用了类似于``黑板-账本``模式来提高效率。这一模式即为`WAL技术`，全程为`Write-Ahead Logging`，关键点:**先写日志，再写磁盘**，也就是前文中先写黑板，再写账本。     

具体步骤如下：    
当有记录需要更新，innoDB先把记录写入redo log中，并更新内存，这是更新操作就算结束了。innoDB引擎会在适当的时候讲操作记录更新到磁盘里，这一动作一般是系统比较闲的时候做的。    
redo log的大小是固定的，共有4个文件组成，每个大小为1G。逻辑上可以将4个文件理解为环形，从头开始写，写到末尾又重新开始新的一轮,如下图所示
![](/assets/img/2018-11/21/mysql-redolog.jpg)    
write pos为当前记录位置，check point为当前擦除点的位置，当记录更新时，check point会随着文件的记录向后移动。擦除后未写入的位置可以记录新的操作。当write pos追上了check point，则需要停下来写入动作，将redo log内容写入磁盘,然后清除check point向后移动。    
有了redo log，innoDB可以知晓每一次操作，保证当数据库发生异常重启时，之前的能够根据redo log恢复之前的记录，这种能力叫做`"crash-safe"`。

## 归档日志：bin log  
  
redo log与bin log日志的区别：    
1. redo log 是属于innoDB引擎所有，bin log是server提供的，所有引擎都可以使用    
2. redo log 属于物理日志，记录`"在某个数据页做了什么修改"`，bin log记录的是该语句逻辑日志，`"将ID=2的这一行c的值+1"` [^1]
3. redo log日志文件是循环使用的，空间有使用完的时刻，bin log是追加记录的，不会覆盖之前的记录    

也就是说，server搭配其他引擎是没有redo log的，因此也就没有了crash-safe能力    

## 更新具体流程   
基于对两个日志文件的了解，再次深入了解更新的流程   
1. 执行器先通过引擎使用树搜索找到ID=2这一行，如果该记录所在的数据页本身就在内存中，则直接返回执行器，否则先从磁盘读入内存，然后返回
2. 执行器拿到数据后将c的值加一，然后通过引擎的写入接口将修改后的数据写入
3. 引擎j将新数据更新到内存中，然后在redo log中记录此次修改，这时redo log中该记录的状态置为prepare，并告知执行器已经更新完成，随时可以提交事务   
4. 执行器生成此次操作的bin log，将bin log写入磁盘中
5. 执行器调用引擎的提交事务接口，引擎将刚刚写入的redo log置为commit状态，更新结束 
   
下图是[《MySQL实战》](https://time.geekbang.org/column/article/68633)提供的流程图：
![](/assets/img/2018-11/21/mysql-update.png)
    浅色代表在innoDB中执行，深色在server中执行    

## 两阶段提交    
从上图可以看出，redo log是分两个阶段来提交的,这是为了保持两个日志逻辑上一致 
如果不用两阶段提交会发生什么呢 利用反证法来看下：   
假设初始ID=2的数据行，c的值为0，现在要执行c+1的操作。
1. **先记录redo log 后记录bin log** 如果刚记录完redo log，还没有记录bin log时，c的值已经记录变为1，这时MySQL服务崩溃重启，根据crash-safe机制，可以用redo log来恢复数据库，恢复后的数据中c的值为1。由于bin log中没有记录这一变化，以后备份bin log时，c的值还是0。如果有一天需要从bin log回复一台备用数据库，由于bin log少了一次更新，则最后恢复出来的c值仍然为0，与原库中值不符合
2. **先记录bin log 后记录redo log**  写完bin log就发生crash，还没来得及写入redo log，崩溃恢复后这个事务是无效的，因此c的值还是0，但是bin log中已经记录了"将c的值+1"的日志，所以用bin log恢复出来的数据多出来一个事务，使得c的值为1，与原库中数据不符。
3. **两阶段提交 记录过bin log回过头提交commit**（可参见评论区知识点）  更新redo log后，还没有记录bin log时崩溃，这时redo log的状态还是prepare，事务并没有提交，而且**bin log中没有记录**，因此由于crash-safe机制，并不会恢复该记录，c的值仍然为0，由于bin log中没有记录，以后从bin log恢复数据时，c的值在此操作中并没有记录变化，因此还是0，与原库中数据一致；另一种情况：更新redo log，也更新了bin log，下一步执行器调用commit接口前崩溃，这时虽然redo log中状态为prepare，但是**从bin log中查到有记录**，所以还是会从redo log中恢复c=1，后面直接从bin log恢复出新的数据库时，因为已经记录c的值+1，所以与原库中的值相同   
### 总结如下：    
两种方式确定记录完整：    
1. redo log状态为 commit
2. redo log状态为prepare**并且**bin log记录完整 （提交commit之前）


## 总结    
这节主要学习了两个日志文件的用法 redo log 用于保证 crash-safe 能力，bin log用于恢复数据的完整性   
 - innodb_flush_log_at_trx_commit 这个参数设置成 1 的时候，表示每次事务的 redo log 都直接持久化到磁盘。这样可以保证 MySQL 异常重启之后数据不丢失。   
 -  sync_binlog 这个参数设置成 1 的时候，表示每次事务的 binlog 都持久化到磁盘。这样可以保证 MySQL 异常重启之后 binlog 不丢失。



----   
### 评论区知识点：    
- binlog没有被用来做崩溃恢复,binlog是可以关的，你如果有权限，可以`"set sql_log_bin=0"`关掉本线程的binlog日志。 所以只依赖binlog来恢复就靠不住的
- @高枕 同学的评论简单精炼的表达了两阶段提交机制下的工作状态：   
  记录日志共有三个过程： 
  1. prepare阶段 
  2. 写binlog 
  3. commit    
    - 当在2之前崩溃时    
重启恢复：后发现没有commit，回滚。备份恢复：没有binlog。备份与原库一致    
    - 当在3之前崩溃时   
重启恢复：虽没有commit，但满足prepare和binlog完整，所以重启后会自动commit。备份：有binlog. 备份与原库一致    

- 来自@黄金的太阳
    - 问：
    1. redo log本身也是文件，记录文件的过程其实也是写磁盘，那和文中提到的离线写磁盘操作有何区别？
    2. 响应一次SQL我理解是要同时操作两个日志文件？也就是写磁盘两次？    
    - 作者回复：
    1. 写redo log是顺序写，不用去“找位置”，而更新数据需要找位置，因此redo log写的速度更快

    2. 其实是3次（redolog两次 binlog 1次）。不过在并发更新的时候会合并写



----
>感谢《MySQL实践》提供的图与知识点    
第三节：[ACID之 事务隔离](https://justed.github.io/2018/11/25/MySQL-3.html)

[^1]:redo log不是记录数据页“更新之后的状态”，而是记录这个页 “做了什么改动”。     
bin log有两种模式，statement 格式的话是记sql语句;   row格式会记录行的内容，记两条，更新前和更新后都有。