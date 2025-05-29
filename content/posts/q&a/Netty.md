## 定义

Netty 是一个基于 NIO 开发的由异步事件驱动的网络应用框架，为了实现高可用的服务器和客户端的开发。

*   基于 NIO
*   异步事件驱动
*   高可用
*   支持多种协议

## 为什么要用 netty

## Netty 的特性总结

*   设计
    *   统一的 api，支持多种传输类型
*   整合了多种主流协议，编解码方便
*   通过 ChannelHandler 方便定制自己的扩展逻辑
*   基于 NIO，性能高效
*   使用的人多，有更多的资料可以使用

## Netty 核心组件

*   Channel
*   回调方法

    *   channleActive
    *   channelInactive
    *   channelRead
    *   ...
*   Future（也是回掉）
    *   ChannelFutureListener.operationComplete
*   事件和ChannelHandler

## BIO、AIO、NIO

*   BIO（Blocking IO）阻塞IO
*   AIO（Asychronous IO）异步IO
*   NIO（NON-Blocking IO）非阻塞IO

