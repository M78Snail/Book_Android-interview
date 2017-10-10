## 冷启动和热启动面试题 {#冷启动和热启动面试题}

---

1、什么是冷启动和热启动

* 冷启动：在启动应用前，系统中没有该应用的任何进程信息
* 热启动：在启动应用时，在已有的进程上启动应用（用户使用返回键退出应用，然后马上又重新启动应用）

2、冷启动和热启动的区别

* 冷启动：创建Application后再创建和初始化MainActivity
* 热启动：创建和初始化MainActivity即可

3、冷启动时间的计算

这个时间值从应用启动（创建进程）开始计算，到完成视图的第一次绘制为止

4、冷启动流程

* Zygote进程中fork创建出一个新的进程
* 创建和初始化Application类、创建MainActivity
* inflate布局、当onCreate/onStart/onResume方法都走完
* contentView的measure/layout/draw显示在界面上

总结：Application构造方法-&gt;attachBaseContext\(\)-&gt;onCreate\(\)-&gt;Activity构造方法-&gt;onCreate\(\)-&gt;配置主题中背景等属性-&gt;onStart\(\)-&gt;onResume\(\)-&gt;测量布局绘制显示在界面上

5、冷启动优化

* 减少第一个界面onCreate\(\)的工作量
* 不要在Application方法中以静态变量方式保存数据
* 不要在Application中参与业务操作
* 不要在Application进行耗时操作
* 减少布局的复杂性和深度
* 不要在主线程中加载资源
* 通过懒加载方式加载第三方SDK



