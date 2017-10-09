> 1、AsyncTask是什么

它本质上就是一个封装了线程池和Handler的异步框架

> 2、AsyncTask使用方法

* 三个参数

  * Params：表示后台任务执行时的参数类型，该参数会传给AysncTask的doInBackground\(\)方法
  * Progress：表示后台任务的执行进度的参数类型，该参数会作为onProgressUpdate\(\)方法的参数
  * Result：表示后台任务的返回结果的参数类型，该参数会作为onPostExecute\(\)方法的参数

* 五个方法

  * onPreExecute\(\)：异步任务开启之前回调，在主线程中执行
  * doInBackground\(\)：执行异步任务，在线程池中执行
  * onProgressUpdate\(\)：当doInBackground中调用publishProgress时回调，在主线程中执行
  * onPostExecute\(\)：在异步任务执行之后回调，在主线程中执行
  * onCancelled\(\)：在异步任务被取消时回调

> 3、AsyncTask工作原理

* [Android进阶——多线程系列之异步任务AsyncTask的使用与源码分析](http://blog.csdn.net/qq_30379689/article/details/53203556)

> 4、AsyncTask引起的内存泄漏

* 原因：非静态内部类持有外部类的匿名引用，导致Activity无法释放
* 解决：
 
  * AsyncTask内部持有外部Activity的弱引用
  * AsyncTask改为静态内部类
  * AsyncTask.cancel\(\)

> 5、AsyncTask生命周期

在Activity销毁之前，取消AsyncTask的运行，以此来保证程序的稳定

> 6、AsyncTask结果丢失

由于屏幕旋转、Activity在内存紧张时被回收等情况下，Activity会被重新创建，此时，旧的AsyncTask持有旧的Activity引用，这个时候会导致AsyncTask的onPostExecute\(\)对UI更新无效

> 7、AsyncTask并行or串行

* AsyncTask在Android 2.3之前默认采用并行执行任务，AsyncTask在Android 2.3之后默认采用串行执行任务
* 如果需要在Android 2.3之后采用并行执行任务，可以调用AsyncTask的executeOnExecutor\(\)



