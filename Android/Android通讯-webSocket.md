#概述
上一篇简单的认识了Socket以及他的使用，在学习过程中看到了WebSocket的身影，于是乎百度了一把，这货也可以做全双工的网络通讯，而且是html5提出来的新东西！程序员嘛！就是要对新的东西充满了好奇！

##WebSocket
引用[API](http://www.runoob.com/html/html5-websocket.html)里面的一句话，WebSocket是HTML5开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。

<i>全双工：简单理解为C,S端可以相互发送和接收数据。</i>

<b>WebSocket和Socket之间有啥关系？</b>

答：李鬼和李逵的关系，他只是名字上带有Socket，他是同Http一样使用TCP协议来传输数据的，但是和Http最大的不同就是他是全双工的。
还不明白的话看下面的解释。

<i>
WebSocket 协议本质上是一个基于 TCP 的协议。
为了建立一个 WebSocket 连接，客户端浏览器首先要向服务器发起一个 HTTP 请求，这个请求和通常的 HTTP 请求不同，包含了一些附加头信息，其中附加头信息"Upgrade: WebSocket"表明这是一个申请协议升级的 HTTP 请求，服务器端解析这些附加的头信息然后产生应答信息返回给客户端，客户端和服务器端的 WebSocket 连接就建立起来了，双方就可以通过这个连接通道自由的传递信息，并且这个连接会持续存在直到客户端或者服务器端的某一方主动的关闭连接。
</i>

他首先是在Html5中被提出来的，使用javaScript与服务端建立全双工通道。但是这套思想也有人做出了jar包提供给java使用。

我们先来看下在Html中他是如何使用的。

##在Html中的使用

要注意有些浏览器现在还不支持WebSocket，我们先用Chrom测试。

占坑

##在Android中的使用（Android 聊天室）

###Android端代码
<b>1 引入java-WebSocket依赖包</b>  

在module/build.gradle 中

```
	//WebSocket 依赖包
    compile 'org.java-websocket:Java-WebSocket:1.3.0'
```

<b>2 逻辑代码</b>

```
public class WebSocketActivity extends AppCompatActivity {

    @Bind(R.id.m_content_et)
    EditText mContentEt;
    @Bind(R.id.m_sent_bt)
    Button mSentBt;
    @Bind(R.id.m_content_tv)
    TextView mContentTv;

    private WebSocketClient mSocketClient;
    private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            mContentTv.setText(mContentTv.getText() + "\n" + msg.obj);
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_web_socket);
        ButterKnife.bind(this);
        init();
    }

    private void init() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    mSocketClient = new WebSocketClient(new URI("ws://10.27.0.197:2017/"), new Draft_10()) {
                        @Override
                        public void onOpen(ServerHandshake handshakedata) {
                            Log.d("picher_log", "打开通道" + handshakedata.getHttpStatus());
                            handler.obtainMessage(0, message).sendToTarget();
                        }

                        @Override
                        public void onMessage(String message) {
                            Log.d("picher_log", "接收消息" + message);
                            handler.obtainMessage(0, message).sendToTarget();
                        }

                        @Override
                        public void onClose(int code, String reason, boolean remote) {
                            Log.d("picher_log", "通道关闭");
                            handler.obtainMessage(0, message).sendToTarget();
                        }

                        @Override
                        public void onError(Exception ex) {
                            Log.d("picher_log", "链接错误");
                        }
                    };
                    mSocketClient.connect();

                } catch (URISyntaxException e) {
                    e.printStackTrace();
                }
            }
        }).start();


        mSentBt.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSocketClient != null) {
                    mSocketClient.send(mContentEt.getText().toString().trim());
                }
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mSocketClient != null) {
            mSocketClient.close();
        }
    }
}
```

这就好了，很简单，接下来看下服务端的代码写法

###服务端代码
<b>1 引入jar包</b>

去下载jar，获取自己配maven
[https://mvnrepository.com/artifact/org.java-websocket/Java-WebSocket/1.3.0](https://mvnrepository.com/artifact/org.java-websocket/Java-WebSocket/1.3.0)

<b>2 编写WebSocketServer</b>

```
public class MWebSocketService extends WebSocketServer {

    public MWebSocketService(int port) throws UnknownHostException {
        super(new InetSocketAddress(port));
    }

    public MWebSocketService(InetSocketAddress address) {
        super(address);
    }

    @Override
    public void onOpen(WebSocket webSocket, ClientHandshake clientHandshake) {
        String address = webSocket.getRemoteSocketAddress().getAddress().getHostAddress();
        String message = String.format("(%s) <进入房间！>", address);
        sendToAll(message);
        System.out.println(message);
    }

    @Override
    public void onClose(WebSocket webSocket, int i, String s, boolean b) {
        String address = webSocket.getRemoteSocketAddress().getAddress().getHostAddress();
        String message = String.format("(%s) <退出房间！>", address);
        sendToAll(message);

        System.out.println(message);

    }

    @Override
    public void onMessage(WebSocket webSocket, String s) {
        //服务端接收到消息
        String address = webSocket.getRemoteSocketAddress().getAddress().getHostAddress();
        String message = String.format("(%s) %s", address, s);
        //将消息发送给所有客户端
        sendToAll(message);
        System.out.println(message);
    }

    private static void print(String msg) {
        System.out.println(String.format("[%d] %s", System.currentTimeMillis(), msg));
    }

    @Override
    public void onError(WebSocket webSocket, Exception e) {
        if (null != webSocket) {
            webSocket.close(0);
        }
        e.printStackTrace();
    }

    public void sendToAll(String message) {
        // 获取所有连接的客户端
        Collection<WebSocket> connections = connections();
        //将消息发送给每一个客户端
        for (WebSocket client : connections) {
            client.send(message);
        }
    }
}
```

<b>3 编写main方法</b>

```
public class WebSocketMainMethod {

    private static int PORT = 2017;

    public static void main(String[] args) throws IOException {
        MWebSocketService socketServer = new MWebSocketService(PORT);
        socketServer.start();

        try {
            String ip = InetAddress.getLocalHost().getHostAddress();
            int port = socketServer.getPort();
            System.out.println(String.format("服务已启动: %s:%d", ip, port));
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }


        InputStreamReader in = new InputStreamReader(System.in);
        BufferedReader reader = new BufferedReader(in);

        while (true) {
            try {
                String msg = reader.readLine();
                socketServer.sendToAll(msg);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```

OK，看下成果图！

##其他

在学习的过程中也发现了不少别人封装好的库，也记录进来吧

一下内容转自 [android中使用webSocket通信](http://blog.csdn.net/u010618194/article/details/69383252)

<b>1.AndroidAsyn</b>

GitHub地址：https://github.com/koush/AndroidAsync
```
dependencies {
    compile 'com.koushikdutta.async:androidasync:2.+'
}
```

<b>2.autobahn</b>
官网：http://autobahn.ws/android/，下到jar包放项目里面就好了

使用起来同样很简单。

```
 private WebSocketConnection mConnect = new WebSocketConnection();
    String url = "ws://192.168.250.38:8181/";
    public void init() {
        try {
            mConnect.connect(url, new WebSocketHandler() {
                @Override
                public void onOpen() {
                    Log.i(TAG, "onOpen: ");
                }
                @Override
                public void onTextMessage(String payload) {
                    Log.i(TAG, "onTextMessage: "+payload);
                }
                @Override
                public void onClose(int code, String reason) {
                    Log.i(TAG, "onClose: " + code + "|" + reason);
                }
            });
        } catch (WebSocketException e) {
            e.printStackTrace();
        }
    }
```

