# OkHttp3源码分析\[复用连接池\]

* 在okhttp中，在高层代码的调用中，使用了类似于引用计数的方式跟踪Socket流的调用，这里的计数对象是StreamAllocation，它被反复执行aquire与release操作\(点击函数可以进入github查看\)，这两个函数其实是在改变Connection中的List&lt;WeakReference&lt;StreamAllocation&gt;&gt;大小。List中Allocation的数量也就是物理socket被引用的计数（Refference Count），如果计数为0的话，说明此连接没有被使用，是空闲的，需要通过下文的算法实现回收；如果上层代码仍然引用，就不需要关闭连接。

* 遍历Deque中所有的RealConnection，标记泄漏的连接

* 如果被标记的连接满足\(空闲socket连接超过5个&&keepalive时间大于5分钟\)，就将此连接从Deque中移除，并关闭连接，返回0，也就是将要执行wait\(0\)，提醒立刻再次扫描

* 如果\(目前还可以塞得下5个连接，但是有可能泄漏的连接\(即空闲时间即将达到5分钟\)\)，就返回此连接即将到期的剩余时间，供下次清理

* 如果\(全部都是活跃的连接\)，就返回默认的keep-alive时间，也就是5分钟后再执行清理

* 如果\(没有任何连接\)，就返回-1,跳出清理的死循环

> okhttp使用了类似于引用计数法与标记擦除法的混合使用，当连接空闲或者释放时，
>
> `StreamAllocation`
>
> 的数量会渐渐变成0，从而被线程池监测到并回收，这样就可以保持多个健康的keep-alive连接，Okhttp的黑科技就是这样实现的。



# OkHttp3源码分析\[DiskLruCache\]

## 总结

1. OkHttp通过对文件进行了多次封装，实现了非常简单的I/O操作
2. OkHttp通过对请求url进行md5实现了与文件的映射，实现写入，删除等操作
3. OkHttp内部维护着清理线程池，实现对缓存文件的自动清理

# OkHttp3源码分析\[任务队列\]

通过上述的分析，我们知道了：

1. OkHttp采用Dispatcher技术，类似于Nginx，与线程池配合实现了高并发，低阻塞的运行
2. Okhttp采用Deque作为缓存，按照入队的顺序先进先出
3. OkHttp最出彩的地方就是在try/finally中调用了
   `finished`
   函数，可以主动控制等待队列的移动，而不是采用锁或者wait/notify，极大减少了编码复杂性



