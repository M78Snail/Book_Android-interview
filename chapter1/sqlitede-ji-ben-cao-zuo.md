SQLite为熟悉SQL语句的程序员提供了相应的函数使用SQL语句，也为不了解SQL语法的程序员提供了简便的增删查改接口：

```java
String path = "数据库路径";  
SQLiteDatabase database = SQLiteDatabase.openOrCreateDatabase(path, null);  
// 执行sql语句  
database.execSQL(sql);  
// 执行带占位符的sql语句  
database.execSQL(sql, bindArgs);  
// 执行查找的sql语句  
database.rawQuery(sql, selectionArgs);  
  
// 执行增删查改  
database.insert(table, nullColumnHack, values);  
database.delete(table, whereClause, whereArgs);  
database.query(table, columns, selection, selectionArgs, groupBy, having, orderBy);  
database.update(table, values, whereClause, whereArgs);  
  
// 开启事务  
database.beginTransaction();  
// 确认事务成功  
database.setTransactionSuccessful();  
// 结束事务  
database.endTransaction(); 
```



