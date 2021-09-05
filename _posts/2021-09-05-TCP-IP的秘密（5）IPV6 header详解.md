---
title: "TCP/IP的秘密（5）IPV6 header详解"
date: 2021-09-05
---


### IPV6包头格式

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37de01db249947c0974be8a220bb5194~tplv-k3u1fbpfcp-watermark.image)

- Version: IP协议版本，4bits，默认是0110；
- Traffic Class: 业务类型，8bits；与v4中的DSCP对应；

- Flow Label: 20bits，新增的字段，流标签，发送方用来标识一系列的数据包需要在网络中被当作一个流业务进行处理；通过流标签、源地址、目的地址就可以唯一标识一个流业务；
- Payload Length: 16bits，专指除去包头的IP包长度，也就是data的长度，但如果有扩展包头，那么拓展包头也算在payload length里面；
- Next Header: 8bit，标识紧跟着本包头的下一个包头的协议类型，下一个包头有可能是传输层协议，比如TCP,UDP等，也有可能是扩展包头；
- Hop Limit：8bits, 和V4中的TTL相同，每通过一个节点减1，如果减到0仍然没有到达目的地，则该IP包会被丢弃；
- Source Address: 128bits，源地址；
- Destination Address: 128bits，目的地地址；如果是路由包头的话，这里的地址有可能不是最终的接收方；

我们来看一个实例：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30da16623def4fb0902d6fea726ebccf~tplv-k3u1fbpfcp-watermark.image)
- Version版本号：6
- Traffic class: 默认值0
- Flow Label流标签：092dcd，通过这个标签和源地址，目的地可以唯一确定一个流；
- Payload Length：有效数据长度664字节；
- Next Header:下一个header是UDP包，可以看到下一个UDP的包头信息；
- Hop Limit:跃点数量，1个，只需要被一个节点处理；
- Source Address:FE80:B002:D7A:E02B:624, 从FF80我们可以得知，这是一个link-local地址；
- Destination Address: FF02::C, 这个地址是一个多播地址，开头8位都是1，scope值=2，意味着多播范围是local-link，所以这是一个从本地link-local地址发向本地链路的多播消息；


### IPV6扩展包头

- 和V4不一样，V6中的扩展包头是一个独立的包头，位于V6默认包头和上层协议包头中间，而且没有长度限制；
- 如果没有扩展包头的话，则V6包头中的Next Header表示的就是上层协议，或者“No Next header"表示这个包只是单纯的网络层信息，没有上层协议包头；

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65c1ec7843c641878d090bbcdf175113~tplv-k3u1fbpfcp-watermark.image)

以下是Next Header字段对应的值：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65dddadef1644cc4bd86128673886457~tplv-k3u1fbpfcp-watermark.image)

IPV6包括6种扩展包头：
1. Hop-by-Hop Options，逐跳寻径选项，会被发送路径上的任何路由器和其他节点解析，这个扩展包头必须紧跟V6的默认包头，它在V6包头中的next header值是0；

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0c0d9f010b744fcbf3d9716227abb2e~tplv-k3u1fbpfcp-watermark.image)

这里面的Hdr Ext Len指的是hop-by-hop选项扩展包头的长度，但不包括最开始的那8个字节；options选项包括一些路由如何处理该数据包的指示；

2. Routing，路由，发送方通过这个包头强制指定该数据包在发往目的地路径上需要访问的路由器；
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc63f29b0c154a8b86e894301f493edd~tplv-k3u1fbpfcp-watermark.image)


3. Fragment 分段， 用来让发送方可以发送路径最大传输单元MTU的数据包给接收方。接收方可以根据这个包头重新组合数据；与IPv4不同，IPv6在发送方对数据进行分段；

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3566f90dc8e3414093ad28161ba2e0e2~tplv-k3u1fbpfcp-watermark.image)

4. Destination Options，目的地选项，除了Destination扩展包头以外，其他包头应该在IP包中最多出现一次；这个扩展包头是用来携带接收方需要解析的选项信息；

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7ca7c99db9d4d1cbfd938f81f01749f~tplv-k3u1fbpfcp-watermark.image)

5. Authentication，验证，用于IPSEC协议，对于IP包的身份验证和完整性保护；

6. Encapsulating Security Payload，封装安全有效负荷，同样用于IPSEC协议安全保护；


### IPV6与IPV4对比

总的来讲，IPV6的包头相比于V4主要有2个特点：
1. 降低了路由处理的复杂度，提升传输效率。
    - 通过省去了checksum，将校验交给传输层；
    - 将数据分段交给发送方而不是路由器；
    - 默认包头长度固定；

2. 扩展性更强。
    - 扩展包头的耦合性更低，几个扩展包头独立分开，
    - 不同的扩展包头功能的格式和容量更灵活；

详细对比如下：

| 对比 | V4 | V6|
| --- | --- | --- |
|数据分段|在路由器中进行分段|在发送方进行分段|
|地址前缀| 使用CIDR标记/NN| 使用子网前缀/NNN|
|相邻节点发现| 使用ARP协议| 使用ICMP V6协议|
|DNS| 支持A地址解析|支持AAAA地址解析|
|DHCP| 支持 | 支持DHCP V6|
|NAT| 支持| 暂不支持，因地址足够，不需要转换|
| 默认包头长度 | 长度不固定，最少是60byte |长度固定40byte|
|字段数量|12个字段|8个字段|
|扩展包头|支持，和默认包头在一起|支持，独立的包头|
|字段对比|version|version|
||TTL|HOP LIMIT|
||IHL| 无|
||Type Of Service|Traffic Class|
||Total Length|Payload Lenght只包括数据和扩展包头长度|
||Host ID|无|
||Fragment flag|无，通过fragment扩展包头实现|
||Fragement offset|无|
||Protocal|Next Header|
||Header checksum| 无，可通过扩展包头实现|
||无|Flow Label|




以上，感谢阅读，如有不准确和错误之处请留言指正，我会立即修正，感谢！

<br/>
<br/>
<hr/>



总结不易，请勿私自转载，否则别怪老大爷不客气

欢迎喜欢技术的小伙伴和我交流，微信1296386616





参考资料：

《Internet Protocol, Version 6 (IPv6) Specification》 IETF
https://datatracker.ietf.org/doc/html/rfc8200


《IPv6学习(2) IPv6报文格式》  Jercey
https://www.cnblogs.com/jersey/archive/2011/11/29/2267492.html

《IPv4 和 IPv6 报头格式说明》  Ricky 
https://ccie.lol/knowledge-base/ipv4-and-ipv6-packet-header/
