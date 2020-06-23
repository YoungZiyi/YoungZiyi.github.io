---
title: mysql技术内幕读书笔记
date: 2020-06-09 08:52:01
categories: 技术
tags: mysql
---


## 1 MySQL体系结构 & 存储引擎

### 1.1 mysql数据库和实例的定义
数据库: 物理操作系统中的文件, 也可能是存放在内存中的文件</br>
数据库实例: 数据库后台进程/线程以及一个共享内存区组成, 它是用来操作数据库文件的</br>
一个数据库实例对应一个数据库文件, 在集群下, 可能多个实例对应一个数据库文件<br>
mysql是单进程多线程架构, mysql实例在操作系统上的表现就是一个进程<br>
启动mysql实例时, mysql会读取配置文件, 如果配置文件不存在, mysql会按编译时的默认参数配置启动实例, 使用以下命令查看配置文件位置<br>
```bash
./mysql --help | grep my.conf
```
多个mysql配置文件, 将以最后一个读到的参数为准, 配置文件中datadir参数指定了数据库文件所在的路径<br>

### 1.2 mysql体系结构
* Connection Pool 
* Management Service & Utilities
* SQL Interface
* Parser
* Optimizer
* Caches & Buffers
* Pluggable Storage Engines
* File System
* Files & Logs

mysql最大特点就是其插件式表存储引擎, 每个引擎都实现了一系列的标准服务, 例如 查询解析器/优化器等等<br>

### 1.3 MySQL 表存储引擎

#### 1.3.1 innoDB存储引擎
InnoDB是使用最广泛的存储引擎, 主要是面向在线事务处理(OLTP)方面的应用, 特点是行锁设计, 非锁定读, 支持外键, 它实现了SQL标准的4种隔离级别, 默认为REPEATABLE级别, 除此之外, 它还提供了 插入缓冲/二次写/自适应哈希索引/预读 这些特性来实现高性能和高可用<br>
InnoDB表中数据的储存采用了聚集的方式, 每张表的存储都按主键的顺序存放, 如果未指定主键, innoDB存储引擎将为每一行生成一个6字节ROWID, 以此为主键<br>

#### 1.3.2 MyISAM存储引擎
myisam不支持事务和全文索引, 不支持行锁, 主要面向在线分析处理(OLAP), myisam表由MYD和MYI组成, MYD存放数据, MYI存放索引, mysql只缓存索引文件, 数据文件的缓存交由操作系统来完成<br>

#### 1.3.3 NDB存储引擎
#### 1.3.4 Memory存储引擎
#### 1.3.5 Archive存储引擎
#### 1.3.6 Faderated存储引擎
#### 1.3.7 Maria存储引擎

### 1.4 各种存储引擎之间的比较
基于mysql8.0文档, 与原书有差异
|  存储引擎  | MyISAM | InnoDB | MEMORY | ARCHIVE | NDB |
|  ----  | ----  |  ----  | ----  |  ----  | ----  |
| B树索引 | Yes | Yes | Yes | No | Yes |
| HASH索引 | No | No | Yes | No | Yes |
| 事务支持 | No | Yes | No | No | No |
| 集群支持 | No | No | No | No | Yes |
| 外键支持 | No | Yes | No | No | No |
| 全文索引 | Yes | Yes(5.6+) | No | No | No |
| 锁粒度 | Table | Row | Table | Row | Row |
| 存储限制 | 256TB | 64TB |	RAM | None | |
| MVCC | No | Yes | No | No | Yes |
| Storage cost | Low | High | N/A | Very Low | Low |
| Memory cost | Low | High | Medium | Low | High |
| Data caches | No | Yes | N/A | No | Yes |
| Index caches | Yes | Yes | N/A | No | Yes |

### 1.5 连接MySQL
连接mysql的本质其实是进程间的通信, 常用的进程间通信有 管道, 命名管道, 命名字, TCP/IP套接字, Unix域间套接字<br>

#### 1.5.1 TCP/IP
#### 1.5.2 命名管道
#### 1.5.3 Unix域套接字

### 1.6 小结
* 数据库和数据库实例的区别
* MySQL体系结构
* 常见数据库引擎的特性


## 2 InnoDB存储引擎

### 2.1 InnoDB存储引擎概述
* 是一款支持ACID事务的MySQL存储引擎
* 支持行锁
* 支持MVCC(?)
* 非锁定读
* 支持外键

### 2.2 InnoDB体系架构
InnoDB由多个内存块组成一个大的内存池负责处理如下工作  
* 维护进程/线程需要访问的多个内部数据结构
* 缓存硬盘上的数据, 写入硬盘的数据也会先缓存到内存中
* redo log缓冲

#### 2.2.1 后台线程
默认情况下, InnoDB存储引擎有7个后台线程, 其中4个IO thread, 1个master thread, 一个锁监控线程, 1个错误监控线程, 我在mysql8.0中通过执行```show engine innodb status```看到IO thread如下:
```
--------
FILE I/O
--------
I/O thread 0 state: waiting for i/o request (insert buffer thread)
I/O thread 1 state: waiting for i/o request (log thread)
I/O thread 2 state: waiting for i/o request (read thread)
I/O thread 3 state: waiting for i/o request (read thread)
I/O thread 4 state: waiting for i/o request (read thread)
I/O thread 5 state: waiting for i/o request (read thread)
I/O thread 6 state: waiting for i/o request (write thread)
I/O thread 7 state: waiting for i/o request (write thread)
I/O thread 8 state: waiting for i/o request (write thread)
I/O thread 9 state: waiting for i/o request (write thread)
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 0
1447 OS file reads, 367 OS file writes, 153 OS fsyncs
1.51 reads/s, 16384 avg bytes/read, 4.46 writes/s, 2.91 fsyncs/s
```
在innoDB plugin版本开始, read thread和write thread分别扩大到4个

