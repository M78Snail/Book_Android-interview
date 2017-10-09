> 1、WebView安全漏洞

* 在Java1.6之前由于存在远程执行代码漏洞，该漏洞源于没有正确使用WebView.addJavascriptInterface方法，导致远程攻击者可以利用Java反射机制利用该漏洞执行任意Java对象的方法。

> 2、WebView销毁步骤

* 当WebView嵌套于其他布局，如LinearLayout时，在销毁Activity之前，在onDestroy中先移除WebView的绑定，然后再将WebView.destroy\(\)，这样就不会导致内存泄漏。

> 3、WebView的jsbridge

* 客户端和服务端之间可以通过Javascript来互相调用各自的方法

> 4、WebViewClient的onPageFinished

* WebViewClient的onPageFinished会在页面完成加载时调用，如果遇到还未加载完就跳转到别的页面有可能会一直调用，可以使用WebChromeClient.onProgressChanged代替。

> 5、WebView后台耗电

* 在WebView加载页面的时候，会自动开启线程去加载，如果不很好的关闭这些线程，就会导致电量消耗加大，可以采用暴力的方法，直接在onDestroy方法中System.exit\(0\)结束当前正在运行中的java虚拟机

> 6、WebView硬件加速

* Android3.0引入硬件加速，默认会开启，WebView在硬件加速的情况下滑动更加平滑，性能更加好，但是会出现白块或者页面闪烁的副作用，建议WebView暂时关闭硬件加速

> 7、WebView内存泄漏

由于WebView是依附于Activity的，Activity的生命周期和WebView启动的线程的生命周期是不一致的，这会导致WebView一直持有对这个Activity的引用而无法释放，解决方案如下

* 独立进程，简单暴力，不过可能涉及到进程间通信（推荐）
* 动态添加WebView，对传入WebView中使用的Context使用弱引用



