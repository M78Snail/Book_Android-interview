> 1、HandlerThread产生背景

当系统有多个耗时任务需要执行时，每个任务都会开启一个新线程去执行耗时任务，这样会导致系统多次创建和销毁线程，从而影响性能。为了解决这一问题，Google提供了HandlerThread，HandlerThread是在线程中创建一个Looper循环器，让Looper轮询消息队列，当有耗时任务进入队列时，则不需要开启新线程，在原有的线程中执行耗时任务即可，否则线程阻塞

> 2、HanlderThread的特点

* HandlerThread本质上是一个线程，继承自Thread
* HandlerThread有自己的Looper对象，可以进行Looper循环，可以创建Handler
* HandlerThread可以在Handler的handlerMessage中执行异步方法
* HandlerThread优点是异步不会堵塞，减少对性能的消耗
* HandlerThread缺点是不能同时继续进行多任务处理，需要等待进行处理，处理效率较低
* HandlerThread与线程池不同，HandlerThread是一个串行队列，背后只有一个线程



