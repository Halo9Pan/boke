---
layout:     post
title:      Netty 简介
date:       2016-03-04
categories: Development
---

[Netty](http://netty.io/)是一个高性能、异步事件驱动的NIO框架，它提供了对TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。  
![Netty](http://netty.io/images/components.png)

### Major Classes
* [Channel](http://netty.io/4.0/api/io/netty/channel/Channel.html)
* [EventLoop](http://netty.io/4.0/api/io/netty/channel/EventLoop.html)
* [ChannelFuture](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html)
* [ChannelPipeline](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html)
* [ChannelHandler](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html)
* [ByteBuf](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)

* Encoder, Decoder  
[MessageToMessageDecoder](http://netty.io/4.0/api/io/netty/handler/codec/MessageToMessageDecoder.html)
[MessageToMessageEncoder](http://netty.io/4.0/api/io/netty/handler/codec/MessageToMessageEncoder.html)
[ByteToMessageDecoder](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html)
[MessageToByteEncoder](http://netty.io/4.0/api/io/netty/handler/codec/MessageToByteEncoder.html)
[MessageToMessageCodec](http://netty.io/4.0/api/io/netty/handler/codec/MessageToMessageCodec.html)

### Transports
|Name    |Package                    |
|--------|---------------------------|
|NIO     |io.netty.channel.socket.nio|
|Epoll   |io.netty.channel.epoll     |
|OIO     |io.netty.channel.socket.oio|
|Local   |io.netty.channel.local     |
|Embedded|io.netty.channel.embedded  |