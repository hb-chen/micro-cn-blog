---
layout: post
title: 2019年Micro的整合工作
date:  2019-06-01 09:00:00
translator: 舒先
translator_url: https://github.com/printfcoder
translator_bio: maintainer of micro & overseeing micro China, senior engineer@huize
---

Micro生态以[**go-micro**](https://github.com/micro/go-micro)微服务开发框架为核心，已经开启她的微服务演近进程，Micro一直专注为微服务核心开发提供方案，通过Micro的抽象结构，大家在开发过程中不需要再关心微服务系统架构的复杂性。

经过几年的时间演进，我们已经把Go-Micro扩展超过了原先的定位，发展出了其它工具、库、插件。这一过程在**虽然解决了问题**与**期望开发者如何（简便地）使用Micro工具链**之间产生了矛盾。所以我们现在正准备把这些工具库整合起来，给大家提供更好的使用体验。

Micro自己的定位是成为微服务开发中的独立开发框架与运行时管理工具。

在讨论整合工作之前，我们回顾一下截止目前的情况。

## 核心焦点

项目刚开始的时候，Go-micro的定位是专注于微服务之间的通信，并且我们一直努力实现这一目的。到目前为止，我们的坚持与专注也正是驱动micro逐渐走向成功的原因。经过这些年的发展，我们收到非常多的需求，这些功能都逐步加入到了Go-Micro中。它们之中大多数与弹性、安全、同步及可配置性有关。

<center>
<img src="https://micro.mu/docs/images/go-micro.svg" style="width: 80%; height: auto;"/>
</center>

把社区要求的那些额外功能加上去有很多好处，不过，我们更希望从一开始就集中精力解决一个问题，所以我们采取了不同的方式，促进社区完成这些事。

## 生态系统与插件

打造产品并不仅仅是实现服务发现、消息编码以及请求-响应。我们一直铭记着，同时也一直希望我们可以通过插件化、可扩展的接口，让开发者们拥有更多选择，以帮助大家开发出更强大的服务平台。
我们在[探索](https://micro.mu/explore/)页面展示了部分使用我们框架micro以及[go-plugins](https://github.com/micro/go-plugins)的开源作品。

<center>
<img src="https://micro.mu/explorer.png" style="width: 80%; height: auto;" />
</center>

Go Plugins已经支持开发者们在系统中集成复杂的工具，比如prometheus监控、zipkin链路追踪或者kafka等等。

## 关于服务接入点

在Micro体系中，Go-Micro作为微服务开发的大心脏，不过，一旦应用开发完成，其它问题就会随之而来，我们如何查询这些服务，如何与之交互，又如何将其对外服务，等等

Go-Micro基于rpc或protobuf协议，并且协议也是可插拔，不依赖运行环境，那我们就需要有某种机制来定位到某个服务，对于Go-Micro本身亦是如此。基于这些想法，我们创建了[**micro**](https://github.com/micro/micro)，它是微服务工具集，可以通过它提供API网关、web管理控制台、命令行CLI、Slack机器人、服务代理等等。 

<center>
<img src="https://micro.mu/runtime3.svg" style="width: 80%; height: auto;"/>
</center>

我们在[**micro**](https://github.com/micro/micro)集成有HTTP API、浏览器界面、Slack命令以及CLI接口，大家可以通过micro管理服务。这些都是我们在构建服务时常见的方法，而且我们需要提供运行时工具使得这些成为可能，不过，现在的重心仍然在通信上。

 
## 其它工具

我们的插件与工具集尽管已经具备很多功能，但是仍然有不少欠缺。社区希望我们能围绕Micro平台持续开发，解决更多的问题，而不是他们在自己公司内部去拼拼揍揍。针对工具库，我们要做到像Go-micro插件那样的抽象化，比如像动态配置，分布同步，支持更多的系统如k8等等。 

针对这些问题，我们已经开发出下列的工具库：

- [micro/go-config](https://github.com/micro/go-config) - 动态配置库
- [micro/go-sync](https://github.com/asim/go-sync) - 分布式同步库
- [micro/kubernetes](https://github.com/micro/kubernetes) - micro的k8s初始化构建工具
- [examples](https://github.com/micro/examples) - 示例应用
- [microhq](https://github.com/microhq) - micro服务预置库

这些仓库、工具都是为了解决社区里提出的问题而创建的。经过4年的发展，对于老用户而言尚能接受，但是对于新用户来说，分散的仓库导致大家体验起来越来越麻烦、晦涩。想到这些过高的门槛，我们得做出改变。

## 整合

过去的几周里我们发布了[**go-micro**](https://github.com/micro/go-micro)数个版本，主要聚焦在我们用户的需求上。我们渐行渐悟，有些仓库本身作为框架的一部分，理应包含在框架之中的，但是因为历史原因彼此是分隔的，所以我们要把它们整合起来，让大家不需要再分心去需其它地方查找集成。

从根本上说，**go-micro**将会演变成微服务开发的全家桶。

我们已经开始把我们所有的库都整合到go-micro中，接下来的几个星期我们会进行重构，然后给大家一个更至简、更丰富的开发体验，比如日志、链路追踪、指标监控、安全认证等等。

<center>
<img src="{{ site.baseurl }}/assets/images/go-micro-repo.png" style="width: 80%; height: auto;" />
</center>

<br>

不过，我们也没有忘记[**micro**](https://github.com/micro/micro)。我们认为，大家构建微服务，就需要有入口去查询接口、执行命令以及管理这些服务。大家一致认为**micro**要发展成Micro体系中的运行时管理工具。我们正着手实现Micro体系开发到管理的闭环，未来我们会有新消息发布。

## 总结

Micro可能是最简单微服务构建框架，她也逐渐成为Golang微服务开发的标准。现在我们通过把Micro仓库中现有的工具与库整合，打造成单一的开发框架与运行时，不再分散到不同库中，让大家在开发微服务时更加简单。

<center>...</center>
<br>

想了解更多，请访问我们的[官方站点](https://micro.mu)，订阅我们的[twitter](https://twitter.com/microhq)、微信公众号MicroHQ、[微博](https://weibo.com/microhq)。
也可以加入[slack](https://micro-services.slack.com)，选择加入中国区Channel

<h6><a href="https://github.com/micro/micro"><i class="fa fa-github fa-2x"></i> Micro</a></h6>
