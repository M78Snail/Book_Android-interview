### Google官方MVP示例 [todo-mvp](https://github.com/googlesamples/android-architecture/tree/todo-mvp/)基础架构项目分析

项目的app/src/main/java目录中代码的组织方式是按照功能模块组织的，每个功能为一个包，每个功能模块的内部分为xActivity、xFragment、xPresenter、xContract四个类文件，其作用如下：

* xActivity主要负责创建View和Presenter实例，并将它们联系起来

* xFragment实现View接口，作为MVP结构中的View

* xPresenter实现Presenter接口，是MVP结构中的Presenter

* xContract作为合同接口，统一管理View和Presenter的接口，便于查看和维护View和Presenter的功能

Model作为单独的模块存放与data目录下

项目的app/src/main/java目录中结构如下：

| ![](/assets/import1.13.png) |
| :---: |


---

##### MVP实现方式-**View和Present**

以添加或编辑任务功能模块\(addedittask\)为例

**1.View和Present基础接口**

* BaseView接口中有setPresenter方法，用于将Presenter实例传入View中，在Presenter的实现类的构造方法中调用。

```java
public interface BaseView<T> {

    void setPresenter(T presenter);

}
```

* BasePresenter接口中有start方法，用于Presenter开始获取数据并调用View进行界面显示，在View的实现类Fragment中的onResume方法中调用。

```java
public interface BasePresenter {

    void start();

}
```

**2.AddEditTaskContract**

AddEditTaskContract是一个合同接口，其中包含了View和Presenter接口，便于查看和维护View和Presenter的功能。

```java
public interface AddEditTaskContract {

    interface View extends BaseView<Presenter> {

        void showEmptyTaskError();

        void showTasksList();

        void setTitle(String title);

        void setDescription(String description);

        boolean isActive();
    }

    interface Presenter extends BasePresenter {

        void createTask(String title, String description);

        void updateTask( String title, String description);

        void populateTask();
    }
}
```

**3.AddEditTaskActivity**

AddEditTaskActivity中创建了View的实现类AddEditTaskFragment的实例和Presenter的实现类AddEditTaskPresenter的实例，并把它们联系起来。

```java
public class AddEditTaskActivity extends AppCompatActivity {
    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.addtask_act);

        ...

        // 创建AddEditTaskFragment实例
        AddEditTaskFragment addEditTaskFragment =
                (AddEditTaskFragment) getSupportFragmentManager().findFragmentById(R.id.contentFrame);
        String taskId = null;
        if (addEditTaskFragment == null) {
            addEditTaskFragment = AddEditTaskFragment.newInstance();
            if(getIntent().hasExtra(AddEditTaskFragment.ARGUMENT_EDIT_TASK_ID)) {
                taskId = getIntent().getStringExtra(
                        AddEditTaskFragment.ARGUMENT_EDIT_TASK_ID);
                actionBar.setTitle(R.string.edit_task);
                Bundle bundle = new Bundle();
                bundle.putString(AddEditTaskFragment.ARGUMENT_EDIT_TASK_ID, taskId);
                addEditTaskFragment.setArguments(bundle);
            } else {
                actionBar.setTitle(R.string.add_task);
            }
            ActivityUtils.addFragmentToActivity(getSupportFragmentManager(),
                    addEditTaskFragment, R.id.contentFrame);
        }

        // 创建Presenter实例，传入Model实例和AddEditTaskFragment实例，建立View和Presenter之间的联系
        new AddEditTaskPresenter(
                taskId,
                Injection.provideTasksRepository(getApplicationContext()),
                addEditTaskFragment);
    }

    ...
}
```

**4.AddEditTaskFragment**

AddEditTaskFragment实现了AddEditTaskContract接口中的View接口，其中有一个Presenter实例，在onResume方法中调用Presenter的start方法。

```java
public class AddEditTaskFragment extends Fragment implements AddEditTaskContract.View {

    private AddEditTaskContract.Presenter mPresenter;

    public static AddEditTaskFragment newInstance() {
        return new AddEditTaskFragment();
    }

    public AddEditTaskFragment() {
        // Required empty public constructor
    }

    @Override
    public void onResume() {
        super.onResume();
        // 调用start方法
        mPresenter.start();
    }

    // 重写setPresenter方法
    @Override
    public void setPresenter(@NonNull AddEditTaskContract.Presenter presenter) {
        mPresenter = checkNotNull(presenter);
    }

    ...
}
```

**5.AddEditTaskPresenter**

