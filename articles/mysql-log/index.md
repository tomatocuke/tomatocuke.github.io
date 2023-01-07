---
title: "区分binlog、redolog、undolog、change buffer"
date: "2021-06-08"
categories:
  - MySQL
toc: true
---


阅读本文前前需要大概明白MySQL的数据结构，内存和磁盘、server层、engine层、页、索引结构、事务、MVCC等。

<!--more-->



#### WAL
 Write Ahead Log 预写日志，用于保证数据操作的原子性和持久性，所有的修改在提交之前都要先写入 log 文件中。
#### binlog
1. server层逻辑日志，各个引擎都能用。
2. 记录对数据库操作的语句的二进制文件。追加写。
3. 用作主从复制和数据恢复

#### redolog
1. engine层物理日志，innodb特有
2. 记录对数据的更改内容。固定大小的几个文件，循环写。
3. 对于数据的修改，先统一在redolog中记录下来，不管是否page页在内存中，都暂时不需要写入磁盘。
4. 日志满了时，将前一部分同步到磁盘对于页行，然后擦除该部分继续使用。

#### undolog
1. engine层逻辑日志，innodb特有
2. 应用于事务和MVCC。
3. 在事务修改记录时，将原数据写入undolog，当rollback或者事务未提交而崩溃后数据的回滚。
4. 在事务提交前，读取数据修改前快照。

#### change buffer
1.  占用的是innodb buffer pool的内存，也有对应的磁盘空间。可以通过innodb_change_buffer_max_size 修改占用innodb buffer pool的百分比
2. 应用于insert、update、delete，减少磁盘页读入内存。
3. 当修改的页不在内存中时，写入change buffer内存，写入redo log日志。 
5. 当相关page页读入内存时引起merge；定时merge；数据库正常关闭merge。
6. 修改后立即查询，会引起page读入内存页马上merge。增加了change buffer写入的开销
7. 唯一索引插入时需要讲索引表读入内存判断是否唯一，没必要使用change buffer
8. 因为写入change buffer的数据也写入了redo log，所以崩溃回复可以找回。

#### 区分binlog和redolog
1. 前者server层，后者engine层。
2. 前者的所有引擎都可用，后者innodb独有。
3. 前者记录的是逻辑操作，后者记录的是更新的内容。
4. 都是日志文件，前者是追加写形成多个文件，后者是固定大小的几个文件循环写。
5. 事务执行时多个操作，一直向redolog中写入，最后提交时才写入binlog。

#### 区分redolog和change buffer
1. 前者为减少写操作，后者为减少读操作
2. 前者是日志文件，后者主要是内存中
3. 修改数据时，如果数据页已经在内存中，只写redolog。不在内存中，才会同时写change buffer和redolog
