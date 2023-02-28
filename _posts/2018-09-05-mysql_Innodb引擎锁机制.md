---
layout: post
title: mysql_innodb引擎锁机制
date: 2018/9/5
categories: blog
tags: [MYSQL]
description: 一个优秀的程序员对于mysql数据库所具备的知识点（面试必备）。

---

## 1、锁实现的目的

为了控制并发带来的读取，修改数据产生数据不一致，出现脏数据的情况。
锁可以解决并发访问的问题，但会影响系统的性能。

## 2、锁的种类

在mysql中：
* 锁按使用的方式可以分为乐观锁以及悲观锁，而悲观锁通过select...for update可以实现mysql的排他锁/lock in share mode 可以实现共享锁；
* 锁按粒度来分的话可以分为表锁，行锁，页锁。在MyISAM引擎中只存在表锁，而innodb既支持表锁，也支持行锁。其中表锁分表共享读锁，表独占写锁。而行锁则有共享锁以及排他锁。

## 3、mysql innodb 锁

* UPDATE、INSERT、DELETE InnoDB会自动给涉及的数据集加排他锁
* select一般不加锁，但select...for update可以实现mysql的排他锁/lock in share mode 可以实现共享锁（select...for update可能会造成死锁）。
* 共享锁主要用在需要数据依存关系时来确认某行记录是否存在，并确保没有人对这个记录进行UPDATE或者DELETE操作。共享锁只要确保该条记录都能读，但不允许出现写操作（update，delete等），所以共享锁也是读锁。
* 排他锁则是对该条数据具有独占权利，可以对该条数据进行读写操作，所有排他锁也是写锁。（所以可以利用这点，将排他锁作为分布式锁）。

## 4、间隙锁

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙(GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。（解决幻读问题）


## 5、注意点

* 行锁必须有索引才能实现，否则会自动锁全表（表锁），那么就不是行锁了。
* 两个事务不能锁同一个索引

        事务A先执行：
        select math from zje where math>60 for update;
         
        事务B再执行：
        select math from zje where math<60 for update；
        这样的话，事务B是会阻塞的。如果事务B把 math索引换成其他索引就不会阻塞，但注意，换成其他索引锁住的行不能和math索引锁住的行有重复。

* 阻塞案例    
        
        先执行
        select  math  from zje where math>60 for update；
        
        再执行
        update zje set math=99 where math=68；

此时第二条执行语句会被阻塞，第一条语句被加上了排他锁，执行第二条语句时需要先释放锁。

* 死锁案例
        
        先执行
        update zje set math=99 where math=59；
        
        再执行
        select  math  from zje where math>60 for update；
        
再执行第一条语句时，数据库会为math为99加上行锁（排他锁），按理来说，第二条语句会避开math=99这条语句，但是系统会考虑第一条语句是否会出现rollback