---
layout: post
title: Micro源码系列 - Go-Micro服务的构造过程
date:  2019-05-23 09:00:00
profile: sx
author: 舒先
author_url: https://github.com/printfcoder
author_bio: Senior Engineer@huize
---

在每一个技术领域，特别是微服务这一分支，**Go-Micro**尝试以最简单的方式帮助大家以最快的速度构建微服务。我们有比较详细但是仍需要大量持续更新、增进的文档，也有足够多的示例代码帮助大家验证每个功能点是否满足大家的业务需求。同时，我们在总结很多朋友的常询问的问题后，发现大家对**Go-Micro**的源码实现都很有兴趣。

故而，在大家对了解如何使用**Go-Micro**来构建服务后，我们专门用这一系列来从设计到源码系统地讲解**Go-Micro**。

## 总架构

首先，我们从**Go-Micro**的架构图开始，回顾一下：

<a href="{{ site.baseurl }}/assets/images/go-micro.svg">
  <img src="{{ site.baseurl }}/assets/images/go-micro.svg" style="width: 100%; height: auto; margin: 0;" />
</a>

如上图所示，**Go-Micro**由三层设计共5大模块组成。

最上层的**service**是基于**Go-Micro**所构建的服务，属于应用层，它属于**Go-Micro**末端。

中层的**Client**与**Server**是第一层中**service**所包含的**请求端**与**响应端**，它们存在于**service**中，处于设计中的**中游**，是**Go-Micro**体系中一切请求与响应的出入口。

最下层的便是**Go-Micro**核心5模块所在，broker负责消息，Codec负责编码，Registry负责注册发现，Selector负责负载均衡，Transport负责接收请求与响应。

**Go-Micro**<u>本质是一个网络库（networking lib）</u>。

## 三段式

**Go-Micro**中原生服务通常由三段组成：*NewService*，*Init*，*Run*。它们功能分别是创建服务，初始化服务，运行服务。

```go
func main() {
    service := micro.NewService(
        micro.Name("go.micro.srv.example"),
    )

    service.Init()

    if err := service.Run(); err != nil {
        log.Fatal(err)
    }
}
```

上面的代码便可以启动一个微服务。接下来，我们便从经典三段入手，先讲解这三段背后各自隐藏的逻辑。

### NewService

