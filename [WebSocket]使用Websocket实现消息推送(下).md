
---
title: 「WebSocket」使用Websocket实现消息推送(下)
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - WebSocket
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x00 WebSocket

上一篇使用 [Java-WebSocket](https://github.com/TooTallNate/Java-WebSocket) 写了一套 `WebSocket` 的Demo，这一篇着重分析下` WebSocket` 的一些实现细节，更加详细的协议细节可参考 [RFC6455][rfc6455]。


## 0x01 WebSocket协议

- `WebSocket` 协议:  可参考[RFC6455][rfc6455]
- 抓包工具：`Charles` 和 `WireShark`
- 测试代码：参考上一篇Demo


## 0x02 数据传输

- 连接和断开

	当 Client 向 Server 发起 ws 连接请求后，会先使用 HTTP 协议建立 TCP 长链接，通过抓包，我们会看到类似这样的信息：
	
	Request：
	
	![Charles Request](http://img.blog.csdn.net/20160910174128237)
	
	Response：
	
	![Charles Response](http://img.blog.csdn.net/20160910175927683)

	这里简单介绍几个关键的Header字段：
	
	- upgrade:websocket
	- connection:Upgrade
	- sec-websocket-version:13
	- sec-websocket-key:xxxxxx
	- sec-websocket-accept:
	
	这些字段的信息可参考[这里](http://www.jianshu.com/p/867274a5e054)。
	
- 数据传输

	当 HTTP 连接建立之后，就可以通过这个长链接进行数据传输了。如果你使用 Charles 进行数据抓包的话，可以看到类似下图中这样的聊天内容：
	
	![Charles websocket Chat](http://img.blog.csdn.net/20160910180038193)

	 在同一个 HTTP 请求中，我们看到不断有数据交互，而不是像之前的 HTTP 请求那样，每发送一条数据，都需要发起一次新的请求。
	
	下面我们来尝试分析一下。这里要用到Wireshark。可能你会为什么换了Wireshark而不用Charles了？原因很简单，Charles给我们呈现的是应用层最终的结果，而传输层的具体的数据包发送接收却不能展示，这个时候就需要借助Wireshark来对TCP数据包进行分析了。
	
	首先，查询 [RFC6455][rfc6455]文档，我们得知 ws 协议的数据包格式如下：

	![RFC6455 Frame](http://img.blog.csdn.net/20160910181202338)
	
	有了这个参考，我们就可以进行数据分析了。下面是我的一次数据收发过程：客户端向服务端发送数据：123456789，然后服务端原样发回，抓包如下：
	
	Sender：
	
	![WireShark sender](http://img.blog.csdn.net/20160911195743125)	
	
	Receiver：

	![WireShark receiver](http://img.blog.csdn.net/20160911195848240)
	
	下面我们着重对这两个数据包进行分析。细心的你可能会发现，截图中有四条数据，如果你对TCP的传输有所了解，第二条和第三条是确认报文。
	
	我们先看下接收到数据。点击数据中的 Data(11bytes) 部分，下方是自动选择了数据部分：8109313233343536373839。（在此之前是TCP报文的Header部分，我们可以忽略）根据 ws 协议，我们将其展开：

	``` shell
	81 09:  10000001 00001001 
	31 32:  00110001 00110010
	33 34:  00110011 00110100
	35 36:  00110101 00110110
	37 38:  00110111 00111000
	39   :  00111001
	``` 	
	0：1，这是最后一帧
	1～3：全为0
	4～7：001，附加数据帧
	8：0，PlayloadData未经过掩码
	 9～15：0001001 = 9 < 125，因此数据长度位9
	 剩下的：313233343536373839即为数据 123456789

	再来看看发送的数据。点击 Data(15bytes) 部分，下方的数据部分：818911eb9db220d9ae8624ddaa8a28，展开：
	
	``` shell
	81 89 : 10000001 10001001 
	11 eb : 00010001 11101011
	9d b2 : 10011101 10110010
	20 d9 : 00100000 11011001
	ae 86 : 10101110 10000110
	24 dd : 00100100 11011101
	aa 8a : 10101010 10001010
	28    : 00101000
	```
	0：1，这是最后一帧
	1～3：全为0
	4～7：001，附加数据帧
	8：1，PlayloadData经过掩码，（所有的由客户端发往服务端的帧此数位都被设置成 1。）
	 9～15：0001001 = 9 < 125，因此数据长度位9
	 11 eb 9d b2：掩码
	 20 d9 ae 86 24 dd aa 8a 28：数据
	关于掩码的计算：
	![掩码的计算](http://img.blog.csdn.net/20160911203233422)
	
- 连接保持
	
	除了正常的数据传输之外，Socket编程还有一个重要的组成部分：心跳保持。ws协议中定义了专门的数据包：
	
	>   Control frames are identified by opcodes where the most significant bit of the opcode is 1.  Currently defined opcodes for control frames include 0x8 (Close), 0x9 (Ping), and 0xA (Pong).  Opcodes 0xB-0xF are reserved for further control frames yet to be defined.
	> Control frames are used to communicate state about the WebSocket. Control frames can be interjected in the middle of a fragmented message.
	>All control frames MUST have a payload length of 125 bytes or less and MUST NOT be fragmented.
	
- 其他
	
	一个协议不可能就这么简单，如果你还想了解更多，可参考[RFC6455][rfc6455]。


## 0xFF 参考

1. http://www.jianshu.com/p/867274a5e054
2. http://www.jianshu.com/p/fc09b0899141
3. https://datatracker.ietf.org/doc/rfc6455/
4. https://developer.mozilla.org/zh-CN/docs/WebSockets/Writing_WebSocket_servers

------
[rfc6455]: https://datatracker.ietf.org/doc/rfc6455/

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


