# 第二十七章：Android 数据库

如果我们要制作提供给用户重要功能的应用程序，那么我们几乎肯定需要一种管理、存储和过滤大量数据的方法。

使用 JSON 可以高效地存储大量数据，但当我们需要有选择地使用数据而不仅仅限制于“保存所有”和“加载所有”的选项时，我们需要考虑其他可用的选项。

一门优秀的计算机科学课程可能会教你处理排序和过滤数据所需的算法，但所需的工作量会相当大，我们能否想出与 Android API 提供的解决方案一样好的解决方案的机会有多大呢？

像往常一样，使用 Android API 中提供的解决方案是最合理的。正如我们所见，`JSON`和`SharedPreferences`类有它们的用途，但在某个时候，我们需要转向使用真正的数据库来解决现实世界的问题。Android 使用 SQLite 数据库管理系统，正如您所期望的那样，有一个 API 可以使其尽可能简单。

在本章中，我们将做以下事情：

+   确切地了解数据库是什么

+   了解 SQL 和 SQLite 是什么

+   学习 SQL 语言的基础知识

+   看一下 Android SQLite API

+   编写在上一章开始的 Age Database 应用程序

# 技术要求

您可以在 GitHub 上找到本章的代码文件[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2027`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2027)。

# 数据库 101

让我们回答一大堆与数据库相关的问题，然后我们就可以开始制作使用 SQLite 的应用程序。

## 什么是数据库？

**数据库**既是存储的地方，也是检索、存储和操作数据的手段。在学习如何使用之前，能够想象数据库是有帮助的。实际上，数据库内部的结构因所涉及的数据库而异。SQLite 实际上将所有数据存储在一个单个文件中。

然而，如果我们将我们的数据视为电子表格，或者有时是多个电子表格，它会极大地帮助我们理解。我们的数据库，就像电子表格一样，将被分成多个列，代表不同类型的数据，和行，代表数据库的条目。

想象一个具有姓名和考试成绩的数据库。看一下这种数据的可视化表示，并想象它在数据库中会是什么样子：

![图 27.1 – 数据库示例](img/Figure_27.1_B16773.jpg)

图 27.1 – 数据库示例

然而，请注意，还有一列额外的数据：一个**ID**列。随着我们的进行，我们将更多地谈论这个。这种类似电子表格的结构称为**表**。如前所述，数据库中可能有多个表。表的每一列都将有一个名称，在与数据库交谈时可以引用该名称。

## 什么是 SQL？

**SQL**代表**Structured Query Language**。这是用于处理数据库的语法。

## 什么是 SQLite？

SQLite 是 Android 所青睐的数据库系统的名称，并且它有自己的 SQL 版本。SQLite 版本的 SQL 需要稍微不同的原因是数据库具有不同的特性。

接下来的 SQL 语法入门将专注于 SQLite。

# SQL 语法入门

在我们学习如何在 Android 中使用 SQLite 之前，我们需要首先学习如何在一般情况下使用 SQLite 的基础知识。

让我们看一些示例 SQL 代码，可以直接在 SQLite 数据库上使用，而不需要任何 Java 或 Android 类；然后我们可以更容易地理解我们的 Java 代码在后面做什么。

## SQLite 示例代码

SQL 有关键字，就像 Java 一样，会引起一些事情发生。以下是一些我们很快将要使用的 SQL 关键字的例子：

+   `INSERT`：允许我们向数据库添加数据

+   `DELETE`：允许我们从数据库中删除数据

+   `SELECT`：允许我们从数据库中读取数据

+   `WHERE`：允许我们指定数据库的部分，匹配特定条件，我们想要在其上使用`INSERT`、`DELETE`或`SELECT`

+   `FROM`：用于指定数据库中的表或列名

注意

SQLite 的关键字远不止这些；要查看完整的关键字列表，请查看此链接：[`sqlite.org/lang_keywords.html`](https://sqlite.org/lang_keywords.html)。

除了关键字之外，SQL 还有**类型**。以下是一些 SQL 类型的示例：

+   **整数**：正好适合存储整数

+   **文本**：非常适合存储简单的姓名或地址

+   **实数**：用于存储大浮点数

注意

SQLite 的类型远不止这些；要查看完整的类型列表，请查看此链接：[`www.sqlite.org/datatype3.html`](https://www.sqlite.org/datatype3.html)。

让我们看看如何将这些类型与关键字结合起来，使用完整的 SQLite 语句创建表格并添加、删除、修改和读取数据。

### 创建表格

我们可能会问为什么我们不先创建一个新的数据库。原因是每个 Android 应用程序默认都可以访问一个 SQLite 数据库。该数据库对该应用程序是私有的。以下是我们在该数据库中创建表格的语句。我已经突出显示了一些部分，以便更清楚地理解语句：

```kt
create table StudentsAndGrades 
   _ID integer primary key autoincrement not null,
   name text not null,
   score int;
```

上述代码创建了一个名为`StudentsAndGrades`的表，其中有一个整数行 ID，每次添加一行数据时都会自动增加（递增）。

该表还将有一个`name`列，其类型为`text`，并且不能为空（`not null`）。

它还将有一个`score`列，其类型为`int`。同时，注意语句以分号结束。

### 向数据库中插入数据

以下是我们如何向数据库插入一行新数据的方式：

```kt
INSERT INTO StudentsAndGrades
   (name, score)
   VALUES
   ("Bart", 23);
```

上述代码向数据库添加了一行。在上述语句之后，数据库将有一个条目，其列（`_ID`，`name`，`score`）的值为（`1`，`Bart`，`23`）。

以下是我们如何向数据库插入另一行新数据的方式：

```kt
INSERT INTO StudentsAndGrades
   (name, score)
   VALUES
   ("Lisa", 100);
```

上述代码添加了一个新的数据行，其列（`_ID`，`name`，`score`）的值为（`2`，`Lisa`，`100`）。

我们的类似电子表格的结构现在看起来如下：

![图 27.2 - 更新后的电子表格](img/Figure_27.2_B16773.jpg)

图 27.2 - 更新后的电子表格

### 从数据库中检索数据

以下是我们如何从数据库中访问所有的行和列：

```kt
SELECT * FROM StudentsAndGrades;
```

上述代码要求每一行和每一列。`*`符号可以理解为“所有”。

我们也可以更加有选择性，就像这段代码所示：

```kt
SELECT score FROM StudentsAndGrades
     where name = "Lisa";
```

上述代码只会返回`100`，这当然是与姓名`Lisa`相关联的分数。

### 更新数据库结构

即使在创建表格并添加数据后，我们仍然可以添加新的列。就 SQL 而言，这很简单，但可能会导致已发布应用程序的用户数据出现一些问题。下一个语句添加了一个名为`age`的新列，其类型为`int`：

```kt
ALTER TABLE StudentsAndGrades
     ADD 
     age int;
```

有许多数据类型、关键字和使用它们的方式，比我们目前所见到的要多。接下来，让我们看一下 Android SQLite API；我们将开始看到如何使用我们的新的 SQLite 技能。

# Android SQLite API

Android API 有许多不同的方式，使得使用我们应用程序的数据库变得相当容易。我们需要首先熟悉的是`SQLiteOpenHelper`类。

## SQLiteOpenHelper 和 SQLiteDatabase

`SQLiteDatabase`类是表示实际数据库的类。然而，`SQLiteOpenHelper`类是大部分操作发生的地方。这个类将使我们能够访问数据库并初始化`SQLiteDatabase`的实例。

此外，`SQLiteOpenHelper`，我们将在我们的 Age Database 应用程序中扩展它，有两个要重写的方法。首先，它有一个`onCreate`方法，当第一次使用数据库时被调用；因此，我们将把用于创建表结构的 SQL 放在其中是有意义的。

我们必须重写的另一个方法是`onUpgrade`，你可能已经猜到，它在我们升级数据库时被调用（使用`ALTER`来改变其结构）。

## 构建和执行查询

随着我们的数据库结构变得更加复杂，以及我们的 SQL 知识的增长，我们的 SQL 语句会变得非常长和笨拙。出现错误的可能性很高。

我们将帮助解决复杂性问题的方法是将查询从各个部分构建成一个字符串。然后我们可以将该字符串传递给执行查询的方法。

此外，我们将使用`final`字符串来表示诸如表和列名之类的东西，这样我们就不会与它们搞混。

例如，我们可以声明以下成员，它们将代表之前虚构示例中的表名和列名。请注意，我们还将为数据库本身命名，并为其设置一个字符串：

```kt
private static final String DB_NAME = "MyCollegeDB";
private static final String TABLE_S_AND_G = " StudentsAndGrades";
public static final String TABLE_ROW_ID = "_id";
public static final String TABLE_ROW_NAME = "name";
public static final String TABLE_ROW_SCORE = "score";
```

请注意在前面的代码中，我们将受益于在类外部访问字符串，因为我们将它们声明为`public`。你可能会认为这违反了封装的规则。的确如此，但当类的意图是尽可能广泛地使用时，这是可以接受的。而且请记住，所有的变量都是 final 的。使用这些字符串变量的外部类不能改变它们或搞乱它们。它们只能引用和使用它们所持有的值。

然后我们可以像下面的示例一样构建一个查询。该示例向我们的假设数据库添加了一个新条目，并将 Java 变量合并到 SQL 语句中：

```kt
String name = "Onkar";
int score = 95;
// Add all the details to the table
String query = "INSERT INTO " + TABLE_S_AND_G + " (" +
         TABLE_ROW_NAME + ", " +
         TABLE_ROW_SCORE +
         ") " +
         "VALUES (" +
         "'" + name + "'" + ", " +
         score +
         ");"; 
```

请注意在前面的代码中，常规的`name`和`score` Java 变量被突出显示。之前的名为`query`的字符串现在是 SQL 语句，与此完全相同：

```kt
INSERT INTO StudentsAndGrades (
   name, score)
   VALUES ('Onkar',95);
```

注意

要学习 Android 编程并不一定要完全掌握前两个代码块。但是，如果你想构建自己的应用程序并构造确切需要的 SQL 语句，理解这些代码块将有所帮助。为什么不学习前两个代码块，以便区分双引号`"`，它们是用`+`连接在一起的字符串的一部分；单引号`'`，它们是 SQL 语法的一部分；常规的 Java 变量；以及字符串和 Java 中 SQL 语句中的不同分号。

在输入查询时，Android Studio 会提示我们变量的名称，这样错误的几率就会降低，尽管它比简单地输入查询更冗长。

现在我们可以使用之前介绍的类来执行查询：

```kt
// This is the actual database
private SQLiteDatabase db;
// Create an instance of our internal CustomSQLiteOpenHelper class
CustomSQLiteOpenHelper helper = new
   CustomSQLiteOpenHelper(context);
// Get a writable database
db = helper.getWritableDatabase();
// Run the query
db.execSQL(query);
```

在向数据库添加数据时，我们将像前面的代码一样使用`execSQL`；在从数据库获取数据时，我们将使用`rawQuery`方法，如下所示：

```kt
Cursor c = db.rawQuery(query, null); 
```

请注意，`rawQuery`方法返回`Cursor`类型的对象。

注意

我们可以用几种不同的方式与 SQLite 交互，它们各有优缺点。我们选择使用原始的 SQL 语句，因为这样可以完全透明地展示我们正在做什么，同时加强我们对 SQL 语言的了解。如果你想了解更多，请参阅下一个提示。

## 数据库游标

除了让我们访问数据库的类和允许我们执行查询的方法之外，还有一个问题，那就是我们从查询中得到的结果如何格式化。

幸运的是，有`Cursor`类。我们所有的数据库查询都会返回`Cursor`类型的对象。我们可以使用`Cursor`类的方法有选择地访问从查询返回的数据，如下所示：

```kt
Log.i(c.getString(1), c.getString(2)); 
```

以前的代码将输出到 logcat 中查询返回的前两列中存储的两个值。决定我们当前正在读取的返回数据的哪一行是`Cursor`对象本身。

我们可以访问`Cursor`对象的许多方法，包括`moveToNext`方法，该方法将`Cursor`移动到下一行，准备读取：

```kt
c.moveToNext();
/*
   This same code now outputs the data in the
   first and second column of the returned 
   data but from the SECOND row.
*/
Log.i(c.getString(1), c.getString(2));
```

在某些情况下，我们将能够将`Cursor`绑定到我们 UI 的一部分（例如`RecyclerView`），就像我们在 Note to Self 应用程序中使用`ArrayList`实例一样，然后将一切留给 Android API。

`Cursor`类还有许多有用的方法，其中一些我们很快就会看到。

注意

这是对 Android SQLite API 的介绍实际上只是触及了它的能力表面。随着我们进一步进行，我们将遇到更多的方法和类。然而，如果您的应用想法需要复杂的数据管理，进一步研究是值得的。

现在我们可以看到所有这些理论是如何结合在一起的，以及我们将如何在 Age Database 应用程序中构建我们的数据库代码结构。

# 编写数据库类

在这里，我们将实践我们迄今为止学到的一切，并完成编写 Age Database 应用程序。在我们之前的部分的`Fragment`类可以与共享数据库进行交互之前，我们需要一个类来处理与数据库的交互和创建。

我们将创建一个通过使用`SQLiteOpenHelper`类来管理我们的数据库的类。它还将定义一些`final`字符串来表示表的名称和其列。此外，它将提供一堆我们可以调用的辅助方法来执行所有必要的查询。在必要时，这些辅助方法将返回一个`Cursor`对象，我们可以用来显示我们检索到的数据。如果我们的应用程序需要发展，添加新的辅助方法将是微不足道的。

创建一个名为`DataManager`的新类，并添加以下成员变量：

```kt
import android.database.sqlite.SQLiteDatabase;
public class DataManager {
    // This is the actual database
    private SQLiteDatabase db;
    /*
        Next we have a public static final string for
        each row/table that we need to refer to both
        inside and outside this class
    */
    public static final String TABLE_ROW_ID = "_id";
    public static final String TABLE_ROW_NAME = "name";
    public static final String TABLE_ROW_AGE = "age";
    /*
        Next we have a private static final strings for
        each row/table that we need to refer to just
        inside this class
    */
    private static final String DB_NAME = "name_age_db";
    private static final int DB_VERSION = 1;
    private static final String TABLE_N_AND_A = 
                                   "name_and_age";
}
```

接下来，我们添加一个构造函数，它将创建我们的自定义版本的`SQLiteOpenHelper`的实例。我们很快将实现这个类作为一个内部类。构造函数还初始化了我们的`SQLiteDatabase`引用`db`成员。

将我们刚刚讨论过的以下构造函数添加到`DataManager`类中：

```kt
public DataManager(Context context) {
   // Create an instance of our internal 
   CustomSQLiteOpenHelper 

   CustomSQLiteOpenHelper helper = new 
      CustomSQLiteOpenHelper(context);
   // Get a writable database
   db = helper.getWritableDatabase();
}
```

现在我们可以添加我们将从 Fragment 类中访问的辅助方法。从`insert`方法开始，它根据传入方法的`name`和`age`参数执行`INSERT` SQL 查询。

将`insert`方法添加到`DataManager`类中：

```kt
// Here are all our helper methods
// Insert a record
public void insert(String name, String age){
   // Add all the details to the table
   String query = "INSERT INTO " + TABLE_N_AND_A + " (" +
                  TABLE_ROW_NAME + ", " +
                  TABLE_ROW_AGE +
                  ") " +
                  "VALUES (" +
                  "'" + name + "'" + ", " +
                  "'" + age + "'" +
                  ");";
   Log.i("insert() = ", query);
   db.execSQL(query);
}
```

下一个名为`delete`的方法将从数据库中删除一条记录，如果它在名称列中具有与传入的`name`参数匹配的值。它使用 SQL`DELETE`关键字来实现这一点。

将`delete`方法添加到`DataManager`类中：

```kt
// Delete a record
public void delete(String name){
   // Delete the details from the table if already exists
   String query = "DELETE FROM " + TABLE_N_AND_A +
                  " WHERE " + TABLE_ROW_NAME +
                  " = '" + name + "';";
   Log.i("delete() = ", query);
   db.execSQL(query);
}
```

接下来，我们有`selectAll`方法，它也如其名称所示。它使用`SELECT`查询并使用`*`参数来实现这一点，该参数相当于单独指定所有列。还要注意，该方法返回一个`Cursor`实例，我们将在一些`Fragment`类中使用。

将`selectAll`方法添加到`DataManager`类中：

```kt
// Get all the records
public Cursor selectAll() {
   Cursor c = db.rawQuery("SELECT *" +" from " +
                TABLE_N_AND_A, null);
   return c;
}
```

现在我们添加一个`searchName`方法，该方法具有一个`String`参数，用于用户想要搜索的名称。它还返回一个包含找到的所有条目的`Cursor`实例。请注意，SQL 语句使用`SELECT`，`FROM`和`WHERE`来实现这一点：

```kt
// Find a specific record
public Cursor searchName(String name) {
   String query = "SELECT " +
                  TABLE_ROW_ID + ", " +
                  TABLE_ROW_NAME +
                  ", " + TABLE_ROW_AGE +
                  " from " +
                  TABLE_N_AND_A + " WHERE " +
                  TABLE_ROW_NAME + " = '" + name + "';";
   Log.i("searchName() = ", query);
   Cursor c = db.rawQuery(query, null);
   return c;
}
```

最后，对于`DataManager`类，我们创建一个内部类，它将是我们的`SQLiteOpenHelper`的实现。这是一个最基本的实现。

我们有一个构造函数，接收一个`Context`对象，数据库名称和数据库版本。

我们还重写了`onCreate`方法，其中包含创建具有`_ID`，`name`和`age`列的数据库表的 SQL 语句。

`onUpgrade`方法在此应用程序中被故意留空。

将内部的`CustomSQLiteOpenHelper`类添加到`DataManager`类中：

```kt
// This class is created when our DataManager is initialized
private class CustomSQLiteOpenHelper extends SQLiteOpenHelper {
   public CustomSQLiteOpenHelper(Context context) {
         super(context, DB_NAME, null, DB_VERSION);
   }
   // This runs the first time the database is created
   @Override
   public void onCreate(SQLiteDatabase db) {
         // Create a table for photos and all their details
         String newTableQueryString = "create table "
                      + TABLE_N_AND_A + " ("
                      + TABLE_ROW_ID
                      + " integer primary key 
                      autoincrement not null,"
                      + TABLE_ROW_NAME
                      + " text not null,"
                      + TABLE_ROW_AGE
                      + " text not null);";
         db.execSQL(newTableQueryString);

   }
   // This method only runs when we increment DB_VERSION
   @Override
   public void onUpgrade(SQLiteDatabase db, 
int oldVersion, int newVersion) {
// Not needed in this app
// but we must still override it
   }
}
```

现在我们可以在我们的`Fragment`类中添加代码来使用我们的新的`DataManager`类。

# 编写 Fragment 类以使用 DataManager 类

将这段突出显示的代码添加到`InsertFragment`类中以更新`onCreateView`方法：

```kt
View v = inflater.inflate(R.layout.content_insert, 
   container, false);
final DataManager dm = 
   new DataManager(getActivity());
Button btnInsert = 
   v.findViewById(R.id.btnInsert);

final EditText editName = 
   v.findViewById(R.id.editName);

final EditText editAge = 
   v.findViewById(R.id.editAge);
btnInsert.setOnClickListener(new View.OnClickListener() {
   @Override
   public void onClick(View v) {
          dm.insert(editName.getText().toString(),
                       editAge.getText().toString());
   }
});
return v;
```

在代码中，我们获取了我们的`DataManager`类的实例和对每个 UI 小部件的引用。然后，在`onClick`方法中，我们使用`insert`方法向数据库添加新的姓名和年龄。要插入的值来自两个`EditText`小部件。

将这段突出显示的代码添加到`DeleteFragment`类中以更新`onCreateView`方法：

```kt
View v = inflater.inflate(R.layout.content_delete, 
   container, false);
final DataManager dm = 
   new DataManager(getActivity());
Button btnDelete = 
   v.findViewById(R.id.btnDelete);

final EditText editDelete = 
   v.findViewById(R.id.editDelete);
btnDelete.setOnClickListener(new View.OnClickListener() {
   @Override
   public void onClick(View v) {
          dm.delete(editDelete.getText().toString());
   }
});
return v;
```

在`DeleteFragment`类中，我们创建了我们的`DataManager`类的实例，然后从我们的布局中获取了`EditText`和`Button`小部件的引用。当按钮被点击时，将调用`delete`方法，传入用户输入的`EditText`小部件中的任何文本的值。`delete`方法搜索我们的数据库是否有匹配项，如果找到，则删除它。

将这段突出显示的代码添加到`SearchFragment`类中以更新`onCreateView`方法：

```kt
View v = inflater.inflate(R.layout.content_search,
   container,false);
Button btnSearch = 
   v.findViewById(R.id.btnSearch);

final EditText editSearch = 
   v.findViewById(R.id.editSearch);

final TextView textResult = 
   v.findViewById(R.id.textResult);
// This is our DataManager instance
final DataManager dm = 
   new DataManager(getActivity());
btnSearch.setOnClickListener(new View.OnClickListener() {
   @Override
   public void onClick(View v) {
          Cursor c = dm.searchName(
                     editSearch.getText().toString());
// Make sure a result was found before using the 
          Cursor
          if(c.getCount() > 0) {
                 c.moveToNext();
textResult.setText("Result = " + 
c.getString(1) + " - " + 
                     c.getString(2));
          }
   }
});
return v;
```

与我们所有不同的`Fragment`类一样，我们创建了`DataManager`类的实例，并获取了布局中所有不同 UI 小部件的引用。在`onClick`方法中，使用`searchName`方法，传入`EditText`小部件的值。如果数据库在`Cursor`实例中返回结果，那么`TextView`小部件使用其`setText`方法输出结果。

将这段突出显示的代码添加到`ResultsFragment`类中以更新`onCreateView`方法：

```kt
View v = inflater.inflate(R.layout.content_results, 
   container, false);
// Create an instance of our DataManager
DataManager dm = 
   new DataManager(getActivity());
// Get a reference to the TextView to show the results
TextView textResults = 
   v.findViewById(R.id.textResults);
// Create and initialize a Cursor with all the results
Cursor c = dm.selectAll();
// A String to hold all the text
String list = "";
// Loop through the results in the Cursor
while (c.moveToNext()){
   // Add the results to the String
   // with a little formatting
   list+=(c.getString(1) + " - " + c.getString(2) + "\n");
}
// Display the String in the TextView
textResults.setText(list);
return v;
```

在这个类中，`Cursor`实例在任何交互发生之前使用`selectAll`方法加载数据。然后通过连接结果将`Cursor`的内容输出到`TextView`小部件中。在连接中的`\n`是在`Cursor`实例中的每个结果之间创建新行的。

# 运行 Age Database 应用程序

让我们运行一些我们应用程序的功能，以确保它按预期工作。

首先，我使用**插入**菜单选项向数据库添加了一个新的名字：

![图 27.3 – 插入菜单](img/Figure_27.3_B16773.jpg)

图 27.3 – 插入菜单

然后我通过查看**结果**选项确认它确实存在：

![图 27.4 – 结果选项](img/Figure_27.4_B16773.jpg)

图 27.4 – 结果选项

之后，我添加了一些更多的姓名和年龄，只是为了填充数据库：

![图 27.5 – 填充数据库](img/Figure_27.5_B16773.jpg)

图 27.5 – 填充数据库

然后我使用了**删除**菜单选项，再次查看**结果**选项，以确保我选择的名字确实被删除了。

![图 27.6 – 删除菜单](img/Figure_27.6_B16773.jpg)

图 27.6 – 删除菜单

然后我搜索了一个我知道存在的名字来测试**搜索**菜单选项：

![图 27.7 – 搜索菜单](img/Figure_27.7_B16773.jpg)

图 27.7 – 搜索菜单

让我们回顾一下本章我们所做的事情。

# 摘要

在本章中，我们涵盖了很多内容。我们学习了关于数据库，特别是 Android 应用程序使用的数据库 SQLite。我们练习了使用 SQL 语言与数据库进行通信的基础知识。

我们已经看到了 Android API 如何帮助我们使用 SQLite 数据库，并实现了我们的第一个使用数据库的工作应用程序。

你已经走了很长的路，已经到达了书的尽头。让我们谈谈接下来可能会发生什么。
