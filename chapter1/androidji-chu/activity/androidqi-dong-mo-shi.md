Android中的Activity由任务栈管理，当我们start一个新的Activity时，就往任务栈中新加入一个栈帧，而当我们finish一个Activity界面时，则往任务栈中移除一个栈帧。Activity具有四种启动模式，我们可以在配置文件中通过修改launchMode修改，启动模式分别是：standard、singleTop、singleTask和singleInstance。

> * **standard** standard为默认Activity的启动模式。在standard启动模式下，无论何时start一个Activity，系统都会往任务栈中加入一个新的栈帧。
>
> * **singleTop** 在singleTop启动模式下，当我们start一个Activity时，系统会先去检测任务栈栈顶的Activity和要启动t的Activity是否相同。如果相同则不进行任何操作，否则往任务栈中加入一个新的栈帧。
>
> * **singleTask** 在singleTask启动模式下，当我们start一个Activity时，系统会先去检测任务栈中是否含有将要启动的Activity。如果含有，则把该Activity所在栈帧的顶部的栈帧移除，使该Activity所在的栈帧处在栈顶，如果没有，则新加入一个栈帧。
>
> * **singleInstance** 在singleInstance启动模式下，当我们start一个新的Activity时，该Activity会在一个新的任务栈中启动。
>
> 和任务栈相关的参数

**TaskAffinity：**指定任务栈的名称，若不指定，Activity自动继承Application也就是包名。TaskAffinity主要和singleTaks和allowTaskReparenting属性配对

1. TaskAffinity和singleTaks：待启动的Activity会运行在与TaskAffinity名字相同的任务栈中；
2. TaskAffinity和allowTaskReparenting：属性为true的话，当Activity【A，属于A应用】启动Activity【B，属于B应用】时，Activity会从A应用的任务栈转移到B应用的任务栈



