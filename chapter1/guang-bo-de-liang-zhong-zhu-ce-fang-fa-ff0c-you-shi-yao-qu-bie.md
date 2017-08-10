Android四大组件之一的broadcast（广播）拥有两种不同的注册方法：静态注册与动态注册

**静态注册**

广播静态注册指把广播相应的信息写在AndroidManifest.xml中，例子如下：

```java
<receiver android:name=".StaticReceiver">    
   <intent-filter>    
      <action android:name="XXX" />   
   </intent-filter>    
</receiver>  
```

静态广播的好处在于：不需程序启动即可接收，可用作自动启动程序

---

**动态注册**

广播动态注册指在程序中调用registerReceiver方法来注册广播，例子如下：

```java
IntentFilter filter = new IntentFilter();  
filter.addAction("XXX");  
DynamicReceiver receiver = new DynamicReceiver();  
registerReceiver(receiver,  filter);  
// 不使用后记得取消注册  
unregisterReceiver(receiver);  
```

动态广播的好处在于：程序适应系统变化做操作，但在程序运行状态才能接收到

  


