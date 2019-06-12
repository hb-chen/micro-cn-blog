---
layout: post
title: Micro源码系列 - Go-Micro服务是如何注册的
date:  2019-06-01 09:00:00
profile: sx
author: 舒先
author_url: https://github.com/printfcoder
author_bio: maintainer of micro & overseeing micro China, senior engineer@huize
---

微服务架构中注册是非常有意思角色，服务中的客户端通过注册机制定位目标服务的具体位置。服务注册中心可以说是服务实例的数据库，在里面有服务的各种信息，包括位置等。服务实例在启动时通过注册机制**注册**到中心，并且在关闭前从中心自动**卸载**。不过光有**注册**与**卸载**两个步骤还不够，在两者之间，我们还需要**健康检查**来确定服务是否可以持续接收请求。

同时，我们需要指出一个大家特别容易犯的错误：很多人都会觉得服务注册就是为了负载均衡，其实不是，<u>服务注册是为了客户端或服务端定位服务实例，并确定选择哪一个服务来发送请求的机制</u>。而负载均衡只是选择服务时如何让各服务之间平衡提供响应的策略，它可能依赖注册，但不是必须，因为有哪些服务可以通过很多方式告之客户端，而并且非一律从注册中心获取。

[前面一章](/blog/cn/2019/05/23/how-does-go-micro-server-be-bulit.html)我们大体讲解了Go-Micro中的服务是如何构建的。接下来我们就从代码层面给大家演示服务注册。

## 调用注册

简单回顾下服务在**Start**时的动作：

```go
func (s *rpcServer) Start() error {
	// ...
	// use RegisterCheck func before register
	if err = s.opts.RegisterCheck(s.opts.Context); err != nil {
		log.Logf("Server %s-%s register check error: %s", config.Name, config.Id, err)
	} else {
		// 注册
		if err = s.Register(); err != nil {
			log.Logf("Server %s-%s register error: %s", config.Name, config.Id, err)
		}
	}

	// ...
	return nil
}
```

我们用默认的[rpc_server](https://github.com/micro/go-micro/blob/master/server/rpc_server.go#L480-L605)来演示，其它如grpc_server等大同小异，不影响理解。

**Start()**方法在检测完信息后便进行注册动作，下面我们分析注册方法**Register**

服务的向中心注册一般可以分为如下几个步骤：

1.解析注册中心地址

2.准备元数据

3.声明节点信息

4.调用注册中心register



## 注册中心

- 声明注册中心
- 注册
- 卸载

## 参考阅读

[服务注册](https://microservices.io/patterns/service-registry.html)