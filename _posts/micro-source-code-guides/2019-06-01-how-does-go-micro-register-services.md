---
layout: post
title: Micro源码系列 - Go-Micro服务是如何注册的
date:  2019-06-01 09:00:00
profile: sx
author: 舒先
author_url: https://github.com/printfcoder
author_bio: maintainer of micro & overseeing micro China, senior engineer@huize
---

微服务架构中注册是非常有意思的角色，服务中的客户端通过注册机制定位目标服务的具体位置。服务注册中心可以说是服务实例的数据库，在里面有服务的各种信息，包括位置等。服务实例在启动时通过注册机制**注册**到中心，并且在关闭前从中心自动**卸载**。不过光有**注册**与**卸载**两个步骤还不够，在两者之间，我们还需要**健康检查**来确定服务是否可以持续接收请求。

同时，我们需要指出一个大家特别容易犯的错误：很多人都会觉得服务注册就是为了负载均衡，其实不是，<u>服务注册是为了客户端或服务端定位服务实例，并确定选择哪一个服务来发送请求的机制</u>。而负载均衡只是选择服务时如何让各服务之间平衡提供响应的策略，它可能依赖注册，但不是必须，因为有哪些服务可以通过很多方式告之客户端，而并且非一律从注册中心获取。

Micro体系中的每一种类型的服务都包含有注册组件（Registry）。当服务启动时，它会把所有描述自身信息的元数据（metadata，比如服务名、地址、transport、编码等等）提取出来，作为关键信息，用于下一步注册成为服务节点。尔后，如果声明有TTL和Interval，则会定期触发重注册机制。

[前面一章](/blog/cn/2019/05/23/how-does-go-micro-server-be-bulit.html)我们大体讲解了Go-Micro中的服务是如何构建的。接下来我们就从代码层面给大家演示服务注册。

## 注册中心接口

