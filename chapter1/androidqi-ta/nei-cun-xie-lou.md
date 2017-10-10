## 内存泄漏面试题 {#内存泄漏面试题}

---

1、Java内存泄漏引起的主要原因

长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄漏

2、Java内存分配策略

* 静态存储区：又称方法区，主要存储全局变量和静态变量，在整个程序运行期间都存在
* 栈区：方法体的局部变量会在栈区创建空间，并在方法执行结束后会自动释放变量的空间和内存
* 堆区：保存动态产生的数据，如：new出来的对象和数组，在不使用的时候由Java回收器自动回收

3、Android解决内存泄漏的例子

* 单例造成的内存泄漏：在单例中，使用context.getApplicationContext\(\)作为单例的context
* 匿名内部类造成的内存泄漏：由于非静态内部类持有匿名外部类的引用，必须将内部类设置为static
* Handler造成的内存泄漏：使用static的Handler内部类，同时在实现内部类中持有Context的弱引用
* 避免使用static变量：由于static变量会跟Activity生命周期一致，当Activity退出后台被后台回收时，static变量是不安全，所以也要管理好static变量的生命周期
* 资源未关闭造成的内存泄漏：比如Socket、Broadcast、Cursor、Bitmap、ListView等，使用完后要关闭
* AsyncTask造成的内存泄漏：由于非静态内部类持有匿名内部类的引用而造成内存泄漏，可以通过AsyncTask内部持有外部Activity的弱引用同时改为静态内部类或在onDestroy\(\)中执行AsyncTask.cancel\(\)进行修复

  


