---
title: "当我们谈论HTTPS时，我们在谈什么(八) Session Ticket, Session ID 和 PSK"
date: 2021-09-05
---


Session Resumption会话恢复，是TLS握手流程的重要部分，有了它的存在，可以大幅的缩短握手流程。TLS中主要依靠session id, session ticket和PSK三种方式进行会话恢复，我们今天就来整体介绍一下。

## Session id & Ticket
这两者都是在TLS1.2中用到的机制，首先看一下引入他们后的效果

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84f3b3dec82f428cb919773b570d89d3~tplv-k3u1fbpfcp-watermark.image)

可以看到红框里的session resumption流程，将TLS握手的9步缩短到了4步。


### Session ID
Session id是由服务器分配的，发给客户端让客户端保存，在客户端发起新会话时，会将这个ID在client hello中发给服务器，用来判断是否恢复之前的session。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54df15011c30428d9524ac2a7df40d6f~tplv-k3u1fbpfcp-watermark.image)

服务器在缓存中查找该id，如果找到了，并且服务器愿意重建该会话的话，那么就会在Sever Hello消息中发送同样的session id

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4add9d834ffa41128a6022d55b5dd180~tplv-k3u1fbpfcp-watermark.image)

之后双方就可以开始用之前协商过的加密模式开始传输数据。

Session id是最传统的方式，它的主要问题是，会话状态保存在服务器端，当大量用户访问服务器时，服务器用来保存会话状态的空间占用就会很大，需要对保存时间，保存机制做优化。

为了解决这个问题，TLS1.2引入了session ticket机制

### Session Ticket

Session Ticket的工作原理很简单：

1. 服务器在获取到客户端的密钥协商参数，并生成session key之后，会将完整的会话状态(session status)进行加密，生成session ticket，注意，**这个信息只有服务器才能够解密**

2. Sever验证了客户端发送Finished消息后，通过"New session ticket"消息将session ticket发给客户端进行保存，服务器端就不再保存会话状态了，下图是这条消息所在的流程步骤：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef0ef99df2d14141a65f90d8d2c2f702~tplv-k3u1fbpfcp-watermark.image)

3. 客户端在下次发起会话时，就可以将Session Ticket放入client hello中，服务器收到后进行解密验证，并从ticket的内容中提取会话状态，用来恢复会话。

##### New Session Ticket
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16e6fd19a3284f28ac62050e50e6dc62~tplv-k3u1fbpfcp-watermark.image)

- Session Ticket LIfetime Hint: 保存时间指示，当时间到期时，客户端必须删除该ticket.
- Session Ticket: 加密后的session status信息，包含了master key，以及cipher suite，这些信息会被用来生成新的session key，保证前向安全。同时这个信息只有服务器才能解密。
- Change Cipher Spec: 告知客户端，开始使用加密传输数据。按照协议，new session ticket消息必须在服务器验证了客户端发送的finished消息后，并在自己发送的change cipher spec之前进行发送
- 按照协议，当客户端收到session ticket后，必须将之前受到过的session id删除

##### Cient Hello
在客户端在发起新会话的时候，会将上次会话收到的session ticket通过client hello发给服务器，让服务器进行解密校验.

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a21d7dac4f340d2b540a9be3d5dc810~tplv-k3u1fbpfcp-watermark.image)

客户端通过Extension: session_ticket来发送session tick，这个信息是加密的，只有服务器才能解密。

可以看到，客户端同时发送了session id。按照协议，当session id与ticket并存时，服务器会忽略session id.

##### Sever Hello
服务器在收到session ticket后，首先会发送响应server hello

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ce013bd167144a9b65a005063c3baf7~tplv-k3u1fbpfcp-watermark.image)

这条消息和平时没有什么不同，但是其中包含一个重要信息，就是Random. 

按照协议，用作恢复session的密钥，会根据session ticket中的master key + 新会话中的client random + server random生成。

所以这条server hello，实际上是把server random发给了客户端。

**安全风险**
1. 之前会话的master key是否泄密，否则新的session key也不安全。
2. 缺乏身份验证机制，无法防止中间人攻击。
3. TLS给出的方案是，恢复会话必须双方都同意，如果有一方认为会话不安全，就都需要重新走一遍完整的握手流程；另外客户端保存session ticket的有效期最长不要超过24小时；

## PSK
在TLS1.3中，引入新的session resumption机制代替了session id和ticket.

PSK--Pre-shared-key预分享密钥。工作机制如下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa9040b07c924539b9dba2e2eca6ff36~tplv-k3u1fbpfcp-watermark.image)
- 双方经过握手和密钥协商，和身份验证之后，生成对称的密钥session key。
- 在收到客户端的finished消息之后，服务器通过"new session ticket"消息，携带PSK发送给客户端，**注意这条消息是使用session key加密的，这就保证了只有通信双方才能读取里面的信息**。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bae02353862344ea8ca55250ed364c91~tplv-k3u1fbpfcp-watermark.image)
- 服务器可能会发送多个new session ticket，包含不同的PSK；
- 在建立新会话时，客户端将PSK信息放入client Hello消息中发送给服务器，报文如下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57ece96909644fc6aecdd90582e0a5b8~tplv-k3u1fbpfcp-watermark.image)

- 如果服务器验证了该PSK的真实性，则会在server hello中的pre_shared_key中将加密的PSK发送给客户端；
- 那么之前的PSK就会被用来生成新的session key，并开始加密传输数据，不需要再发送CA证书等验证消息了。
- 只要证明了双方都持有相同的PSK，不再需要证书认证，就可以证明双方的身份，因此，PSK也是一种身份认证机制。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ae96d46bf6c442c8658aac237a2a6dd~tplv-k3u1fbpfcp-watermark.image)


感谢阅读，如有不准确和错误之处请留言指正，我会立即修正，感谢！

<br/>
<br/>
<hr/>



总结不易，请勿私自转载，否则别怪老大爷不客气

欢迎喜欢技术的小伙伴和我交流，微信1296386616

参考资料：

《TLS Session Resumption》  by Anastasios Arampatzis
https://www.venafi.com/blog/tls-session-resumption

rfc5246 - IETF Tools
https://datatracker.ietf.org/doc/html/rfc5246

rfc5077 - IETF Tools
https://datatracker.ietf.org/doc/html/rfc5077

《The Transport Layer Security (TLS) Protocol Version 1.3》
https://datatracker.ietf.org/doc/html/rfc8446

《TLS 1.3科普——新特性与协议实现》 Wangrange
https://zhuanlan.zhihu.com/p/28850798
