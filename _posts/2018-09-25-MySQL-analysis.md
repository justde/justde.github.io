---
layout: post
title: 'MySql语句执行分析'
date: 2018-09-25
author: Justd
cover: '/assets/img/2018-9/09-25-mysql.png'
tags: MySQL 调优  Linux
---

今天客户那边遇到一个问题：多选文件进行操作，数据量一大后台处理就特别慢，浏览器显示504超时。为了验证问题是否出在sql语句，所以用以下方法来分析：
- 查询SQL执行记录
- explain 分析
- MySQL 语句执行时间

下面会分别介绍三个方法的开启方法。


# 查询SQL执行记录
### 查询日志功能是否开启
```SQL
show variables LIKE 'general%';
```
![](/assets/img/2018-9/09-25/general_log.png)     
 
general_log:日志记录功能是否开启，默认为OFF   
general_log_file:日志存放路径    

### 开启日志功能    
```SQL
set GLOBAL general_log = 'ON'; 
```
然后再次[查询是否开启成功](#查询日志功能是否开启)   

### 在指定路径查看SQL记录


# explain 分析
大部分的性能分析都需要使用到该命令，可以用来查看SQL语句的执行效果，可以帮助选择更好地索引和优化语句。
### 语法
>explain + SQL语句    

输出：
![](/assets/img/2018-9/09-25/explain.png)     


### 参数解析
<table border="1" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">id</span></strong></span></td>
            <td>
                <span style="font-size:12px;">SELECT识别符。这是SELECT的查询序列号</span></td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">select_type</span></strong></span></td>
            <td>
                <p>
                    <span style="font-size:12px;">SELECT类型,可以为以下任何一种:</span></p>
                <ul style="list-style-type:none;">
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">SIMPLE</span></strong></span><span style="font-size:12px;">:简单SELECT(不使用UNION或子查询)</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">PRIMARY</span></strong></span><span style="font-size:12px;">:最外面的SELECT</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">UNION</span></strong></span><span style="font-size:12px;">:UNION中的第二个或后面的SELECT语句</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">DEPENDENT UNION</span></strong></span><span style="font-size:12px;">:UNION中的第二个或后面的SELECT语句,取决于外面的查询</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">UNION RESULT</span></strong></span><span style="font-size:12px;">:UNION
                            的结果</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">SUBQUERY</span></strong></span><span style="font-size:12px;">:子查询中的第一个SELECT</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">DEPENDENT SUBQUERY</span></strong></span><span
                            style="font-size:12px;">:子查询中的第一个SELECT,取决于外面的查询</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">DERIVED</span></strong></span><span style="font-size:12px;">:导出表的SELECT(FROM子句的子查询)</span></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">table</span></strong></span></td>
            <td>
                <p>
                    <span style="font-size:12px;">输出的行所引用的表</span></p>
            </td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">type</span></strong></span></td>
            <td>
                <p>
                    <span style="font-size:12px;">联接类型。下面给出各种联接类型,按照从最佳类型到最坏类型进行排序:</span></p>
                <ul style="list-style-type:none;">
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">system</span></strong></span><span style="font-size:12px;">:表仅有一行(=系统表)。这是const联接类型的一个特例。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">const</span></strong></span><span style="font-size:12px;">:表最多有一个匹配行,它将在查询开始时被读取。因为仅有一行,在这行的列值可被优化器剩余部分认为是常数。const表很快,因为它们只读取一次!</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">eq_ref</span></strong></span><span style="font-size:12px;">:对于每个来自于前面的表的行组合,从该表中读取一行。这可能是最好的联接类型,除了const类型。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">ref</span></strong></span><span style="font-size:12px;">:对于每个来自于前面的表的行组合,所有有匹配索引值的行将从这张表中读取。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">ref_or_null</span></strong></span><span style="font-size:12px;">:该联接类型如同ref,但是添加了MySQL可以专门搜索包含NULL值的行。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">index_merge</span></strong></span><span style="font-size:12px;">:该联接类型表示使用了索引合并优化方法。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">unique_subquery</span></strong></span><span style="font-size:12px;">:该类型替换了下面形式的IN子查询的ref:
                            value IN
                            (SELECT primary_key FROM single_table WHERE some_expr)
                            unique_subquery是一个索引查找函数,可以完全替换子查询,效率更高。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">index_subquery</span></strong></span><span style="font-size:12px;">:该联接类型类似于unique_subquery。可以替换IN子查询,但只适合下列形式的子查询中的非唯一索引:
                            value IN (SELECT key_column FROM single_table WHERE some_expr)</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">range</span></strong></span><span style="font-size:12px;">:只检索给定范围的行,使用一个索引来选择行。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">index</span></strong></span><span style="font-size:12px;">:该联接类型与ALL相同,除了只有索引树被扫描。这通常比ALL快,因为索引文件通常比数据文件小。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">ALL</span></strong></span><span style="font-size:12px;">:对于每个来自于先前的表的行组合,进行完整的表扫描。</span></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">possible_keys</span></strong></span></td>
            <td>
                <p>
                    <span style="font-size:12px;">指出MySQL能使用哪个索引在该表中找到行</span></p>
            </td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">key</span></strong></span></td>
            <td>
                <span style="font-size:12px;">显示MySQL实际决定使用的键(索引)。如果没有选择索引,键是NULL。</span></td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">key_len</span></strong></span></td>
            <td>
                <span style="font-size:12px;">显示MySQL决定使用的键长度。如果键是NULL,则长度为NULL。</span></td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">ref</span></strong></span></td>
            <td>
                <span style="font-size:12px;">显示使用哪个列或常数与key一起从表中选择行。</span></td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">rows</span></strong></span></td>
            <td>
                <span style="font-size:12px;">显示MySQL认为它执行查询时必须检查的行数。多行之间的数据相乘可以估算要处理的行数。</span></td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">filtered</span></strong></span></td>
            <td>
                <span style="font-size:12px;">显示了通过条件过滤出的行数的百分比估计值。</span></td>
        </tr>
        <tr>
            <td>
                <span style="color:#000000;"><strong><span style="font-size:12px;">Extra</span></strong></span></td>
            <td>
                <p>
                    <span style="font-size:12px;">该列包含MySQL解决查询的详细信息</span></p>
                <ul style="list-style-type:none;">
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">Distinct</span></strong></span><span style="font-size:12px;">:MySQL发现第1个匹配行后,停止为当前的行组合搜索更多的行。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">Not exists</span></strong></span><span style="font-size:12px;">:MySQL能够对查询进行LEFT
                            JOIN优化,发现1个匹配LEFT
                            JOIN标准的行后,不再为前面的的行组合在该表内检查更多的行。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">range checked for each record (index map: #)</span></strong></span><span
                            style="font-size:12px;">:MySQL没有发现好的可以使用的索引,但发现如果来自前面的表的列值已知,可能部分索引可以使用。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">Using filesort</span></strong></span><span style="font-size:12px;">:MySQL需要额外的一次传递,以找出如何按排序顺序检索行。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">Using index</span></strong></span><span style="font-size:12px;">:从只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的列信息。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">Using temporary</span></strong></span><span style="font-size:12px;">:为了解决查询,MySQL需要创建一个临时表来容纳结果。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">Using where</span></strong></span><span style="font-size:12px;">:WHERE
                            子句用于限制哪一个行匹配下一个表或发送到客户。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">Using sort_union(...), Using union(...), Using
                                    intersect(...)</span></strong></span><span style="font-size:12px;">:这些函数说明如何为index_merge联接类型合并索引扫描。</span></li>
                    <li style="list-style-position:outside;list-style-type:disc;">
                        <span><strong><span style="font-size:12px;">Using index for group-by</span></strong></span><span
                            style="font-size:12px;">:类似于访问表的Using index方式,Using
                            index for group-by表示MySQL发现了一个索引,可以用来查 询GROUP BY或DISTINCT查询的所有列,而不要额外搜索硬盘访问实际的表。</span></li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>





# MySQL 语句执行时间
show profile 以及show profiles语句可以显示当前会话过程中执行SQL语句的性能信息。

### 查看profile是否开启
```SQL
show variables like '%profil%';
```
![](/assets/img/2018-9/09-25/profiling.png)   
profiling:OFF 默认此功能关闭

### 设置开启状态
``` SQL
set profiling = 1;
```
再次查看[是否开启](#查看profile是否开启)    
![](/assets/img/2018-9/09-25/open-profiling.png)     
已经是开启状态


### 执行sql语句后进行分析
执行完后，输入
```SQL
show profiles；
```
即可查看所有的sql的执行时间    
![](/assets/img/2018-9/09-25/sql.png)     

```SQL
show profile for query 1 
``` 
 查看第1个sql语句的执行的各个操作的耗时详情。   
 ![](/assets/img/2018-9/09-25/time.png)     


```SQL
show profile cpu, block io, memory,swaps,context switches,source for query 6;
```
可以查看出一条SQL语句执行的各种资源消耗情况，比如CPU，IO等

```SQL
show profile all for query 6 查看第6条语句的所有的执行信息。
```

### 测试完毕后，关闭参数：
``` SQL
mysql> set profiling=0
```