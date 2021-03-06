---
title: mysql相关锁
date: 2018-04-11 23:02:59
categories: [数据库]
tags: [mysql, lock]
---

## 乐观锁

+ 乐观地认为数据没有别处修改，在完成业务的时候再拿锁（其实不会上锁，只是判断有无修改）。
+ mysql没有提供具体乐观锁，是要程序自己实现。

### 做法

用version字段（主要用这个），或时间戳字段：

+ select data as old_data， version as old_version from ... where ...；
+ 处理data、version；
+ update set date = new_data，version = new_version where version = old_version。
+ updated row > 0，则成功提交事务；否则回滚。

## 悲观锁

+ 悲观地认为数据会被别处修改，因此先确保获取锁成功再进行业务操作。（一锁二查三更新）
+ 表锁、行锁、共享锁、排它锁，都是悲观锁，都是操作前先上锁。
+ mysql提供这种机制，直接用就行。

### 跟乐观锁开销比较

+ 取锁成功情况下，乐观锁开销较小；
+ 取锁失败情况下，需要回滚，则乐观锁开销较大。

总结：写操作少时，用乐观锁；写操作多时，用悲观锁。

### 表锁、行锁

+ 表锁锁定整个表，开销小，加锁快；不会出现死锁；锁定力度大，锁冲突概率高，并发度低；
+ 行锁锁定若干行，开销大，加锁慢；会出现死锁；锁定力度小，锁冲突概率低，并发度高。

### 共享锁、排他锁、自旋锁

+ 共享锁：又叫读锁，加锁后，其他线程可读，不可写；
+ 排它锁：又叫写锁，加锁后，其他线程不可读、不可写；
+ 自旋锁：跟排它锁类似，但不会引起调用者睡眠。

#### 排他锁、自旋锁，比较

##### 原理

+ 排它锁：线程->sleep（加锁）->running（解锁），过程有上下文切换，cpu抢占，信号量发送等；
+ 自旋锁：线程->running（加锁->解锁），死循环检测锁的标志位。

##### 开销

初始开销，排它锁较高；但时间越长，排它锁不变，自旋锁线性增长。

##### 场景

+ 互斥锁，适用于临界区锁时间较长的操作：临界区有IO操作，临界区竞争激烈，单核处理器等；
+ 自旋锁，适用于临界区锁时间非常短，且CPU资源不紧张，一般用于多核的服务器。
