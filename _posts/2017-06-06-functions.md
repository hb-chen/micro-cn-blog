---
layout: post
title: Micro中函数式编程(Function)
date:   2017-06-06 09:00:00	date:  2019-11-19 23:26:00
profile: crazybber
author: Edward
author_url: https://github.com/crazybber
author_bio: maintainer of micro & micro china
---
<br>
随着技术的发展，我们的编程模型也在发展，我们已经从单体式服务转向微服务
并且，最近开始将这种分离，更进一步推向函数式编程。

Micro希望通过[go-micro](https://github.com/micro/go-micro)为微服务提供可插拔的框架来简化分布式系统的开发，并且Go-micro历来包括高级[Service](https://godoc.org/github.com/micro/go-micro#Service)
接口，封装了对微服务更偏底层要求。

现在，我们将介绍在Go-Micro中执行一次Service 的函数功能[Function](https://godoc.org/github.com/micro/go-micro#Function)实现。

<script src="https://gist.github.com/asim/bfbaf036c90761879dbf6e939e5172e4.js"></script>

### 灵感

Ben Firshman去年开源了一个名为[Funker](https://github.com/bfirsh/funker)的项目:函数(Function)即容器，这个概念非常简单但也非常聪明。

函数(Function)可以很简单地成为一种方法的程序，可以在网络上侦听请求并在执行一次后退出，可以利用kubernetes、docker swarm服务进行生命周期管理，发现等。

这激发了我们将函数(Function)纳入go-micro的灵感。

### 为什么需要函数式编程

函数编程模型是微服务的进一步发展，随着我们需求规模的增加，无论是在技术上还是组织上，我们都需要使系统和团队解耦，以便他们可以分别独立运行。

在过去的五年中，我们已经看到微服务作为满足这些扩展需求的一种方式而出现。微服务架构模式当然并不是什么新鲜事物，但是我们现在已经开始定义最佳实践，以帮助我们构建更好的软件。

在简化分布式系统开发和解决软件问题方面，函数编程使我们进入了一个新的可能性领域。
回到unix的哲学理念:“做一件事情，做事情做好”，相比于微服务，函数式编程，更能体现了这一哲学理念。

虽然基础架构可以帮助我们构建可扩展的系统，但需要知道，微服务和函数式编程是软件体系结构模式和编程模型，因此，我们需要能够帮助我们使用这些模式编写软件的工具。

### 函数式编程示例

这是使用go-micro编写函数的简单示例。

如您所知，它看起来几乎与服务定义相同。那是因为除了一个小细节，它们的底下是完全相同的，函数在执行处理程序或订阅程序后退出。

功能为您提供与服务相同的功能，从而使您可以利用所有现有的微生态系统工具。

<script src="https://gist.github.com/asim/7d70cf1160ad1279597f12985fe3fbd5.js"></script>

### 运行函数

如前所述，micro中的功能是一次执行服务，该功能将在完成请求后退出。然后这个
提出了一个问题，我们如何保持功能运行？

开源社区中有大量用于流程生命周期管理的现有工具，因此可以随意使用任何您喜欢的工具流程管理工具。

但是，微型工具包现在包含一个名为[** micro run **](https://micro.mu/docs/run.html)的便捷工具。

这是运行函数的方法：

```bash
micro run -r github.com/micro/examples/function
```

** micro run ** 命令可从源获取，构建和执行。 -r标志告诉它在退出时重新启动功能。
目前，它是用于运行基于微服务的Service和Function的简单实验工具。从源代码到在一个命令中运行。

等该功能更加稳定后，我们将为run命令，发布个专题文章。

### 总结

函数式编程是微服务的自然扩展，是帮助简化分布式系统开发的下一个编程模型，Micro将函数(Function)视为头等公民。

虽然函数已添加到go-micro中，但这并不意味着您必须使用它们编写100％的软件，重要的是要了解应用场景，搞清楚什么时候应该使用，单体服务、微服务或函数。
希望在不久的将来，我们能看到更多有关微服务下的函数式编程与现有系统和Serverless工具集成的信息。

<center> <p> ... </p> </center>

如果您想了解有关我们提供的服务或微服务的更多信息，请访问[网站](https://micro.mu/blog/cn)或
请访问[GitHub](https://github.com/micro/micro-in-cn)。

在[Twitter](https://twitter.com/microhq)上关注我们，或加入[Slack](http://slack.micro.mu)社区的中文频道。
