测试：

依次打开3个Activity（ActivityA，ActivityB，ActivityC），并在第3个Activity中终止App

**System.exit（0）**

重写Application方法：

```java
public class MyApplication extends Application {  

    public void exit() {  
        System.exit(0);  
    }  

}
```

切记在AndroidManifest中修改使用的Application

在第三个Activity中调用该方法：

```java
public class ActivityC extends Activity {  

    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity);  
        Button button = (Button)findViewById(R.id.btn);  
        button.setText("Finish app");  
        button.setOnClickListener(new OnClickListener() {  

            @Override  
            public void onClick(View v) {  
                // 退出App  
                ((MyApplication)getApplication()).exit();  
            }  
        });  
    }  
}
```

失败，应用进程被杀死，然而过会应用重启了……并且ActivityA和ActivityB都被“复活了”，只杀死了ActivityC。

---

**android.os.Process.killProcess\(android.os.Process.myPid\(\)\)**

失败，与System.exit（0）一样，虽然杀死了进程，但过会就被重启了，并且ActivityA和ActivityB都被“复活了”。

---

**ActivityManager am= \(ActivityManager\) this.getSystemService\(Context.ACTIVITY\_SERVICE\);**

**am.killBackgroundProcesses\(this.getPackageName\(\)\);**

失败，毫无反应。

---

**自定义BaseActivity维护Activity列表**

```java
public class BaseActivity extends Activity {  
  
    // 维护一个Activity软引用的列表  
    private static List<SoftReference<Activity>> list = new ArrayList<SoftReference<Activity>>();  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        list.add(new SoftReference<Activity>(this));  
    }  
  
    @Override  
    protected void onDestroy() {  
        super.onDestroy();  
        list.remove(new SoftReference<Activity>(this));  
    }  
  
    /** 
     * 关闭所有的Activity 
     */  
    public void finishAll() {  
        for (SoftReference<Activity> sr : list) {  
            if (sr.get() != null) {  
                sr.get().finish();  
            }  
        }  
    }  
  
}  
```

对于ActivityA、ActivityB和ActivityC继承BaseActivity而不是Activity，在ActivityC中调用finishAll方法即可关闭所有Activity进而退出App。

**finishAffinity**\(\)  
直接关闭相同任务栈中的所用Activity，与上一个方法效果差不多，但是是Android自带的，方便多了。

综上所述，测试结果如下：

1. System.exit（0）：只能关闭当前Activity，关闭进程可能导致数据存储问题，不推荐

2. Android.os.Process.killProcess\(android.os.Process.myPid\(\)\)：同上

3. ActivityManager am= \(ActivityManager\) this.getSystemService\(Context.ACTIVITY\_SERVICE\);

   am.killBackgroundProcesses\(this.getPackageName\(\)\)：测试无效

4. 自定义BaseActivity维护Activity列表：可以关闭依次启动的所用Activity，进而退出整个App

5. finishAffinity\(\)：可以关闭同一个任务栈中的所有Activity，Android自带方法，比较方便



