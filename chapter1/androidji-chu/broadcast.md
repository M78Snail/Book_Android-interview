> 1、Broadcast Receiver是什么

Broadcast是四大组件之一，是一种广泛运用在应用程序之间传输信息的机制，通过发送Intent来传送我们的数据

> 2、Broadcast Receiver的使用场景

* 同一App具有多个进程的不同组件之间的消息通信
* 不同App之间的组件之间的消息通信

> 3、Broadcast Receiver的种类

* 普通广播
* 有序广播
* 本地广播
* Sticky广播

> 4、Broadcast Receiver的实现

* 静态注册：注册后一直运行，尽管Activity、进程、App被杀死还是可以接收到广播
* 动态注册：跟随Activity的生命周期

> 5、Broadcast Receiver实现机制

* 自定义广播类继承Broadcast Receiver，复写onReceiver\(\)

* 通过Binder向AMS进行广播注册

* 广播发送者通过Binder向AMS发送广播

* AMS查找符合条件的广播发送到Broadcast Receiver相应的消息循环队列当中

* 消息循环队列拿到广播，回调Broadcast Receiver的onReceiver\(\)

> 6、LocalBroadcastManager特点

* 本地广播只在本App内传播，更加安全隐私，不用担心隐私泄露

* 其他App不能向你的App发送该广播，不必担心安全漏洞被泄漏

* 本地广播比全局广播更高效

* 以上三点都是源于其内部通过Handler实现的  



