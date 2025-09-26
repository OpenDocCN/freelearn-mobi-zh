# 第十一章。处理数据

不时地，对于应用程序来说，存储或操作数据是很重要的。随着 LINQ 的出现，操作现在变得极其简单。然而，问题是您需要以某种方式存储数据。幸运的是，使用 SQLite 这也同样简单。

我们将在本章中介绍以下主题：

+   使用 SQLite

+   设置 SQLite 辅助类

+   使用 LINQ

+   在 iOS 上使用 LINQ 的危险

# 使用 SQLite

SQLite 是一个非常简单但功能强大的数据库系统。本书的范围不包括为您提供使用 SQLite 的高级课程，但了解如何设置和使用该系统将有所帮助。

## 安装和设置 SQLite

安装可以通过两种方式之一进行；要么你可以从 Xamarin 组件市场安装（它很有用，因为它为你提供了示例，还包括 Android 版本）或者你可以手动下载并安装软件。由于没有额外的库（SQLite 作为一个单独的 C#文件提供），两种方式都很好。

一旦您已经复制了 C#文件或安装了组件，`SQLite.net`实现就准备好使用了。只需在源文件顶部插入以下`using`指令即可，操作完成：

```swift
using SQLite;
```

## 数据库基础

考虑数据库的一个简单方法是将它比作一个古老的卡片索引系统（通常称为**卡片索引系统**）。信息（数据）可以添加、更新、读取或删除——SQLite 在移动环境中提供了这些功能。数据存储在一个包含表格的文件中，这些表格（表格可以被认为是存放卡片的盒子）包含了您想要的信息。

在可以使用卡片索引系统之前，必须定义存储方法。最简单的方法是创建一个包含原始类型的类。SQLite 只能存储某些类型的数据：`integer`、`real`、`text`、`none`和`numeric`。不允许其他类型——这包括数组集合（如`List<T>`）。编程中通常使用的类型（如`string`和`double`）被**映射**到这些内部类型。

一个表也需要一个主键。这是通常自动递增的主要索引键。在数据类中，这将使用`[PrimaryKey, AutoIncrement]`定义。

### 一个简单的数据库类

作为演示，可以使用一个简单的数据库类：

```swift
using SQLite;
public class demoRow
{
    public demoRow ()
    {}
    [PrimaryKey, AutoIncrement]
    public int ID
    {get; set;}
    public string Name
    {get;set;}
    public double Value
    {get;set;}
    public override string ToString()
    {
        return string.Format("[demoRow : ID={0}, Name={1},Value={2}]", ID, Name, Value);
    }
}
```

然后，可以将`demoTable`变量引入数据库。

### 创建到数据库的连接

在数据库可以使用之前，需要设置与服务器的连接。就像任何类一样，数据库类也需要设置。我将其命名为`DataManager`。SQLite 需要一个指向数据库文件的路径。

```swift
DataManager dm = new DataManager("path_to_database");
dm.Setup(); // calls the creation of the database code
```

上述代码设置了一个`DataManager`实例。为了使数据库能够在整个应用程序中使用，以下内容应添加到`AppDelegate`类中：

```swift
private static DataManager dm{ get; set; }
```

在 `DataManager` 类中，需要一个锁（以防止任何时刻对数据库进行多个操作），以及数据库路径的本地副本。

```swift
public DataManager(string path)
{
    dataLock = new object();
    dataBasePath = path;
}

private string dataBasePath;
private object dataLock;

public string DataPath{
    get
    {
        return dataBasePath;
    }
}
```

最后，需要在数据库中设置表。

```swift
public bool Setup()
{
    lock(dbLock)
    {
        try
        {
            using (SQLiteConnection sqlCon = newSQLiteConnection(DBPath))
            {
                 sqlCon.CreateTable<demoRow>();
            }
            return true;
        }
        catch (SQLiteException ex)
        {
            throw ex;
        }
        catch (Exception ex)
        {
            throw ex;
        }
    }
}
```

你会注意到，你需要定义一个包含以下常量的类：

```swift
public const string DBClauseSyncOff = "PRAGMA SYNCHRONOUS=OFF;";
public const string DBClauseVacuum = "VACUUM;";
```

`DBClauseVacuum` 常量用于最后的执行查询。`DBClauseSyncOff` 常量用于第一个。

到目前为止，你可能已经注意到了使用 SQLite 的某些方面。它被用作类中的正常方法。这样是可以的。

# 设置 SQLite 辅助类

通常使用 SQLite 需要你全局创建并存储连接（以节省设备资源并减少安全问题的可能性），然后在每次调用数据库时设置一组查询和问题。即使是简单的数据库，这也可能导致大量重复的代码，代码行数越多，出现错误的可能性就越大。

