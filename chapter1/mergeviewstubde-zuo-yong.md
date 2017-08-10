Merge和ViewStub均为Android中的xml优化标签，用于对Android的View布局进行优化。

**Merge**

merge标签应用于xml的顶层标签，主要应对于layout嵌套浪费的现象。在Android layout文件中需要一个顶级容器来容纳其他的组件，而不能直接放置多个组件，通过使用merge标签作为顶层容器，我们可以删减多余或者额外的层级，从而优化整个Android Layout的结构。

**activity\_merge\_test.xml：**

```java
<?xml version="1.0" encoding="utf-8"?>  
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  

    <TextView   
        android:id="@+id/text_view"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent"  
        android:gravity="top|center_horizontal"  
        android:text="text_view"/>  

    <TextView   
        android:id="@+id/text_view_2"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent"  
        android:gravity="bottom|center_horizontal"  
        android:text="text_view_2"/>  

</FrameLayout>
```

**MergeTestActivity：**

```java
public class MergeTestActivity extends Activity {  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_merge_test);  
    }  
}
```

效果图如下：

| ![](/assets/import1.14.3.png) |
| :---: |


由hierarchyviewer分析可得（此处只分析与Activity界面相关的部分）：

| ![](/assets/import1.14.4.png) |
| :---: |


此时已经减少了一层layout嵌套，我们通过使用merge标签，达到了优化的目的。

**ViewStub**  
ViewStub即占位符，用于处理动态觉得显示某个View的情况。在开发应用程序的时候，我们经常会在运行时动态根据条件来决定显示哪个View或某个布局。那么最通常的想法就是把可能用到的View都写在上面，先把它们的可见性都设为View.GONE，然后在代码中动态的更改它的可见性。这样的做法的优点是逻辑简单而且控制起来比较灵活。但是它的缺点就是，耗费资源。虽然把View的初始可见View.GONE但是在Inflate布局的时候View仍然会被Inflate，也就是说仍然会创建对象，会被实例化，会被设置属性。也就是说，会耗费内存等资源。而使用ViewStub的话在inflate布局的时候不会被inflate，ViewStub的inflate操作被延迟到了直到我们调用其inflate方法。

ViewStub的xml文件设置如下：

```java
<ViewStub  
    android:id="@+id/view_stub"  
    android:inflatedId="@+id/my_view"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:layout="@layout/view_view_stub" />
```

其中，inflateId属性表示ViewStub被inflate后重新被赋予的id值，layout属性指定了调用inflate方法时inflate的具体布局。

找到ViewStub并调用inflate：

```java
ViewStub viewStub = (ViewStub)findViewById(R.id.view_stub);  
viewStub.inflate();
```

值得注意的是：

1. ViewStub只能Inflate一次，之后ViewStub对象会被置为空。按句话说，某个被ViewStub指定的布局被Inflate后，就不会够再通过ViewStub来控制它了。

2. ViewStub只能用来Inflate一个布局文件，而不是某个具体的View。



