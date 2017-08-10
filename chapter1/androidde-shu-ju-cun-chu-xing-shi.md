在Android中的数据存储形式主要有以下几种：  
**SharedPreferrences**  
SharedPreferrences主要用于存储一些少量的简单的应用程序配置信息。SharedPreferrences以明文键值对的形式把数据存储在一个xml文件上，该文件位于/data/data//shared\_prefs目录下。因此，SharedPreferrences只适合用于存储一些简单的数据，不适合存储复杂的或敏感的数据。

**File**  
Android和Java一样，同样支持使用文件流来保存和访问文件。除了在手机内置存储空间上存储文件外，Android还支持读写SD卡上的文件：只要获取相应的权限后，调用Environment的getExternalStorageDirectory方法即可获取路径。

**SQLite数据库**  
Android系统集成了一个轻量级的数据库：SQLite。SQLite是一个嵌入式数据库引擎，专门适用于资源有限的设备上适量数据的存取。在Android上我们一般使用SQLiteOpenHelper辅助类来操作SQLite数据库。

