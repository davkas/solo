title: Netty综述
date: 2015-08-19 20:12:23
tag: hadoop
update: 2015-08-21 20:12:23
comments: true
categories: hadoop
---


#### 1. Netty架构
------
##### 传输服务（Transport Service）
- Socket & Datagram
- HTTP Tunnel
- In-VM Pipe
##### 协议支持（rotocol Support）
------
- HTTP&WebSocket
- SSL StartTLS
- Google Protobuf
- zlib/gzip Compression
- Large File Transfer
- RTSP
- Legacy Text、Binary Protocols with Unit Testability
##### 核心功能（Core）
------
- Extensible Event Model
- Universal Communication API
- Zero-Copy-Capable Rich Byte Buffer

<!--more-->

#### 2. Netty服务器端流程
------
##### 一个Netty服务器程序的构建主要由两部分组成：
- 配置服务器的功能，如线程，端口等
- 实现服务器处理程序，它包含业务逻辑，决定当有一个请求连接或接受数据时的处理逻辑
#####使用Netty编写服务器程序的实现的过程如下：
- 创建ServerBootstrap实例引导绑定和启动服务器
- 创建NioEventLoopGroup对象来处理事件，如接受新连接、接受数据、写数据等。
- 制定InetSocketAddress，服务器监听此端口
- 设置childHandler执行所有的连接请求
- 调用ServerBootStrap.bind()方法绑定服务器
- 编写Handler的实例类完成业务逻辑的处理


#### 3. Netty客户端流程
------
##### 一个Netty客户端服程序的实现过程如下：
- 创建Bootstrap对象用来引导启动客户端
- 创建EventLoopGroup对象并设置到Bootstrap中，EventLoopGroup可以理解为是一个线程池，这个线程池用来处理连接、接受数据、发送数据
- 创建InetSocketAddress并设置到Bootstrap中，InetSocketAddress是指定连接的服务器地址
- 添加一个ChannelHandler，客户端成功连接服务器后就会被执行
- 调用Bootstrap.connect()来连接服务器
- 关闭EventLoopGroup来释放资源

#### 4. Netty中关于“著名的epoll缺陷”
---
Java Nio是根据操作系统不同，针对Nio的Selector有不同的实现，在Linux下面使用EPollSelectorProvider(2.6以上)或PollSelecotrProvider
>epoll是Linux内核为处理大批量文件描述符而作了改进的poll，是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率。另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。epoll除了提供select/poll那种IO事件的水平触发（Level Triggered）外，还提供了边缘触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少epoll_wait/epoll_pwait的调用，提高应用程序效率。
>

在Java Nio中，当客户端断开连接时，系统报告一个查询失败的错误，但是用户还能够正常的获得请求结果。
在使用epoll时，客户端使用close()断开连接时，在服务器端会触发一个epoll事件。在低于 2.6.17 版本的内核中，这个 epoll 事件一般是 EPOLLIN，即 0x1，代表连接可读。
连接池检测到某个连接发生 EPOLLIN 事件且没有错误后，会认为有请求到来，将连接交给上层进行处理。这样一来，上层尝试在对端已经 close() 的连接上读取请求，只能读到 EOF，会认为发生异常，报告一个错误。

因此在使用 2.6.17 之前版本内核的系统中，无法依赖封装 epoll 的底层连接库来实现对对端关闭连接事件的检测，只能通过上层读取数据时进行区分处理。

不过，2.6.17 版本内核中增加了 EPOLLRDHUP 事件，代表对端断开连接，关于添加这个事件的理由可以参见 “[Patch][RFC] epoll and half closed TCP connections”。

在使用 2.6.17 之后版本内核的服务器系统中，对端连接断开触发的 epoll 事件会包含 EPOLLIN | EPOLLRDHUP，即 0x2001。有了这个事件，对端断开连接的异常就可以在底层进行处理了，不用再移交到上层。

重现这个现象的方法很简单，首先 telnet 到 server，然后什么都不做直接退出，查看在不同系统中触发的事件码。

注意，在使用 2.6.17 之前版本内核的系统中，sys/epoll.h 的 EPOLL_EVENTS 枚举类型中是没有 EPOLLRDHUP 事件的，所以带 EPOLLRDHUP 的程序无法编译通过。

自4.0.16起, Netty为Linux通过JNI的方式提供了native socket transport.
使用native socket transport很简单，只需将相应的类替换即可

------
#### Netty的解决策略：
1) 根据该BUG的特征，首先侦测该BUG是否发生；
2) 将问题Selector上注册的Channel转移到新建的Selector上；
3) 老的问题Selector关闭，使用新建的Selector替换。

### 参考文献
[Netty系列之Netty可靠性分析]( http://www.infoq.com/cn/articles/netty-reliability)
