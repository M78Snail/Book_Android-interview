> 1、Handler是什么

* Handler作为消息的主要处理者，在MessageQueen和Looper之间的桥梁，一方面Handler将Message发送到消息队列中，另一方面又作为消息的接收处理者。

> 2、Handler使用方法

* post\(runnable\)

* sendMessage\(message\)

> 3、Android通信机制中Message、Handler、MessageQueen、Looper的之间的关系？

MessageQueen是一个消息循环队列，可以存贮Handler发送过来的Message消息，内部是由一个单项的链表构成的队列，维护着进队和出队的操作，对应enqueueMessage\(\) 和next\(\)方法。

Message是一个Bean对象，其中的属性用来记录各种Message信息。

Looper是一个循环器，它不停的取出MessageQueen中的消息，最重要的prepare\(\)和loop\(\)方法。在prepare\(\)方法中会进行Looper的初始化会关联一个MessageQueen，并将Looper保存在ThreadLocal中，保证一个线程中只有唯一一个Looper对象；在loop\(\)方法中使用MessagQueen的next\(\)方法取出Message，判断是否为空，如果为空则阻塞，不为空，则将Message分发给它关联的Handler对象的handlerMessage\(\)方法。

> 4、Handler引起的内存泄漏

* 原因：非静态内部类持有外部类的匿名引用，导致Activity无法释放
* 解决：
  * Handler内部持有外部Activity的弱引用
  * Handler改为静态内部类
  * Handler.removeCallback\(\)



