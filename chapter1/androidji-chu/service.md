> Service和Thread的区别

* Service是Android四大组件之一，运行在独立进程的主线程中，不可以执行耗时操作；Thread是CPU分配的最小执行单位，可以开启子线程执行耗时操作
* Service在不同Activity中可以获得自身实例，可以方便的对Service进行操作。Thread在不同的Activity中难以获得自身实例，如果Activity被销毁，Thread也会被销毁。

> Service有哪些启动方法，有什么区别，怎样停用Service？

在Service的生命周期中，被回调的方法比Activity少一些，只有onCreate, onStart, onDestroy,onBind和onUnbind。

通常有两种方式启动一个Service,他们对Service生命周期的影响是不一样的。

1. 通过startService

   Service会经历 onCreate 到onStart，然后处于运行状态，stopService的时候调用onDestroy方法。如果是调用者自己直接退出而没有调用stopService的话，Service会一直在后台运行。

2. 通过bindService  

   Service会运行onCreate，然后是调用onBind， 这个时候调用者和Service绑定在一起。调用者退出了，Srevice就会调用onUnbind-&gt;onDestroyed方法。

    所谓绑定在一起就共存亡了。调用者也可以通过调用unbindService方法来停止服务，这时候Srevice就会调用onUnbind-&gt;onDestroyed方法。



