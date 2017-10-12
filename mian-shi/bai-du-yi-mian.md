> Bitmap 使用时候注意什么？

1. 注意图片的压缩，避免出现OOM
2. 不要在UI线程中加载图片
3. 在ListView中加载图片：①使用异步加载 ②快速滑动时不加载图片
4. 使用三级缓存

> Oom 是否可以try catch ？

* 在某些情况下，我们需要事先评估那些可能发生OOM的代码，对于这些可能发生OOM的代码，加入catch机制，可以考虑在catch里面尝试一次降级的内存分配操作。例如decode bitmap的时候，catch到OOM，可以尝试把采样比例再增加一倍之后，再次尝试decode。

> 内存泄露如何产生？

1. **具有close方法的对象切记调用close方法释放资源** 常见的有Cursor和各种流等对象。博主就曾经因为Cursor对象忘记close，在进行了多次数据库查询后APP就因为OOM而崩溃了。
2. **动态注册的广播不使用后记得取消注册**  
   取消注册后可释放Context中持有的相关的广播过滤器资源。

3. **bitmap的recycle方法**  
   在Android3.0版本以前，bitmap对象的构造涉及到JVM的两块内存区域：一块是Bitmap对象所在的Java堆，一块是Bitmap对象持有的native层资源所在的Native堆。在Java堆上分配的资源由于JVM的GC机制的关系我们不需要去特别关心，但在Native堆上分配的资源就需要我们显式地调用bitmap的native方法recycle去回收了。然而在Android3.0版本后bitmap内存资源分配都在Java堆上，因此是否调用recycle方法对内存影响就不大了。

4. **listView的Adapter中注意convertView的使用**  
   在Adapter的getView方法中，有一个重要的参数就是convertView。convertView是一个View参数，它代表了一个回收的View（如果没有就为null）。Android的ListView虽然看起来有很多项，但其实真正的ItemView的数目只有屏幕上显示的哪几个，当一个ItemView从可见变为不可见时，他就会被回收掉，在下次调用getView方法时传入的convertView就是某个被回收的View。因此在getView方法中，当convertView不为null时，我们应该使用convertView而不是重新创建一个View。更高效的方式是把convertView结合一个自定义的ViewHolder一起使用，可以有效的减少findViewById的时间。

5. **非静态内部类handler**  
   Handler常见的应用场景是多线程的通信，此时要是Handler使用不当就相当容易造成内存泄露。在多线程通信的场景下，子线程通常都会持有Handler对象的引用，如果一个Handler对象在Activity中不是静态的，那么该Handler对象就会持有该Activity对象的引用，从而导致Activity组件无法正常的被GC回收，导致了内存泄露。因此一般而言我们应该把Handler对象置为Activity中的静态变量，通过软引用或者弱引用的方式来获取Activity及其内部域，避免持有Activity的强引用导致该组件无法回收。

6. **集合对象清理**  
   对于一些集合中不再使用的对象，我们应该把它们移除出我们的集合。避免由于集合持有那些对象的引用导致它们无法被JVM的GC所回收。

   还有一些内存优化的方法，个人认为有以下几个方面：

   * Java引用的灵活使用
   * 图片缓存
   * Bitmap压缩加载

> 适配器模式，装饰者模式，外观模式的异同？

* 适配器模式是让两个不相融的类打动可以互融的作用
* 装饰者模式是给一个对象动态的添加某些属性
* 外观模式模式是为各个子系统提供一个高度统一的接口给用户使用

> ANR 如何产生？

在Android里，应用程序的响应性是由Activity Manager和WindowManager系统服务监视的。当它监测到以下情况中的一个时，Android就会针对特定的应用程序显示ANR：

1. 在5秒内没有响应输入的事件（例如，按键按下，屏幕触摸）
2. BroadcastReceiver在10秒内没有执行完毕

造成以上两点的原因有很多，比如在主线程中做了非常耗时的操作，比如说是下载，io异常等。

> Stringbuffer 与Stringbuilder 的区别？

Stringbuffer线程安全，效率相对低一些，Stringbuilder线程不安全，效率相对高一些

> 如何保证线程安全？

1. synchronized
2. ThreadLoacl（本质上获取Thread的ThreadLocalMap产生一个副本）
3. ReentrantLock和Condition（模拟阻塞队列）
4. Semaphore信号量（模拟食堂打饭，3个窗口5个人打饭）
5. CyclicBarrier循环栅栏（多个线程等待，达到某个数量放行）
6. 闭锁CountDownLatch（多个线程先执行，执行数量达到一定后再执行）

> Java四中引用

强，软，弱，虚。

> Jni 用过么？
>
> 多进程场景遇见过么？
>
> sqlite升级，增加字段的语句

ALTER TABLE tableName ADD COLUMN message VARCHAR

> bitmap recycler 相关
>
> 强引用置为null，会不会被回收？
>
> glide 使用什么缓存？

* **缓存Resource**使用三层缓存，包括：

  1. 一级缓存：缓存被回收的资源，使用LRU算法（Least Frequently Used，最近最少使用算法）。当需要再次使用到被回收的资源，直接从内存返回。
  2. 二级缓存：使用弱引用缓存正在使用的资源。当系统执行gc操作时，会回收没有强引用的资源。使用弱引用缓存资源，既可以缓存正在使用的强引用资源，也不阻碍系统需要回收无引用资源。
  3. 三级缓存：磁盘缓存。网络图片下载成功后将以文件的形式缓存到磁盘中。

* **Bitmap缓存算法**

  在Glide中，使用BitmapPool来缓存Bitmap,使用的也是LRU算法。当需要使用Bitmap时，从Bitmap的池子中取出合适的Bitmap,若取不到合适的，则再新创建。当Bitmap使用完后，不直接调用Bitmap.recycler\(\)回收，而是放入Bitmap的池子。

> Glide 内存缓存如何控制大小？

> 如何保证多线程读写文件的安全？



