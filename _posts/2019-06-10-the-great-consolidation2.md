---
layout: post
title: 2019年Micro的整合工作
date:  2019-06-01 09:00:00
translator: 舒先
translator_url: https://github.com/printfcoder
translator_bio: maintainer of micro & overseeing micro China, senior engineer@huize
---

<br>
Micro生态以[**go-micro**](https://github.com/micro/go-micro)微服务开发框架为核心，已经开启她的微服务演近进程，Micro一直专注为微服务核心开发提供方案，通过Micro的抽象结构，大家在开发过程中不需要再关心微服务系统架构的复杂性。

经过几年的时间演进，我们已经把Go-Micro扩展超过了原先的定位，发展出了其它工具、库、插件。这一过程在**虽然解决了问题**与**期望开发者如何（简便地）使用Micro工具链**之间产生了矛盾。所以我们现在正准备把这些工具库整合起来，给大家提供更好的使用体验。

Micro的基本定位是成为微服务开发中的独立开发框架与运行时管理工具。

在讨论整合工作之前，我们回顾一下截止目前的情况。

## 核心焦点

Go-micro started out as primarily being focused on the communication aspects of microservices and we've tried to remain true to that. 
This opinionated approach and focus is what's really driven the success of the framework thus far. Over the years 
we've had numerous requests to solve the next day problems for building production ready software in go-micro itself. 
Much of this has been related to scalability, security, synchronisation and configuration.

<center>
<img src="https://micro.mu/docs/images/go-micro.svg" style="width: 80%; height: auto;"/>
</center>

While there would have been merit to add the additional features requested we really wanted to stay very focused 
on solving one problem really well at first. So we took a different approach to promoting the community to do this.

## 生态系统与插件

Going to production involves much more than vanilla service discovery, message encoding and request-response. 
We really understood this but wanted to enable users to choose the wider platform requirements via pluggable and 
extensible interfaces. Promoting an ecosystem through an [explorer](https://micro.mu/explore/) which aggregates 
micro based open source projects on GitHub along with extensible plugins via the [go-plugins](https://github.com/micro/go-plugins) repo.

<center>
<img src="https://micro.mu/explorer.png" style="width: 80%; height: auto;" />
</center>

Go Plugins has generally been a great success by allowing developers to offload significant complexity to systems built 
for those requirements e.g prometheus for metrics, zipkin for distributed tracing and kafka for durable messaging.

## 关于服务接入点

Go Micro was really at the heart of microservice development but as services were written the next questions moved on to; 
how do I query these, how do I interact with them, how do I serve them by traditional means...

Given go-micro used an rpc/protobuf based protocol that was both pluggable and runtime agnostic, we needed some way to address 
this in a way that was true to go-micro itself. This led to the creation of [**micro**](https://github.com/micro/micro), 
the microservice toolkit. Micro provides an api gateway, web dashboard, cli, slack bot, service proxy and much more.

<center>
<img src="https://micro.mu/runtime3.svg" style="width: 80%; height: auto;"/>
</center>

The micro toolkit acted as interaction points via http api, browser, slack commands and command line interface. These 
are common ways in which we query and build on applications and it was important for us to provide a runtime that 
really enabled this. Yet still it really focused on communication above all else.
 
## Additional Tools

While the plugins and the toolkit helped users of micro significantly, it still lacked in key areas. It was clear that our community 
wanted us to solve further problems around platform tooling for product development rather than having to do it individually at 
their various companies. We needed the same types of abstractions go-micro provided for things like dynamic configuration, 
distributed synchronization and broader solutions for systems like Kubernetes.

For these we created:

- [micro/go-config](https://github.com/micro/go-config) - a dynamic configuration library
- [micro/go-sync](https://github.com/asim/go-sync) - a distributed synchronisation library
- [micro/kubernetes](https://github.com/micro/kubernetes) - micro on kubernetes initialisation
- [examples](https://github.com/micro/examples) - example usage code for micro
- [microhq](https://github.com/microhq) - a place for prebuilt microservices

These were a few of the repos, libraries and tools used to attempt to solve the wider requirements of our community. Over the 
4 years the number of repos grew and the getting started experience for new users became much more difficult. The barrier to 
entry increased dramatically and with that we knew something needed to change.

## Consolidation

In the past few weeks we've realised [**go-micro**](https://github.com/micro/go-micro) was really the focal point for most of our users 
developing microservices. It's become clear they want additional functionality as part of this library and as a self described 
framework we really need to embrace this by solving those next day concerns without asking a developer to go anywhere else. 

Essentially **go-micro** will be the all encompassing and standalone framework for microservice development.

We've started the consolidation process by moving all our libraries into go-micro and we'll continue to refactor over the 
coming weeks to provide a simpler default getting started experience while also adding further features for logging, tracing, metrics, 
authentication, etc.

<center>
<img src="{{ site.baseurl }}/assets/images/go-micro-repo.png" style="width: 80%; height: auto;" />
</center>

We're not forgetting about [**micro**](https://github.com/micro/micro) either though. In our minds after you've built your microservices 
there's still a need for a way to query, run and manage them. **Micro** is by all accounts going to be the runtime for micro 
service development. We're working on providing a simpler way to manage the end to end flow of microservice development and 
should have more to announce soon.

## Summary

Micro is the simplest way to build microservices and slowly becoming the defacto standard for Go based microservice development in 
the cloud. We're making that process even simpler by consolidating our efforts into a single development framework and runtime. 

<center>...</center>
<br>
To learn more check out the [website](https://micro.mu), follow us on [twitter](https://twitter.com/microhq) or 
join the [slack](https://micro-services.slack.com) community.

<h6><a href="https://github.com/micro/micro"><i class="fa fa-github fa-2x"></i> Micro</a></h6>
