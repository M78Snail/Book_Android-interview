## 内存管理面试题 {#内存管理面试题}

---

1、Android内存管理机制

* 分配机制
* 管理机制

2、内存管理机制的特点

* 更少的占用内存
* 在合适的时候，合理的释放系统资源
* 在系统内存紧张的时候，能释放掉大部分不重要的资源
* 能合理的在特殊生命周期中，保存或还原重要数据

3、内存优化方法

* Service完成任务后应停止它，或用IntentService（因为可以自动停止服务）代替Service
* 在UI不可见的时候，释放其UI资源
* 在系统内存紧张的时候，尽可能多的释放非重要资源
* 避免滥用Bitmap导致内存浪费
* 避免使用依赖注入框架
* 使用针对内存优化过的数据容器
* 使用ZIP对齐的APK
* 使用多进程



