## android中layout\_weight的理解

重点有两个

1. layout\_weight表示LinearLayout中额外空间的划分\(可能扩大应用layout\_weight前的大小也可能缩小\)。
2. 按比例\(layout\_weight大小的比例\)

**以下说的都以 android:orientation="horizontal" 为例**

看了一下源码,虽说不太懂,但了解了下大概意思,按照自己的理解总结一下,直接写一下简化的代码吧\(下面的代码是LinearLayout源文件中一部分的精简,变量名称含义可能不准确,为叙述方便暂作此解释\):

```
//Either expand children with weight to take up available space or
// shrink them if they extend beyond our current bounds
int delta = widthSize - mTotalLength;
if (delta != 0 && totalWeight > 0.0f) {
    float weightSum = mWeightSum > 0.0f ? mWeightSum : totalWeight;
    for (int i = 0; i < count; ++i) {
        final View child = getVirtualChildAt(i);

        if (child == null || child.getVisibility() == View.GONE) {
            continue;
        }

        final LinearLayout.LayoutParams lp =
                (LinearLayout.LayoutParams) child.getLayoutParams();

        float childExtra = lp.weight;
        if (childExtra > 0) {
            int share = (int) (childExtra * delta / weightSum);
　　　　　　　weightSum -= childExtra;
　　　　　　  delta  -= share;
            int childWidth = child.getMeasuredWidth() + share;
            if (childWidth < 0) {
                childWidth = 0;
            }
        }
    }
}
```

变量含义

* widthSize:           LinearLayout的宽度
* mTotalLength:     所有子View的宽度的和\(还没用考虑layout\_weight\)
* totalWeight:        所有子View的layout\_weight的和
* mWeihtSUm:　　 LinearLayout的android:weightSum属性

过程分析:

首先计算出额外空间\(可以为负\)如果额外空间不为0并且有子View的layout\_weight不为0的话按layout\_weight分配额外空间:

```
int delta = widthSize - mTotalLength;
if (delta != 0 && totalWeight > 0.0f) {
  ...
}
```

**如果LinearLayout设置了weightSum则覆盖子View的layout\_weight的和:**

```
float weightSum = mWeightSum > 0.0f ? mWeightSum : totalWeight;
```

然后遍历LinearLayout的子元素,如果不为null且Visibility不为GONE的话,取得它的LayoutParams,如果它的layout\_weight大于0,根据weightSum与它的weight计算出分配给它的额外空间

```
if (childExtra > 0) {
    int share = (int) (childExtra * delta / weightSum);
　　 weightSum -= childExtra;
　　 delta -= share;

    int childWidth = child.getMeasuredWidth() + share;
    if (childWidth < 0) {
        childWidth = 0;
    }
}
```

网上有解释说layout\_weight表示重要程度,表示划分额外空间的优先级,通过代码可以知道这种观点是错误的.layout\_weight表示划分的比例,至于当View的layout\_width为fill\_parent时layout\_weight比例相反的问题按我的理解可以作以下解释:

比如说如下XML:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:background="#00ff00"
    android:weightSum="0"
    android:orientation="horizontal" >

    <Button
        android:id="@+id/imageViewLoginState"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:text="1" >
    </Button>

    <Button
        android:id="@+id/imageViewLoginState1"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:text="2" >
    </Button>

    <Button
        android:id="@+id/imageViewLoginState2"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_weight="2"
        android:text="3" >
    </Button>

</LinearLayout>
```

按一般理解,3个Button的比例应该为1:1:2,但实际情况是这样的:

![](/assets/import_weight.png)

按我的理解,系统是这样设置按钮的大小的,变量名按前面代码的意义:

假设Container即LinearLayout的宽度为PARENT\_WIDTH

三个按钮的宽度都是FILL\_PARENT,所以在应用layout\_weight之前,三个按钮的宽度都为PARENT\_WIDTH

所以额外空间:

```
delta = PARENT_WIDTH - 3 * PARENT_WIDTH = -2 * PARENT
```

因为LinearLayout没有设置android:weightSum\(默认为0,设置为0就当没设置吧\),所以 mWeightSum = 1 + 1 +2 =4

所以:

第一个按钮的宽度为

```
PARENT_WIDTH + share = PARENT_WIDTH + (layout_weight * delta / mWeightSum)

                                                = PARENT_WIDTH + (1 * (-2 * PARENT_WIDTH) /4)

                                                = 1 /2 *PARENT_WIDTH
