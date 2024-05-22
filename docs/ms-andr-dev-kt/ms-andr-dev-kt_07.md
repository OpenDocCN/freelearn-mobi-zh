# 第七章：使用数据库

在上一章中，我们获得了访问 Android 系统功能所需的关键权限。在我们的情况下，我们获得了位置权限。在本章中，我们将通过向数据库插入数据来继续。我们将插入来自 Android 位置提供程序的位置数据。为此，我们将定义适当的数据库模式和管理类。我们还将定义用于访问位置提供程序以获取位置数据的类。

在本章中，我们将涵盖以下主题：

+   SQLite 简介

+   描述数据库

+   CRUD 操作

# SQLite 简介

为了保存我们应用程序的数据，我们将需要一个数据库。在 Android 中，可以使用 SQLite 进行离线数据存储。

SQLite 是开箱即用的，这意味着它已经包含在 Android 框架中。

# 好处

SQLite 的好处是它功能强大、快速、可靠，并且有一个庞大的社区在使用它。如果您遇到任何问题，很容易找到解决方案，因为社区中的某人很可能已经解决了这些问题。SQLite 是一个独立的、嵌入式的、功能齐全的、公共领域的 SQL 数据库引擎。

我们将使用 SQLite 来存储所有我们的 Todos 和 Notes。为此，我们将定义我们的数据库、访问它的机制以及数据管理。我们不会直接暴露一个裸的数据库实例，而是会适当地包装它，以便轻松地插入、更新、查询或删除数据。

# 描述我们的数据库

我们将首先通过定义其表和列以及适当的数据类型来描述我们的数据库。我们还将定义简单的模型来表示我们的数据。为此，请创建一个名为`database`的新包：

```kt
     com.journaler.database 
```

然后，创建一个名为`DbModel`的新的 Kotlin 类。`DbModel`类将表示我们应用程序的所有数据库模型的矩阵，并且只包含 ID，因为 ID 是一个必填字段，并且将用作主键。确保您的`DbModel`类看起来像这样：

```kt
    package com.journaler.database 

    abstract class DbModel { 
      abstract var id: Long 
    } 
```

现在，当我们定义了我们的起点后，我们将定义实际包含数据的数据类。在我们现有的名为`model`的包中，创建新的类--`DbEntry`、`Note`和`Todo`。`Note`和`Todo`将扩展`Entry`，而`Entry`又扩展了`DbModel`类。

`Entry`类的代码如下：

```kt
    package com.journaler.model 

    import android.location.Location 
    import com.journaler.database.DbModel 

    abstract class Entry( 
      var title: String, 
      var message: String, 
      var location: Location 
    ) : DbModel() 
    Note class: 
    package com.journaler.model 

    import android.location.Location 

    class Note( 
      title: String, 
      message: String, 
      location: Location 
    ) : Entry( 
        title, 
        message, 
        location 
        ) { 
         override var id = 0L 
        } 
```

您将注意到我们将当前地理位置作为存储在我们的笔记中的信息，以及`title`和笔记`message`内容。我们还重写了 ID。由于新实例化的`note`尚未存储到数据库中，因此其 ID 将为零。存储后，它将更新为从数据库获取的 ID 值。

`Todo`类：

```kt
    package com.journaler.model 

    import android.location.Location 

    class Todo( 
      title: String, 
      message: String, 
      location: Location, 
      var scheduledFor: Long 
    ) : Entry( 
        title, 
        message, 
        location 
    ) { 
    override var id = 0L 
    } 
```

`Todo`类将比`Note`类多一个字段--用于安排`todo`的`timestamp`。

现在，在我们定义了数据模型之后，我们将描述我们的数据库。我们必须定义负责数据库初始化的数据库助手类。数据库助手类必须扩展 Android 的`SQLiteOpenHelper`类。创建`DbHelper`类，并确保它扩展了`SQLiteOpenHelper`类：

