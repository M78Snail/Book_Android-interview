**标记清除**  
标记清除算法分两步执行：

1. 暂停用户线程，通过GC Root使用可达性算法标记存活对象
2. 清除未被标记的垃圾对象

标记清除算法缺点如下：

1. 效率较低，需要暂停用户线程
2. 清除垃圾对象后内存空间不连续，存在较多内存碎片,标记算法如今使用的较少了

**复制算法**  
复制算法也分两步执行，在复制算法中一般会有至少两片的内存空间（一片是活动空间，里面含有各种对象，另一片是空闲空间，里面是空的）：

1. 暂停用户线程，标记活动空间的存活对象
2. 活动空间的存活对象复制到空闲空间去，清除活动空间 复制算法相比标记清除算法，优势在于其垃圾回收后的内存是连续的。

复制算法相比标记清除算法，优势在于其垃圾回收后的内存是连续的。

但是复制算法的缺点也很明显：

1. 需要浪费一定的内存作为空闲空间
2. 如果对象的存活率很高，则需要复制大量存活对象，导致效率低下

复制算法一般用于年轻代的Minor GC，主要是因为年轻代的大部分对象存活率都较低

**标记整理**  
标记整理算法是标记清除算法的改进，分为标记、整理两步：

1. 暂停用户线程，标记所有存活对象
2. 移动所有存活对象，按内存地址次序一次排列，回收末端对象以后的内存空间

标记整理算法与标记清除算法相比，整理出的内存是连续的；而与复制算法相比，不需要多片内存空间。  
然而标记整理算法的第二步整理过程较为麻烦，需要整理存活对象的引用地址，理论上来说效率要低于复制算法。  
因此标记整理算法一般引用于老年代的Major GC