AddEditTaskPresenter实现了AddEditTaskContract接口中的Presenter接口，其中有一个View实例，在构造方法中调用View的setPresenter方法与View建立联系。

```java
public class AddEditTaskPresenter implements AddEditTaskContract.Presenter,
        TasksDataSource.GetTaskCallback {

    // Model实例，完成数据获取  
    @NonNull
    private final TasksDataSource mTasksRepository;

    @NonNull
    private final AddEditTaskContract.View mAddTaskView;

    @Nullable
    private String mTaskId;

    public AddEditTaskPresenter(@Nullable String taskId, @NonNull TasksDataSource tasksRepository,
            @NonNull AddEditTaskContract.View addTaskView) {
        mTaskId = taskId;
        mTasksRepository = checkNotNull(tasksRepository);
        mAddTaskView = checkNotNull(addTaskView);

        mAddTaskView.setPresenter(this);
    }

    // 重写start方法，开始获取数据
    @Override
    public void start() {
        if (mTaskId != null) {
            populateTask();
        }
    }

    ...
}
```

---

##### MVP实现方式-Model

| ![](/assets/import1.14.png) |
| :---: |


TasksDataSource定义了Model的回调接口和方法

```java
public interface TasksDataSource {

    interface LoadTasksCallback {

        void onTasksLoaded(List<Task> tasks);

        void onDataNotAvailable();
    }

    interface GetTaskCallback {

        void onTaskLoaded(Task task);

        void onDataNotAvailable();
    }

    void getTasks(@NonNull LoadTasksCallback callback);

    void getTask(@NonNull String taskId, @NonNull GetTaskCallback callback);

    void saveTask(@NonNull Task task);

    void completeTask(@NonNull Task task);

    void completeTask(@NonNull String taskId);

    void activateTask(@NonNull Task task);

    void activateTask(@NonNull String taskId);

    void clearCompletedTasks();

    void refreshTasks();

    void deleteAllTasks();

    void deleteTask(@NonNull String taskId);
}
```

**1.TasksLocalDataSource**

TasksLocalDataSource是TasksDataSource的本地数据源实现类，用于从本地数据库获取数据，采用单例模式，并且有两个帮助类TasksDbHelper和TasksPersistenceContract用于从操作数据库

**2.TasksRemoteDataSource**

TasksRemoteDataSource是TasksDataSource的远程数据源实现类，用于从服务端获取数据（demo里只是模仿从服务端获取），采用单例模式

**3.TasksRepository**

Repository也是TasksDataSource的实现类，其中持有两个两个TasksDataSource对象，一般为一个TasksLocalDataSource对象和一个TasksRemoteDataSource对象，用于统一管理获取数据的方式，采用单例模式

```java
public class TasksRepository implements TasksDataSource {

    private static TasksRepository INSTANCE = null;

    // 远程数据源对象
    private final TasksDataSource mTasksRemoteDataSource;

    // 本地数据源对象
    private final TasksDataSource mTasksLocalDataSource;

    // 存放缓存数据
    Map<String, Task> mCachedTasks;

    // 缓存数据是否可用
    boolean mCacheIsDirty = false;

    // 构造方法
    private TasksRepository(@NonNull TasksDataSource tasksRemoteDataSource,
                            @NonNull TasksDataSource tasksLocalDataSource) {
        mTasksRemoteDataSource = checkNotNull(tasksRemoteDataSource);
        mTasksLocalDataSource = checkNotNull(tasksLocalDataSource);
    }

    // 获取TasksRepository实例，采用单例模式
    public static TasksRepository getInstance(TasksDataSource tasksRemoteDataSource,
                                              TasksDataSource tasksLocalDataSource) {
        if (INSTANCE == null) {
            INSTANCE = new TasksRepository(tasksRemoteDataSource, tasksLocalDataSource);
        }
        return INSTANCE;
    }

    public static void destroyInstance() {
        INSTANCE = null;
    }

    // 获取数据
    @Override
    public void getTasks(@NonNull final LoadTasksCallback callback) {
        checkNotNull(callback);

        // 如果缓存可用，则立即返回内存中的数据
        if (mCachedTasks != null && !mCacheIsDirty) {
            callback.onTasksLoaded(new ArrayList<>(mCachedTasks.values()));
            return;
        }

        if (mCacheIsDirty) {
            // 如果缓存数据已经不可用，则从服务端更新数据
            getTasksFromRemoteDataSource(callback);
        } else {
            // 从数据库获取数据，如果没有，则从服务端获取
            mTasksLocalDataSource.getTasks(new LoadTasksCallback() {
                @Override
                public void onTasksLoaded(List<Task> tasks) {
                    refreshCache(tasks);
                    callback.onTasksLoaded(new ArrayList<>(mCachedTasks.values()));
                }

                @Override
                public void onDataNotAvailable() {
                    getTasksFromRemoteDataSource(callback);
                }
            });
        }
    }

    // 从服务端获取数据
    private void getTasksFromRemoteDataSource(@NonNull final LoadTasksCallback callback) {
        mTasksRemoteDataSource.getTasks(new LoadTasksCallback() {
            @Override
            public void onTasksLoaded(List<Task> tasks) {
                refreshCache(tasks);
                refreshLocalDataSource(tasks);
                callback.onTasksLoaded(new ArrayList<>(mCachedTasks.values()));
            }

            @Override
            public void onDataNotAvailable() {
                callback.onDataNotAvailable();
            }
        });
    }
}
```