```kt
    package com.journaler.database 

    import android.database.sqlite.SQLiteDatabase 
    import android.database.sqlite.SQLiteOpenHelper 
    import android.util.Log 
    import com.journaler.Journaler 

    class DbHelper(val dbName: String, val version: Int) :   
    SQLiteOpenHelper( 
      Journaler.ctx, dbName, null, version 
    ) { 

      companion object { 
        val ID: String = "_id" 
        val TABLE_TODOS = "todos" 
        val TABLE_NOTES = "notes" 
        val COLUMN_TITLE: String = "title" 
        val COLUMN_MESSAGE: String = "message" 
        val COLUMN_SCHEDULED: String = "scheduled" 
        val COLUMN_LOCATION_LATITUDE: String = "latitude" 
        val COLUMN_LOCATION_LONGITUDE: String = "longitude" 
      } 

      private val tag = "DbHelper" 

      private val createTableNotes =  """ 
        CREATE TABLE if not exists $TABLE_NOTES 
           ( 
             $ID integer PRIMARY KEY autoincrement, 
             $COLUMN_TITLE text, 
             $COLUMN_MESSAGE text, 
             $COLUMN_LOCATION_LATITUDE real, 
             $COLUMN_LOCATION_LONGITUDE real 
           ) 
          """ 

      private val createTableTodos =  """ 
        CREATE TABLE if not exists $TABLE_TODOS 
           ( 
              $ID integer PRIMARY KEY autoincrement, 
              $COLUMN_TITLE text, 
              $COLUMN_MESSAGE text, 
              $COLUMN_SCHEDULED integer, 
              $COLUMN_LOCATION_LATITUDE real, 
              $COLUMN_LOCATION_LONGITUDE real 
           ) 
         """ 

       override fun onCreate(db: SQLiteDatabase) { 
        Log.d(tag, "Database [ CREATING ]") 
        db.execSQL(createTableNotes) 
        db.execSQL(createTableTodos) 
        Log.d(tag, "Database [ CREATED ]") 
       } 

      override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int,
      newVersion: Int) { 
        // Ignore for now. 
      } 

    } 
```

我们的`companion`对象包含了表和列名称的定义。我们还定义了用于创建表的 SQL。最后，SQL 在`onCreate()`方法中执行。在下一节中，我们将进一步进行数据库管理，并最终插入一些数据。

# CRUD 操作

CRUD 操作是用于创建、更新、选择或删除数据的操作。它们是用一个名为`Crud`的接口定义的，并且它将是通用的。在`database`包中创建一个新的接口。确保它涵盖所有 CRUD 操作：

```kt
     interface Crud<T> where T : DbModel { 

       companion object { 
        val BROADCAST_ACTION = "com.journaler.broadcast.crud" 
        val BROADCAST_EXTRAS_KEY_CRUD_OPERATION_RESULT = "crud_result" 
       } 

      /** 
       * Returns the ID of inserted item. 
       */ 
      fun insert(what: T): Long 

      /** 
       * Returns the list of inserted IDs. 
       */ 
      fun insert(what: Collection<T>): List<Long> 

      /** 
      * Returns the number of updated items. 
      */ 
      fun update(what: T): Int 

      /** 
      * Returns the number of updated items. 
      */ 
      fun update(what: Collection<T>): Int 

      /** 
      * Returns the number of deleted items. 
      */ 
      fun delete(what: T): Int 

      /** 
      * Returns the number of deleted items. 
      */ 
      fun delete(what: Collection<T>): Int 

      /** 
      * Returns the list of items. 
      */ 
      fun select(args: Pair<String, String>): List<T> 

      /** 
      * Returns the list of items. 
      */ 
      fun select(args: Collection<Pair<String, String>>): List<T> 

      /** 
      * Returns the list of items. 
      */ 
      fun selectAll(): List<T> 

    } 
```

要执行 CRUD 操作，有两种方法版本。第一个版本是接受*实例集合*的版本，第二个版本是*接受单个项目*的版本。让我们通过创建一个名为`Db`的 Kotlin 对象来创建 CRUD 具体化。创建一个对象使我们的具体化成为一个完美的单例。`Db`对象必须实现`Crud`接口：

```kt
     package com.journaler.database 

     import android.content.ContentValues 
     import android.location.Location 
     import android.util.Log 
     import com.journaler.model.Note 
     import com.journaler.model.Todo 

     object Db { 

      private val tag = "Db" 
      private val version = 1 
      private val name = "students" 

      val NOTE = object : Crud<Note> { 
        // Crud implementations 
      } 

      val TODO = object : Crud<NoteTodo { 
         // Crud implementations 
      } 
    }  
```

# 插入 CRUD 操作

插入操作将新数据添加到数据库中。其实现如下：

