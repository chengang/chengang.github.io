---
title: Websocket协议rfc阅读散记
date: 2014-12-16T18:40:55+00:00

---
websocket协议(rfc6455)见此地址: tools.ietf.org/html/rfc6455
  
记录几个觉得比较有意思的点如下：

**1、双工通信**
  
双工通过把数据切为多个frame实现；
  
数据收完了回去的时候带上自己想发的数据。

**2、Control frame**
  
Control frame可以插在Data frame之间发送，标准要求接收方优先处理Control frame；
  
Control frame只包括三个：Close、Ping、Pong。

**3、可以主动发pong**
  
在对面没有发ping的时候，可以主动声明自己依然存活。

**4、Mask**
  
每个frame中必须包含一个key，payload必须用这个key做mask。
  
每个frame的key必须不相同且无法预测；
  
Server端响应时，必须不做mask。
  
（标准中提到这样设计是安全考虑。
  
主要是防止cache污染攻击：
  
攻击者用一对自己控制的Server和Client，通过伪造origin/host和响应的方式达到污染cache的目的。
  
这个设计主要是为网络上现存的很多实现不完善的无法分辨websocket协议的HTTP设施而考虑。）

**5、防syn flood**
  
同样2个的host和port之间，不允许并发两个CONNECTING（正在连接）状态；
  
但允许多个连接同时存在，比如浏览器开了多个Tab的情况；
  
标准中说这个设计是为了防止syn flood类的攻击。确实，如果没有建立连接就直接拒绝后续的syn的话，的确可以达到一定的防守效果。

**6、断线重连**
  
意外关闭后重连时，每次重试的间隔时间交由客户端决定。
  
建议随着重试失败次数增加，间隔越来越长。

**7、扩展支持**
  
协议设计中支持用户在协议的同一层扩展协议：
  
a>有extension字段供用户声明自己的协议；
  
b>在opcode、playload data等多个地方留下了自定义空间；
  
c>留下了status code 3000-3999用于公有扩展，status code 4000-4999用于私有扩展（不许注册）。
  
（但貌似无法改变1对1的传输结构）

**8、平台限制**
  
记得要遵守平台的限制，比如说不要被一个超级大frame撑爆内存。

**9、身份认证**
  
websocket不提供任何身份认证机制，但可以使用HTTP的basic auth、TLS认证或者基于cookies的方式；

**10、安全传输**
  
一个是origin头的机制。
  
传输加密通过TLS实现；

**11、错误处理**
  
任何非法数据都应该直接导致断开TCP连接。如果是在握手阶段收到的非法数据，可以回一个合适的HTTP response。

**12、opcode列表**

<pre>|Opcode  | Meaning                             | Reference |
    -+--------+-------------------------------------+-----------|
     | 0      | Continuation Frame                  | RFC 6455  |
    -+--------+-------------------------------------+-----------|
     | 1      | Text Frame                          | RFC 6455  |
    -+--------+-------------------------------------+-----------|
     | 2      | Binary Frame                        | RFC 6455  |
    -+--------+-------------------------------------+-----------|
     | 8      | Connection Close Frame              | RFC 6455  |
    -+--------+-------------------------------------+-----------|
     | 9      | Ping Frame                          | RFC 6455  |
    -+--------+-------------------------------------+-----------|
     | 10     | Pong Frame                          | RFC 6455  |
    -+--------+-------------------------------------+-----------|
</pre>

**13、URI的样子**

<pre>ws-URI = "ws:" "//" host [ ":" port ] path [ "?" query ]
 wss-URI = "wss:" "//" host [ ":" port ] path [ "?" query ]
</pre>

**14、frame结构**

<pre>0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-------+-+-------------+-------------------------------+
     |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
     |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
     |N|V|V|V|       |S|             |   (if payload len==126/127)   |
     | |1|2|3|       |K|             |                               |
     +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
     |     Extended payload length continued, if payload len == 127  |
     + - - - - - - - - - - - - - - - +-------------------------------+
     |                               |Masking-key, if MASK set to 1  |
     +-------------------------------+-------------------------------+
     | Masking-key (continued)       |          Payload Data         |
     +-------------------------------- - - - - - - - - - - - - - - - +
     :                     Payload Data continued ...                :
     + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
     |                     Payload Data continued ...                |
     +---------------------------------------------------------------+
</pre>

**15、另外学了一个之前漏掉的关于HTTP的小知识**
  
Ergo, the following are
     
equivalent:

Sec-WebSocket-Extensions: foo
           
Sec-WebSocket-Extensions: bar; baz=2

is exactly equivalent to

Sec-WebSocket-Extensions: foo, bar; baz=2
