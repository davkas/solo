title: Netty核心概念
date: 2015-08-19 20:12:23
tag: Netty
update: 2015-08-19 20:12:23
comments: true
categories: Netty
---
## Netty核心类图
Netty的核心类图如下

![alt text](/images/Netty_Class_Graph.jpg)

Netty中核心的概念有：

- BootStrap
- ChannelFactory
- ChannelPipelineFactory
- ChannelPipeline
- ChannelHandler
- ChannelEvent
- ChannelSink

<!-- more -->

## Netty实现过程

> 一个Netty程序开始于Bootstrap类，Bootstrap类是Netty提供的一个可以通过简单配置来设置或"引导"程序的一个很重要的类。Netty中设计了Handlers来处理特定的"event"和设置Netty中的事件，从而来处理多个协议和数据。事件可以描述成一个非常通用的方法，因为你可以自定义一个handler,用来将Object转成byte[]或将byte[]转成Object；也可以定义个handler处理抛出的异常。
        你会经常编写一个实现ChannelInboundHandler的类，ChannelInboundHandler是用来接收消息，当有消息过来时，你可以决定如何处理。当程序需要返回消息时可以在ChannelInboundHandler里write/flush数据。可以认为应用程序的业务逻辑都是在ChannelInboundHandler中来处理的，业务罗的生命周期在ChannelInboundHandler中。
        Netty连接客户端端或绑定服务器需要知道如何发送或接收消息，这是通过不同类型的handlers来做的，多个Handlers是怎么配置的？Netty提供了ChannelInitializer类用来配置Handlers。ChannelInitializer是通过ChannelPipeline来添加ChannelHandler的，如发送和接收消息，这些Handlers将确定发的是什么消息。ChannelInitializer自身也是一个ChannelHandler，在添加完其他的handlers之后会自动从ChannelPipeline中删除自己。
        所有的Netty程序都是基于ChannelPipeline。ChannelPipeline和EventLoop和EventLoopGroup密切相关，因为它们三个都和事件处理相关，所以这就是为什么它们处理IO的工作由EventLoop管理的原因。
        Netty中所有的IO操作都是异步执行的，例如你连接一个主机默认是异步完成的；写入/发送消息也是同样是异步。也就是说操作不会直接执行，而是会等一会执行，因为你不知道返回的操作结果是成功还是失败，但是需要有检查是否成功的方法或者是注册监听来通知；Netty使用Futures和ChannelFutures来达到这种目的。Future注册一个监听，当操作成功或失败时会通知。ChannelFuture封装的是一个操作的相关信息，操作被执行时会立刻返回ChannelFuture。

## BootStrap
Netty程序起始于BootStrap类,BootStrap 类是Netty提供的通过配置设置和引导程序的类。Netty通过扩展Handlers处理特定的事件，以此处理协议和数据。
## Channels
在Netty里，Channel是通讯的载体，而ChannelHandler负责Channel中的逻辑处理。

ChannelPipeline可以理解为ChannelHandler的容器：一个Channel包含一个ChannelPipeline，所有ChannelHandler都会注册到ChannelPipeline中，并按顺序组织起来。

一个Channel关联一个单一的EventLoop,一个EventLoopGroup中包含多个EventLoop。在Netty中一个EventLoop可能会绑定到多个channle，因而在使用过程中应该尽量不使用同步机制。使用同步机制可能会使得一个EventLoop阻塞，从而造成其他的Channle中的数据处理造成阻塞。

在Netty中，ChannelEvent是数据或者状态的载体，例如传输的数据对应MessageEvent，状态的改变对应ChannelStateEvent。当对Channel进行操作时，会产生一个ChannelEvent，并发送到ChannelPipeline。ChannelPipeline会选择一个ChannelHandler进行处理。这个ChannelHandler处理之后，可能会产生新的ChannelEvent，并流转到下一个ChannelHandler。

## EventLoop
EventLoop继承自EventExecutor，EventLoopGroup。从这点可以看出EventLoop首先是一个Executor，任务执行器。 


## 参考文献
[Netty源码解读（三）Channel与Pipeline](http://ifeve.com/channel-pipeline/)