```kt
    val NOTE = object : Crud<Note> { 
      ... 
      override fun insert(what: Note): Long { 
        val inserted = insert(listOf(what)) 
        if (!inserted.isEmpty()) return inserted[0] 
        return 0 
      } 

     override fun insert(what: Collection<Note>): List<Long> { 
       val db = DbHelper(name, version).writableDatabase 
       db.beginTransaction() 
       var inserted = 0 
       val items = mutableListOf<Long>() 
       what.forEach { item -> 
         val values = ContentValues() 
         val table = DbHelper.TABLE_NOTES 
         values.put(DbHelper.COLUMN_TITLE, item.title) 
         values.put(DbHelper.COLUMN_MESSAGE, item.message) 
         values.put(DbHelper.COLUMN_LOCATION_LATITUDE,
           item.location.latitude) 
         values.put(DbHelper.COLUMN_LOCATION_LONGITUDE,
           item.location.longitude) 
         val id = db.insert(table, null, values) 
           if (id > 0) { 
             items.add(id) 
             Log.v(tag, "Entry ID assigned [ $id ]") 
               inserted++ 
             } 
           } 
           val success = inserted == what.size 
           if (success) { 
                db.setTransactionSuccessful() 
           } else { 
                items.clear() 
           } 
            db.endTransaction() 
            db.close() 
            return items 
          } 
          ... 
    } 
    ... 
    val TODO = object : Crud<Todo> { 
      ... 
      override fun insert(what: Todo): Long { 
        val inserted = insert(listOf(what)) 
        if (!inserted.isEmpty()) return inserted[0] 
        return 0 
      } 

      override fun insert(what: Collection<Todo>): List<Long> { 
        val db = DbHelper(name, version).writableDatabase 
        db.beginTransaction() 
        var inserted = 0 
        val items = mutableListOf<Long>() 
        what.forEach { item -> 
          val table = DbHelper.TABLE_TODOS 
          val values = ContentValues() 
          values.put(DbHelper.COLUMN_TITLE, item.title) 
          values.put(DbHelper.COLUMN_MESSAGE, item.message) 
          values.put(DbHelper.COLUMN_LOCATION_LATITUDE,
          item.location.latitude) 
          values.put(DbHelper.COLUMN_LOCATION_LONGITUDE,
          item.location.longitude) 
          values.put(DbHelper.COLUMN_SCHEDULED, item.scheduledFor) 
            val id = db.insert(table, null, values) 
            if (id > 0) { 
              item.id = id 
              Log.v(tag, "Entry ID assigned [ $id ]") 
              inserted++ 
            } 
           } 
           val success = inserted == what.size 
           if (success) { 
                db.setTransactionSuccessful() 
           } else { 
               items.clear() 
           } 
           db.endTransaction() 
           db.close() 
           return items 
          } 
         ... 
     } 
     ... 
```

# 更新 CRUD 操作

更新操作将更新我们数据库中的现有数据。其实现如下：

```kt
    val NOTE = object : Crud<Note> { 
       ... 
       override fun update(what: Note) = update(listOf(what)) 

       override fun update(what: Collection<Note>): Int { 
         val db = DbHelper(name, version).writableDatabase 
         db.beginTransaction() 
         var updated = 0 
         what.forEach { item -> 
           val values = ContentValues() 
           val table = DbHelper.TABLE_NOTES 
           values.put(DbHelper.COLUMN_TITLE, item.title) 
           values.put(DbHelper.COLUMN_MESSAGE, item.message) 
           values.put(DbHelper.COLUMN_LOCATION_LATITUDE,
           item.location.latitude) 
           values.put(DbHelper.COLUMN_LOCATION_LONGITUDE,
           item.location.longitude) 
           db.update(table, values, "_id = ?", 
           arrayOf(item.id.toString())) 
                updated++ 
           } 
           val result = updated == what.size 
           if (result) { 
             db.setTransactionSuccessful() 
           } else { 
             updated = 0 
           } 
           db.endTransaction() 
           db.close() 
           return updated 
          } 
          ... 
        } 
        ... 
      val TODO = object : Crud<Todo> { 
        ... 
        override fun update(what: Todo) = update(listOf(what)) 

        override fun update(what: Collection<Todo>): Int { 
          val db = DbHelper(name, version).writableDatabase 
          db.beginTransaction() 
          var updated = 0 
          what.forEach { item -> 
             val table = DbHelper.TABLE_TODOS 
             val values = ContentValues() 
             values.put(DbHelper.COLUMN_TITLE, item.title) 
             values.put(DbHelper.COLUMN_MESSAGE, item.message) 
             values.put(DbHelper.COLUMN_LOCATION_LATITUDE,
             item.location.latitude) 
            values.put(DbHelper.COLUMN_LOCATION_LONGITUDE,
            item.location.longitude) 
            values.put(DbHelper.COLUMN_SCHEDULED, item.scheduledFor) 
            db.update(table, values, "_id = ?",  
            arrayOf(item.id.toString())) 
               updated++ 
            } 
            val result = updated == what.size 
            if (result) { 
              db.setTransactionSuccessful() 
            } else { 
              updated = 0 
            } 
            db.endTransaction() 
            db.close() 
            return updated 
            } 
           ... 
      } 
     ...  
```

# 删除 CRUD 操作

删除操作将从数据库中删除现有数据。其实现如下：

