
## 0x00 Websocket

联系客服功能在项目中很难避免，一般有下面三种实现方式：

- 使用http的get方式轮询
- 接入第三方IM系统
- 自己的IM系统
	- 基于socket
	- 基于websocket

第一种方式，最low的，实现简单，但是浪费用户流量；第二种方式，接入简单，功能强大，但是可能需要一定的成本（比如付费）；第三种方式，需要一定的开发成本（服务器托管费用忽略）。对于第三种情况的 socket，实现IM的文字加音视频聊天，做过的话你可以也会直接懵逼。但是，简单的文字聊天还好，不过你还是需要去定义一些协议来实现这样一个功能。如果我们使用websocket，那事情就变的简单很多。

>WebSocket一种在单个 TCP 连接上进行全双工通讯的协议。WebSocket通信协议于2011年被IETF定为标准RFC 6455，并被RFC7936所补充规范，WebSocketAPI被W3C定为标准。

>WebSocket 是独立的、创建在 TCP 上的协议，和 HTTP 的唯一关联是使用 HTTP 协议的101状态码进行协议切换，使用的 TCP 端口是80，可以用于绕过大多数防火墙的限制。

>WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端直接向客户端推送数据而不需要客户端进行请求，在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并允许数据进行双向传送。

>目前常见的浏览器如 Chrome、IE、Firefox、Safari、Opera 等都支持 WebSocket，同时需要服务端程序支持 WebSocket。

