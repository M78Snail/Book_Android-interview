### Activity的生命周期如下图所示： {#toc_1}

| ![](/assets/import.png) |
| :---: |


| 方法 | 操作 |
| :---: | :---: |
| onCreate\(\) | 初始化操作 |
| onStart\(\) | 在可见之前被调用 |
| onResume\(\) | 可见状态，位于栈顶 |
| onPause\(\) | 处理一些消耗cpu操作，保存关键数据 |

### Fragment生命周期如下图所示： {#toc_2}

| ![](/assets/import1.1.png) |
| :---: |


| 方法 | 操作 |
| :---: | :---: |
| onAttach\(\) | 当fragment被加入到activity调用此方法，可以获得activity |
| onCreateView\(\) | 创建layout |
| onDestroyView\(\) | fragment移除视图 |