```kt
    val NOTE = object : Crud<Note> { 
      ... 
      override fun delete(what: Note): Int = delete(listOf(what)) 
         override fun delete(what: Collection<Note>): Int { 
         val db = DbHelper(name, version).writableDatabase 
         db.beginTransaction() 
         val ids = StringBuilder() 
         what.forEachIndexed { index, item -> 
         ids.append(item.id.toString()) 
           if (index < what.size - 1) { 
              ids.append(", ") 
           } 
         } 
         val table = DbHelper.TABLE_NOTES 
         val statement = db.compileStatement( 
           "DELETE FROM $table WHERE ${DbHelper.ID} IN ($ids);" 
         ) 
         val count = statement.executeUpdateDelete() 
         val success = count > 0 
         if (success) { 
           db.setTransactionSuccessful() 
           Log.i(tag, "Delete [ SUCCESS ][ $count ][ $statement ]") 
         } else { 
            Log.w(tag, "Delete [ FAILED ][ $statement ]") 
         } 
          db.endTransaction() 
          db.close() 
          return count 
        } 
        ... 
     } 
     ... 
     val TODO = object : Crud<Todo> { 
       ... 
       override fun delete(what: Todo): Int = delete(listOf(what)) 
       override fun delete(what: Collection<Todo>): Int { 
         val db = DbHelper(name, version).writableDatabase 
         db.beginTransaction() 
         val ids = StringBuilder() 
         what.forEachIndexed { index, item -> 
         ids.append(item.id.toString()) 
            if (index < what.size - 1) { 
                ids.append(", ") 
            } 
        } 
        val table = DbHelper.TABLE_TODOS 
        val statement = db.compileStatement( 
          "DELETE FROM $table WHERE ${DbHelper.ID} IN ($ids);" 
        ) 
        val count = statement.executeUpdateDelete() 
        val success = count > 0 
        if (success) { 
           db.setTransactionSuccessful() 
           Log.i(tag, "Delete [ SUCCESS ][ $count ][ $statement ]") 
        } else { 
           Log.w(tag, "Delete [ FAILED ][ $statement ]") 
        } 
         db.endTransaction() 
         db.close() 
         return count 
        } 
        ... 
    } 
    ...  
```

# 选择 CRUD 操作

选择操作将从数据库中读取并返回数据。其实现如下：

