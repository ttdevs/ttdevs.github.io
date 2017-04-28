
---
title: 「Java」Java中TCP和UDP总结
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Java
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x00 还是先说点啥

之前写过一些关于TCP和UDP数据传输的代码，比如使用TCP传输音视频数据包，P2P打洞中使用UDP等。写好之后就直接丢下了，没有总结下都。最近准备找工作，再拿来温习下。

暂时把自己的定位很明确，就是android应用层的开发，所以关于TCP/UDP的实现细节，暂时也不想去深究。但是心里清楚这个必须去看的，有时间推荐大家看看《TCP/IP详解》或者网上有很多大牛的总结。  


## 0x01 TCP

不知道为什么，这个总结不想写的太细，不贴代码写的细又不知道能总结啥，好纠结，可能就是认识有限吧，公司要是有个架构就好了。不说了，还是安安稳稳的写总结吧。TCP这个可能是我们用的比较多的或者说我用的比较多的，主要的工作还是进行大量数据的传输和心跳保持（想想去年面试的时候都不知道啥是心跳，汗……）。对于心跳保持，就是一个简单的小心跳包；大量数据的传输，这个也总结不了啥东西，代码就那样，就是一些细节说一下。

### 客户端

这个是我们关注的最多的，也是作为一个手机APP主要关注的，因为我们就是client。

#### 创建client
    
`new Socket(ip, port);`

我们可以通过上面的方式创建一个socket，如果失败，会抛出IOException。参数中的IP和Port是目标服务器的IP和端口号。若你想得到本地的IP和PORT也可通过这个socket拿到。当然，创建socket还有多种方法，比如 `new Socket(proxy)`，如果有需要你可以查阅相关资料。

拿到socket对象之后我们接着要做的事情就是从这个socket中拿到输入和输出流，这样我们才可以进行数据的收发：

``` java
InputStream is = mSocket.getInputStream();
OutputStream out = mSocket.getOutputStream();
```

通过输出流，我们可以使用 is.read(receiveBuffer) 和 out.write(data); 来进行数据的收发。这里给两个简单实例：

#### 接收数据

  ``` java
    @Override
    public void execute() {
        try {
            int count = is.read(receiveBuffer);
            if (count == -1) {
                notifyError();
            }
            byte[] data = getPacket(receiveBuffer, count, is);
            mReceiverQueue.put(data);
        } catch (InterruptedException e) {
            e.printStackTrace();
            notifyError();
        } catch (IOException e) {
            e.printStackTrace();
            notifyError();
        }
    }
  ```

假如我们的数据包协议格式如下：  

![](http://img.blog.csdn.net/20140222112122625?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

这个时候，如果进行大量数据传输的时候我们从InputStream中一次读取的数据可能不是一个完整的数据包。这个时候我们需要做下面的判断：

a、读取的数据长度是否达到包头大小，若没有包头长，继续读，直到达到或者超过包头大小
b、从包头中解出长度字段，循环读取，直到读到长度字段标示长度为止
c、校验数据，去除包头，取出包体
d、重复上述步骤

#### 发送数据
    
``` java
@Override
public void execute() {
	try {
		byte[] data = mSenderQueue.take();
		out.write(data);
	} catch (InterruptedException e) {
		e.printStackTrace();
		notifyError();
	} catch (IOException e) {
		e.printStackTrace();
		notifyError();
	}
}
```

mSenderQueue和mReceiverQueue一样，都是阻塞队列。发送的时候，我们从队列中取出要发送的数据，然后通过输出流写入即可。这个地方比发送简单了一些。可能你已经注意到，我的发送和接收都用了阻塞队列，这个原因就是考虑到大量数据的时候做一个缓冲，如果没有缓冲，可能导致代码的阻塞。另外就是当我们的阻塞队列充满的时候可以手动丢弃一些数据，这个就是具体应用了。  

### 服务端

这个可以简单了解下，可以使用各种语言实现，比如Java、C++等。这里简单介绍Java的实现。

首先，我们创建一个ServerSocket对象并指定一个端口号，通过这个对象的accept()方法等待客户的连接，这个方法是阻塞的；当有客户端连接上来之后，我们就拿到了连接的socket，拿到这个socket之后的操作就和客户端一样了，最后看一下关键代码：

``` java
try {
	ServerSocket serverSocket = new ServerSocket(9559);
	while (true) {
		Socket socket = serverSocket.accept();
		// new ServiceSocketThread(socket).start();
	}
} catch (Exception e) {
	e.printStackTrace();
}
```

因为服务端不可能只与一个客户端连接，因此上面的代码写在一个死循环中。拿到socket之后起一个新的线程来处理这个socket。当然，实际情况可能不是这样，为每一个连接创建一个新的线程是很耗费资源的。

最后，还有一点需要注意的是：从Socket中拿到InputStream和OutputStream之后如果关闭它们，socket也将随之关闭(未验证)。


## 0x02 UDP

这个东西用的很少，就是当初测试P2P的时候用过。能想到的问题就是数据大小的问题，比如发送数据我们的数据定义为多大合适。但是最后没有实际的项目验证，在此也不好回答。先贴一段代码出来：

``` java
public class UDPServer extends BaseThread {

    /**
     * 发送队列大小
     */
    public static final int SENDQUEUESIZE = 10;
    /**
     * 接收队列大小
     */
    public static final int RECEIVEQUEUESIZE = 10;
    /**
     * UDP接收缓存大小
     */
    public static final int RECEIVERBUFFERSIZE = 1024;
    /**
     * UDP接收缓存大小
     */
    public static final int RECEIVERPACKETSIZE = 1024 * 64;

    private int count = 0;
    private DatagramPacket receivePacket;
    private DatagramSocket mSocket;

    @Override
    public boolean prepare() {
        receivePacket = new DatagramPacket(new byte[RECEIVERPACKETSIZE], RECEIVERPACKETSIZE);
        try {
            mSocket = new DatagramSocket(9559);
        } catch (SocketException e) {
            System.out.println(String.format("udp connect init error: %s", e.getMessage()));
            return false;
        }
        return true;
    }

    @Override
    public void execute() {
        try {
            mSocket.receive(receivePacket);

            byte[] data = receivePacket.getData();
            int length = receivePacket.getLength();
            int offset = receivePacket.getOffset();
            System.out.print(++count
                    + String.format("length:%d|%d, offset:%d, data: %s \n", length, data.length, offset, new String(data, "gbk")));
            System.out.println(data[1024]);
        } catch (SocketException e) {
            System.out.println(String.format("udp connect init error: %s", e.getMessage()));
        } catch (IOException e) {
            System.out.println(String.format("udp connect init error: %s", e.getMessage()));
        }
    }
}
```

`byte[] data = receivePacket.getData();` 

这个地方拿到的data是缓冲区的大小，他们地址是一样的，这个大家可以试试就知道。至于这个data中有多少数据，就需要我们通过 `receivePacket.getLength();` 拿到。

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


