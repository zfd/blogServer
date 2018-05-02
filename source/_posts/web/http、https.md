---
title: http、https
date: 2018-05-01 16:01:41
categories: web
tags: [http, https]
---

## http

> HyperText Transfer Protocol，超文本传输协议。

### http请求

> CRLF：carriage-return line-feed，回车、换行，\r\n

+ 请求行：请求方法 url 协议/版本 CRLF
+ 请求头：key ： value CRLF（N行）
+ 空行
+ 请求体：get请求没有正文，post请求正文是参数；

### http响应

+ 状态行：协议/版本 状态码 状态码描述 CRLF
+ 响应头：key ： value CRLF（N行）
+ 空行
+ 响应体：html/json等

### 状态码

#### 1XX，指定客户端的响应动作

+ 100，continue，，客户端应当继续发送请求；
+ 101，switching protocols，服务器通过Upgrade消息头通知客户端采用不同协议，下个请求将转换协议；

#### 2XX，服务端成功处理了请求

+ 200，ok，请求成功；
+ 201，created，请求已经被实现，而且有一个新的资源已经依据请求的需要而建立，且其URI已经随Location头信息返回；
+ 202，accepted，服务器已接受请求，但尚未处理；
+ 203，non-authoritative information，文档已经正常地返回，但一些应答头可能不正确，因为使用的是文档的拷贝，非权威性信息；
+ 204，no content，服务器成功处理了请求，但没有返回任何内容；
+ 205，reset content，没有新内容返回，但是浏览器要清除表单域；

#### 3XX，重定向类

+ 300，multiple choices/多重选择，被请求的资源可以在多个地方找到，并返回列表供选择（如有首选项，则在location中指定）；
+ 301，moved permanently，请求永久重定向；
+ 302，found，请求临时重定向；
+ 304，not modified，客户端有缓存，可以直接使用；
+ 305，use proxy，请求必须通过指定的代理才能被访问（location中指定）；

#### 4XX，客户端请求错误

+ 400，bad request，客户端请求中有语法错误；
+ 401，unauthorized，未授权，客户端请求需要身份验证；
+ 403，forbidden，服务器拒绝该请求，权限不足等；
+ 404，not found，服务器找不到该请求的任何资源；
+ 405，method not allow，请求的方法不允许；

#### 5XX，服务端错误

+ 500，internal server error，服务器错误；
+ 501，not implemented，服务端不支持请求中要求的功能；
+ 502，bad gateway，网关错误，如：php-fpm挂了；
+ 503，service unavailable，服务器在维护或已过载，无法处理请求；
+ 504，gateway timeout，网关超时；
+ 505，http version not supported，不支持该http版本；

### http的缺点

+ 窃听风险：黑客可以获取通信的内容；
+ 篡改风险：黑客可以修改通信的内容；
+ 冒充风险：黑客可以冒充他人身份参与通信；

## https

### 对称加密、非对称加密

对称加密效率更高，非对称加密更安全。

### https工作流程

1. 客户端发出https请求；
1. 服务器返回ca证书（公钥、数字签名、相关信息等）；
1. 客户端验证证书；
1. 客户端生成对称秘钥（随机字符串），用公钥加密，并发送服务器；
1. 服务器用私钥解密，得到对称秘钥，然后用对称秘钥加密请求的返回内容；
1. 客户端用对称秘钥解密返回内容，并处理；
1. 后续请求都用该对称秘钥加密、解密通信内容。

### https优点

+ 解决窃听风险：通信内容加密；
+ 解决篡改风险：具有校验机制，一旦被篡改，通信双方会立刻发现；
+ 解决冒充风险：配备证书，防止身份被冒充；

### https缺点

+ ssl证书费用高，增加部署、维护的工作；
+ https请求，握手次数增加，降低用户访问速度；
+ http重定向到https，增加了用户的访问耗时；
+ https涉及的安全算法会增加cpu资源的消耗；
