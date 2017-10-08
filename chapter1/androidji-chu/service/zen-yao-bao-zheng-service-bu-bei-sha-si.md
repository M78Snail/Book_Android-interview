```
  要想使Service存活下来，我们就必须保证Service所在的进程不被杀掉，一般来说有以下方法：
```



> 1. 在onStartCommand回调方法中返回START\_STICKY，那么该进程被杀掉后系统会试图重启它
>
> 2. 设置配置文件中application的persistent属性，把应用提升为系统级别应用，免疫low memory killer  
>    **前台进程&gt;可使进程&gt;次要服务进程&gt;后台进程&gt;内容供应节点&gt;空进程**
>
> 3. 在Service的onDestroy方法中重启该Service，不过如果进程被直接杀掉这种方法就无效了
>
> 4. 通过监听特殊的系统广播（如屏幕变化、电量变化、网络变化等）去不断重启Service
>
> 5. 使用AlarmManager定时重复开启Service
>
> 6. 通过设置Service的process属性，把Service放在子进程中，避免与主进程一起被回收
>
> 7. 开启一个另外的进程与Service进程互相监视，双方要是有任意一方被杀掉则重启