```kt
     val NOTE = object : Crud<Note> { 
        ... 
        override fun select( 
            args: Pair<String, String> 
        ): List<Note> = select(listOf(args)) 

        override fun select(args: Collection<Pair<String, String>>):
        List<Note> { 
          val db = DbHelper(name, version).writableDatabase 
          val selection = StringBuilder() 
          val selectionArgs = mutableListOf<String>() 
          args.forEach { arg -> 
              selection.append("${arg.first} == ?") 
              selectionArgs.add(arg.second) 
          } 
          val result = mutableListOf<Note>() 
          val cursor = db.query( 
              true, 
              DbHelper.TABLE_NOTES, 
              null, 
              selection.toString(), 
              selectionArgs.toTypedArray(), 
              null, null, null, null 
          ) 
          while (cursor.moveToNext()) { 
          val id = cursor.getLong(cursor.getColumnIndexOrThrow
          (DbHelper.ID)) 
          val titleIdx = cursor.getColumnIndexOrThrow
          (DbHelper.COLUMN_TITLE) 
          val title = cursor.getString(titleIdx) 
          val messageIdx = cursor.getColumnIndexOrThrow
          (DbHelper.COLUMN_MESSAGE) 
          val message = cursor.getString(messageIdx) 
          val latitudeIdx = cursor.getColumnIndexOrThrow( 
             DbHelper.COLUMN_LOCATION_LATITUDE 
          ) 
          val latitude = cursor.getDouble(latitudeIdx) 
          val longitudeIdx = cursor.getColumnIndexOrThrow( 
             DbHelper.COLUMN_LOCATION_LONGITUDE 
          ) 
          val longitude = cursor.getDouble(longitudeIdx) 
          val location = Location("") 
          location.latitude = latitude 
          location.longitude = longitude 
          val note = Note(title, message, location) 
          note.id = id 
          result.add(note) 
        } 
          cursor.close() 
          return result 
       } 

       override fun selectAll(): List<Note> { 
         val db = DbHelper(name, version).writableDatabase 
         val result = mutableListOf<Note>() 
         val cursor = db.query( 
            true, 
            DbHelper.TABLE_NOTES, 
            null, null, null, null, null, null, null 
         ) 
         while (cursor.moveToNext()) { 
                val id = cursor.getLong(cursor.getColumnIndexOrThrow
               (DbHelper.ID)) 
                val titleIdx = cursor.getColumnIndexOrThrow
                (DbHelper.COLUMN_TITLE) 
                val title = cursor.getString(titleIdx) 
                val messageIdx = cursor.getColumnIndexOrThrow
                (DbHelper.COLUMN_MESSAGE) 
                val message = cursor.getString(messageIdx) 
                val latitudeIdx = cursor.getColumnIndexOrThrow( 
                  DbHelper.COLUMN_LOCATION_LATITUDE 
                ) 
                val latitude = cursor.getDouble(latitudeIdx) 
                val longitudeIdx = cursor.getColumnIndexOrThrow( 
                   DbHelper.COLUMN_LOCATION_LONGITUDE 
                ) 
                val longitude = cursor.getDouble(longitudeIdx) 
                val location = Location("") 
                location.latitude = latitude 
                location.longitude = longitude 
                val note = Note(title, message, location) 
                note.id = id 
                result.add(note) 
              } 
             cursor.close() 
             return result 
            } 
            ... 
          } 
          ... 
       val TODO = object : Crud<Todo> { 
        ... 
        override fun select(args: Pair<String, String>): List<Todo> =
        select(listOf(args)) 

        override fun select(args: Collection<Pair<String, String>>): 
        List<Todo> { 
          val db = DbHelper(name, version).writableDatabase 
          val selection = StringBuilder() 
          val selectionArgs = mutableListOf<String>() 
          args.forEach { arg -> 
             selection.append("${arg.first} == ?") 
             selectionArgs.add(arg.second) 
          } 
          val result = mutableListOf<Todo>() 
          val cursor = db.query( 
             true, 
             DbHelper.TABLE_NOTES, 
             null, 
             selection.toString(), 
             selectionArgs.toTypedArray(), 
             null, null, null, null 
            ) 
            while (cursor.moveToNext()) { 
                val id = cursor.getLong(cursor.getColumnIndexOrThrow
                (DbHelper.ID)) 
                val titleIdx = cursor.getColumnIndexOrThrow
                (DbHelper.COLUMN_TITLE) 
                val title = cursor.getString(titleIdx) 
                val messageIdx = cursor.getColumnIndexOrThrow
                (DbHelper.COLUMN_MESSAGE) 
                val message = cursor.getString(messageIdx) 
                val latitudeIdx = cursor.getColumnIndexOrThrow( 
                    DbHelper.COLUMN_LOCATION_LATITUDE 
                ) 
                val latitude = cursor.getDouble(latitudeIdx) 
                val longitudeIdx = cursor.getColumnIndexOrThrow( 
                    DbHelper.COLUMN_LOCATION_LONGITUDE 
                ) 
                val longitude = cursor.getDouble(longitudeIdx) 
                val location = Location("") 
                val scheduledForIdx = cursor.getColumnIndexOrThrow( 
                    DbHelper.COLUMN_SCHEDULED 
                ) 
                val scheduledFor = cursor.getLong(scheduledForIdx) 
                location.latitude = latitude 
                location.longitude = longitude 
                val todo = Todo(title, message, location, scheduledFor) 
                todo.id = id 
                result.add(todo) 
               } 
              cursor.close() 
              return result 
            } 

            override fun selectAll(): List<Todo> { 
            val db = DbHelper(name, version).writableDatabase 
            val result = mutableListOf<Todo>() 
            val cursor = db.query( 
              true, 
              DbHelper.TABLE_NOTES, 
              null, null, null, null, null, null, null 
            ) 
            while (cursor.moveToNext()) { 
                val id = cursor.getLong(cursor.getColumnIndexOrThrow
                (DbHelper.ID)) 
                val titleIdx = cursor.getColumnIndexOrThrow
                (DbHelper.COLUMN_TITLE) 
                val title = cursor.getString(titleIdx) 
                val messageIdx = cursor.getColumnIndexOrThrow
                (DbHelper.COLUMN_MESSAGE) 
                val message = cursor.getString(messageIdx) 
                val latitudeIdx = cursor.getColumnIndexOrThrow( 
                    DbHelper.COLUMN_LOCATION_LATITUDE 
                ) 
                val latitude = cursor.getDouble(latitudeIdx) 
                val longitudeIdx = cursor.getColumnIndexOrThrow( 
                    DbHelper.COLUMN_LOCATION_LONGITUDE 
                ) 
                val longitude = cursor.getDouble(longitudeIdx) 
                val location = Location("") 
                val scheduledForIdx = cursor.getColumnIndexOrThrow( 
                    DbHelper.COLUMN_SCHEDULED 
                ) 
                val scheduledFor = cursor.getLong(scheduledForIdx) 
                location.latitude = latitude 
                location.longitude = longitude 
                val todo = Todo(title, message, location, scheduledFor) 
                todo.id = id 
                result.add(todo) 
              } 
              cursor.close() 
               return result 
             } 
             ... 
        } 
        ... 
```

