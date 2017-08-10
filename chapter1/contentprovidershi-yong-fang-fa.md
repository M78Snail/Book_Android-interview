为了在应用程序之间交换数据，Android提供了ContentProvider。当一个应用程序需要把自己的数据暴露给其他程序使用时，该应用程序就可以通过ContentProvider来实现，而其他程序则使用ContentResolver来操作ContentProvider暴露的数据。

实现ContentProvider的Java代码如下：

```java
public class MyProvider extends ContentProvider {  
  
    @Override  
    public boolean onCreate() {  
        // 第一次创建时调用，如果创建成功则返回true  
        // 可以在这里打开数据库什么的  
        return true;  
    }  
  
    @Override  
    public String getType(Uri uri) {  
        // 返回ContentProvider所提供数据的MIME类型  
        return null;  
    }  
  
    @Override  
    public Cursor query(Uri uri, String[] projection, String selection,  
            String[] selectionArgs, String sortOrder) {  
        // 实现查询方法  
        return null;  
    }  
      
    @Override  
    public Uri insert(Uri uri, ContentValues values) {  
        // 实现插入方法，返回插入条数  
        return null;  
    }  
  
    @Override  
    public int delete(Uri uri, String selection, String[] selectionArgs) {  
        // 实现删除方法，返回删除条数  
        return 0;  
    }  
  
    @Override  
    public int update(Uri uri, ContentValues values, String selection,  
            String[] selectionArgs) {  
        // 实现更新方法，返回更新条数  
        return 0;  
    }  
  
}  
```

当我们实现了自己的ContentProvider后，还需要去AndroidManifest.xml中注册才行：

```java
<provider   
    android:name=".MyProvider"  
    android:authorities="com.example.test.provider"  
    android:exported="true" />  
```

配置ContentProvider时我们需要设置如下属性：

* name：类名
* authorities：为ContentProvider指定一个对应的Uri，其他程序通过这个Uri来找到该ContentProvider
* exported：允许ContentProvider被其他应用调用 当其他应用需要访问我们提供的ContentProvider的使用，只需使用ContentResolver并传入相应的Uri即可：

```java
public class MyActivity extends Activity {  
  
    private static String TAG = "MyActivity";  
      
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        ContentResolver resolver = getContentResolver();  
        // 传入对应的Uri进行增删查改操作  
        resolver.query(uri, projection, selection, selectionArgs, sortOrder);  
        resolver.insert(url, values);  
        resolver.delete(url, where, selectionArgs);  
        resolver.update(uri, values, where, selectionArgs);  
    }  
      
}  
```

除此之外，我们还可以使用ContentObserver来为ContentProvider添加观察者。我们在其他程序的ContentResolver中注册ContentObserver，当ContentProvider发生改变时，我们在数据改动的使用调用：

```java
getContext().getContentResolver().notifyChange(uri, null);  
```

那么此时ContentObserver中的onChange回调方法就会被调用，我们就可以简单地监听ContentProvider的数据改变了。