辅助类封装了你需要的所有功能，并且很容易编写。

## 编写辅助类方法

正如我一开始提到的，数据库允许你从/向表中读写数据。首先，能够读取数据是有意义的。以下有两个类：第一个返回 `List<demoRow>`，另一个返回给定 ID 的名称。

```swift
Public List<demoRow> getAllListOfRows()
{
    lock (dbLock)
    {
        using (SQLiteConnection sqlCon = newSQLiteConnection(this.DBPath))
        {
            sqlCon.Execute(Constants.DBClauseSyncOff);
            sqlCon.BeginTransaction();
            List<demoRow> toReturn = new List<demoRow>();
            toReturn = sqlCon.Query<demoRow>("SELECT * FROMdemoRow");
                return toReturn.Count != 0 ? toReturn : newList<demoRow>();
        }
    }
}

public string getNameForID(int id)
{
    lock (dbLock)
    {
        using (SQLiteConnection sqlCon = newSQLiteConnection(DBPath))
        {
            sqlCon.Execute(Constants.DBClauseSyncOff);sqlCon.BeginTransaction();string toReturn = string.Empty;toReturn = sqlCon.ExecuteScalar<string>("SELECT NameFROM demoRow WHERE ID=?", id);
                return !string.IsNullOrEmpty(toReturn) ?toReturn : "No name found";
        }
    }
}
```

两种方法之间几乎没有区别，除了数据库调用。列表版本需要一个查询——这用于非原始类型的数据，并期望返回数据。`ExecuteScalar` 方法期望返回原始类型的数据行。

访问这些辅助方法与访问任何其他方法相同。

```swift
string name = dm.getNameForID(3);
```

或者

```swift
List<demoRow> dT = dm.getListOfTables();
```

## 向数据库添加数据

数据添加以插入或更新形式出现，与读取不同，数据添加需要捕获失败的情况，并且数据库需要回滚到添加数据尝试之前的状态。

幸运的是，插入或更新可以在一个方法中处理。

```swift
public void AddOrUpdateTable(demoRow dTRow){
    lock (dataLock)
    {
        using (SQLiteConnection sqlCon = new SQLiteConnection( DBPath))
        {
            sqlCon.Execute(Constants.DBClauseSyncOff);
            sqlCon.BeginTransaction();
            try
            {
                if (sqlCon.Execute("UPDATE dataRow SET " +"ID=?, " + "Name=?, " +"Value=? WHERE " +"ID=?",dRow.ID,dRow.Name,dRow.Value,dRow.ID) == 0)
                {
                    sqlCon.Insert(dRow, typeof(demoRow));}
                sqlCon.Commit();
            }
            catch (Exception ex)
            {
                Console.WriteLine("Error in AddOrUpdateTable :{0}-{1}", ex.Message, ex.StackTrace);sqlCon.Rollback();
             }
        }
    }
}
```

再次，为了使前面的代码能够正常工作，需要将 `demoRow` 类的单个实例传递到方法中。使用 `demoRow` 的 `List` 很容易处理。

```swift
public void AddOrUpdateTables(List<demoRow>rows)
{
    foreach(demoReow row in rows)
        AddOrUpdateTable(row);
}
```

# 使用 LINQ 进行数据操作

持续访问数据库会对它所在的系统造成压力，如果你考虑任何时刻正在使用的数据库数量，很快就会清楚，不断击打 SQL 服务的开销会很高。

LINQ 允许将数据库样式查询应用于数据，并输出结果。虽然有许多优秀的书籍（其中许多是免费的），但我将介绍一些在 iOS 应用程序中使用 LINQ 的便捷方法。

虽然 LINQ 非常强大，但在 iOS 上编码时，并不是所有使用 LINQ 的操作都不会出现问题。主要原因在于 iPhone 的工作方式。简单来说，iOS 想要知道将要发生什么**提前时间**（**AOT**）。它不喜欢惊喜，并且真的不喜欢它不知道的数据结构。例如：

```swift
List<string> data = new List<string>();
```

之前的代码不应该引起任何问题。但换一个角度思考——`List`的大小将会有多大？当你提前工作时，空间分配通常是有限的。（例如，一个`int`值的数组可能有 20 个值的内存预留，没有更多，AOT 系统喜欢这样——它提前知道需要预留 20 个`int`。）

考虑到这一点，当在应用程序中使用 LINQ 时，你可能会遇到随机崩溃——这不太可能，但它可能发生，并且值得提及。

## LINQ – 快速浏览

对于我的示例，我将使用本章中已经使用的`demoRow`表。

```swift
List<demoRow> myRow = dm.getListOfRows();
```

