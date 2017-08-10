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



