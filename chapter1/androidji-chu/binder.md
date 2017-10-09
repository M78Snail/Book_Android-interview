> 一.Linux内核基本知识

* 进程隔离/虚拟地址空间：进程间是不可以共享数据的，相当于被隔离，每个进程分配不同的虚拟地址中
* 系统调用：Linux内核对应用有访问的权限，用户只能在应用层通过系统调用，调用系统中的某些程序
* binder驱动：它负责各个用户的进程，通过binder通信内核来进行交互的模块

> 二.为什么使用Binder

* 性能上，相比传统的Socket性能更高
* 安全性高，支持协议双方相互校验

> 三.Binder通信模型

![](/assets/import_Binder.png)

* Service端通过Binder驱动在ServiceManager的查找表中注册object对象的add方法
* Client端通过Binder驱动在ServiceManager的查找表中查找object对象的add方法，并返回proxyd的add方法
* proxy的add方法是一个空方法，proxy也不是一个真正Object对象，是通过Binder封装的一个代理对象
* Client端调用add方法，Client端通过Binder驱动将proxy对象的add方法来请求ServiceManager来找到Service端真正的add方法

> 四.AIDL

* 客户端通过AIDL的Stub.asInterface\(\)方法，拿到Proxy代理类
* 通过调用Proxy代理类的方法，将参数封包后，调用底层transact\(\)方法
* transact\(\)方法会回调onTransact\(\)方法进行参数的解封
* 在onTransact\(\)方法中调用服务端的方法，并将结果返回



