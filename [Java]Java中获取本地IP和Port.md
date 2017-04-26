
## 0x00 问题

项目中要用到DatagramSocket，同时也要获取本地的IP和PORT，直接创建并获取端口获取的IP总是0.0.0.0，代码如下：

``` java 
private static void testGetDatagramSocket() {
	try {
		DatagramSocket socket = new DatagramSocket();
		System.out.println(socket.getLocalSocketAddress());
		socket.close();
	} catch (SocketException e) {
		e.printStackTrace();
	}
}
``` 

输出结果：

`0.0.0.0/0.0.0.0:55816`


## 0x01 解决

查了些资料，比如查询可用端口并绑定，没有现成方法，好像实现不了。个中原因也没太多时间去细究，好像是还没和网卡关联（纯属个人YY）。由于项目对本地端口没有特别
要求，自己写了一个方法获取DatagramSocket，如下：

``` java
/**
 * 从最大端口开始向下遍历，有可能端口就返回
 *
 * @return DatagramSocket, 为null的可能性极小
 */
public static DatagramSocket getDatagramSocket() {
    DatagramSocket socket = null;
    int port = 65535;
    while (port > 0) {
        try {
            socket = new DatagramSocket(new InetSocketAddress(InetAddress.getLocalHost(), --port));
            System.out.println(port);
            break;
        } catch (SocketException e) {
            e.printStackTrace();
            continue;
        } catch (UnknownHostException e) {
            e.printStackTrace();
            continue;
        }
    }
    return socket;
}
```
  
出现异常继续运行，代价使用者自行斟酌。


## 0x02 测试

``` java
public class MainTest {

    public static void main(String[] args) {
        testGetDatagramSocket();
        testGetUDPSocketInfo();
        testGetLocalHost();
    }

    private static void testGetDatagramSocket() {
        try {
            DatagramSocket socket = new DatagramSocket();
            System.out.println(socket.getLocalSocketAddress());
            socket.close();
        } catch (SocketException e) {
            e.printStackTrace();
        }
    }

    private static void testGetUDPSocketInfo() {
        DatagramSocket socket = getDatagramSocket();
        System.out.println(socket.getLocalSocketAddress());
        socket.close();
    }

    /**
     * 从最大端口开始向下遍历，有可能端口就返回
     *
     * @return DatagramSocket, 为null的可能性极小
     */
    public static DatagramSocket getDatagramSocket() {
        DatagramSocket socket = null;
        int port = 65535;
        while (port > 0) {
            try {
                socket = new DatagramSocket(new InetSocketAddress(InetAddress.getLocalHost(), --port));
                System.out.println(port);
                break;
            } catch (SocketException e) {
                e.printStackTrace();
                continue;
            } catch (UnknownHostException e) {
                e.printStackTrace();
                continue;
            }
        }
        return socket;
    }

    private static void testGetLocalHost() {
        try {
            InetAddress address = InetAddress.getLocalHost();
            System.out.println(address.getHostAddress());
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
    }
}
```
    

  
  

  

