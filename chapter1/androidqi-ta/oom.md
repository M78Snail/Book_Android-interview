### OOM

1、什么是OOM

OOM指Out of memory（内存溢出），当前占用内存加上我们申请的内存资源超过了Dalvik虚拟机的最大内存限制就会抛出Out of memory异常

2、OOM相关概念

* 内存溢出：指程序在申请内存时，没有足够的空间供其使用
* 内存泄漏：指程序分配出去的内存不再使用，无法进行回收
* 内存抖动：指程序短时间内大量创建对象，然后回收的现象

3、解决OOM

Bitmap相关

* 图片压缩
* 加载缩略图
* 在滚动时不加载图片
* 回收Bitmap
* 使用inBitmap属性
* 捕获异常

其他相关

* listview重用convertView、使用lru
* 避免onDraw方法执行对象的创建
* 谨慎使用多进程



