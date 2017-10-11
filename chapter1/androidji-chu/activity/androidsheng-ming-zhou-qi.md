### Activity的生命周期如下图所示： {#toc_1}

| ![](/assets/import.png) |
| :---: |


| 方法 | 操作 |
| :---: | :---: |
| onCreate\(\) | 初始化操作 |
| onStart\(\) | 在可见之前被调用 |
| onResume\(\) | 可见状态，位于栈顶 |
| onPause\(\) | 处理一些消耗cpu操作，保存关键数据 |

特殊情况分析：

1. 针对一个特定Activity启动：onCreate\(\)-&gt;onStart\(\)-&gt;onResume\(\)
2. 切换到新Activity或桌面：onPause\(\)-&gt;onStop\(\),若新Activity采用透明主题，不会调用onStop\(\);
3. onStart和onStop是从是否可见的角度来回调的，onResume和onPause是从是否位于栈顶来考虑的。

异常情况下的生命周期分析：

1. 系统发生一些异常现象（如：旋转屏幕），所使用的保存系统状态的方法。

   1. 在onStop之前调用onSaveInstance\(\)方法【不一定是在onPause之前或之后】来保存一些数据，系统会自动保存一些状态
   2. 在onStart之前会调用onRestoreInstance\(\)方法，onCreate\(\)也可以来进行数据的恢复
   3. onRestoreInstance\(\)方法中的Bundle参数只要被调用一定有值，但是onCreate中的参数有可能是null，推荐使用前者.

2. 


