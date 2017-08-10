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

**2、下面就要开启一个线程 去不断读取服务器那边传过来的数据 采用Thread去实现**

```java
private class ReceiveThread extends Thread {  
        private byte[] buf;  
        private String str = null;  
  
        @Override  
        public void run() {  
            while (true) {  
                try {  
                    // LogUtil.e(TAG, "监听中...:"+socket.isConnected());  
                    if (socket!=null && socket.isConnected()) {  
  
                        if (!socket.isInputShutdown()) {  
                            BufferedReader inStream = new BufferedReader(  
                                    new InputStreamReader(  
                                            socket.getInputStream()));  
                            String content = inStream.readLine();                              
                            if (content == null)  
                                continue;  
                            LogUtil.e(TAG, "收到信息:" + content);  
                            LogUtil.e(TAG, "信息长度:"+content.length());  
                            if (!content.startsWith("CMD:"))  
                                continue;  
                            int spacePos = content.indexOf(" ");  
                            if (spacePos == -1)  
                                continue;  
                            String cmd = content.substring(4, spacePos);  
                            String body = content.substring(spacePos).trim();  
                            LogUtil.e(TAG, "收到信息(CMD):" + cmd);  
                            LogUtil.e(TAG, "收到信息(BODY):" + body);  
                            if (cmd.equals("LOGIN"))  
                           {  
                                // 登录  
                                ReceiveLogin(body);  
                                continue;  
                            }  
                              if (cmd.equals("KEEPLIVE")) {  
                                if (!body.equals("1")) {  
                                    Log.e(TAG, "心跳时检测到异常，重新登录!");  
                                    socket = null;  
                                    KeepAlive();  
                                } else {  
                                    Date now = Calendar.getInstance().getTime();  
                                    lastKeepAliveOkTime = now;  
                                }  
                                continue;  
                            }  
                        }  
                    } else {  
                        if(socket!=null)  
                            LogUtil.e(TAG, "链接状态:" + socket.isConnected());  
                    }  
  
                } catch (Exception e) {  
                    LogUtil.e(TAG, "监听出错:" + e.toString());  
                    e.printStackTrace();  
                }  
            }  
 }  
```

**3 、 Socket 是否断开了 断开了 需要重新去连接**

```java
public void KeepAlive() {  
        // 判断socket是否已断开,断开就重连  
        if (lastKeepAliveOkTime != null) {  
            LogUtil.e(  
                    TAG,  
                    "上次心跳成功时间:"  
                            + DateTimeUtil.dateFormat(lastKeepAliveOkTime,  
                                    "yyyy-MM-dd HH:mm:ss"));  
            Date now = Calendar.getInstance().getTime();  
            long between = (now.getTime() - lastKeepAliveOkTime.getTime());// 得到两者的毫秒数  
            if (between > 60 * 1000) {  
                LogUtil.e(TAG, "心跳异常超过1分钟,重新连接:");  
                lastKeepAliveOkTime = null;  
                socket = null;  
            }  
  
        } else {  
            lastKeepAliveOkTime = Calendar.getInstance().getTime();  
        }  
  
        if (!checkIsAlive()) {  
            LogUtil.e(TAG, "链接已断开,重新连接.");  
            connect();  
            if (loginPara != null)  
                Login(loginPara);  
        }
}  

//此方法是检测是否连接  
boolean checkIsAlive() {  
        if (socket == null)  
            return false;  
            try {  
            socket.sendUrgentData(0xFF);  
        } catch (IOException e) {  
            return false;  
        }  
        return true;  
  
}  
//然后发送数据的方法  
public void sendmessage(String msg) {  
        if (!checkIsAlive())  
            return;  
        LogUtil.e(TAG, "准备发送消息:" + msg);  
        try {  
            if (socket != null && socket.isConnected()) {  
                if (!socket.isOutputShutdown()) {  
                    PrintWriter outStream = new PrintWriter(new BufferedWriter(  
                            new OutputStreamWriter(socket.getOutputStream())),  
                            true);  
  
                    outStream.print(msg + (char) 13 + (char) 10);  
                    outStream.flush();  
                }  
            }  
            LogUtil.e(TAG, "发送成功!");  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
}  

```



