---
title: nginx、CGI、fastCGI、php-cgi、php-fpm
date: 2018-02-07 17:12:45
categories: 服务端
tags: [nginx, CGI, fastCGI, php-fpm]
---

## 名词解释


+ `nginx`：web server，内容分发者，只能处理静态文件，动态脚本只能交给php自己处理。
+ `CGI`：通用网关接口（common gateway interface），是 Web Server 与 Web Application 之间数据交换的一种协议。
+ `FastCGI`：同 CGI，是一种通信协议，但比 CGI 在效率上做了一些优化。同样，SCGI 协议与 FastCGI 类似。
+ `PHP-CGI`：是 PHP（Web Application）对 Web Server 提供的 CGI协议的接口程序。
+ `PHP-FPM`：是 PHP（Web Application）对 Web Server 提供的 FastCGI 协议的接口程序，额外还提供了相对智能一些任务管理。


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
fast-cgi监听127.0.0.1:9000地址
        |
        |
www.example.com/index.php请求到达127.0.0.1:9000
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
