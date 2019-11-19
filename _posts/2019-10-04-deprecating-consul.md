---
layout: post
title: Etcd替代Consul计划
date:  2019-10-04 09:00:00
translator: 舒先
translator_url: https://github.com/printfcoder
translator_bio: maintainer of micro & overseeing micro China, senior engineer@oppo
---

过去4年Micro使用Consul作为默认的（推荐）注册中心，事实上，在一段时间内，Consul在Micro中是唯一依赖的外部基建服务。

随着云原生技术的出现与演进，我们发现在其之上推荐使用Consul时，或多或少都会出现问题。并不是说Consul不好，只是说从使用的反映上看，我们得迁移到不同的方案上去。

我们使用Consul的Tag属性来保存编码、压缩后的元数据、服务接口等信息，很遗憾，基于Consul的API，并没有提供其它机制让我们来保存这些信息，诚然，使用标签看上去是非常不合理的，有点滥用的意思，并且也造成了Raft一致性的问题。比如**注册/重注册**到集群上的其他节点时，一致性的问题非常明显。

Consul的`tags/metadata`API使用起来并不那么方便，而我们需要的是一个聚焦分布式服务注册的套件。

Etcd经过K8S的洗礼，是个非常不错的选择。它有标准的**Get/Put/Delete**存储接口来保存我们的二进制元数据，对于数据的格式，Etcd并没有限制。

从2014年以来，K8S成为容器编排的基建平台，同时，它选择使用Etcd作为其K-V存储，Etcd作为Raft一致性的K-V，经过几年的演进，通过大家的真实场景使用与测试，大家把大坑都踩完了，V2版本已经在开发。

现在对Go-Micro来讲，也是时候把默认的服务注册发现中心从Consul迁移到其它服务上了。

过去的一个星期，我们把Etcd转正成默认的注册机制，同时接下来的几周内把Consul移到go-plugins中，以后需要借助社区的力量维护Consul相关的组件。而Micro团队会聚焦在Etcd上。

我们清楚的知道，很多用户使用Go-Micro都是基于Consul注册的，所以这个迁移计划自然会破坏现有的系统，鉴于此，我们会在Etcd版本加上V2标签，这样大家就可以在基于V1版本使用Consul。

<center>...</center>
<br>

想了解更多，请访问我们的[官方站点](https://micro.mu/blog/cn)，订阅我们的[twitter](https://twitter.com/microhq)、微信公众号MicroHQ、[微博](https://weibo.com/microhq)。
也可以加入[slack](https://micro-services.slack.com)，选择加入中国区Channel

<h6><a href="https://github.com/micro/micro"><i class="fa fa-github fa-2x"></i> Micro</a></h6>