#### 2.2.2 内存
innoDB存储引擎运行时的内存由一下几个部分组成, 缓冲池(buffer pool), 重做日志缓冲池(redo log buffer), 以及额外的内存池(additional memory pool), 他们都可以通过配置文件来设置大小  
innoDB总是将数据库文件按页读取到缓冲池, 按最近最少使用(LRU)的算法保留缓冲池中的数据, 当修改数据库文件时, 也是首先修改缓存中的数据页, 再按一定的频率刷新到硬盘, 在刷新到硬盘之前, 缓存中被修改后的页即为脏页  

查看innoDB状态如下: 
```
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 137428992
Dictionary memory allocated 454130
Buffer pool size   8192
Free buffers       7115
Database pages     1069
Old database pages 406
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 899, created 170, written 227
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 1069, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
```
Buffer pool size表示缓冲帧(buffer frame)总数, 每个缓冲帧16K, Free buffers表示当前空闲的缓冲帧, Database pages表示已经使用的缓冲帧, Old database pages表示LRU列表中old部分的page数量, Modified db pages表示脏页的数量  
缓冲池中有三种数据, 分别是数据页, 日志缓冲, 额外内存池, 数据页类型有: 索引页, 数据页, undo页, 插入缓冲, 自适应哈希索引, InnoDB存储的锁信息, 数据字典信息等; 日志缓冲例如redo log, 一般先放入缓冲区, 然后按一定频率刷新到日志文件; 额外的内存池(略)  

### 2.3 master thread
InnoDB主要的工作都是在master thread线程上完成  

#### 2.3.1 master thread源码分析
master thread内部由几个循环组成, 主循环(loop), 后台循环(background loop), 刷新循环(flush loop), 暂停循环(suspend loop), master thread根据运行状态在几个循环中切换  
主循环loop: 其中有两个主要操作, 每秒钟操作, 每十秒操作  

每秒钟的操作包含:  
* 日志缓冲刷新到文件, 即使事务还未提交(因此, 无论多大的事务在提交的时候也是很快的, 因为它分散到每一秒的循环操作中)
* 合并插入缓冲
* 最多刷新100个脏页到硬盘
* 如果当前没有用户活动, 切换到background loop

每十秒的操作包含:  
* 刷新100个脏页到硬盘
* 合并最多5个插入缓冲
* 日志缓冲刷新到硬盘
* 删除无用的undo页
* 刷新100个或者10个脏页到硬盘
* 产生一个检查点

当当前没有用户活动或者数据库关闭时, 就会切换到background loop, 该循环包含以下操作：
* 删除无用的undo页
* 合并20个插入缓冲
* 跳回到主循环
* 不断刷新100个页

flush loop负责刷新页， 如果flush loop也没事做，则会切换到suspend loop将master loop挂起等待事件发生

#### 2.3.2 master thread的潜在问题

### 2.4 关键特性
innoDB的关键特性包括：插入缓冲，两次写，自适应哈希索引

#### 2.4.1 插入缓冲
insert buffer和数据页一样，他是物理页的一部分。在插入行数据的时候，是按照主键也就是聚集索引的顺序来插入的，这样避免了磁盘随机读。但是，如果表中还有其他非聚集索引，非聚集索引的插入就不是按顺序进行的了，需要离散的访问非聚集索引页，产生随机读。插入韩冲的作用是，对非聚集索引的插入或更新操作首先会判断插入的非聚集索引的索引页是否在缓冲池中，如果有，则直接插入，否则放入一个插入缓冲区中，再以一定的频率将插入缓冲合并到索引的叶子节点中，可能同时将多个插入缓冲合并到一个操作。  
insert buffer需要满足两个条件：  
* 索引是辅助索引
* 索引不是唯一的

辅助索引不能是唯一的原因是，当插入数据时，直接放到insert buffer中，不需要检查索引树，而当辅助索引时唯一的时候，为了保证唯一必须去查找索引树，那么就会产生随机读，插入缓冲也就没有意义了。

insert buffer可能产生的问题是，当写密集时，插入缓冲会占用过多的缓冲池内存，而且如果这时发生宕机，重启恢复的时间会很长，需要将插入缓冲合并到索引树中。

#### 2.4.2 两次写
#### 2.4.3 自适应哈希索引

### 2.5 启动/关闭与恢复

### 2.6 InnoDB Plugin = 新版本的InnoDB存储引擎

### 2.7 小结


## 3 文件

### 3.1 参数文件
#### 3.1.1 什么是参数
#### 3.1.2 参数类型

### 3.2 日志文件
#### 3.2.1 错误日志
#### 3.2.2 慢查询日志
#### 3.2.3 查询日志
#### 3.2.4 二进制日志

### 3.3 套接字文件
### 3.4 pid文件
### 3.5 表结构定义文件

### 3.6 InnoDB存储引擎文件
#### 3.6.1 表空间文件
#### 3.6.2 重做日志文件

### 3.7 小结
