---
title: "TCP/IP的秘密（4）IPV6 Anycast & Multicast"
date: 2021-09-05
---


书接上回, 本文详细介绍IPV6的anycast和multicast地址类型, 并汇总整理IPV6对比V4地址的区别和联系

### Anycast任播地址
- 地址来自于单播的地址空间，并且语法与单播并无二至，所以在给节点分配anycast地址时必须明确设置为anycast类型；

- 使用anycast地址发送数据包时，数据包会被发送到“最近的”接口，这个“最近”是按照不同的路由协议计算的，有可能是hop或距离；

- 其中有一类任播地址格式是特殊定义的，就是路由器anycast地址，格式如下图；

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48541d08c47745a3a12881c4cc09bed7~tplv-k3u1fbpfcp-watermark.image)

- 其中子网前缀用来标识一条特定的链路link，这个前缀和该链路上其他接口的unicast地址是一样的。发往该路由器anycast地址的数据包会被送到这个子网中路径最短的路由器上，子网内的所有路由器都必须支持“路由器anycast地址”。

### Multicast多播地址（组播）
- multicast地址用来分配给一组通常位于不同节点上的接口，数据会发送给拥有这个地址的所有接口。一个接口可以拥有多个multicast地址，节点可以同时监听多个组播地址；

- 地址格式，前8位固定位1，即ff00::/8，

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9da525ef546411d8f1afacc2b3aba7c~tplv-k3u1fbpfcp-watermark.image)

- flag字段为 0|R|P|T：
    - 第一位是保留位，必须是0；
    - 第二位R是汇聚点的多播地址指示，详情可以参见RFC3956
    - 第三位P是基于单播前缀的多播地址格式（Unicast-Prefix-based），详情参见RFC3306；
    - 第四位T是指生命周期，0标识该地址是IANA分配的永久有效的多播地址，1标识该地址是临时的多播地址；

- scope指示该地址的路由范围：包括本地接口Interface-Local(用于环回的多播），本地链路(link-Local)，本地管理（需要管理员配置），本地站点（Site-Local），本地组织（organization-Local, 用于多个站点多播）.

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae077da9682c4db89ac491860026d35c~tplv-k3u1fbpfcp-watermark.image)

- 预定义的多播地址：
    - FF01::1(Interface-Local本地接口范围所有节点组播地址)
    - FF01::2(Interface-Local本地接口范围所有路由器组播地址)
    - FF02::1(Link-local本地链路范围所有节点组播地址)
    - FF02::2(Link-local本地链路范围所有路由器组播地址)
    - FF05::2(Site-local本地站点范围所有路由器组播地址)
    - **Solicited-Node Address**，请求节点地址，FF02:0:0:0:0:1:FFXX:XXXX，后6位是该节点的unicast或anycast的最后6位十六进制数；比如2000::01:800A:200E:8C6C，那么它的请求节点地址就是FF02::1:FF0E:8C6C，后6位相同；
    
### 一个节点必须配置的地址
1. 对于主机host来说，以下地址必须配置：
    - 每个接口1个link-local地址；
    - 任意额外的unicast单播和anycast任播地址
    - 本地环回地址；
    - 所有预定义节点组播地址，包括：link-local, site-local, interface-local
    - 每个unicast和anycast对应的请求节点地址(solicted-node address)
    - 节点归属的所有其他多播组群的地址

2. 对于路由器来说，以下地址必须配置：
    - 每个配置了路由的接口需要配置Subnet-Router Anycast addresses路由器任播地址
    - 所有其他的任播地址
    - 所有预定义路由器组播地址
    
    
### **IPV6与V4对比**
仅从地址的维度, 我们来看一下两个协议的区别和联系

| 项目 | V4 |V6|
| --- | --- |---|
|对象|按照节点或设备分配|按照接口分配,一个节点可以有多个不同类型的地址|
| 容量 | 32bits | 128bits|
|格式|xxxx.xxxx.xxxx.xxxx|使用冒号分割，完整格式，简短格式|
|CIDR|支持，使用/n标识二进制网络号位数|前缀prefix, /N代表二进制网络号位数|
|广播地址|支持|不支持，使用multicast地址代替|
|多播地址|支持|支持,且增加了scope和flag字段, 且预设了多个组播地址, 接口会默认加入|
|anycast|no|支持, 和unicast公用地址, 且规定了针对路由的anycast地址格式|
|私网地址|支持,划分单独网段|支持,本地唯一地址, 指定网段, 且地址是全网唯一|
|环回地址|支持|支持|
|兼容性|通过NAT,BGP等协议支持与IPV6通信|设置专门网段来映射IPV4地址|


以上! 下一篇详细介绍IPV6报头的内容, 感谢阅读，如有不准确和错误之处请留言指正，我会立即修正，感谢！

<br/>
<br/>
<hr/>



总结不易，请勿私自转载，否则别怪老大爷不客气

欢迎喜欢技术的小伙伴和我交流，微信1296386616


参考资料：

《IP Version 6 Addressing Architecture》 IETF
https://datatracker.ietf.org/doc/html/rfc4291

《IPv6 Global Unicast Address Format》 IETF
https://datatracker.ietf.org/doc/html/rfc3587

《Internet Protocol Version 6 Address Space》 IANA
https://www.iana.org/assignments/ipv6-address-space/ipv6-address-space.xhtml

《第 3 章 IPv6 介绍（概述）》 Oracle Corporation
https://docs.oracle.com/cd/E19253-01/819-7058/6n91g7dd4/index.html