每个 CRUD 操作都将使用我们的`DbHelper`类获取数据库实例。我们不会直接暴露它，而是通过我们的 CRUD 机制来利用它。每次操作后，数据库都将被关闭。我们只能通过访问`writableDatabase`来获取可读数据库或者像我们的情况一样获取`WritableDatabase`实例。每个 CRUD 操作都作为一个 SQL 事务执行。这意味着我们将通过在数据库实例上调用`beginTransaction()`来开始它。通过调用`endTransaction()`来完成事务。如果我们在之前没有调用`setTransactionSuccessful()`，则不会应用任何更改。正如我们已经提到的，每个 CRUD 操作都有两个版本--一个包含主要实现，另一个只是将实例传递给另一个。要执行对数据库的插入，重要的是要注意我们将在数据库实例上使用`insert()`方法，该方法接受我们要插入的表名和代表数据的内容值（`ContentValues`类）。`update`和`delete`操作类似。我们使用`update()`和`delete()`方法。在我们的情况下，对于数据删除，我们使用了包含删除 SQL 查询的`compileStatement()`。

我们在这里提供的代码有点复杂。我们直接指向了与数据库相关的事项。所以，请耐心阅读代码，慢慢来，花时间来研究它。我们鼓励您利用我们已经提到的 Android 数据库类，以您自己的方式创建自己的数据库管理类。

# 将事物联系在一起

我们还有一步！那就是实际使用我们的数据库类并执行 CRUD 操作。我们将扩展应用程序以创建笔记，并专注于插入。

在我们向数据库中插入任何内容之前，我们必须提供一种机制来获取当前用户位置，因为这对于`notes`和`todos`都是必需的。创建一个名为`LocationProvider`的新类，并将其定位在`location`包中，如下所示：

```kt
     object LocationProvider { 
       private val tag = "Location provider" 
       private val listeners =   CopyOnWriteArrayList
       <WeakReference<LocationListener>>() 

       private val locationListener = object : LocationListener { 
       ... 
       } 

      fun subscribe(subscriber: LocationListener): Boolean { 
        val result = doSubscribe(subscriber) 
        turnOnLocationListening() 
        return result 
      } 

      fun unsubscribe(subscriber: LocationListener): Boolean { 
        val result = doUnsubscribe(subscriber) 
        if (listeners.isEmpty()) { 
            turnOffLocationListening() 
        } 
        return result 
      } 

      private fun turnOnLocationListening() { 
      ... 
      } 

      private fun turnOffLocationListening() { 
      ... 
      } 

      private fun doSubscribe(listener: LocationListener): Boolean { 
      ... 
      } 

      private fun doUnsubscribe(listener: LocationListener): Boolean { 
       ... 
      } 
    } 
```

我们公开了`LocationProvider`对象的主要结构。让我们来看看其余的实现：

`locationListener`实例代码如下：

```kt
     private val locationListener = object : LocationListener { 
        override fun onLocationChanged(location: Location) { 
            Log.i( 
                    tag, 
                    String.format( 
                            Locale.ENGLISH, 
                            "Location [ lat: %s ][ long: %s ]",
                            location.latitude, location.longitude 
                    ) 
            ) 
            val iterator = listeners.iterator() 
            while (iterator.hasNext()) { 
                val reference = iterator.next() 
                val listener = reference.get() 
                listener?.onLocationChanged(location) 
            } 
         } 

        override fun onStatusChanged(provider: String, status: Int,
        extras: Bundle) { 
            Log.d( 
                    tag, 
                    String.format(Locale.ENGLISH, "Status changed [ %s
                    ][ %d ]", provider, status) 
            ) 
            val iterator = listeners.iterator() 
            while (iterator.hasNext()) { 
                val reference = iterator.next() 
                val listener = reference.get() 
                listener?.onStatusChanged(provider, status, extras) 
            } 
        } 

        override fun onProviderEnabled(provider: String) { 
            Log.i(tag, String.format("Provider [ %s ][ ENABLED ]",
            provider)) 
            val iterator = listeners.iterator() 
            while (iterator.hasNext()) { 
                val reference = iterator.next() 
                val listener = reference.get() 
                listener?.onProviderEnabled(provider) 
            } 
        } 

        override fun onProviderDisabled(provider: String) { 
            Log.i(tag, String.format("Provider [ %s ][ ENABLED ]",
            provider)) 
            val iterator = listeners.iterator() 
            while (iterator.hasNext()) { 
                val reference = iterator.next() 
                val listener = reference.get() 
                listener?.onProviderDisabled(provider) 
            } 
          } 
         } 
```

`LocationListener`是 Android 的接口，其目的是在`location`事件上执行。我们创建了我们的具体化，基本上会通知所有订阅方有关这些事件的信息。对我们来说最重要的是`onLocationChanged()`：

