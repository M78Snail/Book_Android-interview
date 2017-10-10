## 其他优化面试题 {#其他优化面试题}

---

1、Android不用静态变量存储数据

* 静态变量等数据由于进程已经被杀死而被初始化
* 使用其他数据传输方式：文件/sp/contentProvider

2、SharePreference安全问题

* 不能跨进程同步
* 文件不宜过大

3、内存对象序列化

* Serializeble：是java的序列化方式，Serializeble在序列化的时候会产生大量的临时对象，从而引起频繁的GC
* Parcelable：是Android的序列化方式，且性能比Serializeble高，Parcelable不能使用在要将数据存储在硬盘上的情况

4、避免在UI线程中做繁重的操作

