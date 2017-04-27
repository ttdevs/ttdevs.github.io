
## 0x00 心跳

本来以为写完了，结果最近和一个同事在讨论心跳的事情，这里再做一个补充。先说我的结论：

- WebSocket协议已经设计了心跳，这个功能可以到达检测链接是否可用
- 心跳是用来检测链接是否可用的，不一定支持携带数据，可要看具体实现
- 如果非要心跳中带上复杂数据，那这个可作为应用层的一个功能自己去实现。

![心跳逻辑](http://img.blog.csdn.net/20170317140005670?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## 0x01 WebSocket协议的控制帧

上一篇的最后简单提到了心跳，下面是对websocket协议控制帧的描述：

    5.5.  Control Frames
    
       Control frames are identified by opcodes where the most significant
       bit of the opcode is 1.  Currently defined opcodes for control frames
       include 0x8 (Close), 0x9 (Ping), and 0xA (Pong).  Opcodes 0xB-0xF are
       reserved for further control frames yet to be defined.
    
       Control frames are used to communicate state about the WebSocket.
       Control frames can be interjected in the middle of a fragmented
       message.
    
       All control frames MUST have a payload length of 125 bytes or less
       and MUST NOT be fragmented.
    

- Ping的协议头是0x9，Pong的协议头是0xA
- 控制帧最大载荷为125bytes且不能拆分


## 0x02 WebSocket协议的心跳

下面再来看看对心跳的规定：

    5.5.2.  Ping
    
       The Ping frame contains an opcode of 0x9.
    
       A Ping frame MAY include "Application data".
       // 注：Ping帧中可能会携带数据
    
       Upon receipt of a Ping frame, an endpoint MUST send a Pong frame in
       response, unless it already received a Close frame.  It SHOULD
       respond with Pong frame as soon as is practical.  Pong frames are
       discussed in Section 5.5.3.
       // 注：在收到Ping帧后，端点必须发送Pong帧响应，除非已经收到了Close帧。在实际中应尽可能快的响应。
    
       An endpoint MAY send a Ping frame any time after the connection is
       established and before the connection is closed.
    
       NOTE: A Ping frame may serve either as a keepalive or as a means to
       verify that the remote endpoint is still responsive.
    
    5.5.3.  Pong
    
       The Pong frame contains an opcode of 0xA.
    
       Section 5.5.2 details requirements that apply to both Ping and Pong
       frames.
    
       A Pong frame sent in response to a Ping frame must have identical
       "Application data" as found in the message body of the Ping frame
       being replied to.
       // 注：在响应Ping帧的的Pong帧中，必须携和被响应的Ping帧中相同的数据。
       
       If an endpoint receives a Ping frame and has not yet sent Pong
       frame(s) in response to previous Ping frame(s), the endpoint MAY
       elect to send a Pong frame for only the most recently processed Ping
       frame.

从上面的描述我们可以得到如下结论：

- 心跳包中可能会携带数据
- 当收到Ping帧的时候需要立即返回一个Pong帧
- 在连接建立之后，随时都可以发送Ping帧
- 心跳是用来测试链接是否存在和对方是否在线
- 在响应Ping帧的的Pong帧中，必须携和被响应的Ping帧中相同的数据


## 0x03 测试

和之前一样，自己本地搭建的服务器，用的库是 `org.java_websocket`。
在其源码中我们可以找到这样一段：

``` java
package org.java_websocket;

public abstract class WebSocketAdapter implements WebSocketListener {
    ...

    public void onWebsocketPing(WebSocket conn, Framedata f) {
        FramedataImpl1 resp = new FramedataImpl1(f);
        resp.setOptcode(Opcode.PONG);
        conn.sendFrame(resp);
    }

    public void onWebsocketPong(WebSocket conn, Framedata f) {
    }

    ...
}
```

客户端也可以使用这个库，相同的逻辑，代码也是这一份。

然后我们再换一个库，`com.squareup.okhttp3:okhttp-ws:3.4.2`，他的实现如下：

``` java

package okhttp3.internal.ws;

public abstract class RealWebSocket implements WebSocket {
  ...
  public RealWebSocket(boolean isClient, BufferedSource source, BufferedSink sink, Random random,
      final Executor replyExecutor, final WebSocketListener listener, final String url) {
    this.listener = listener;

    writer = new WebSocketWriter(isClient, sink, random);
    reader = new WebSocketReader(isClient, source, new FrameCallback() {
      @Override public void onMessage(ResponseBody message) throws IOException {
        listener.onMessage(message);
      }

      @Override public void onPing(final Buffer buffer) {
        replyExecutor.execute(new NamedRunnable("OkHttp %s WebSocket Pong Reply", url) {
          @Override protected void execute() {
            try {
              writer.writePong(buffer);
            } catch (IOException ignored) {
            }
          }
        });
      }

      @Override public void onPong(Buffer buffer) {
        listener.onPong(buffer);
      }

      @Override public void onClose(final int code, final String reason) {
        readerSentClose = true;
        replyExecutor.execute(new NamedRunnable("OkHttp %s WebSocket Close Reply", url) {
          @Override protected void execute() {
            peerClose(code, reason);
          }
        });
      }
    });
  }
...
}
```

在处理Ping帧的时候，也是将协议字段改为Pong然后返回。

对心跳的测试代码已经上传到Github：[ttdevs](https://github.com/ttdevs)
[WebSocketActivity.java](https://github.com/ttdevs/android/blob/master/app/src/main/java/com/ttdevs/android/WebSocketActivity.java)
[WebSocketOKActivity.java](https://github.com/ttdevs/android/blob/master/app/src/main/java/com/ttdevs/android/WebSocketOKActivity.java)
[SocketServer.java](https://github.com/ttdevs/android/blob/master/modules/webscoket/src/main/java/com/ttdevs/webscoket/SocketServer.java)

在实际的测试中，可能会遇到一些异常，比如在我们自己的生产环境：当客户端发送带了简单数据的Ping帧后，服务器立马返回Pong帧，但是它会将携带的数据丢弃。这个就是服务端的问题了。


## 0xFF 参考

1. RFC6455: https://datatracker.ietf.org/doc/rfc6455/

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