```kt
    turnOnLocationListening(): 

    private fun turnOnLocationListening() { 
       Log.v(tag, "We are about to turn on location listening.") 
       val ctx = Journaler.ctx 
       if (ctx != null) { 
            Log.v(tag, "We are about to check location permissions.") 

            val permissionsOk = 
            ActivityCompat.checkSelfPermission(ctx,
            Manifest.permission.ACCESS_FINE_LOCATION) ==  
            PackageManager.PERMISSION_GRANTED  
            &&  
            ActivityCompat.checkSelfPermission(ctx, 
            Manifest.permission.ACCESS_COARSE_LOCATION) ==
            PackageManager.PERMISSION_GRANTED 

            if (!permissionsOk) { 
                throw IllegalStateException( 
                "Permissions required [ ACCESS_FINE_LOCATION ]
                 [ ACCESS_COARSE_LOCATION ]" 
                ) 
            } 
            Log.v(tag, "Location permissions are ok. 
            We are about to request location changes.") 
            val locationManager =
            ctx.getSystemService(Context.LOCATION_SERVICE)
            as LocationManager 

            val criteria = Criteria() 
            criteria.accuracy = Criteria.ACCURACY_FINE 
            criteria.powerRequirement = Criteria.POWER_HIGH 
            criteria.isAltitudeRequired = false 
            criteria.isBearingRequired = false 
            criteria.isSpeedRequired = false 
            criteria.isCostAllowed = true 

            locationManager.requestLocationUpdates( 
                    1000, 1F, criteria, locationListener, 
                    Looper.getMainLooper() 
            ) 
            } else { 
             Log.e(tag, "No application context available.") 
          } 
        } 
```

要打开位置监听，我们必须检查权限是否得到了正确的满足。如果是这样，那么我们将获取 Android 的`LocationManager`并为位置更新定义`Criteria`。我们将我们的标准定义为非常精确和准确。最后，我们通过传递以下参数来请求位置更新：

+   `long minTime`

+   `float minDistance`

+   `Criteria criteria`

+   `LocationListener listener`

+   `Looper looper`

正如您所看到的，我们传递了我们的`LocationListener`具体化，它将通知所有订阅的第三方有关`location`事件的信息：

```kt
     turnOffLocationListening():private fun turnOffLocationListening() 
     { 
       Log.v(tag, "We are about to turn off location listening.") 
       val ctx = Journaler.ctx 
       if (ctx != null) { 
         val locationManager =  
         ctx.getSystemService(Context.LOCATION_SERVICE)
         as LocationManager 

         locationManager.removeUpdates(locationListener) 
        } else { 
            Log.e(tag, "No application context available.") 
        } 
     } 
```

+   我们通过简单地移除我们的监听器`instance.doSubscribe()`来停止监听位置。

```kt
      private fun doSubscribe(listener: LocationListener): Boolean { 
        val iterator = listeners.iterator() 
        while (iterator.hasNext()) { 
          val reference = iterator.next() 
          val refListener = reference.get() 
          if (refListener != null && refListener === listener) { 
                Log.v(tag, "Already subscribed: " + listener) 
                return false 
            } 
         } 
         listeners.add(WeakReference(listener)) 
         Log.v(tag, "Subscribed, subscribers count: " + listeners.size) 
         return true 
      }  
```

+   `doUnsubscribe()`方法代码如下：

```kt
      private fun doUnsubscribe(listener: LocationListener): Boolean { 
        var result = true 
        val iterator = listeners.iterator() 
        while (iterator.hasNext()) { 
            val reference = iterator.next() 
            val refListener = reference.get() 
            if (refListener != null && refListener === listener) { 
                val success = listeners.remove(reference) 
                if (!success) { 
                    Log.w(tag, "Couldn't un subscribe, subscribers
                    count: " + listeners.size) 
                } else { 
                    Log.v(tag, "Un subscribed, subscribers count: " +
                    listeners.size) 
                } 
                if (result) { 
                    result = success 
                } 
               } 
             } 
            return result 
        } 
```

这两种方法负责订阅和取消订阅位置更新给感兴趣的第三方。

我们已经拥有了所需的一切。打开`NoteActivity`类并扩展如下：

