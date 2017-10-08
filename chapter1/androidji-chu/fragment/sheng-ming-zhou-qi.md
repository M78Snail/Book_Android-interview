> 一.在Fragment中调用Activity中的方法getActivity

```
View listView = getActivity().findViewById(R.id.list);
```

> 二.获得fragment的引用要用[**FragmentManager**](http://developer.android.com/reference/android/app/FragmentManager.html)之后可以调用[`findFragmentById()`](http://developer.android.com/reference/android/app/FragmentManager.html#findFragmentById%28int%29) 或者 [`findFragmentByTag()`](http://developer.android.com/reference/android/app/FragmentManager.html#findFragmentByTag%28java.lang.String%29)

```
ExampleFragment fragment = (ExampleFragment) getFragmentManager().findFragmentById(R.id.example_fragment);
```



