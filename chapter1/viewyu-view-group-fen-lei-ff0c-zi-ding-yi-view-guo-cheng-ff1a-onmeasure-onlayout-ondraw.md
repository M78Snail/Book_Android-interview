View是Android中基本的UI单元，占据屏幕的一块矩形区域，可用于绘制并能处理事件，而ViewGroup是View的子类，他能包含多个View，并让他们在其中按照一定的规则排列。VIew与ViewGroup的设计使用了组合模式。

自定义View我们一般需要重写一下三个方法：

* onMeasure：用于测量自定义View的大小，在方法中必须调用setMeasureDimension方法
* onLayout：用于确定自定义View布局
* onDraw：用于绘制自定义View本身



