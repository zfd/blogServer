---
title: nginx、CGI、fastCGI、php-cgi、php-fpm
date: 2018-02-07 17:12:45
categories: 服务器
tags: [nginx, CGI, fastCGI, php-fpm]
---

## 名词解释

+ `nginx`：web server，内容分发者，只能处理静态文件，动态脚本只能交给php自己处理。
+ `CGI`：通用网关接口（common gateway interface），是 Web Server 与 Web Application 之间数据交换的一种协议。
+ `FastCGI`：同 CGI，是一种通信协议，但比 CGI 在效率上做了一些优化。同样，SCGI 协议与 FastCGI 类似。
+ `PHP-CGI`：是 PHP（Web Application）对 Web Server 提供的 CGI 协议的接口程序。
+ `PHP-FPM`：是 PHP（Web Application）对 Web Server 提供的 FastCGI 协议的接口程序，它直接管理多个 PHP-CGI 进程。

## fastCGI

fastCGI可以理解为一个常驻型的cgi，其主要行为是将CGI解释器进程保持在内存中，并因此获得较高的性能。

### 作用

+ 解决cgi并发重复fork的问题；
+ 支持分布式运算；

## php-fpm

就是 php-cgi 的进程管理器，修改php.ini后，可以重新fork php-cgi进程，实现平滑重启。

### 运行模式

+ static：静态分配，启动时分配固定的worker进程；
+ ondemand：按需分配，当收到用户请求时fork worker进程；
+ dynamic：动态分配，启动时分配固定进程，用户请求增加，按照设定范围浮动；

### 运行原理

一个master进程，管理多个worker进程；

master进程：

1. 初始化cgi；
1. 初始化php环境；
1. 初始化php-fpm；
1. 运行，主进程fork子进程，主进程阻塞，事件循环；

worker进程：

1. 接收请求；
1. 处理请求；
1. 返回请求结果；

## nginx，php-fpm工作流程

```
 www.example.com
        |
        |
      Nginx
        |
        |
路由到www.example.com/index.php
        |
        |
加载nginx的fast-cgi模块
        |
        |
www.example.com/index.php请求到达127.0.0.1:9000（fast-cgi协议）
        |
        |
php-fpm 监听127.0.0.1:9000
        |
        |
php-fpm 接收到请求，启用worker进程处理请求
        |
        |
php-fpm 处理完请求，返回给nginx
        |
        |
nginx将结果通过http返回给浏览器

```