可以看到，在实例化TasksRepository时需要传入两个TasksDataSource对象；在获取数据时先从缓存中获取，如果缓存中没有，则从数据库获取，数据库没有，再从服务端获取，若为强制刷新，则直接从服务端获取。

---

### Model使用

**1.TasksDataSource的实例化**

在MVP中，Presenter对象持有Model对象，如：

```java
public class AddEditTaskPresenter implements AddEditTaskContract.Presenter,
        TasksDataSource.GetTaskCallback {

    // TasksDataSource对象
    @NonNull
    private final TasksDataSource mTasksRepository;

    // 实例化时传入TasksDataSource对象
    public AddEditTaskPresenter(@Nullable String taskId, @NonNull TasksDataSource tasksRepository,
            @NonNull AddEditTaskContract.View addTaskView) {
        mTaskId = taskId;
        mTasksRepository = checkNotNull(tasksRepository);
        mAddTaskView = checkNotNull(addTaskView);

        mAddTaskView.setPresenter(this);
    }
}
```

Presenter在实例化是需要传入TasksDataSource对象，Presenter是在Activity中完成实例化的

```java
public class AddEditTaskActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.addtask_act);

        ...

        // 实例化Presenter，传入TasksDataSource对象
        new AddEditTaskPresenter(
                taskId,
                Injection.provideTasksRepository(getApplicationContext()),
                addEditTaskFragment);
    }
}
```

可以看到TasksDataSource对象是由Injection类的provideTasksRepository方法生成

**2.Gradle动态配置**

在main/java目中并没有Injection类，通过查看app/build.gradle文件可以知道，项目有两个生产版本：mock和prod，如下：

```java
productFlavors {
        mock {
            applicationIdSuffix = ".mock"
        }
        prod {

        }
}
```

在项目的目录结构中也有对应的src/mock和src/prod两个目录目录

| ![](/assets/import1.14.2.png) |
| :---: |


在mock和prod中分别有一个Injection类，这里利用了gradle中的productFlavor创建了多个版本，其中每个版本都可以在工程src目录下建立与main同级的文件夹，文件夹名字就是ProductFlavor的名字，可以在其中创建java目录并添加相应的java文件，不用的版本可以有相同名字不同实现的类，具体可查看[Android Plugin for Gradle](http://developer.android.com/intl/zh-cn/tools/building/plugin-for-gradle.html)

prod目录下的Injection

```java
public class Injection {

    public static TasksRepository provideTasksRepository(@NonNull Context context) {
        checkNotNull(context);
        return TasksRepository.getInstance(TasksRemoteDataSource.getInstance(),
                TasksLocalDataSource.getInstance(context));
    }
}
```

mock目录下的Injection

```java
public class Injection {

    public static TasksRepository provideTasksRepository(@NonNull Context context) {
        checkNotNull(context);
        return TasksRepository.getInstance(FakeTasksRemoteDataSource.getInstance(),
                TasksLocalDataSource.getInstance(context));
    }
}
```

可以看到：

* prod的Injection创建TasksRepository实例时传入的是TasksRemoteDataSource实例和TasksLocalDataSource实例
* mock的Injection创建TasksRepository实例时传入的是FakeTasksRemoteDataSource实例和TasksLocalDataSource实例，其中FakeTasksRemoteDataSource是在mock目录下创建的TasksDataSource的实现类。 

通过这种方式可以实现不同版本注入不同的TasksDataSource实现类。

  




