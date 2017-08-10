#### 所谓的心跳包就是客户端定时放送简单的信息给服务器端，告诉它我还在而已。代码就是每隔几分钟发送一个固定信息给服务器端，服务器端回复一个固定信息。如果服务器端几分钟后没有收到客户端信息则视客户端断开。比如有些通信软件长时间不适用，要想知道它的状态是在线还是离线，就需要心跳包，定时发包收包。

心跳包之所以叫心跳包是因为：它像心跳一样每隔固定时间发一次，以此来告诉服务器，这个客户端还活在。事实上这是为了保持长连接，至于这个包的内容，是没有什么特别规定的，不过一般都是很小的包，或者只包含包头的一个空包。

在TCP机制里面，本身是存在有心跳包机制的，也就是TCP选项:SO\_KEEPALIVE. 系统默认是设置的2小时的心跳频率。

心跳包的机制，其实就是传统的长连接。或许有的人知道消息推送的机制，消息推送也是一种长连接 ，是将数据有服务器端推送到客户端这边从而改变传统的“拉”的请求方式。下面我来介绍一下安卓和客户端两个数据请求的方式

1. push 这个也就是有服务器推送到客户端这边 现在有第三方技术 比如极光推送。
2. pull 这种方式就是客户端向服务器发送请求数据（http请求）

**1、首先服务器和客户端有一次“握手”**

```java
public void connect()  {  
        LogUtil.e(TAG, "准备链接...");  
        InetAddress serverAddr;  
        try {  
            socket = new Socket(Config.Host, Config.SockectPort);  
            _connect = true;  
            mReceiveThread = new ReceiveThread();  
            receiveStop = false;  
            mReceiveThread.start();  
            LogUtil.e(TAG, "链接成功.");  
  
        } catch (Exception e) {  
            LogUtil.e(TAG, "链接出错." + e.getMessage().toString());  
            e.printStackTrace();  
        }  
}  
```

  


　　2、pull 这种方式就是客户端向服务器发送请求数据（http请求）

  


**1、首先服务器和客户端有一次“握手”**