在返回的`List`中某个地方有一个名为`Fred Moriarty`的名字，我想从`List`中获取具有该名字的类的实例。我知道列表中只有一个这样的实例。

```swift
var demo = myRow.SingleOrDefault(t=>t.Name == "Fred Moriarty");
```

之前的代码获取表并返回单个实例。如果找不到该名字，则返回`null`（或默认值）。

假设我的列表在`Name`字段中有多个`Fred Bloggs`实例。

```swift
var bloggs = myTRow.Where(t=>t.Name == "Fred Bloggs").ToList();
```

在 LINQ 出现之前，这将会是一项相当缓慢的工作，并且可能需要如下代码：

```swift
List<demoRow> bloggs = new List<demoRow>();
foreach(demoRow blog in bloggs)
{
    if (blog.Name == "Fred Bloggs")
        bloggs.Add(blog);
}
```

之前的示例非常简单。LINQ 也可以执行非常复杂的操作，例如：

```swift
var res = from inviter in ContactListfrom tester in inviteswhere inviter.UserID == tester.UserIdselect tester;
```

这里有两个列表（`ContactList`和`invites`）。LINQ 查询创建了两个循环，并在外层循环的`UserID`与内层循环的`UserId`匹配时选择实例测试器。结果是几乎瞬间的。

### LINQ 中的 SELECT 和 WHERE – 常见的混淆原因

每个人在某个时候都会遇到的一个非常常见的 LINQ 错误是将`WHERE`语法与`SELECT`语法混淆。

`WHERE`语法是一个条件（例如，`WHERE ID==311`或`WHERE A==B`）。它只返回设置的条件。在以下示例中，`testClass`是一个列表，它包含一个有字符串`Value`的类。

```swift
var retString = testClass.Where(t=>t.Value == "fred").ToList();
```

`retString`变量将包含所有来自`testString`的结果的`List<string>`，其中`Value == "fred"`。

`SELECT`语法返回传入对象中所有项的内容。结果可能是项本身，也可能是其他内容。

```swift
var retString = inString.Select(t=>t.ToUpper()).ToList();
```

之前的代码将`inString`的内容转换为大写，并输出到一个列表中。

```swift
var retString = inString.Select(t=>t.Value, Func<inString,outString>).ToList();
```

这里`Func<inString, outString>`将`inString`的元素转换为`outString`，并将元素作为`outString`列表输出。

### 在 LINQ 中使用 Select

看以下示例，并记住`Select`执行的是转换操作：

```swift
string[] teams = {"Liverpool", "Everton", "Oldham", "Leeds"};
var result = teams.Select(t=>t.ToUpper());
foreach (string team in result)
{
    Console.WriteLine(team);
}
```

这将是输出：

```swift
LIVERPOOL
EVERTON
OLDHAM
LEEDS.
```

*但是发生了什么？* `Select` 语句在字符串数组 `teams` 上执行。然后它指定了一个 lambda 表达式 (`t=>`)，它将表达式转换为大写 (`ToUpper()`).

`Select` 语句（如前代码所示）也有一个重载的方法。同样，它也是一个转换方法。

```swift
string[] teams = {"Liverpool", "Everton", "Oldham", "Leeds"};
var trans = teams.Select(teams, index) =>new {index, str = teams.Substring(0, index)});
```

如果代码运行并使用了合适的输出，你会看到 `E`，`Ol`，`Lee`——起始索引是 0，所以没有看到任何字符。

### 替换 SQL 为 LINQ

根据您的需求，用一系列 `List<T>` 类替换 SQLite 数据库可能是一个更好的选择，并使用 LINQ 来替换 SQL 查询。例如，如果您有一个非常简单的数据库（例如我们的 `demoTable` 数据库），它具有非常有限的用于操作的范围，那么使用类列表可能是一个更好的主意，您可以像通常那样向其中添加内容，并使用 LINQ 执行查询。由于没有对数据库服务器的打击，这可能会从应用程序中获得更快的响应时间。

对于结构更复杂的表格，其中 SQL 本身执行对另一个表的 `JOIN` 操作，或者涉及复杂的操作，使用 LINQ 可能不会得到一个可用的系统。

LINQ 可以执行 `JOIN` 条件，以及 SQLite 可以执行的大多数其他功能——只是不那么容易。

但是，请注意，iPhone 上的 LINQ 可能会决定直接崩溃，而 SQLite 不会。

# 摘要

iPhone 上的数据存储可以很简单，也可能充满问题。为了安全和可靠性，iPhone 上首选 SQLite 选项。对于速度，LINQ 无敌，但您必须确保在使用 LINQ 时在物理设备上测试 LINQ 项目。
