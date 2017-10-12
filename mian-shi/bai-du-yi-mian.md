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



