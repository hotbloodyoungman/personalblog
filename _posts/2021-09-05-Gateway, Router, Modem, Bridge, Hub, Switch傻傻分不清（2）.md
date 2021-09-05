---
title: "Gateway, Router, Modem, Bridge, Hub, Switch傻傻分不清（2）"
date: 2021-09-05
---


书接上回，本文详细介绍Router

## Router路由器
1. 路由器的作用域是两个不同的网段或VLAN；同一个网段内的通信不需要用到路由器；

2. 路由器的两个功能：路由和转发；所谓路由，就是找到一条从源IP到目标IP间一条“最佳路径”，就像我们平时用的导航功能一样；转发，就是将数据包从一个节点发送到另一个节点；

3. 路由器是根据IP地址进行路由，即工作在TCP协议的第三层，网络层；它不关心层二和层一的传输介质，底层的传输方式对于路由器来说是透明的；

4. 路由器每一个端口连接的子网都是一个广播域，也可以说路由器分割了广播域；

5. 接入公网的路由器会与NAT，调制解调器一起，合并为默认网关；

### 路由协议
路由协议使路由器可以解析数据包，维护本地路由表，并于其他路由器交换路由信息。针对不同的应用场景，路由器使用不同的路由协议，常见的路由协议包括：RIP(Routing Information Protocol), OSPF(Open Shortest Path First), BGP(Border Gateway Protocol)

**路由表routing table**，这是所有路由协议的核心：
- 路由表的作用是，标示如果要去某一个地址，下一步应该向哪里走；

- 他的内容包括但不限于：本地路由器状态，本地子网信息，本地端口到下一跳端口的路径信息，路由路径状态，路由路径成本，周边路由器状态等等；

- 根据以上信息，路由器在收到数据包后，可以根据他的源IP和目的地IP，选择是把它发给本地子网还是发给下一个路由器，并根据表中的信息选择最佳路径上的路由器进行转发；

- 路由器之间可以互相广播自己的路由表；

**RIP路由信息协议：**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66d387076fd94eef9c6654c5c128db74~tplv-k3u1fbpfcp-watermark.image)
- RIP是应用层协议，使用UDP传输；

- 基于距离矢量算法的路由协议，这里的距离就是“hop count跃点数量”，直接相连的节点的距离是1，远端节点每增加一级节点距离就+1，最佳路径即距离短的路径；

- 协议规定可达的最大距离是15跳，超出15跳的目的网络认为是不可达的；所以RIP一般用于小规模的网络；

- 路由表中记录着源网络到目标网络的路径以及距离（跃点数）；

- 路由器会定时将自己的路由表通过广播的方式发给所有相邻节点；

- RIP的最大缺点除了限制了网络规模以外，就是简单化了路径距离，只以跳数为计量而忽略了实际链路的状况，比如带宽，负载和延迟等等；

**OSPF开放式最短路径优先**
- 路由器维护一张链路状态数据库link state database，这个数据库就是路由所在网络的拓扑图

- 基于shortest path first (SPF) algorithm最短路径优先算法
    - 当路由器加入网络后，它会向全网广播hello消息用来发现相邻节点
    - 之后将自己到直接连接的相邻路由器的连接速率进行广播
    - 其他节点收到广播后，会更新自己的链路状态数据库，并向其他路由器广播，这样网络中的每个路由器就都有了完整的全网路由信息；

- 当有链路状态发生更新或改变的时候，路由器只会广播变化的部分，而不会广播整个链路表，这样也节省了带宽消耗；
- 同时，为了进一步减小广播风暴，OSPF还引入了area区域概念，将一个大网络分成多个不同的area，同一个area的链路状况只会在本area内广播；
- 基于以上特点，OSPF协议适用在大型网络

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e51ced6f7b34f369dca0373c0271c0e~tplv-k3u1fbpfcp-watermark.image)

**BGP边界网关协议**
- 提到边界就要引入一个autonomous system自治系统的概念，它是用来定义边界的。自治系统指的是一组拥有相同的路由策略的，由一个或多个网络运营商管理的一个或多个IP网络。大型AS有自己唯一的编号ASN；
- 自治系统也有不同种类，多出口的自治系统（Multihomed AS，连接多个其他AS，但不允许中转路由）；末端自治系统（Stub AS,只连接一个其他AS）；中转自治系统(Transit AS，连接多个AS且允许中转路由)。

- 自治系统内部的路由协议称为内部网关协议IGP，与外部其他AS交互的路由协议的称为外部网关协议EGP，前面讲到的RIP和OSPF都属于IGP，而BGP则属于EGP；

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ec6ef2f2ace4b8b84cd519003f6bf83~tplv-k3u1fbpfcp-watermark.image)

- BGP是应用层协议，路由信息使用TCP传输以保证可靠性；

- 采用BGP的路由器在初始化时必须手动建立邻居关系（Peer relation)

- BGP基于路径矢量Path-Vector算法
    - 路由器会广播自己相邻的可达路由器信息，而收到这条信息的路由器会把自己加到这条路径中，再向其他相邻路由器发送；
    - 如果发现自己已经在这条路径中的话，就会丢弃这条消息；这样可以有效避免环路loop的产生；
    - 路径信息是由AS号组成
    - 最终，路由表都包含了所有目的网络，下一跳和完整的路径信息；

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d945b0adc6d424083f62a36afb21063~tplv-k3u1fbpfcp-watermark.image)

### 单臂路由Router-on-a-Stick
单臂路由属于路由器一个特殊的应用场景, 即在一个端口上实现数据包的输入和输出：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/927ef3debd2d4f6692b2d258af5dcb34~tplv-k3u1fbpfcp-watermark.image)

- 为了在局域网内部实现访问控制，可以将一个大型局域网划分为多个虚拟局域网VLAN，不同vlan之间不能相互通信，必须通过路由器进行转发；

- 路由器使用一个物理端口，通过配置多个“逻辑接口”来实现在不同vlan之间互相通信；

- VLAN1的数据包通过交换机发送给默认网关，经过网关路由后，转发到另一个vlan2的IP，并通过同一台交换机进行转发；

- 使用层三交换机也可以实现同样的效果，我们下一节会重点介绍交换机的相关知识；

以上，感谢阅读，如有不准确和错误之处请留言指正，我会立即修正，感谢！

<br/>
<br/>
<hr/>



总结不易，请勿私自转载，否则别怪老大爷不客气

欢迎喜欢技术的小伙伴和我交流，微信1296386616


参考资料：

《OSPF》 Nokia
https://infocenter.nokia.com/public/7210SAS203R1A/index.jsp?topic=%2Fcom.sas.protocols.k%2Fhtml%2Fk_ospf.html

《Path-vector routing protocol》 wikipedia
https://en.wikipedia.org/wiki/Path-vector_routing_protocol

《Practical BGP》   Russ White, Danny McPherson, Srihari Sangli
https://www.informit.com/articles/article.aspx?p=331613&seqNum=2

《网络协议之路由协议》  harkecho
https://www.huaweicloud.com/articles/11964767.html
