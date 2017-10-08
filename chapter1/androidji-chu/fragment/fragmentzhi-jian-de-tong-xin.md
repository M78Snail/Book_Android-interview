### Fragment生命周期如下图所示： {#toc_2}

| ![](/assets/import1.1.png) |
| :---: |


| 方法 | 操作 |
| :---: | :---: |
| onAttach\(\) | 当fragment被加入到activity调用此方法，可以获得activity |
| onCreateView\(\) | 创建layout |
| onDestroyView\(\) | fragment移除视图 |
| onActivityCreated\(\) | 当Activity的onCreate\(\)函数调用时调用 |

Activity的状态决定了Fragment可能接收到的回调函数。

1. 当activity接收到它的onCreate\(\)回调函数，那么这个activity中的fragment最多接收到了onActivityCreated\(\)；

2. 当Activity处于Resumed状态时，Fragment才可以自由的进行添加删除，也就是说只有处于onResume\(\)状态时，fragment状态可以改变；

3. 当activity离开Resumed状态，fragment的生命周期被activity控制。



