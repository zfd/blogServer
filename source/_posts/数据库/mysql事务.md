---
title: mysql事务
date: 2018-04-25 09:44:17
categories: [数据库]
tags: [mysql, acid]
---

## 四个特性（ACID）

+ 原子性（atomicity）：事务包含的所有操作，要么全部成功，要么全部失败；
+ 一致性（consistency）：事务执行前后，mysql的状态是一致的；
+ 隔离性（isolation）：并发时，多个事务互相不干扰；
+ 持久性（durability）：事务一旦提交，对于数据库的数据改变就是永久的。

## 开启命令

```
begin // start transaction

...

commit; // rollback;
```

## 隔离级别

|隔离级别|脏读|不可重复读|幻读|
|---|---|---|---|
|未提交读（read uncommitted）|O|O|O|
|已提交读（read committed）|X|O|O|
|可重复读（repeatable read）|X|X|O|
|可串行化（serializable）|X|X|X|

### 脏读（dirty read）

事务A读取了事务B未提交的数据。

### 不可重复读（nonRepeatable read）

一个事务中，多次查询某个数据却返回不同的值；这是由于事务A查询间隔内，另一个事务修改并提交了该数据。

### 幻读

一个事务中，读取到了别的事务插入的数据，导致前后不一致。

## 锁类型

### 表锁

对一张表加锁，并发能力低下（读/写锁）。

### 行锁

只锁住特定行的数据，并发能力强。

```
//2种一致性锁定读操

//共享锁
select * from table where ? lock in share mode;

//排它锁
select * from table where ? for update;
```

### 间隙锁（GAP锁）

使用索引对行锁两边的区间加锁，避免其他事务在这两个区间插入数据。

### next-key锁

就是行锁+间隙锁，用来避免幻读。

### 多版本并发控制（MVCC）

乐观锁，通过版本号控制，最后提交检查版本号再提交事务。

## 锁的应用

### 避免脏读

脏读：事务A读到事务B未提交的数据；

解决：

+ 事务级别，已提交读以上，都只会读取已提交数据；
+ 未提交读级别下，可以用select加排它锁，防止其他事务读；

### 避免不可重复读

不可重复读：事务中多次查询数据的值不一致。

处理：

+ 可重复读之下，使用mvcc解决，读取一个快照数据；
+ 已提交读之上，可以对select加共享锁，防止其他事务写；

### 避免幻读

使用next-key锁（间隙锁+行锁），innodb自动加。

用法：(隔离级别：可重复读)

```
begin;
select * from xxx where id between 10 and 15 lock in share mode;//for update
...
commit;//rollback;
```

+ 防止间隙插入新数据；
+ 防止已有数据，更新成间隙内的数据；

可串行化：是默认读加共享锁，写加排他锁，读写互斥。
