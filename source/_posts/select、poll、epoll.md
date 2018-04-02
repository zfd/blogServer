---
title: select、poll、epoll
date: 2018-03-30 14:39:42
categories: 服务端
tags: [select, poll, epoll]
---

## 相关概念

### 一个IO操作流程

一个read（O）操作的2个阶段：

+ 对于socket，等待从网络收到数据，并且在数据到达后，复制数据到内核缓冲区；
+ 从内核缓冲区复制数据到用户进程缓冲区，以便进程处理。

### 同步、异步

是一种通信机制，关注的是IO操作的结果返回方式。

+ 同步：提交请求->等待服务器回应->处理完毕，中间一直等待；
+ 异步：提交请求->处理其他事情，服务器回应->通知回调->处理完毕。

### 阻塞、非阻塞

是一种调用机制，关注的是IO操作的执行状态。

+ 阻塞：调用方等待IO操作完成后返回；
+ 非阻塞：调用方不需要等待IO操作的完成就立即返回。

### 用户态、内核态

> 内核态：控制计算机硬件资源，并提供上层应用程序运行的环境；
> 用户态：上层应用程序的活动空间，应用程序运行必须依赖内核提供的资源，如CPU资源、存储资源、I/O资源等。

### 文件描述符（file descriptor，简称fd）

> 在Linux下面一切皆文件，fd是内核为文件所创的索引，所有I/O操作都要调用fd来执行。

### 回调函数（callback）

> 回调函数就是一个通过函数指针调用的函数。

+ 定义一个回调函数；
+ 提供函数实现的一方在初始化的时候，将回调函数的函数指针注册给调用者；
+ 当特定的事件或条件发生的时候，调用者使用函数指针调用回调函数对事件进行处理。

## 网络通信模型

+ 阻塞式IO

+ 非阻塞式IO

+ IO多路复用

+ 信号驱动IO

+ 异步IO

## IO多路复用

> I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。

### select

#### 函数

```c++
#include <sys/select.h>
#include <sys/time.h>

/**
 * 参数：
 * maxfdp: 待监听的最大fd数+1。
 * readSet：待监听的可读fd集合
 * writeSet：待监听的可写fd集合
 * exceptSet：待监听的异常fd集合
 * timeval：指定超时，NULL为一直等到，0为不等待
 *
 * 返回：
 * 就绪描述符的数目，超时返回0，出错返回-1，
 * 正常返回后，对应fd_set会设置相关满足条件的fd。
 */
int select(int maxfdp, fd_set *readset, fd_set *writeset, fd_set *exceptset, const struct timeval *timeout);

//fd_set相关操作
void FD_ZERO(fd_set *fdset); //清空集合
void FD_SET(int fd, fd_set *fdset);//将一个给定的文件描述符加入集合之中
void FD_CLR(int fd, fd_set *fdset);//将一个给定的文件描述符从集合中删除
int FD_ISSET(int fd, fd_set *fdset);//检查集合中指定的文件描述符是否可以读写

//相关结构
struct timeval {
    long tv_sec;//seconds
    long tv_usec;//microseconds
};

#define __NFDBITS (8 * sizeof(unsigned long)) //32位编译器，unsigned long为4个字节
#define __FD_SETSIZE 1024
#define __FDSET_LONGS (__FD_SETSIZE/__NFDBITS)

typedef struct {
    unsigned long fds_bits [__FDSET_LONGS];
} __kernel_fd_set;
```

#### 缺点

+ 单个进程监听的fd数量有限，最多为1024；（能改，但是改后影响效率）；
+ 每次调用select，都需要遍历所有fd，才能发现哪些发生了事件，效率慢；
+ 内存复制开销大，需要从用户空间、内核空间来回拷贝fd_set；
+ 触发方式是水平触发，应用程序如果没有完成对一个就绪的fd进行IO操作，那么下次select调用还是会将这些fd通知进程；

### poll

poll的实现和select非常相似，只是描述fd集合的方式不同，poll使用pollfd结构而不是select的fd_set结构，其他的都差不多。

就少了select的fd数量限制，其他缺点仍存在。

#### 函数

```C++
# include <poll.h>
int poll ( struct pollfd * fds, unsigned int nfds, int timeout);
```

### epoll

