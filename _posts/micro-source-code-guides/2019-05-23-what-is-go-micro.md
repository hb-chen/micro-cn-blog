---
layout: post
title: 什么是Go-Micro
date:  2019-05-23 09:00:00
profile: sx
author: 舒先
author_url: https://github.com/printfcoder
author_bio: Senior Engineer@huize
---

该系统博客我们会专门讲解**Go-Micro**的源码。我们**Go-Micro**的架构图开始。

<a href="{{ site.baseurl }}/assets/images/cli.png">

如上图所示，**Go-Micro**由三层设计共5大模块组成。

最上层的**service**是基于**Go-Micro**所构建的服务，属于应用层，它属于**Go-Micro**末端。

中层的**Client**与**Server**是第一层中**service**所包含的**请求端**与**响应端**，它们存在于**service**中，处于设计中的**中游**，是**Go-Micro**体系中一切请求与响应的出入口。

最下层的便是**Go-Micro**核心所在，broker负责消息，Codec负责编码，Registry负责注册发现，Selector负责负载均衡，Transport负责接收请求与响应。

**Go-Micro**本质是一个网络库（networking lib）。