[来自维基百科 https://zh.wikipedia.org/wiki/WebSocket](https://zh.wikipedia.org/wiki/WebSocket)

websocket只是一种协议，类似http，因此与语言无关。这里我使用java来做服务端，同时提供android和html的客户端，通过一个简单的demo来介绍websocket的使用，接下来的一篇会对websocket进行分析。


## 0x01 服务端

这里我使用 [Java-WebSocket](https://github.com/TooTallNate/Java-WebSocket) 这个库来实现 websocket server。需要的jar包位于项目的 `/Java-WebSocket/dist/java-websocket.jar` 位置。效果如下：

![server](http://img.blog.csdn.net/20160826172546089)

代码如下：

``` java
public class SocketServer extends WebSocketServer {
    private static final int PORT = 2333;

    public static void main(String[] args) {
        SocketServer server = new SocketServer(PORT);
        server.start();

        try {
            String ip = InetAddress.getLocalHost().getHostAddress();
            int port = server.getPort();
            print(String.format("服务已启动: %s:%d", ip, port));
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }

        InputStreamReader in = new InputStreamReader(System.in);
        BufferedReader reader = new BufferedReader(in);

        while (true) {
            try {
                String msg = reader.readLine();
                server.broadcastMessage(msg);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public SocketServer(int port) {
        super(new InetSocketAddress(port));
    }

    public SocketServer(InetSocketAddress address) {
        super(address);
    }

    @Override
    public void onOpen(WebSocket webSocket, ClientHandshake clientHandshake) {
        String address = webSocket.getRemoteSocketAddress().getAddress().getHostAddress();
        String message = String.format("(%s) <加入>", address);
        broadcastMessage(message);
        print(message);
    }

    @Override
    public void onClose(WebSocket webSocket, int code, String reason, boolean remote) {
        String address = webSocket.getRemoteSocketAddress().getAddress().getHostAddress();
        String message = String.format("(%s) <离开>", address);
        broadcastMessage(message);
        print(message);
    }

    @Override
    public void onMessage(WebSocket webSocket, String msg) {
        String address = webSocket.getRemoteSocketAddress().getAddress().getHostAddress();
        String message = String.format("(%s) %s", address, msg);
        broadcastMessage(message);
        print(message);
    }

    @Override
    public void onError(WebSocket webSocket, Exception e) {
        if (null != webSocket) {
            if (!webSocket.isClosed()) {
                webSocket.close(0);
            }
        }
        e.printStackTrace();
    }

    /**
     * 广播收到消息
     *
     * @param msg
     */
    private void broadcastMessage(String msg) {
        Collection<WebSocket> connections = connections();
        synchronized (connections) {
            for (WebSocket client : connections) {
                client.send(msg);
            }
        }
    }

    private static void print(String msg) {
        System.out.println(String.format("[%d] %s", System.currentTimeMillis(), msg));
    }
}
```


## 0x02 客户端

- Android
	
	这里也是用java-websocket这个库，效果如下：
	
	![android](http://img.blog.csdn.net/20160826172640594)

	核心代码：
	
	``` java
	public class WebSocketActivity extends BaseActivity {
    private static final int STATUS_CLOSE = 0;
    private static final int STATUS_CONNECT = 1;
    private static final int STATUS_MESSAGE = 2;

    @Bind(R.id.etIP)
    EditText etIP;
    @Bind(R.id.etPort)
    EditText etPort;
    @Bind(R.id.tvStatus)
    TextView tvStatus;
    @Bind(R.id.tvMsg)
    TextView tvMsg;
    @Bind(R.id.rgVersion)
    RadioGroup rgVersion;
    @Bind(R.id.etMessage)
    EditText etMessage;
    @Bind(R.id.svContent)
    ScrollView svContent;
    @Bind(R.id.viewMain)
    View viewMain;

    @Bind(R.id.btConnect)
    Button btConnect;
    @Bind(R.id.btDisconnect)
    Button btDisconnect;
    @Bind(R.id.btSend)
    Button btSend;

    @OnClick({R.id.btConnect, R.id.btDisconnect, R.id.btSend})
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.btConnect:
                connectToServer();
                break;
            case R.id.btDisconnect:
                if (null != mClient) {
                    mClient.close();
                }
                break;
            case R.id.btSend:
                if (null != mClient) {
                    String msg = etMessage.getText().toString();
                    if (!TextUtils.isEmpty(msg)) {
                        try {
                            mClient.send(msg);
                        } catch (NotYetConnectedException e) {
                            e.printStackTrace();
                            return;
                        }
                        etMessage.setText("");
                    }
                }
                break;

            default:
                break;
        }
    }

    private Client mClient;
    private Handler mHandle = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            String message = String.format("[%d] %s\n", System.currentTimeMillis(), msg.obj.toString());
            tvMsg.append(message);

            switch (msg.what) {
                case STATUS_CONNECT:
                    btConnect.setEnabled(false);
                    btDisconnect.setEnabled(true);
                    btSend.setEnabled(true);
                    break;
                case STATUS_CLOSE:
                    btConnect.setEnabled(true);
                    btDisconnect.setEnabled(false);
                    btSend.setEnabled(false);
                    break;
                case STATUS_MESSAGE:
                    // TODO: 16/8/24
                    break;
                default:
                    break;
            }
            svContent.postDelayed(new Runnable() {
                @Override
                public void run() {
                    svContent.fullScroll(View.FOCUS_DOWN);
                }
            }, 100);
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_web_socket);
        ButterKnife.bind(this);


        System.setProperty("java.net.preferIPv6Addresses", "false");
        System.setProperty("java.net.preferIPv4Stack", "true");
    }

    private void connectToServer() {
        String ip = etIP.getText().toString();
        String port = etPort.getText().toString();
        if (TextUtils.isEmpty(ip) || TextUtils.isEmpty(port)) {
            Snackbar.make(viewMain, "IP and Port 不能为空", Snackbar.LENGTH_LONG).show();
            return;
        }
        String address = String.format("ws://%s:%s", ip, port);
        Draft draft = null;
        switch (rgVersion.getCheckedRadioButtonId()) {
            case R.id.rbDraft10:
                draft = new Draft_10();
                break;
            case R.id.rbDraft17:
                draft = new Draft_17();
                break;
            case R.id.rbDraft75:
                draft = new Draft_75();
                break;
            case R.id.rbDraft76:
                draft = new Draft_76();
                break;

            default:
                draft = new Draft_17();
                break;
        }
        try {
            URI uri = new URI(address);
            mClient = new Client(uri, draft);
            mClient.connect();
        } catch (URISyntaxException e) {
            e.printStackTrace();
            return;
        }

        tvStatus.setText(address);
    }

    private class Client extends WebSocketClient {

        public Client(URI serverURI) {
            super(serverURI);
        }

        public Client(URI serverUri, Draft draft) {
            super(serverUri, draft);
        }

        @Override
        public void onOpen(ServerHandshake handShakeData) {
            Message msg = new Message();
            msg.what = STATUS_CONNECT;
            msg.obj = String.format("[Welcome：%s]", getURI());
            mHandle.sendMessage(msg);
        }

        @Override
        public void onMessage(String message) {
            Message msg = new Message();
            msg.what = STATUS_MESSAGE;
            msg.obj = message;
            mHandle.sendMessage(msg);
        }

        @Override
        public void onClose(int code, String reason, boolean remote) {
            Message msg = new Message();
            msg.what = STATUS_CLOSE;
            msg.obj = String.format("[Bye：%s]", getURI());
            mHandle.sendMessage(msg);
        }

        @Override
        public void onError(Exception ex) {
            ex.printStackTrace();
        }
    }
}
	```
	
- Html
	
	H5使用了这个库：[sstephenson/prototype](https://github.com/sstephenson/prototype)，效果如下：

	![html5](http://img.blog.csdn.net/20160826172728638)

	核心代码如下：
	
	``` html 
    <script type="text/javascript">
        document.observe("dom:loaded", function () {
            function log(text) {
                var value = (new Date).getTime();
                value += ": ";
                value += text.escapeHTML();
                value += $("log").innerHTML;
                $("log").innerHTML = value;
            }

            if (!window.WebSocket) {
                alert("浏览器不支持,换一个吧");
            }

            var ws;

            $("connectForm").observe("submit", function (e) {
                e.stop();
                ws = new WebSocket($F("uri"));
                ws.onopen = function () {
                    log("连接到服务器\n");
                }
                ws.onmessage = function (e) {
                    log("服务器信息: " + e.data + "\n");
                }
                ws.onclose = function () {
                    log("断开服务器连接\n");

                    $("uri", "connect").invoke("enable");
                    $("close").disable();
                    ws = null;
                }

                $("uri", "connect").invoke("disable");
                $("close").enable();
            });
            $("close").observe("click", function (e) {
                e.stop();
                if (ws) {
                    ws.close();
                    ws = null;
                }
            });
            $("sendForm").observe("submit", function (e) {
                e.stop();
                if (ws) {
                    var message = $("message");
                    ws.send(message.value);
                    message.value = "";
                    message.focus();
                }
            });

        });
    </script>
	```
	
所有的代码都可以到github查看：[ttdevs](https://github.com/ttdevs/android/tree/master/modules/webscoket)。

下一篇将通过对websocket协议的分析，来对其做进一步的了解。



## 0xFF 参考

1. https://github.com/sstephenson/prototype
2. https://github.com/TooTallNate/Java-WebSocket/
3. http://www.cnblogs.com/wlfcolin/p/5193583.html

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

