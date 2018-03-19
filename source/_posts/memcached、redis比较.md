---
title: memcached、redis比较
date: 2018-03-19 17:03:13
categories: [服务端]
tags: [nosql, memcached, redis]
---

## memcached优点

+ 存储数据量大于10w时，性能更高；
+ 存储结构简单（key/value），对内存利用率更高；

## redis优点

+ 存储小数据，性能更高；
+ 支持结构类型更丰富；
+ 支持持久化，存储数据更安全；
+ 支持数据备份，主从模式；

## 总结

有持久化需求或者对数据结构和处理有更高要求，选择redis；其他简单的key/value存储，选择memcache。
