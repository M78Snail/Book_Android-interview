> 事件分发流程

* 当点击事件发生时，首先Activity会将点击事件传递给Window，再从Window传递给顶层View。TouchEvent会首先传递给最顶层View的**dispatchTouchEvent事件分发**，如果**dispatchTouchEvent返回true**，则会由当前view的**dispatchTouchEvent来消费事件；如果返回false，**则会传递给上层父view的**onTouchEvent **进行消费，如果返回默认的**super.dispatchTouchEvent\(ev\)**,事件会自动的分发给当前View的**onIntercepTouchEvent**方法。如果 interceptTouchEvent 返回 true ，也就是拦截掉了，则交给它的 onTouchEvent 来处理，如果 interceptTouchEvent 返回 false ，那么就传递给子 view ，由子 view 的 dispatchTouchEvent 再来开始这个事件的分发。如果事件传递到某一层的子 view 的 onTouchEvent 上了，这个方法返回了 false ，那么这个事件会从这个 view 往上传递，都是 onTouchEvent 来接收。而如果传递到最上面的 onTouchEvent 也返回 false 的话，这个事件就会“消失”，而且接收不到下一次事件。

> View的渲染机制

UI对象—-&gt;CPU处理为多维图形,纹理 —–通过OpeGL ES接口调用GPU—-&gt; GPU对图进行光栅化\(Frame Rate \) —-&gt;硬件时钟\(Refresh Rate\)—-垂直同步—-&gt;投射到屏幕![](/assets/import_xuanran.png)

* Android系统每隔16ms发出VSYNC信号，触发对UI进行渲染，那么整个过程如果保证在16ms以内就能达到一个流畅的画面。

* 如果系统发生的VSYNC信号，而此时无法进行渲染，还在做别的操作，那么就会导致丢帧的现象， （大家在察觉到APP卡顿的时候，可以看看logcat控制台，会有drop frames类似的警告）。这样的话，绘制就会在下一个16ms的时候才进行绘制， 即使只丢一帧，用户也会发现卡顿的。

* 为了理解App是如何进行渲染的，我们必须了解手机硬件是如何工作，那么就必须理解什么是VSYNC。

  在讲解VSYNC之前，我们需要了解两个相关的概念：

  1.Refresh Rate：代表了屏幕在一秒内刷新屏幕的次数，这取决于硬件的固定参数，例如60Hz。 2.Frame Rate：代表了GPU在一秒内绘制操作的帧数，例如30fps，60fps。

* GPU会获取图形数据进行渲染，然后硬件负责把渲染后的内容呈现到屏幕上，他们两者不停的进行协作。不幸的是，刷新频率和帧率并不是总能够保持相同的节奏。如果发生帧率与刷新频率不一致的情况， 就会容易出现Tearing的现象\(画面上下两部分显示内容发生断裂，来自不同的两帧数据发生重叠\)。通常来说，帧率超过刷新频率只是一种理想的状况，在超过60fps的情况下，GPU所产生的帧数据会因为等待VSYNC的刷新信息而被Hold住， 这样能够保持每次刷新都有实际的新的数据可以显示。但是我们遇到更多的情况是帧率小于刷新频率。在这种情况下，某些帧显示的画面内容就会与上一帧的画面相同。糟糕的事情是，帧率从超过60fps突然掉到60fps以下， 这样就会发生LAG，JANK，HITCHING等卡顿掉帧的不顺滑的情况。这也是用户感受不好的原因所在。

> 编译打包的过程

1. **第一步：打包资源文件，生成R.java文件**

   编译R.java类需要用到AndroidSDK提供的aapt工具

2. **第二步：处理AIDL文件，生成对应的.java文件**

3. **第三步：编译Java文件，生成对应的.class文件**

4. **第四步：把.class文件转化成Davik VM支持的.dex文件**

   **将工程bin目录下的class文件编译成classes.dex，Android虚拟机只能执行dex文件!**

5. **第五步：打包生成未签名的.apk文件**

6. **第六步：对未签名.apk文件进行签名**

7. **第七步：对签名后的.apk文件进行对齐处理（不进行对齐处理是不能发布到Google Market的）**

> Android有多个资源文件夹，应用在不同分辨率下是如何查找对应文件夹下的资源的，描述整个过程

Activity.setContentView\(int resourceID\) -&gt;

PhoneWindow.setContentView\(int resourceID\) -&gt;

LayoutInflater.inflate\(int resource, ViewGroup root\) -&gt;

LayoutInflater.inflate\(int resource, ViewGroup root, boolean attachToRoot\) -&gt;

Resources.getLayout\(int id\) -&gt;

Resources.loadXmlResourceParser\(int id, String type\) -&gt;

**Resources.getValue\(int id, TypedValue outValue, boolean resolveRefs\) -&gt;**

AssetManager.getResourceValue\(int ident, int density, TypedValue outValue, boolean resolveRefs\) -&gt;

AssetManager.loadResourceValue\(int ident, short density, TypedValue outValue, boolean resolve\)

1. 最后加载的是哪个xml是由Resources.getValue\(int id, TypedValue outValue, boolean resolveRefs\)调用完成之后的outValue.string决定的，因为outValue.string的值就是你的资源文件的具体路径，如：

   1\) xxx/values/xxx.xml

   2\) xxx/layout-sw600dp/xxx.xml

2. AssetManager.loadResourceValue\(\)调的是frameworks/base/core/jni/android\_util\_AssetManager.cpp里面的native方法， 如何获得正确的outValue值，在native方法俩面主要有以下几步：

   1\) 调用frameworks/base/libs/utils/ResourceTypes.cpp 的ResTable::getResource\(\)，遍历所有资源文件

   2\) 在ResTable::getResource\(\)里面调用ResTable::getEntry\(\)来确定资源文件来自哪个entry，即layout,或者layout-swdp，由此可见，ResTable::getEntry\(\)是我们这个问题的关键

   3\) 在ResTable::getEntry\(\)里面：

   a\) 首先获取本设备的configurion信息，屏幕分辨率，屏幕大小，locale，横竖屏等。

   b\) 根据得到的本设备的configurion信息，过滤掉不适应本设备的entry，比如设备是800x480的，那么超过此分辨率的资源\(例：layout-sw600dp\)就要被过滤掉，实现在frameworks/base/include/utils/ResourceTypes.h中ResTable\_config的match函数中

   c\) 对过滤后的resource进行最佳适配，找到最符合的entry文件。因为之前已经将不符合的，即大分辨率的entry已经被过滤掉了，所以这里就找剩下的最大的就是最佳适配的。实现在frameworks/base/include/utils/ResourceTypes.h中ResTable\_config的isBetterThan\(\)函数中。



