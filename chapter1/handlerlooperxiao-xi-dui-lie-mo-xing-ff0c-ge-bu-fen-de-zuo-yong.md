简单提提Handler、Looper模型中各部分的作用，主要有以下三部分：

* MessageQueue：消息队列，存储待处理的消息
* Looper：封装了消息队列与Handler，线程绑定，使用loop方法循环处理消息
* Handler：消息处理的辅助类，里面封装了消息的投递、处理和获取等一系列操作



