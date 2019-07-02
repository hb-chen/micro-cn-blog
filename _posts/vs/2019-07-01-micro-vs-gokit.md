---
layout: post
title: Micro与Go-Kit对比
date:  2019-7-01 09:00:00
profile: sx
author: 舒先
author_url: https://github.com/printfcoder
author_bio: Senior Engineer@huize
---

我们想做一些有意思的对比，看到Go-Kit将近我们两倍的Star，心里是不服的，是时候来一篇，比比谁更厉害了！

## 总体概览

首先，我们进行功能上的对比

|功能|Micro|Go-Kit|
|---|:---:|:---:|
|专门API Gateway|[有](https://micro.mu/docs/api.html)|[需手动实现](https://github.com/go-kit/kit/tree/master/examples/apigateway)|
|负载均衡|支持|支持|

## 分层设计

二者都不是常见的MVC结构。

### Go-kit

Go-Kit虽然也由三层设计，但是它们并非MVC。

1. Transport layer
2. Endpoint layer
3. Service layer

请求从第1层逐层往下到第三层，响应则是相反的路径

### Micro

Micro则相对复杂一些，大致见下图：

<p><a href="https://micro.mu/blog/cn/assets/images/go-micro.svg">
  <img src="https://micro.mu/blog/cn/assets/images/go-micro.svg" style="width: 100%; height: auto; margin: 0;" />
</a></p>

#### Go-Micro

#### Micro

## 总结

简而总之，Go-Kit从诞生到现在，更多是为了集成到应用平台之中，而Micro则是一个平台，二者适应的场景不同。


## 参考资料

[go-kit faq](https://gokit.io/faq/)