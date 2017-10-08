> 一.在Fragment中调用Activity中的方法getActivity

```
View listView = getActivity().findViewById(R.id.list);
```

> 二.获得fragment的引用要用[**FragmentManager**](http://developer.android.com/reference/android/app/FragmentManager.html)之后可以调用[`findFragmentById()`](http://developer.android.com/reference/android/app/FragmentManager.html#findFragmentById%28int%29) 或者 [`findFragmentByTag()`](http://developer.android.com/reference/android/app/FragmentManager.html#findFragmentByTag%28java.lang.String%29)

```
ExampleFragment fragment = (ExampleFragment) getFragmentManager().findFragmentById(R.id.example_fragment);
```

> 三.创建事件回调

1.在FragmentA中创建回调接口

```java
public static class FragmentA extends ListFragment {
    ...
    // Container Activity must implement this interface
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
}
```

2.Activity继承接口，要在Fragment的onAttach\(\)【这个方法在fragment 被加入到activity中时由系统调用】中通过将传入的activity强制类型转换，实例化一个OnArticleSelectedListener对象：

```java
public static class FragmentA extends ListFragment {
    OnArticleSelectedListener mListener;
    ...
    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mListener = (OnArticleSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString() + " must implement OnArticleSelectedListener");
        }
    }
    ...
}
```