```

然后更新weightSum与delta：

```
weightSum -= childExtra;(=3)
delta  -= share;(=-3/2 * PARENT_WIDTH)
```

第二个按钮的宽度为：

```
PARENT_WIDTH + share = PARENT_WIDTH + (layout_weight * delta / mWeightSum)

                                                = PARENT_WIDTH + (1 * (-3 / 2 * PARENT_WIDTH) /3)

                                                = 1 /2 *PARENT_WIDTH
```

更新weightSum与delta：

```
weightSum -= childExtra;(=2)
delta  -= share;(=-PARENT_WIDTH)
```

第三个按钮的宽度为：

```
PARENT_WIDTH + share = PARENT_WIDTH + (layout_weight * delta / mWeightSum)

                                                = PARENT_WIDTH + (2 * (- PARENT_WIDTH) /2)

                                                = 0
```

所以最终的而已就是前两个按钮平分LinearLayout,第三个按钮消失了.

大致过程是这样,但不全对,比如如果上例中LinearLayout的weightSum设置为2的话,前两个按钮的宽度为0,但当计算第三个按钮的宽度时mWeightSum = 0,但layout\_weight \* delta / mWeightSum无法计算,不知道系统怎么处理的,在我的能力之外了,weightSum为2时的效果图:

![](/assets/import_weight1.png)  
weightSum为3时的效果图:

![](/assets/import_weight2.png)

SDK中说明的是,layout\_weight表示额外空间怎么划分,要注意额外2字,要有额外的空间才可以将按比例将其分配给设置了layout\_weight的子View,所以,如果LinearLayout设置为WRAP\_CONTENT的话是没有额外的空间的,layout\_weight就没有用处,所只要layout\_width不设置为WRAP\_CONTENT就行,也可以设置为具体的值,如果值太小的话,额外空间为负,可能压缩子控件,使其大小比XML文件中定义的小,例如:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="100dp"
    android:layout_height="wrap_content"
    android:background="#00ff00"
    android:orientation="horizontal" >

    <Button
        android:id="@+id/button1"
        android:layout_width="60dp"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:text="1" >
    </Button>

    <Button
        android:id="@+id/button2"
        android:layout_width="60dp"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:text="2" >
    </Button>

    <Button
        android:id="@+id/button3"
        android:layout_width="60dp"
        android:layout_height="fill_parent"
        android:layout_weight="2"
        android:text="3" >
    </Button>

</LinearLayout>
```

额外空间：

```
delta = 100- 3 * 60 = -80

mWeightSum = 1 + 1 +2 =4
```

所以:

第一个按钮的宽度为：

```
60+ share = 60 + (layout_weight * delta / mWeightSum)

                = 60 + (1 * (-80) /4) = 40
```

然后：

```
weightSum -= childExtra;(=3)
delta  -= share;(=-60)
```

第二个按钮的宽度为：

```
60 + share = 60 + (layout_weight * delta / mWeightSum)

                = 60 + (2 * (-60) /3)

                = 40
```

然后：

```
weightSum -= childExtra;(=2)
delta  -= share;(=-40)
```

第三个按钮的宽度为：

```
60 + share = 60 + (layout_weight * delta / mWeightSum)

                 = 60 + (2 * (-40) /2)

                 = 20
```

![](/assets/import——weight3.png)

以下代码也说明了layout\_weight表示额外空间的分配:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="200dp"
    android:layout_height="wrap_content"
    android:background="#00ff00"
    android:orientation="horizontal" >

    <Button
        android:id="@+id/button1"
        android:layout_width="60dp"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:text="1" >
    </Button>

    <Button
        android:id="@+id/button2"
        android:layout_width="40dp"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:text="2" >
    </Button>

   
</LinearLayout>

```

额外空间为100，所以Button1的宽度为60+100/2=110，Button2的宽度为40+100/2=90。

