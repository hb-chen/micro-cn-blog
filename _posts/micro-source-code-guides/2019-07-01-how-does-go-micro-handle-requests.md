---
layout: post
title: Micro源码系列 - Go-Micro如何处理请求
date:  2019-07-01 09:00:00
profile: sx
author: 舒先
author_url: https://github.com/printfcoder
author_bio: maintainer of micro & overseeing micro China, senior engineer@huize
---

我们过去用了两篇博客给大家介绍了Go-Micro是如何创建服务与注册服务的，服务启动后，它又是如何处理（handle）请求的呢？那接下来我们就介绍Go-Micro的Handler是怎么接收请求的。

## 接口声明

一般情况下，Micro服务使用proto文件来声明接口，并通过protoc指令与micro插件（）

## 注册接口

## handler

### router

## 侦听

**Micro**中负责传输、处理连接的模块是Transport，它在**Go-Micro**模块中的位置如下，在server.Start方法中，

// todo --go-micro图片

```go
err := ts.Accept(s.ServeConn)
```

// printfstack 调用链

### ServerConn

## 总结

## Micro源码系列

1. [Go-Micro服务的构造过程](https://micro.mu/blog/cn/2019/05/23/how-does-go-micro-server-be-bulit.html)
2. [Go-Micro注册解读](https://micro.mu/blog/cn/2019/06/01/how-does-go-micro-register-services.html)
3. [Go-Micro请求处理](https://micro.mu/blog/cn/2019/07/01/how-does-go-micro-handle-requests.html)
4. [Go-Micro客户端解读（in progress）]

## 参考阅读

[服务注册](https://microservices.io/patterns/service-registry.html)