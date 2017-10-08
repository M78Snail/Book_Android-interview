> Fragment为什么被称为第五大组件

1. Fragment比起Activity更加节省内存，切换模式更加舒服；
2. 使用频率高
3. 拥有自己的生命周期，必须依附于Activity

> 在Activity创建Fragment的方式

静态：

```java
<?xml version="." encoding="utf-"?>
 <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:tools="http://schemas.android.com/tools"
   android:id="@+id/activity_main"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   android:orientation="horizontal"
   tools:context="com.example.administrator.fragmenttest.MainActivity">
   <fragment
     android:layout_width="dp"
     android:layout_height="match_parent"
     android:layout_weight=""
     android:name="com.example.administrator.fragmenttest.LeftFragment"
     tools:layout="@layout/left_layout" />
   <fragment
     android:layout_width="dp"
     android:layout_height="match_parent"
     android:layout_weight=""
     android:name="com.example.administrator.fragmenttest.RightFragment"
     tools:layout="@layout/right_layout" />
 </LinearLayout>
```

动态：

```java
public class MainActivity extends AppCompatActivity {
   //声明本次使用到的java类
   FragmentManager fragmentManager;
   FragmentTransaction fragmentTransaction;
   RightFragment rightFragment;
   @Override
   protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_main);
     /*在activity对应java类中通过getFragmentManager()
     *获得FragmentManager，用于管理ViewGrop中的fragment
     * */
     fragmentManager=getFragmentManager();
     /*FragmentManager要管理fragment（添加，替换以及其他的执行动作）
     *的一系列的事务变化，需要通过fragmentTransaction来操作执行
     */
     fragmentTransaction = fragmentManager.beginTransaction();
     //实例化要管理的fragment
     rightFragment = new RightFragment();
     //通过添加（事务处理的方式）将fragment加到对应的布局中
     fragmentTransaction.add(R.id.right,rightFragment);
     //事务处理完需要提交
     fragmentTransaction.commit();
   }
 }
```

> FragmentPageAdapter和FragmentPageStateAdapter的区别

* FragmentPageAdapter在每次切换页面的时候，是将Frament进行分离，适合页面少的Fragment，对内存不会有太大的影响
* FragmentPageStateAdapter在每次切换页面的时候，是将Frament进行回收，适合页面多的Fragment，这样就不会有消耗太多内存



