> 1、IntentService是什么

IntentService是继承自Service并处理异步请求的一个类，其内部采用HandlerThread和Handler实现的，在IntentService内有一个工作线程来处理耗时操作，其优先级比普通Service高。当任务完成后，IntentService会自动停止，而不需要手动调用stopSelf\(\)。另外，可以多次启动IntentService，每个耗时操作都会以工作队列的方式在IntentService中onHandlerIntent\(\)回调方法中执行，并且每次只会执行一个工作线程

> 2、IntentService使用方法

* 创建Service继承自IntentService
* 覆写构造方法和onHandlerIntent\(\)方法
* 在onHandlerIntent\(\)中执行耗时操作



