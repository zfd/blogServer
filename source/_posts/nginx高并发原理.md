---
title: nginx高并发原理
date: 2018-02-03 15:11:53
categories: 服务端
tags: [server, nginx, epoll]
---

Nginx 采用的是多进程 && 多路IO复用模型。

## 多进程的工作模式

* Nginx启动后，会有一个master进程和多个项目独立的worker进程。
* master进程接收来自外界信号，然后向worker进程发送，每个进程都有可能处理这个连接。
* master进程监控worker进程运行状态，worker进程异常退出时，启动新的进程。

注：worker进程数一般为cpu核数，因为更多的worker进程数会导致相互竞争cpu，从而带来不必要的上下文切换。

### 惊群现象 && 解决方法

现象：
master进程创建一个sockfd（socket文件描述符），然后fork子进程，子进程继承父进程的sockfd，之后子进程accept后再创建已连接描述符，最后与客户端通信。
由于，所有子进程都继承父进程的sockfd，当客户端连接进来时，所有子进程都会收到通知并抢着与他建立连接，这就叫`惊群现象`。
大量的进程被激活又挂起，只有一个进程可以accept这个连接，当然会消耗系统资源。

处理：
nginx提供一个accept_mutex，这是一个加在accept上的一把互斥锁。
同一时刻只有一个子进程去accept，这里就不会有惊群问题了。

### worker进程工作流程

accept客户端的连接->读取请求->解析请求->处理请求->返回结果给客户端->断开连接。

一个请求，完全由一个worker进程处理，好处：

* 节省锁带来的开销；每个worker进程都是独立的进程，不共享资源，不需要加锁。
* 独立进程，减少风险，进程之间相互不会影响。

## 多路IO复用模型epoll

epoll通过在linux内核中申请一个简易的文件系统（数据结构：B+树），工作原理：

1. 调用 `int epoll_create(int size)` 建立一个epoll对象，内核会创建一个eventpoll结构体，用于存放通过epoll_ctl()向epoll对象中添加进来的事件，这些事件都会挂载在红黑树中。
1. 调用 `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)` 在 epoll 对象中为 fd 注册事件，所有添加到epoll中的事件都会与设备驱动程序建立回调关系，也就是说，当相应的事件发生时会调用这个sockfd的回调方法，将sockfd添加到eventpoll 中的双链表。
1. 调用 `int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout)` 来等待事件的发生，timeout 为 -1 时，该调用会阻塞知道有事件发生

