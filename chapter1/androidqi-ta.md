### ANR

1、什么是ANR

Application Not Responding，页面无响应的对话框

2、发生ANR的条件

应用程序的响应性是由ActivityManager和WindowManager系统服务监视的，当ANR发生条件满足时，就会弹出ANR的对话框

* Activity超过5秒无响应
* BroadcastReceiver超过10秒无响应
* Service超过20秒无响应

3、造成ANR的主要原因

主线程被IO操作阻塞

* Activity的所有生命周期回调都是执行在主线程的
* Service默认执行在主线程中
* BoardcastReceiver的回调onReceive\(\)执行在主线程中
* AsyncTask的回调除了doInBackground，其他都是在主线程中
* 没有使用子线程Looper的Handler的handlerMessage，post\(Runnable\)都是执行在主线程中

4、如何解决ANR

* 使用AsyncTask处理耗时IO操作
* 使用Thread或HandlerThread提高优先级
* 使用Handler处理工作线程的耗时操作
* Activity的onCreate和onResume回调尽量避免耗时操作

---

### OOM

1、什么是OOM

OOM指Out of memory（内存溢出），当前占用内存加上我们申请的内存资源超过了Dalvik虚拟机的最大内存限制就会抛出Out of memory异常

2、OOM相关概念

* 内存溢出：指程序在申请内存时，没有足够的空间供其使用
* 内存泄漏：指程序分配出去的内存不再使用，无法进行回收
* 内存抖动：指程序短时间内大量创建对象，然后回收的现象

3、解决OOM

Bitmap相关

* 图片压缩
* 加载缩略图
* 在滚动时不加载图片
* 回收Bitmap
* 使用inBitmap属性
* 捕获异常

其他相关

* listview重用convertView、使用lru
* 避免onDraw方法执行对象的创建
* 谨慎使用多进程

---

### Bitmap

1、recycle

* 在安卓3.0以前Bitmap是存放在堆中的，我们只要回收堆内存即可
* 在安卓3.0以后Bitmap是存放在内存中的，我们需要回收native层和Java层的内存
* 官方建议我们3.0以后使用recycle方法进行回收，该方法也可以不主动调用，因为垃圾回收器会自动收集不可用的Bitmap对象进行回收
* recycle方法会判断Bitmap在不可用的情况下，将发送指令到垃圾回收器，让其回收native层和Java层的内存，则Bitmap进入dead状态
* recycle方法是不可逆的，如果再次调用getPixels\(\)等方法，则获取不到想要的结果

2、LruCache原理

LruCache是个泛型类，内部采用LinkedHashMap来实现缓存机制，它提供get方法和put方法来获取缓存和添加缓存，其最重要的方法trimToSize是用来移除最少使用的缓存和使用最久的缓存，并添加最新的缓存到队列中

3、计算inSampleSize

```
public static int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;
    if (height > reqHeight || width > reqWidth) {
        if (width > height) {
            inSampleSize = Math.round((float)height / (float)reqHeight);
        } else {
            inSampleSize = Math.round((float)width / (float)reqWidth);
        }
    }
    return inSampleSize;
}
```

4、缩略图

```
public static Bitmap thumbnail(String path,int maxWidth, int maxHeight, boolean autoRotate) {
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    Bitmap bitmap = BitmapFactory.decodeFile(path, options);
    options.inJustDecodeBounds = false;
    int sampleSize = calculateInSampleSize(options, maxWidth, maxHeight);
    options.inSampleSize = sampleSize;
    options.inPreferredConfig = Bitmap.Config.RGB_565;
    options.inPurgeable = true;
    options.inInputShareable = true;
    if (bitmap != null && !bitmap.isRecycled()) {
        bitmap.recycle();
    }
    bitmap = BitmapFactory.decodeFile(path, options);
    return bitmap;
}
```

5、保存Bitmap

```
public static String save(Bitmap bitmap,Bitmap.CompressFormat format, int quality, File destFile) {
    try {
        FileOutputStream out = new FileOutputStream(destFile);
        if (bitmap.compress(format, quality, out)) {
            out.flush();
            out.close();
        }
        if (bitmap != null && !bitmap.isRecycled()) {
            bitmap.recycle();
        }
        return destFile.getAbsolutePath();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
```

6、保存到SD卡

```
public static String save(Bitmap bitmap,Bitmap.CompressFormat format, int quality, Context context) {
    if (!Environment.getExternalStorageState()
            .equals(Environment.MEDIA_MOUNTED)) {
        return null;
    }
    File dir = new File(Environment.getExternalStorageDirectory()
            + "/" + context.getPackageName() + "/save/");
    if (!dir.exists()) {
        dir.mkdirs();
    }
    File destFile = new File(dir, UUID.randomUUID().toString());
    return save(bitmap, format, quality, destFile);
}
```

7、三级缓存

* 网络缓存
* 本地缓存
* 内存缓存

---

**UI卡顿面试题**

1、UI卡顿原理

View的绘制帧数保持60fps是最佳，这要求每帧的绘制时间不超过16ms（1000/60），如果安卓不能在16ms内完成界面的渲染，那么就会出现卡顿现象

2、UI卡顿的原因分析

* 在UI线程中做轻微的耗时操作，导致UI线程卡顿
* 布局Layout过于复杂，无法在16ms内完成渲染
* 同一时间动画执行的次数过多，导致CPU和GPU负载过重
* overDraw，导致像素在同一帧的时间内被绘制多次，使CPU和GPU负载过重
* View频繁的触发measure、layout，导致measure、layout累计耗时过多和整个View频繁的重新渲染
* 频繁的触发GC操作导致线程暂停，会使得安卓系统在16ms内无法完成绘制
* 冗余资源及逻辑等导致加载和执行缓慢
* ANR

3、UI卡顿的优化

* 布局优化
  * 使用include、ViewStub、merge
  * 不要出现过于嵌套和冗余的布局
  * 使用自定义View取代复杂的View
* ListView优化
  * 复用convertView
  * 滑动不加载
* 背景和图片优化
  * 缩略图
  * 图片压缩
* 避免ANR
  * 不要在UI线程中做耗时操作 



