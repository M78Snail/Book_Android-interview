* 通过Binder对象

> 当Activity通过调用bindService\(Intent service, ServiceConnection conn,int flags\),我们可以得到一个Service的一个对象实例，然后我们就可以访问Service中的方法

```
**新建一个Service类**
```

```java
public class MsgService extends Service {  
    /** 
     * 进度条的最大值 
     */  
    public static final int MAX_PROGRESS = 100;  
    /** 
     * 进度条的进度值 
     */  
    private int progress = 0;  

    /** 
     * 增加get()方法，供Activity调用 
     * @return 下载进度 
     */  
    public int getProgress() {  
        return progress;  
    }  

    /** 
     * 模拟下载任务，每秒钟更新一次 
     */  
    public void startDownLoad(){  
        new Thread(new Runnable() {  

            @Override  
            public void run() {  
                while(progress < MAX_PROGRESS){  
                    progress += 5;  
                    try {  
                        Thread.sleep(1000);  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  

                }  
            }  
        }).start();  
    }  


    /** 
     * 返回一个Binder对象 
     */  
    @Override  
    public IBinder onBind(Intent intent) {  
        return new MsgBinder();  
    }  

    public class MsgBinder extends Binder{  
        /** 
         * 获取当前Service的实例 
         * @return 
         */  
        public MsgService getService(){  
            return MsgService.this;  
        }  
    }  
}
```

```
Intent intent = new Intent("com.example.communication.MSG_ACTION");    
bindService(intent, conn, Context.BIND_AUTO_CREATE);
```

> 上面的代码我们就在Activity绑定了一个Service,上面需要一个ServiceConnection对象，它是一个接口，我们这里使用了匿名内部类

```
ServiceConnection conn = new ServiceConnection() {  

        @Override  
        public void onServiceDisconnected(ComponentName name) {  

        }  

        @Override  
        public void onServiceConnected(ComponentName name, IBinder service) {  
            //返回一个MsgService对象  
            msgService = ((MsgService.MsgBinder)service).getService();  

        }

}
```

> 我们每次都要主动调用getProgress\(\)来获取进度值，然后隔一秒在调用一次getProgress\(\)方法，你会不会觉得很被动呢？可不可以有一种方法当Service中进度发生变化主动通知Activity，答案是肯定的，我们可以利用回调接口实现Service的主动通知

```
/** 
     * 注册回调接口的方法，供外部调用 
     * @param onProgressListener 
     */  
    public void setOnProgressListener(OnProgressListener onProgressListener) {  
        this.onProgressListener = onProgressListener;  
    }  

    ...activity中
   /** 
     * 模拟下载任务，每秒钟更新一次 
     */  
    public void startDownLoad(){  
        new Thread(new Runnable() {  

            @Override  
            public void run() {  
                while(progress < MAX_PROGRESS){  
                    progress += 5;  

                    //进度发生变化通知调用方  
                    if(onProgressListener != null){  
                        onProgressListener.onProgress(progress);  
                    }  

                    try {  
                        Thread.sleep(1000);  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  

                }  
            }  
        }).start();  
    }  
    ...
ServiceConnection conn = new ServiceConnection() {  
        @Override  
        public void onServiceDisconnected(ComponentName name) {  

        }  

        @Override  
        public void onServiceConnected(ComponentName name, IBinder service) {  
            //返回一个MsgService对象  
            msgService = ((MsgService.MsgBinder)service).getService();  

            //注册回调接口来接收下载进度的变化  
            msgService.setOnProgressListener(new OnProgressListener() {  

                @Override  
                public void onProgress(int progress) {  
                    mProgressBar.setProgress(progress);  

                }  
            });  

        }  
    };
```

* 通过broadcast\(广播\)的形式

> Service向Activity发送消息，可以使用广播，当然Activity要注册相应的接收器。比如Service要向多个Activity发送同样的消息的话，用这种方法就更好



