---
layout:	post
title:	Micro 1.0.0 release and beyond
date:	2019-04-01 09:00:00
---
<br>
在过去的四年里，我们花了大量精力简化微服务的开发。为了达到这个目标，我们推出了[**Go Micro**](https://github.com/micro/go-micro)框架及基于**Go Micro**的服务治理工具集[**Micro**](https://github.com/micro/micro)。

<img src="https://micro.mu/docs/images/go-micro.svg" style="max-width: 100%; margin: 0;" />

## 版本 1.0.0

上个月我们发布了**Go Micro**与**Micro****version 1.0.0**。在Micro发展历程中，这是非常重要的里程碑。从2015年开始，Micro就在很多公司中真实项目上运行，并且变得越来越重要，比如Micro的捐献商德国在线租车平台Sixt，它们产线上有几百个微服务。

开发团队可以通过Micro将复杂的分布式与云端应用的抽象化，把它们构建成弹性的微服务。Micro插件化、对运行环境零依赖。

<center>
<img src="https://micro.mu/micro-diag.svg" style="max-width: 100%; margin: 0;" />
</center>

<br>
We've considered Micro production ready for a long time but the release of 1.0.0 solidifies the maturity and stability of our tooling. And 
we believe it's the right time for everyone to adopt Micro as the defacto standard for microservice development.

## 使用情况

Micro在合理成长has largely grown organically. We've not yet actively engaged in speaking at conferences, meetups or any other form of outreach. Instead 
we focused on solving a real problem and it's shown in the numbers.

<center>
<img src="{{ site.baseurl }}/assets/images/stars.png" style="max-width: 75%; margin: 0;" />
</center>
<br>
跟踪大家的使用情况目前比较困难，但是我们通过在论坛中看到了大家的共呜与反馈，我们很清楚大家希望在Golang体系有一个更简单构建微服务的方式。

## 1.0之后的规则

我们宣告1.0.0版本发布并不只是说发布稳定版、或者可以在产线中运行，也更意味着当前版本不会受未来变动API影响。This now also allows us to take stock of all the learnings of Micro's usage of the past 4 
years, how technology has evolved in the industry and what version 2 might start to look like.

Micro刚启动的时候，k8s也只是才进入孵化期，而gRPC才发布不久。现在新的技术或者理念比如service-mesh等等又冒了出来。

因为Micro是插件化的，更简化的顶层抽象，使得其比较容易面对开发者技术选型。2.0版本Micro会提供更多无缝集成和丝滑的体验。

Some of these ideas will revolve around using gRPC by default, allowing a drop-in experience on kubernetes and potentially a runtime 
for those who don't want to deal with the complexity of cloud-native systems or any dependency management at all.

We'll also be thinking about how to move beyond Go to support multiple languages.

## Collaboration

Slack has served us well for realtime collaboration but we need a medium aligned with open source to push this much further, to be far more inclusive 
and to provide a historic record for newcomers to explore easily.

We're going to work with the community by using GitHub to create an open source location to share ideas, discussion and the roadmap for the 
[**Development**](https://github.com/micro/development) of features for 2.0 and beyond.

To all those interested in contributing and collaboration, create an issue for feature requests, a pull request to share design ideas and we'll work 
together to shape the roadmap.

## 鸣谢

我想在最后向Micro社区还有使用Micro、支持Micro的人们说声感谢，谢谢大家这4年来的支持。这段时间走得很艰难，不过好在困难都是值得的，而且我们还有很多事要做。没有社区就没有现在的Micro。 We're 1.6k+ members 
strong in Slack with thousands more across other forums.

<center>
<blockquote class="twitter-tweet" data-cards="hidden" data-lang="en"><p lang="en" dir="ltr">Today I released v1 of micro and go-micro. 4 years of hard work. Thanks to all that supported me along the way. <a href="https://t.co/blI1pJ3hBl">https://t.co/blI1pJ3hBl</a></p>&mdash; Asim Aslam (@chuhnk) <a href="https://twitter.com/chuhnk/status/1102992210088378369?ref_src=twsrc%5Etfw">March 5, 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</center>
<br>
Thank you for all your support and contributions. We hope we can return the favour by providing everyone the most inclusive and collaborative 
place for all things microservices.

<center>...</center>
<br>
Micro始终坚持极简构建微服务的理念。如果你也在想微服务开发，那就上车吧。

访问我们的网站了解更多，[micro.mu](https://micro.mu)

关注我们：[twitter](https://twitter.com/microhq)，[微博](https://weibo.com/microhq)

讨论：[slack](https://micro-services.slack.com)

线下：[meetup](https://www.meetup.com/Micro-Services-Network/)

<h6><a href="https://github.com/micro/micro"><i class="fa fa-github fa-2x"></i> Micro</a></h6>
