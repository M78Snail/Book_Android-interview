Thread、AsyncTask和IntentService都与多线程有关，当我们在Android中涉及并发编程时（进行网络请求、加载较大的文件等操作）就需要使用。  
**Thread**  
Java中的子线程，可以通过传入Runnable接口或继承Thread重写run方法新建。

**AsyncTask**  
由Java线程池改造的异步任务工具

**IntentService**  
Android四大组件之一的Service默认是在主线程中运行的，IntentService是Service的子类，有如下特点：  
1. IntentService会创建单独的worker线程来处理所有的Intent请求  
2. 当所有请求处理完成后，IntentService会自动停止  
3. 为Service的onBind方法提供了默认实现，返回null  
4. 为Service的onStartCommand方法提供了默认实现，把请求Intent添加到队列中  
实现IntentService的例子如下：

```java
public class MyIntentService extends IntentService {  
      
    public MyIntentService(String name) {  
        super(name);  
    }  
  
    @Override  
    protected void onHandleIntent(Intent intent) {  
        // IntentService会创建单独的worker线程来处理此处的代码  
  
    }  
  
}  
```

我们只需重写onHandleIntent方法即可，该方法的代码会在一个子线程中运行。

