View的绘制机制是从ViewRoot的performTraversals方法开始，它经过measure,layout,draw三个过程，最终将一个View绘制出来，其中measure用来测量View的宽和高，layout用来确定view在父容器的位置，draw将view绘制出来。

> 1. 理解MeasureSpec

MeasureSpec在很大程度上决定一个View的大小，在测量过程中，系统会将View的LayoutParams根据父容器所施加的规则转换成对应的MeasureSpec，然后再根据这个MeasureSpec测量出View的大小。

> 2.SpecMode 三类

1. **UNSPECIFIED**
   父容器不对View有任何影响，要多大给多大，这种情况一般存在于系统内部
2. **EXACTLY**
   父容器已经检测出View所需的精确大小，这个时候View大小就是SpecSize指定的大小。它对应于match\_parent和具体的数值
3. **AT\_MOST**
   父容器指定了一个可用大小，即SpecSize，View不能大于这个值，具体是多少看用户自己需求，对应于wrap\_content3.匹配模式

> 3.匹配模式

* 如果View是match\_parent，而父容器是精准模式，那么View是精准模式并且为父容器的剩余空间
* 如果View是match\_parent，而父容器是最大模式，那么View也是最大模式并且不会超过父容器大小
* 如果View是wrap\_content，那么不论父容器为什么模式，View总是最大模式并且不超过剩余空间大小

![](/assets/import_specmode.png)

> 4.onMeasure\(\)方法

直接继承View的控件需要重写onMeasure方法并设置wrap\__content时的自身大小，否则wrap\__content

