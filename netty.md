# Netty

## 介绍

网络模型

IO复用模型

![img](http://www.52im.net/data/attachment/forum/201809/05/213041mtejdsoeojfjy7dd.jpeg)

NIO

Zero Copy

1. **ByteBufAllocator**.ioBuffer(int) Netty的ByteBuf分配器直接创建非堆内存避免缓冲区的二次拷贝，通过“零拷贝”来提升读写性能。
2. **CompositeByteBuf**  ByteBuf的包装器，它将多个ByteBuf组合成一个集合，然后对外提供统一的ByteBuf接口
3. **FileChannel**.tranferto() 利用操作系统，直接将文件系统缓存传输到目标channel，无需copy

### 架构图

主从Reactor模式

![img](http://www.52im.net/data/attachment/forum/201809/06/200759gg777fr7v7wzcr7r.jpeg)

Netty组从模式

![img](https://upload-images.jianshu.io/upload_images/1500839-55f5b1d5ddc13581.jpg)

### 主要组件

- ServerBootstrap 服务器端引导
- Bootstrap 客户器端引导
- Channel 网络组件，用于处理网络IO操作
- ChannelFuture 网络异步结果组件， 用于获取异步网络组件的结果，通常注册一个监听器来获取异步通知
- Selector IO多路复用组件，通过 Selector 一个线程可以监听多个连接的 Channel 事件
- NioEventLoop 事件处理组件 ，其维护一个线程及任务队列
- NioEventLoopGroup 事件处理器生命周期管理组件
- ChannelHandler IO操作接口
- ChannelHandlerContext  IO操作上下文，与ChannelHandler绑定
- ChannelPipeline  ChannelHandler流水线

### 线程模型

基于主从 Reactors 多线程模型

![img](https://upload-images.jianshu.io/upload_images/1500839-9c4e284f0dc58d97.jpg)

### 主要流程

#### Boss NioEventLoopGroup

1.  监听网络连接， 轮询Accept事件
2. 处理 Accept I/O 事件，与 Client 建立连接，生成 NioSocketChannel，并将 NioSocketChannel 注册到某个 Worker NioEventLoop 的 Selector 上
3. 处理任务队列

#### Worker NioEventLoopGroup

1. 轮询 Read、Write 事件
2. 处理 I/O 事件，即 Read、Write 事件，在 NioSocketChannel 可读、可写事件发生时进行处理
3. 处理任务队列中的任务，runAllTasks



### 样例

```
publicstaticvoidmain(String[] args) {
    // 创建mainReactor
    NioEventLoopGroup boosGroup = newNioEventLoopGroup();
    // 创建工作线程组
    NioEventLoopGroup workerGroup = newNioEventLoopGroup();
	finalServerBootstrap serverBootstrap = newServerBootstrap();
	serverBootstrap
        // 组装NioEventLoopGroup
        .group(boosGroup, workerGroup)
        // 设置channel类型为NIO类型
        .channel(NioServerSocketChannel.class)
        // 设置连接配置参数
        .option(ChannelOption.SO_BACKLOG, 1024)
        .childOption(ChannelOption.SO_KEEPALIVE, true)
        .childOption(ChannelOption.TCP_NODELAY, true)
        // 配置入站、出站事件handler
        .childHandler(newChannelInitializer<NioSocketChannel>() {
        	@Override
        	protectedvoidinitChannel(NioSocketChannel ch) {
        		// 配置入站、出站事件channel
        		ch.pipeline().addLast(...);
        		ch.pipeline().addLast(...);
        	}
        });
        // 绑定端口
        intport = 8080;
        serverBootstrap.bind(port).addListener(future -> {
        	if(future.isSuccess()) {
        		System.out.println(newDate() + ": 端口["+ port + "]绑定成功!");
        	} else{
        		System.err.println("端口["+ port + "]绑定失败!");
        	}
        });
}
```

## TCP

### 拆包与粘包

> 1. 要发送的数据大于TCP缓冲区剩余的大小，发生拆包
> 2. 要发送的数据大于MSS(最大报文长度)，发生拆包
> 3. 要发送的数据小于TCP缓冲区的大小，TCP将多次写入缓冲区的数据一次发送出去，发生粘包
> 4. 接收数据端的应用层没有及时读取接收缓冲区中的数据，将发生粘包

### 解决方案

#### 固定长度的拆包器 

> FixedLengthFrameDecoder

该解码器会每次读取固定长度的消息，如果当前读取到的消息不足指定长度，那么就会等待下一个消息到达后进行补足。

编码器实现要保证每次的数据的长度， 不足部分要补足。



#### 行拆包器 

LineBasedFrameDecoder

#### 分隔符拆包器 

DelimiterBasedFrameDecoder

对象拆包器

public class ObjectDecoder extends LengthFieldBasedFrameDecoder

public class ObjectEncoder extends MessageToByteEncoder<Serializable> 

#### 基于数据包长度的拆包器

LengthFieldBasedFrameDecoder与LengthFieldPrepender

https://blog.csdn.net/agzhchren/article/details/91359168