```kt
     class NoteActivity : ItemActivity() { 
       private var note: Note? = null 
       override val tag = "Note activity" 
       private var location: Location? = null 
       override fun getLayout() = R.layout.activity_note 

      private val textWatcher = object : TextWatcher { 
        override fun afterTextChanged(p0: Editable?) { 
            updateNote() 
        } 

        override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2:
        Int, p3: Int) {} 
        override fun onTextChanged(p0: CharSequence?, p1: Int, p2:
        Int, p3: Int) {} 
      } 

      private val locationListener = object : LocationListener { 
        override fun onLocationChanged(p0: Location?) { 
            p0?.let { 
                LocationProvider.unsubscribe(this) 
                location = p0 
                val title = getNoteTitle() 
                val content = getNoteContent() 
                note = Note(title, content, p0) 
                val task = object : AsyncTask<Note, Void, Boolean>() { 
                    override fun doInBackground(vararg params: Note?):
                    Boolean { 
                        if (!params.isEmpty()) { 
                            val param = params[0] 
                            param?.let { 
                                return Db.NOTE.insert(param) > 0 
                            } 
                        } 
                        return false 
                    } 

                    override fun onPostExecute(result: Boolean?) { 
                        result?.let { 
                            if (result) { 
                                Log.i(tag, "Note inserted.") 
                            } else { 
                                Log.e(tag, "Note not inserted.") 
                            } 
                        } 
                     } 
                  } 
                task.execute(note) 
              } 
          } 

         override fun onStatusChanged(p0: String?, p1: Int, p2:
         Bundle?) {} 
         override fun onProviderEnabled(p0: String?) {} 
         override fun onProviderDisabled(p0: String?) {} 
        } 

        override fun onCreate(savedInstanceState: Bundle?) { 
          super.onCreate(savedInstanceState) 
          note_title.addTextChangedListener(textWatcher) 
          note_content.addTextChangedListener(textWatcher) 
        } 

       private fun updateNote() { 
         if (note == null) { 
          if (!TextUtils.isEmpty(getNoteTitle()) &&
          !TextUtils.isEmpty(getNoteContent())) { 
             LocationProvider.subscribe(locationListener) 
          } 
         } else { 
            note?.title = getNoteTitle() 
            note?.message = getNoteContent() 
            val task = object : AsyncTask<Note, Void, Boolean>() { 
                override fun doInBackground(vararg params: Note?):
            Boolean { 
              if (!params.isEmpty()) { 
                 val param = params[0] 
                 param?.let { 
                   return Db.NOTE.update(param) > 0 
                  } 
                } 
                  return false 
              } 

              override fun onPostExecute(result: Boolean?) { 
                result?.let { 
                   if (result) { 
                       Log.i(tag, "Note updated.") 
                   } else { 
                       Log.e(tag, "Note not updated.") 
                   } 
                 } 
               } 
            } 
            task.execute(note) 
          } 
       } 

       private fun getNoteContent(): String { 
         return note_content.text.toString() 
       } 

       private fun getNoteTitle(): String { 
         return note_title.text.toString() 
       } 

     } 
```

我们在这里做了什么？让我们从上到下解释一切！我们添加了两个字段——一个包含我们正在编辑的当前`Note`实例，另一个包含当前用户位置信息。然后，我们定义了一个`TextWatcher`实例。`TextWatcher`是一个监听器，我们将分配给我们的`EditText`视图，每次更改时，适当的更新方法将被触发。该方法将创建一个新的`note`类，并将其持久化到数据库中（如果不存在），或者如果存在，则执行数据更新。

由于在没有位置数据可用之前，我们不会插入笔记，因此我们定义了我们的`locationListener`将接收到的位置放入位置字段中，并取消订阅自身。然后，我们将获取`note`标题和其主要内容的当前值，并创建一个新的`note`实例。由于数据库操作可能需要一些时间，我们将以异步方式执行它们。为此，我们将使用`AsyncTask`类。`AsyncTask`类是 Android 的类，旨在用于大多数异步操作。该类定义了输入类型、进度类型和结果类型。在我们的情况下，输入类型是`Note`。我们没有进度类型，但我们有一个结果类型`Boolean`，即操作是否成功。

主要工作是在`doInBackground()`具体化中完成的，而结果在`onPostExecute()`中处理。正如你所看到的，我们正在使用我们最近为数据库管理定义的类在后台执行插入操作。

如果你继续查看，我们接下来要做的事情是将`textWatcher`分配给`onCreate()`方法中的`EditText`视图。然后，我们定义了我们最重要的方法——`updateNote()`。它将更新现有的笔记，如果不存在，则插入一个新的笔记。同样，我们使用`AsyncTask`在后台执行操作。

构建你的应用程序并运行它。尝试插入`note`。观察你的 Logcat。你会注意到与数据库相关的日志，如下所示：

```kt
    I/Note activity: Note inserted. 
    I/Note activity: Note updated. 
    I/Note activity: Note updated. 
    I/Note activity: Note updated. 
```

如果你能看到这些日志，那么你已经成功在 Android 中实现了你的第一个数据库。我们鼓励你扩展代码以支持其他 CRUD 操作。确保`NoteActivity`支持`select`和`delete`操作。

# 总结

在本章中，我们演示了如何在 Android 中持久化复杂数据。数据库是每个应用程序的核心，所以 Journaler 也不例外。我们涵盖了在 SQLite 数据库上执行的所有 CRUD 操作，并为每个操作提供了适当的实现。在我们的下一章中，我们将演示另一种持久化机制，用于较不复杂的数据。我们将处理 Android 共享首选项，并将使用它们来保存我们应用程序的简单小数据。
