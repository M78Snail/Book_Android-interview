> Service和Thread的区别

* Service是Android四大组件之一，运行在独立进程的主线程中，不可以执行耗时操作；Thread是CPU分配的最小执行单位，可以开启子线程执行耗时操作
* Service在不同Activity中可以获得自身实例，可以方便的对Service进行操作。Thread在不同的Activity中难以获得自身实例，如果Activity被销毁，Thread也会被销毁。



