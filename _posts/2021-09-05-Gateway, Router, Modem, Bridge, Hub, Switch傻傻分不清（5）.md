---
title: "Gateway, Router, Modem, Bridge, Hub, Switch傻傻分不清（5）"
date: 2021-09-05
---


书接上回，通过前面几篇，我们已经对这几个通信网元有了全面地了解，是时候给这个系列做一个总结了。

## 横向对比

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea1297fd235642e69b68eaaf04bfb868~tplv-k3u1fbpfcp-watermark.image)

**1. 工作层级**
- Router识别IP包，工作在第三层网络层；
- Modem改变信号物理格式，工作在第一层物理层；
- Hub对信号进行放大和复制转发，工作在第一层物理层；

- Bridge可以根据MAC地址判决转发，工作在第二层数据链路层；
- Switch可以识别MAC地址，工作在第二层数据链路层；
- 层三交换机可以通过添加路由模块实现识别IP并进行路由，工作在第三层网络层；
- 层四交换机可以根据TCP/UDP数据包来进行负载均衡等高级调度，工作在第四层传输层；

**2. 工作场景**
- Router工作在网络边缘，通过路由和转发功能，实现连个不同网段的互联互通；
- Modem工作在网络边缘，通过转化信号格式来实现在不同介质上传输；
- Hub工作在局域网的核心，通过信号放大和广播实现一个网段内的互联互通；
- Bridge工作在两个局域网的中心，桥接同一网段的两个局域网；
- Switch工作在局域网核心，实现局域网内端到端高速通信

**3. 广播域与冲突域**
- Router可以分割广播域，路由器的一个端口就是一个广播域；
- Hub将所有连接到其的host共同组成一个广播域和一个冲突域；
- Bridege可以分割冲突域，网桥的一个端口就是一个冲突域，但网桥不能分割广播域，连接到网桥的局域网都处于同一个广播域；
- Switch和网桥一样，可以进一步分割冲突域，但不能分割广播域；
- Switch可以通过划分VLAN的方式分割广播域，一个VLAN就是一个广播域；

**4. 双工模式**
- Router是全双工模式，可以同时进行收发数据；同时也支持半双工模式；
- Modem是全双工模式，可以同时进行调制和解调；
- Hub是半双工模式，不能同时发送和接收，会造成冲突；
- Bridge同样也是半双工模式，不能同时发送和接收；
- Switch是全双工模式，端口间可以同时收发数据；同时也支持半双工模式；

## 实际端到端通信流程

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e226f7408ba9426e8a95149da71a2221~tplv-k3u1fbpfcp-watermark.image)

我在HTTP的秘密系列中讲解了浏览器上网的过程，从HTTP讲到DNS, 讲到TCP，又深入到了TLS安全协议，在更深入地了解物理层到网络层的通信设备和机制，那么我们再整体看一下一次真实的互联网通信端到端流程：

### 一次完整的互联网通信过程

1. 接入网络：当我们的手机或电脑接入网络，接入方式可以是咖啡厅wifi，手机热点，公司的网线，还是运营商的4G网络，这时我们和网络建立的只是物理连接，也就是层一；

2. 获取IP：
    - 接入网络后，网卡会通过端口发送广播消息，请求DHCP服务器（Dynamic Host Configuration Protocol动态主机配置协议）分配IP地址，请求消息中包含主机的MAC地址信息；
    - 一般来说，DHCP服务会集成在路由器中，或是在运营商的机房里；
    - 当DHCP服务器收到请求后，会自动从IP地址池中选择可用的IP地址分配给主机，并告知路由器地址，子网掩码，DNS地址，租约时长等信息，下面是DHCP消息截图：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aab4da98d15c4751b114b6b055745822~tplv-k3u1fbpfcp-watermark.image)

3. 获取网关（路由器）MAC地址及局域网内邻居的MAC地址：
    - 光知道路由器IP地址是不够的，我们还需要知道MAC地址才能建立数据链路层的连接；
    - 这是通过ARP协议实现的，主机会发送ARP广播消息，询问路由器IP所对应的MAC地址，路由器收到广播消息后，会将自己的MAC地址发送给主机，这样双方的层二连接就可以建立了；
    - 主机不光需要知道路由器的MAC地址，还会在局域网内发送ARP广播，询问同网段的邻居的MAC地址，用于局域网内的通信；
    
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c34fc4ea4ec44888e0306abffd7901a~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75df1a83e258460eabc9093da076e77d~tplv-k3u1fbpfcp-watermark.image)

4. 主机此时就可以通过路由器访问互联网内容了，当在浏览器中填写网址后，主机会先向DNS服务器发送请求获取目标IP地址；

5. 首先建立到目标IP的TCP连接，将目标IP地址放入到IP包中，此时有一个重要的判断，判断目标IP是否在局域网中：
    - 如果在一个网段，则可以直接通过MAC地址进行通信；
    - 如果不在一个网段，则必须通过路由器才能通信，这是协议规定；
    - 路由器会负责将IP包转发给目标IP，所以此时主机会将路由器的MAC地址放入数据帧中，由路由器再决定下一跳的MAC地址：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78268fd76a0b47a09b71270d22893edd~tplv-k3u1fbpfcp-watermark.image)

6. 路由器会将数据包发送给Modem进行调制，调制到光纤或电话铜线上进行传输，传输到下一跳，直到到达目的地路由器；

6. 目标IP响应主机请求，响应消息同样是经过路由器转发给主机：
    - IP层的源IP为目标IP，但数据链路层的帧头，却是路由器的MAC地址，因为在局域网内是由路由器负责转发：
    
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9f52602929b47c6a863250336c46cb0~tplv-k3u1fbpfcp-watermark.image)

8. **这里要注意的是**，虽然我们在IP包中看到的是本机的IP地址，但通过网段10.0.0.N可以得知，这只是一个私网地址，是不能用来做公网通信的，所以路由器在不知不觉中做了一个NAT地址转换：
    - 路由器收到我们发往公网的IP包解析后，将其中的源IP改为公网IP，再发向下一个路由；
    - 并将收到目标IP响应的IP包中的IP地址，改为我们的客户端IP地址，再发给客户端；
    - 所以这也是为什么我们在公网查询本机IP时和我们本地看到的不一致的原因；
    - 在路由器管理软件中，我们可以看到自己的公网地址，这个公网地址是所有局域网主机共用的；
    
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13c5d38b12e64fb3aa592e4acca14994~tplv-k3u1fbpfcp-watermark.image)
    
感谢阅读，如有不准确和错误之处请留言指正，我会立即修正，感谢！

<br/>
<br/>
<hr/>



总结不易，请勿私自转载，否则别怪老大爷不客气

欢迎喜欢技术的小伙伴和我交流，微信1296386616
