---
title: "Gateway, Router, Modem, Bridge, Hub, Switch傻傻分不清（1)"
date: 2021-09-05
---


上面这些名词，从事互联网的都应该耳熟能详，他们是构建通信网络的基础节点，没有他们，可能连局域网都撑不起来。那么，在错综复杂的网络结构中，他们的位置，作用和区别都是什么呢？该如何利用他们实现互联互通呢？

## 网关Gateway
先从网关说起，因为网关和其他几种通信设备不一样，它是一个抽象的概念，有多种含义，具体要看上下文。

**网关是什么？**
网关是各类"网络关口"的统称，网络关口可以理解为网络数据进出用的一道门，根据这道门两边不同的对象，以及这道门本身的功能，可以分为以下几类：

1. 传输类网关，如路由器Router等，在网络拓扑中位于子网边界，发挥连接传输的作用；
    - 路由器作为默认网关，可以连接两个不同网段，为不同网段的主机提供路由和转发服务；
    - 可以说路由器是网关，但不能说网关是路由器，以偏概全了；

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8051de7d256b4f37bf6baa39c2627cc7~tplv-k3u1fbpfcp-watermark.image)

2. 转换类网关或翻译类网关，如媒体网关，信令网关，NAT网关等。这类网关发挥的是转化器和翻译器的作用。
    - 如媒体网关是将媒体数据在不同的传输和编码技术之间进行转换；
    - 信令网关将一种协议信令翻译成另一种协议信令，用来在另一类网络中进行通信，如TDM转化为SIP；
    - NAT网关则是将私网地址转换为了公网地址或指定端口；

3. 安全类网关，如防火墙，或身份认证网关，在不同网络之间，提供安全防护的功能，起到过滤和门禁的作用；

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b701fd8460fb43caaac007ca4cda2496~tplv-k3u1fbpfcp-watermark.image)

4. 代理类网关，如API网关，作为服务器的反向代理，处理并响应客户端提出的各种传入API调用请求，而不是由服务器直接响应客户端的请求；

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8bbba65dd68462ab1c7ae2161598218~tplv-k3u1fbpfcp-watermark.image)

在实际部署中，不同种类的网关往往不是一个人在战斗，拿LTE网络架构中的SGW（serving Gateway)和PGW（Packet Data Network Gateway)为例：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0cbbdc3337c456495fc960ff09b23d4~tplv-k3u1fbpfcp-watermark.image)

Serving Gateway服务网关：主要负责的是用户数据包的路由和转发，这就包括了用户移动性管理，系统内和系统间的切换，同时还负责S1接口与S11接口数据格式转换，传输层数据优先级管理，以及跨运营商的计费；

PGW数据网络网关：主要建立用户到数据网络的连接，包括DHCP动态分配IP地址，数据包深度过滤，传输层数据优先级管理，用户传输速率控制，计费等等；

### NAT网关
NAT网关Network Address Translation gateway，作用是将私网IP和端口转换为公网IP和端口，这个转换是双向的，解决公网IP受限的问题，并起到隐藏私网网络拓扑结构的作用。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfd1c0a8bda445c5b05bacc4b023c9e7~tplv-k3u1fbpfcp-watermark.image)

主要分为3类：
1. 静态NAT：将私网IP一对一地转化为公网IP，这个转化关系是固定的，这是最原始的NAT形态，只能起到保护私网拓扑的作用，主要用于外网访问特定的服务器或服务器组；

2. 动态NAT：私网IP共享一个公网IP池，每次新建公网连接都会动态分配一个公网IP，转换关系依然是一对一，可以少量节省公网IP地址；

3. 端口地址转化PAT(Port Address Transloation): 将多个私网IP都转换为一个公网IP地址，并将源端口转换到该公网IP的指定端口上，这样就实现了大量节省公网地址的目的，并通过端口映射实现了数据双向流通；
    - 实际工作中，NAT网关会将服务器发送的数据包进行解析，并将其中的源IP和源端口修改为路由器或NAT网关的IP地址和对应端口，再发送给外网用户，而外网用户的响应也是发送给NAT网关，再由网关转发给源服务器，在这过程中，NAT网关相当于一个反向代理的作用；

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a4c991263bb4fb88b728d58fe26ef7a~tplv-k3u1fbpfcp-watermark.image)


参考资料：

《3GPP TS 23.401 version 8.16.0 Release 8》  ETSI
https://www.etsi.org/deliver/etsi_ts/123400_123499/123401/08.16.00_60/ts_123401v081600p.pdf