注册组件接口[**registry**](https://github.com/micro/go-micro/blob/master/registry/registry.go)中：

```go
package registry
// ...
type Registry interface {
        Init(...Option) error
        Options() Options
        Register(*Service, ...RegisterOption) error
        Deregister(*Service) error
        GetService(string) ([]*Service, error)
        ListServices() ([]*Service, error)
        Watch(...WatchOption) (Watcher, error)
        String() string
}
```

在go-micro包中，共有4种注册实现consul、gossip、mdns、memory，前两个都是基于hashicorp公司的协议，mdns则是基于组网广播实现，memory则是本地实现。

- consul 依赖hashicorp的组件，但是功能强大、完整
- gossip 基于SWIM协议广播，零依赖
- mdns 轻量、零依赖，但是对环境有要求，某些环境不支持mdns的无法正常使用
- memory 本地解决方案，不可跨主机访问

另外在[**go-plugins**](https://github.com/micro/go-plugins/tree/master/registry)中有其它注册实现，比如etcd、eureka、k8s、nats、zk等等

大体解释下接口中每个方法的作用

- Init 初始化
- Options 获取配置选项
- Register 注册服务
- Deregister 卸载服务
- GetService 获取指定服务
- ListServices 列出所有服务
- Watch watcher 负责侦听变动
- String 注册信息转成字符串描述

可见，接口定义的注册组件是几乎完全自包含，它自行注册、卸载、侦听等，服务不需要关心自己如何注册、卸载，只需要将注册中心的实现作为Option导入自身启动即可

> 通过定义注册组件接口，我们便可以将服务与注册中心解耦

## 声明注册中心

我们知道Go-Micro可以通过命令行参数--**registry**或者方法参数**micro.Registry**来指定服务注册中心，但是**Register**方法中并没有选择注册中心的过程，我们看下Go-Micro在构建服务时的动作：

命令行参数：

```bash
go run main.go --registry=consul
```

Go-Micro预置有4种命令行参数：

[cmd.go](https://github.com/micro/go-micro/blob/master/config/cmd/cmd.go#L189-L194)

```go
DefaultRegistries = map[string]func(...registry.Option) registry.Registry{
                "consul": consul.NewRegistry,
                "gossip": gossip.NewRegistry,
                "mdns":   mdns.NewRegistry,
                "memory": rmem.NewRegistry,
}
```

在识别命令行传入参数后，Micro就会匹配DefaultRegistries中的key，然后把注册组件附加给服务。

> *Env方式大同小异，这里不表

方法参数，通过micro.Registry传入：

```go
        micReg := consul.NewRegistry(registryOptions)
        service := micro.NewService(
                // ...
                micro.Registry(micReg),
                // ...
        )
```

因为Registry是自包含的，故而我们只需要将其传入服务，让服务调用即可。

## 服务启动

简单回顾下服务在**Start**时的动作，我们用默认的[rpc_server](https://github.com/micro/go-micro/blob/master/server/rpc_server.go#L480-L605)来演示，其它如grpc_server等大同小异，不影响理解。

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
        
        // Interval、卸载代码，下面我们会讲到
        return nil
}
```

**Start()**方法在检测完信息后便进行注册动作，下面我们分析注册方法**Register**

Micro服务在注册时有两个关键点，元数据、自定义handler

服务向中心注册一般可以分为如下几个步骤：

1.解析注册中心地址

2.准备元数据

3.声明节点信息

4.声明endpoint handlers

5.声明服务

6.注册

整个流程我们缩略成一个二维集合图：

<a href="https://micro.mu/blog/cn/assets/images/registry-pie.png">
  <img src="/blog/cn/assets/images/registry-pie.png" style="width: 100%; height: auto; margin: 0;" />
</a>

接下来我们分析一下注册流程代码，大家请配合上面的集合图阅读，方便理解

```go
func (s *rpcServer) Register() error {
        // 解析注册中心地址
        // 忽略这部分代码

        // 准备元数据
        md := make(metadata.Metadata)
        for k, v := range config.Metadata {
                md[k] = v
        }

        // 声明节点信息
        node := &registry.Node{
                Id:       config.Name + "-" + config.Id,
                Address:  addr,
                Port:     port,
                Metadata: md,
        }

        node.Metadata["transport"] = config.Transport.String()
        node.Metadata["broker"] = config.Broker.String()
        node.Metadata["server"] = s.String()
        node.Metadata["registry"] = config.Registry.String()
        node.Metadata["protocol"] = "mucp"

        s.RLock()
        
        // 声明endpoint，map元素顺序是随机的，故而使用key排序，方便每个同名服务之间显示一致
        var handlerList []string
        for n, e := range s.handlers { 
                if !e.Options().Internal {
                        handlerList = append(handlerList, n)
                }
        }
        sort.Strings(handlerList)

        var endpoints []*registry.Endpoint
        for _, n := range handlerList {
                endpoints = append(endpoints, s.handlers[n].Endpoints()...)
        }
         
        // 忽略部分代码

        // 声明服务信息
        service := &registry.Service{
                Name:      config.Name,
                Version:   config.Version,
                Nodes:     []*registry.Node{node},
                Endpoints: endpoints,
        }

        s.Lock()
        registered := s.registered
        s.Unlock()

        // 构建注册选项
        rOpts := []registry.RegisterOption{registry.RegisterTTL(config.RegisterTTL)}

        // 注册
        if err := config.Registry.Register(service, rOpts...); err != nil {
                return err
        }
    
        // 忽略部分订阅代码

        s.registered = true
}
```

以上便是服务向注册中心注册时的主要流程代码，整个注册过程非常简单。服务注册完后，我们还要定期检查与声明生存周期，也即是Interval与TTL（Time-To-Live）机制。

## Interval

与Register注册一样，Interval由服务触发，而不是由Registry触发，因为Registry已经暴露了Register接口，而Interval的工作只是定时重新调用Register方法，如果再把Interval放到其中，便会导致每个Registry实现都会有相同的Interval代码。

我们再回顾一下上面说到的**Start**方法，Start方法中除了注册之外，还有循环重注册的逻辑，这一部分就是利用Interval指定的值，不间断重复向注册中心注册，以达到在线的目的：

```go
func (s *rpcServer) Start() error {
	// 忽略部分代码

	go func() {
		t := new(time.Ticker)

		// 仅在声明了Interval时才会执行，每隔Interval指定的时间，发送一次信号
		if s.opts.RegisterInterval > time.Duration(0) {
			t = time.NewTicker(s.opts.RegisterInterval)
		}

		// return error chan
		var ch chan error

	Loop:
		for {
			select {
			// 当接收到Interval信号时重新执行注册操作
			case <-t.C:
				s.RLock()
				registered := s.registered
				s.RUnlock()
				if err = s.opts.RegisterCheck(s.opts.Context); err != nil && registered {
					log.Logf("Server %s-%s register check error: %s, deregister it", config.Name, config.Id, err)
					// deregister self in case of error
					if err := s.Deregister(); err != nil {
						log.Logf("Server %s-%s deregister error: %s", config.Name, config.Id, err)
					}
				} else {
					if err := s.Register(); err != nil {
						log.Logf("Server %s-%s register error: %s", config.Name, config.Id, err)
					}
				}
			// 直到接收到退出信号，才停止重注册
			case ch = <-s.exit:
				t.Stop()
				close(exit)
				break Loop
			}
		}

		// 忽略部分卸载、关连接的代码
	}()

	return nil
}
```

> 当重注册循环停止时，相当于服务不再生效，故而需要卸载、停止侦听连接请求等操作。

## TTL

TTL与Register不同，它由注册组件执行，并非以服务直接调用。故而不同的注册中心组件有不同的实现。我们这里不深入讨论，后继如果有机会，我们再讨论每个中心的TTL机制。

## 卸载

服务卸载相当于注册的逆过程。 

```go
func (s *rpcServer) Deregister() error {
	config := s.Options()
	
	// 忽略部分地址解析代码

	node := &registry.Node{
		Id:      config.Name + "-" + config.Id,
		Address: addr,
		Port:    port,
	}

	service := &registry.Service{
		Name:    config.Name,
		Version: config.Version,
		Nodes:   []*registry.Node{node},
	}

	if err := config.Registry.Deregister(service); err != nil {
		return err
	}

	s.Lock()

	if !s.registered {
		s.Unlock()
		return nil
	}

	s.registered = false

	// 忽略部分订阅代码
	return nil
}
```

卸载的过程很简单，把服务名、版本号、节点信息向注册组件调用**Deregister**即可。

> 因为一个应用实例可能注册多个服务，故而，我们需要将服务名传过去，让注册组件停止对某个服务的侦听工作。

## 总结

我们在本篇中从源码的角度简单给大家介绍Go-Micro服务的注册流程，不过，我们并没有深入各注册中心组件去详解，这也超过本文的范畴，会让文章变得很重，大家有兴趣可以去查看各注册中心的客户端代码。

## Micro源码系列

1. [Go-Micro服务的构造过程](https://micro.mu/blog/cn/2019/05/23/how-does-go-micro-server-be-bulit.html)
2. [Go-Micro注册解读](https://micro.mu/blog/cn/2019/06/01/how-does-go-micro-register-services.html)
3. [Go-Micro请求处理](https://micro.mu/blog/cn/2019/07/01/how-does-go-micro-handle-requests.html)
4. [Go-Micro客户端解读（in progress）]

## 参考阅读

[服务注册](https://microservices.io/patterns/service-registry.html)
