---
title: mysql技术内幕
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