**NewService**负责创建新服务。它的参数是[**micro.Option**](https://github.com/micro/go-micro/blob/master/micro.go#L42)函数数组，此类函数专用来装配启动参数或配置选项。

```go
func NewService(opts ...Option) Service {
    return newService(opts...)
}
```

整个NewService有三个步骤：

- [初始化配置选项](#1.初始化配置选项)
- [包装客户端](#2.包装客户端)
- [组装服务](#3.组装服务)

<a href="{{ site.baseurl }}/assets/images/go-micro-service.png">
  <img src="{{ site.baseurl }}/assets/images/go-micro-service.png" style="width: 100%; height: auto; margin: 0;" />
</a>

参考上图，对2，3步骤可以有更好的理解。服务中有两个对象Client与Server，分别负责发送请求与接收请求。

构建Service服务核心代码：

[service.go](https://github.com/micro/go-micro/blob/master/service.go#L21-L34)

```go
func newService(opts ...Option) Service {
    
    // 初始化配置选项
    options := newOptions(opts...)

    // 包装客户端
    options.Client = &clientWrapper{
        options.Client,
        metadata.Metadata{
            HeaderPrefix + "From-Service": options.Server.Options().Name,
        },
    }

    // 组装服务
    return &service{
        opts: options,
    }
}
```

#### 1.初始化配置选项

配置选项通过Option函数来传递

[**Option**](https://github.com/micro/go-micro/blob/master/micro.go#L42)

```go
type Option func(*Options)
```

我们就用**micro.Name("go.micro.srv.example")**来说明

[options.go](https://github.com/micro/go-micro/blob/master/options.go#L133-L137)

```go
// Name of the service
func Name(n string) Option {
    return func(o *Options) {
        o.Server.Init(server.Name(n))
    }
}
```

上述代码中接收服务名*n*后即返回一个回参为Option的匿名函数。函数中的**o.Server.Init(server.Name(n))**，我们这里不讲，大家只需要知道是服务的配置初始化逻辑，我们把它放到后面的专门讲解Server（MockServer，RpcServer，HttpServer）的章节里讲解。

更多选项配置函数参考：[options.go](https://github.com/micro/go-micro/blob/master/options.go)

在上面的**newService**代码中，第一部分就是初始化传入的配置：

```go
func newService(opts ...Option) Service {
    
    // 初始化配置选项
    options := newOptions(opts...)

    // ...
}
```

我们再来看看[**newOptions**](https://github.com/micro/go-micro/blob/master/options.go#L36-L52)方法

```go
func newOptions(opts ...Option) Options {
    opt := Options{
        Broker:    broker.DefaultBroker,
        Cmd:       cmd.DefaultCmd,
        Client:    client.DefaultClient,
        Server:    server.DefaultServer,
        Registry:  registry.DefaultRegistry,
        Transport: transport.DefaultTransport,
        Context:   context.Background(),
    }

    for _, o := range opts {
        o(&opt)
    }

    return opt
}
```

可见，初始化配置中，预置有基础组件默认配置。再调用**opts**中传入的参数，逐个执行定制的配置函数。

Option函数的参数是Options，里面维护服务配置信息，比如注册、命令行信息等。

每个Option函数执行完后即返回配置对象Options。

#### 2.包装客户端

包装客户端在配置初始化之后，客户端由包装器[clientWrapper](https://github.com/micro/go-micro/blob/master/wrapper.go#L10-L13)封装。

```go
func newService(opts ...Option) Service {
    
    options.Client = &clientWrapper{
        options.Client,
        metadata.Metadata{
            HeaderPrefix + "From-Service": options.Server.Options().Name,
        },
    }
}
```

包装器继承了客户端，并自带由元数据组成的头：

[wrapper.go](https://github.com/micro/go-micro/blob/master/wrapper.go#L10-L13)

```go
type clientWrapper struct {
    client.Client
    headers metadata.Metadata
}
```

go-micro中的Client都有三个能力：

- 调用RPC Call
- 流式请求 Stream
- 广播消息 Publish

```go
func (c *clientWrapper) Call(ctx context.Context, req client.Request, rsp interface{}, opts ...client.CallOption) error {
    ctx = c.setHeaders(ctx)
    return c.Client.Call(ctx, req, rsp, opts...)
}

func (c *clientWrapper) Stream(ctx context.Context, req client.Request, opts ...client.CallOption) (client.Stream, error) {
    ctx = c.setHeaders(ctx)
    return c.Client.Stream(ctx, req, opts...)
}

func (c *clientWrapper) Publish(ctx context.Context, p client.Message, opts ...client.PublishOption) error {
    ctx = c.setHeaders(ctx)
    return c.Client.Publish(ctx, p, opts...)
}
```

其实Service中Client并没有过多改变原生Client行为，只是在其发送请求或广播前把元数据放到上下文中传向下游服务传递。见setHeaders：

[**setHeaders**](https://github.com/micro/go-micro/blob/master/wrapper.go#L15-L28)

```go
func (c *clientWrapper) setHeaders(ctx context.Context) context.Context {
   
    // 复制元数据信息
    mda, _ := metadata.FromContext(ctx)
    md := metadata.Copy(mda)

    // 预置头信息塞到元数据
    for k, v := range c.headers {
        if _, ok := md[k]; !ok {
            md[k] = v
        }
    }

    // 元数据塞到上下文中
    return metadata.NewContext(ctx, md)
}
```

#### 3.组装服务

**Go-Micro**中的server非常简单

```go
type service struct {
    opts Options
    once sync.Once
}
```

服务结构中只有配置项与单次锁。

以上就是service的构造过程。其最核心的地方便是[初始化配置选项](#1.初始化配置选项)、[包装客户端](#2.包装客户端)。本章的主要内容是介绍服务构建流程，所以不会深入讲解配置项，后面的章节会讲。

### Init

初始化函数在构造服务之后，由服务调用，它的工作主要是再次渲染传入Init的Option配置项，然后再初始化命令行参数。

```go
func (s *service) Init(opts ...Option) {
    // process options
    for _, o := range opts {
        o(&s.opts)
    }

    s.once.Do(func() {
        // 初始化命令行参数，会覆盖的参数
        _ = s.opts.Cmd.Init(
            cmd.Broker(&s.opts.Broker),
            cmd.Registry(&s.opts.Registry),
            cmd.Transport(&s.opts.Transport),
            cmd.Client(&s.opts.Client),
            cmd.Server(&s.opts.Server),
        )
    })
}
```

由于在**NewService**中加载过一次传入的配置项，而Init中如果有配置项传入，会改变之此前的值，故而，在**Go-Micro**体系中，配置项的生效顺序由小到大依次是：**默认值 < 硬编码值（随构造函数传入）< 命令行参数**。

我们再用**micro.Name("go.micro.srv.example")**来说明，假设我们通过下列4种方式指定服务名：

- 默认值

```go
service := micro.NewService()
```

- 随参传入

```go
service := micro.NewService(
        micro.Name("go.micro.api.new"), 
        )
```

- 随参传入 Init

```go
service.Init(micro.Name("go.micro.api.init"),)
```

- 命令行参数

```bash
go run main.go --server_name=go.micro.api.cmd
```

它们各自打印的服务名分别是：

- server
- go.micro.api.new
- go.micro.api.init
- go.micro.api.cmd

**Init**函数的本质是<u>再次加载配置，并确保权重最高的命令行参数生效</u>。

### Run

在服务构造与初始化完成后，便可以执行**Run**函数，它负责新增协程把服务启动（Start），然后通过select指令阻塞并侦听相关的关闭信号，随后执行关闭程序。

[**Run**](https://github.com/micro/go-micro/blob/master/service.go#L115-L131)

```go
func (s *service) Run() error {
    
    // 启动服务
    if err := s.Start(); err != nil {
        return err
    }

    ch := make(chan os.Signal, 1)
    signal.Notify(ch, syscall.SIGTERM, syscall.SIGINT, syscall.SIGQUIT)

    select {
    // 侦听kill信号
    case <-ch:
    // 侦听上下文关闭信号
    case <-s.opts.Context.Done():
    }

    return s.Stop()
}
```

在上述代码中我们看到，服务启动后由**signal.Notify**方法将三个信号量绑定到信道上：

- syscall.SIGTERM 结束进程
- syscall.SIGINT  ctrl + c
- syscall.SIGQUIT ctrl + \

题外，类Unix标准系统中，只有如下几个信号绑定了键盘

> SIGHUP、SIGINT、SIGTTOU、SIGTTIN、SIGQUIT、SIGTSTP

#### 服务启动

服务的启动由**Run**函数中的启动方法**Start**完成

```go
func (s *service) Run() error {
    
    // 启动服务
    if err := s.Start(); err != nil {
        return err
    }

    // ...
}
```

[**Start**](https://github.com/micro/go-micro/blob/master/service.go#L73-L91)

```go
func (s *service) Start() error {
    
    // 启动勾子
    for _, fn := range s.opts.BeforeStart {
        if err := fn(); err != nil {
            return err
        }
    }

    if err := s.opts.Server.Start(); err != nil {
        return err
    }

    // 启动完成勾子
    for _, fn := range s.opts.AfterStart {
        if err := fn(); err != nil {
            return err
        }
    }

    return nil
}
```

#### 启动勾子

启动函数Start中前后各有启动开始和结束两个勾子选项，可以在NewService或Init中将其传入：

```go
    service.Init(
        micro.BeforeStart(func() error {
            log.Log(1)
            return nil
        }),
        micro.BeforeStart(func() error {
            log.Log(2)
            return nil
        }),
        micro.BeforeStop(func() error {
            return nil
        }),
    )
```

需要多个就传入多个，在渲染配置时会将它们附加到指定启动函数数组中：

```go
func BeforeStart(fn func() error) Option {
    return func(o *Options) {
        // 有多个则会附加进去
        o.BeforeStart = append(o.BeforeStart, fn)
    }
}
```

#### 服务启动

```go
func (s *service) Start() error {
    
    // ...

    if err := s.opts.Server.Start(); err != nil {
        return err
    }


    // ...
}
```

服务启动是通过在配置项的中的Server对象的**Start**方法触发。

整个**Start**分为以下几个步骤

1. 注册调试接口endpoint，Debug.{Method}
2. transport 分配地址
3. broker 分配地址并连接
4. register 注册
5. 侦听连接请求与关闭信号
6. 循环保活

我们来看关键代码（删除了部分逻辑，增加注释）

```go
func (s *rpcServer) Start() error {
    
    // 1. 注册调试接口endpoint，Debug.{Method}
    registerDebugHandler(s)
    config := s.Options()

    // 2. transport 分配地址
    ts, err := config.Transport.Listen(config.Address)
    if err != nil {
        return err
    }

    // 默认端口是0，意味着会随机分配，但是我们需要反填随机端口，以让请求能够到达
    s.Lock()
    addr := s.opts.Address
    s.opts.Address = ts.Addr()
    s.Unlock()

    // 3. broker 分配地址并连接
    if err := config.Broker.Connect(); err != nil {
        return err
    }

  
    // 4. register 注册
    if err := s.Register(); err != nil {
        log.Log("Server register error: ", err)
    }

    exit := make(chan bool)

    // 新协程侦听请求与关闭信号
    go func() {
        for {
            // listen for connections
            err := ts.Accept(s.ServeConn)

            select {
            // check if we're supposed to exit
            case <-exit:
                return
            // check the error and backoff
            default:
                if err != nil {
                    log.Logf("Accept error: %v", err)
                    time.Sleep(time.Second)
                    continue
                }
            }

            // no error just exit
            return
        }
    }()

    // 6. 循环保活
    go func() {
        t := new(time.Ticker)

        // only process if it exists
        if s.opts.RegisterInterval > time.Duration(0) {
            // new ticker
            t = time.NewTicker(s.opts.RegisterInterval)
        }

        // return error chan
        var ch chan error

    Loop:
        for {
            select {
            // 重新注册，保持节点在线
            case <-t.C:
                if err := s.Register(); err != nil {
                    log.Log("Server register error: ", err)
                }
            // 侦听退出信号
            case ch = <-s.exit:
                t.Stop()
                close(exit)
                break Loop
            }
        }

        // 退出后往注册中心卸载节点
        if err := s.Deregister(); err != nil {
            log.Log("Server deregister error: ", err)
        }

        // 等待所有请求都返回
        if wait(s.opts.Context) {
            s.wg.Wait()
        }

        // 关闭侦听
        ch <- ts.Close()

        // 关闭broker
        config.Broker.Disconnect()

        // 归还原始值，不重要，只是为了回归原始
        s.Lock()
        s.opts.Address = addr
        s.Unlock()
    }()

    return nil
}
```

我们在代码中注释了具体过程。但是其中非常重要的环节，比如register注册、broker连接等我们不深入讲解，会放到其实章节中演示，这也是为了让本章主题更向服务构建与启动靠拢。

## 总结

我们在这一章节中讲了**Go-Micro**服务是如何构造与启动的，下一章我们会讲解micro服务的注册是如何实现的。

## Micro源码系列

1. [Go-Micro服务的构造过程](https://micro.mu/blog/cn/2019/05/23/how-does-go-micro-server-be-bulit.html)
2. [Go-Micro注册解读(in progress)]

## Micro 中文资源

1. [中文示例集](https://github.com/micro-in-cn/all-in-one)
2. [中文教程](https://github.com/micro-in-cn/tutorials)
3. [中文博客](https://micro.mu/blog/cn)
4. [Micro服务治理控制台](https://github.com/micro-in-cn/platform-web)