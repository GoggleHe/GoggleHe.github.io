---
layout: post
title: Netty入门
categories: [网络编程]
description: Netty入门
keywords: IO, Netty
---

# Netty

## JDK和Netty字节缓冲的比较

### JDK ByteBuffer

- **长度固定**：一旦分配完成，容量不能动态扩展和收缩
- 只有一个**position**指针记录读写位置
- 写`byteBuffer`的内容，是按**position**指针当前位置顺序写
- 读取`byteBuffer`的内容，是读写**position**到**capacity**之间的数据
  - 所以读取时必须调用`flip()`令**position**为0，才能读到正确数据

### Netty ByteBuf

- `ByteBuf`则是**通过两个位置指针来协助缓冲区的读写操作**，读操作使用`readerIndex`、写操作使用`writerIndex`
- **读操作**的时候：**readIndex++，读取的内容为，readIndex - writeIndex，但是不会超过writeIndex。** 之后0~readIndex的数据视为`discard`无用，调用`discardReadBytes`方法可以释放掉。
- **写操作**的时候：writeIndex++，write ~ capacity之间的数据都是可以写的

## Channel

- 客户端与服务端之间连接的抽象
  - 显示当前通道状态，比如开启状态，连接状态等。
  - 提供通道配置参数，例如设定接收缓冲区的大小。
  - 支持多种IO操作，如读操作、写操作、连接和绑定等。同时，它也可以处理与通道相关联的所有IO事件和请求。

在Netty中，`AbstractChannel` 是整个家族的基类，它的子类包括` AbstractNioChannel`、`AbstractOioChannel`、`AbstractEpollChannel` 等，每一种都代表了不同的 I/O 模型和协议类型。常用的 Channel 实现类有：

- NioServerSocketChannel 异步 TCP 服务端。
- NioSocketChannel 异步 TCP 客户端。
- OioServerSocketChannel 同步 TCP 服务端。
- OioSocketChannel 同步 TCP 客户端。
- NioDatagramChannel 异步 UDP 连接。
- OioDatagramChannel 同步 UDP 连接。

### attr

Netty中的 Channel.attr() 方法让我们可以在 Channel 上绑定和使用自定义属性。这些属性会在整个 Channel 的生命周期中保持，也就是说，可以在任何时候，任何地方，通过同样的 Channel.attr()方法访问获取到这些属性。

实际上，Attr 是一个接口，Channel 通过类似于 Map 的机制来维护它们。使用 AttributeKey 非常简单，首先我们需要创建一个 AttributeKey 对象，例如：

```java
private static final AttributeKey<Integer> LOGIN_COUNT = AttributeKey.valueOf("loginCount");
```

上面的代码创建了一个 AttributeKey 对象，它的键名是 "loginCount"，值的类型是 Integer。我们可以在 Channel 上使用 AttributeKey 存储或获取 Integer 类型的属性，例如：

```java
Channel channel = ...; // 获取一个 Channel 对象
channel.attr(LOGIN_COUNT).set(1); // 存储一个 Integer 类型的属性
Integer loginCount = channel.attr(LOGIN_COUNT).get(); // 获取 Integer 类型的属性
```

### 获取channel的状态

```java
boolean isOpen(); //如果通道打开，则返回true
boolean isRegistered();//如果通道注册到EventLoop，则返回true
boolean isActive();//如果通道处于活动状态并且已连接，则返回true
boolean isWritable();//当且仅当I/O线程将立即执行请求的写入操作时，返回true。
```

### writeAndFlush流程解析

- write()：将数据写到`ChannelOutboundBuffer`的缓存中
- flush()：真正将数据写到socket缓冲区

#### 比较两个类的`writeAndFlush()`

- `ChannelHandlerContext `:是从 pipeline 链中的当前节点开始往前找到第一个 outBound 类型的 handler 把对象往前进行传播，如果这个对象确认不需要经过其他 outBound 类型的 handler 处理，就使用这个方法
- `Channel `是从 pipeline 链中的最后一个 outBound 类型的 handler 开始，把对象往前进行传播，如果你确认当前创建的对象需要经过后面的 outBound 类型的 handler，那么就调用此方法

## ChannelGroup

- Netty提供的维护多个Channel的类，提供一些对于Channel的批量操作
- 支持添加、移除、遍历、等操作
- 支持向所有维护的Channel发送信息，信息由于网络原因会乱序

## EventLoop

### 继承实现关系

![netty-start-uml-EventLoop](/images/posts/netty-start-uml-EventLoop.png)

### 特点

- 单线程执行：EventLoop本质是一个只会由一个线程执行的线程池
- 多路复用：即一条线程可以监听多个 Channel，EventLoop 使用底层操作系统提供的 I/O 模型（例如 Java NIO）实现多路复用，从而实现并发处理多个连接的能力。
- 事件驱动：EventLoop 通过监听与 Channel 相关的 I/O 事件（例如读、写、连接等），并将其转化为事件对象（例如 `ChannelReadEvent`、`ChannelWriteEvent` 等）进行处理。这种事件驱动模型可以让 Netty 高效地响应 I/O 事件，避免了轮询等不必要的操作。
- 非阻塞操作：EventLoop 中所有的操作都是非阻塞的，包括 I/O 操作和异步任务的执行。这样可以确保整个系统始终处于高效运行状态，并提高了系统的可伸缩性。

## EventLoopGroup

本质是个线程池，负责管理一个或多个EventLoop，每个EventLoop负责处理其分配的所有Channel的生命周期中发生的事件

服务端使用时会创建Boss Group和Worker Group，Boss负责监听客户端连接，Worker负责已连接客户端的读写操作

Boss Group监听到新连接请求时，会将其注册到某个EventLoop的Selector上，并将其关联到对于的Channel对象。之后Worker Group将负责处理该已建立连接的全部读写操作

EventLoopGroup支持优雅关闭，调用`shutdownGracefully()`，将不在接收新任务，旧任务全部完成后安全关闭

## ChannelReadEvent等