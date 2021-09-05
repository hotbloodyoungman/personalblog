---
title: "DNS的秘密（3）Anycast任播"
date: 2021-09-05
---


书接上回，从真实的DNS寻址过程中，我们看到第一步向LDNS查询根域名服务器时，一共返回了13个结果：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49c13b43a0da469082b8cf4b35cea42b~tplv-k3u1fbpfcp-watermark.image)

很多文章中就写，全世界只有13台根服务器（🤣）

这其实是个很大的误区，试想如果只有13台服务器，如何能承受得起每天数十亿次的查询请求呢？

DNS除了应用分级查询，缓存等措施外，还使用了anycast任播技术。


## Anycast
中文翻译叫“任播”，与“unicast单播”, “broadcast广播”，“multicast多播”，统为四种寻址方式。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71e2227bb7464590aae72083b781619a~tplv-k3u1fbpfcp-watermark.image)

Anycast是一种互联网寻址和路由方法，它允许在不同位置的多台服务器共享一个IP地址，并通过判决算法将数据路由至期望的服务器，比如时延最短的，负载最轻的，或距离最近的等等；

这个技术最成功的应用就是DNS和CDN。

13台跟域名服务器虽然对应13个IP地址，但每个IP都对应着散落在世界各地不同数量的服务器，下图是截至发稿日期的全世界DNS服务器分布，全世界目前有1402台根域名服务器：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e261d37ef6c94aff88997111d5afc084~tplv-k3u1fbpfcp-watermark.image)

每个根节点下都有不同数量的服务器，这样做的好处是：
1. 可以缩短响应延时，要知道DNS寻址只是打开网页的第一步，要尽快缩短这个时延，通过选择距离客户端最近的服务器，可以在物理上减小信令的传输距离，或者选择hop数最少的服务器，减少路由转发次数来缩短时延；

2. 增加系统可靠性，如果共享IP的一个服务器出现问题，那么请求会被转发到其他同IP服务器上，这样整个系统的可靠性就增加了；

3. 应对DDOS攻击，我们知道DNS服务器是最容易收到DDOS攻击的，使用anycast之后，可以均衡查询请求到临近服务器，分散负载；

4. 系统扩展性增强，DNS系统可以更容易的增加服务器容量，不需要额外的IP地址；

因为这些好处，目前不仅仅是根域名服务器使用anycast，顶级域名服务器和权威域名服务器都在使用；

以上就是Anycast的关键内容和工作原理，感谢阅读，如有不准确和错误之处请留言指正，我会立即修正，感谢！

<br/>
<br/>
<hr/>



总结不易，请勿私自转载，否则别怪老大爷不客气

欢迎喜欢技术的小伙伴和我交流，微信1296386616




参考资料：

root-servers.org
https://root-servers.org/

《什么是 Anycast DNS？》  CloudFlair
https://www.cloudflare.com/zh-cn/learning/dns/what-is-anycast-dns/

Wikipedia
https://en.wikipedia.org/wiki/Anycast

《根域名服务器只有13台？》 命运之轮
https://zhuanlan.zhihu.com/p/107492241
