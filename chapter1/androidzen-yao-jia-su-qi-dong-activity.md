个人认为，影响Activity启动时间的主要有两个地方：

> 1. onCreate、onStart、onResume等回调方法的执行时间
> 2. Activity对应的界面的inflate时间

对于第一点，我们应该尽量减少在这些回调方法中执行耗时操作（涉及数据库，图片等），如果一定要执行耗时操作，可以考虑新开子线程处理。  
对于第二点，我们应该合理使用各种xml的优化标签，并界面上减少View的嵌套层数与绘制时间。

  

