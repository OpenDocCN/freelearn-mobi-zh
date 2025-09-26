# 第四章 数据管理

在本章中，我们将涵盖以下主题：

+   创建文件

+   创建 SQLite 数据库

+   插入和更新数据

+   查询 SQLite 数据库

+   使用现有的 SQLite 数据库

+   使用序列化存储数据

+   使用 XML 存储数据

+   使用 LINQ to XML 管理 XML 数据

# 简介

几乎每个应用程序都需要在文件系统中具有永久数据存储。在本章中，我们将讨论不同的数据存储方式。我们将了解如何在 iPhone 应用程序中创建 SQLite 数据库并使用它来管理数据。此外，我们还将学习如何在项目中使用现有的数据库。

SQLite ([`www.sqlite.org`](http://www.sqlite.org) ) 是一个自包含的事务型数据库系统。每个数据库都保存在一个独立的文件中，没有数据库服务器。在 iOS 中，SQLite 支持是原生的。

接下来，我们将看到如何序列化和将对象保存到文件系统中，以及如何使用 LINQ to XML 与 XML 文件一起使用。

# 创建文件

在这个配方中，我们将学习如何在 iOS 设备的文件系统中创建文件。

## 准备工作

在 MonoDevelop 中创建一个新的**iPhone 单视图应用程序**项目，命名为`FileCreationApp`。打开`FileCreationAppViewController.xib`文件，并在其视图中添加一个`UILabel`和一个`UIButton`。

## 如何操作...

在`FileCreationAppViewController`类中输入以下代码：

```swift
public override void ViewDidLoad(){
string filePath = Path.Combine (Environment.GetFolderPath (Environment.SpecialFolder.Personal), "MyFile.txt");
using (StreamWriter sw = new StreamWriter (filePath)){
sw.WriteLine ("Some text in file!");
}
this.btnShow.TouchUpInside += delegate {
using (StreamReader sr = new StreamReader (filePath)){
this.labelMessage.Text = sr.ReadToEnd ();
}
}
};

```

## 工作原理...

从这段代码中可以看出，我们可以像在桌面应用程序中一样使用`System.IO`命名空间中的标准类。我们首先做的事情是为我们将要保存的文件设置一个路径。我们在以下行中这样做：

```swift
string filePath = Path.Combine (Environment.GetFolderPath (Environment.SpecialFolder.Personal), "MyFile.txt");

```

在 iOS 中，我们无法访问整个文件系统，甚至在应用程序包内部也不行。如果我们尝试在无法访问的文件夹中写入，将会发生异常。因此，我们使用静态的`Environment.GetFolderPath(SpecialFolder)`方法来检索`Personal`特殊文件夹，这对应于我们的应用程序的`Documents`文件夹。注意`Path.Combine(string, string)`的使用，它将两个字符串组合并返回一个路径。之后，我们创建`StreamWriter`类的新实例，并使用其`WriteLine(string)`方法在文件中写入一些文本。

```swift
using (StreamWriter sw = new StreamWriter (filePath)){
sw.WriteLine ("Some text in file!");
}

```

要从文件中检索文本，我们创建`StreamReader`类的新实例，并使用其`ReadLine`方法读取文本：

```swift
using (StreamReader sr = new StreamReader (filePath)){
this.labelMessage.Text = sr.ReadToEnd ();
}

```

## 更多内容...

如果我们想要写入或读取二进制数据，我们可以使用`FileStream`类。

### 文档文件夹

应用程序的`Documents`文件夹仅与该应用程序相关。如果从设备中卸载应用程序，其内容也会被删除。我们在这个文件夹中既有读又有写权限。

## 参见

在本章中：

+   *使用序列化存储数据*

# 创建 SQLite 数据库

在这个配方中，我们将学习如何创建 SQLite 数据库文件。

## 准备工作

在 MonoDevelop 中创建一个新的**iPhone 单视图应用程序**，命名为`CreateSQLiteApp`。在其视图中添加一个`UILabel`和一个`UIButton`。

## 如何操作...

1.  将项目引用添加到`Mono.Data.Sqlite`程序集，并在命名空间上添加相应的`using`指令：

    ```swift
    using Mono.Data.Sqlite;

    ```

1.  在`CreateSQLiteAppViewController`类中输入以下方法：

    ```swift
    private void CreateSQLiteDatabase (string databaseFile){
    try{
    if (!File.Exists (databaseFile)){
    SqliteConnection.CreateFile (databaseFile);
    using (SqliteConnection sqlCon = new SqliteConnection (String.Format ("Data Source = {0};", databaseFile))){
    sqlCon.Open ();
    using (SqliteCommand sqlCom = new SqliteCommand (sqlCon)){
    sqlCom.CommandText = "CREATE TABLE Customers (ID INTEGER PRIMARY KEY, FirstName VARCHAR(20), LastName VARCHAR(20))";
    sqlCom.ExecuteNonQuery ();
    }
    sqlCon.Close ();
    }
    this.lblMessage.Text = "Database created!";
    } else {
    this.lblMessage.Text = "Database already exists!";
    }
    } catch (Exception ex) {
    this.lblMessage.Text = String.Format ("Sqlite error: {0}", ex.Message);
    }
    }

    ```

1.  然后在`ViewDidLoad`方法中添加以下代码：

    ```swift
    string sqlitePath = Path.Combine (Environment.GetFolderPath (Environment.SpecialFolder.Personal), "MyDB.db3");
    this.btnCreate.TouchUpInside += delegate {
    this.CreateSQLiteDatabase (sqlitePath);
    };

    ```

### 它是如何工作的...

iOS 为 SQLite 数据库提供了原生支持。

1.  我们可以使用 Mono 的`Mono.Data.Sqlite`命名空间来访问 SQLite 数据库，如下所示：

    ```swift
    using Mono.Data.Sqlite;

    ```

1.  在`CreateSQLiteDatabase`方法内部，我们首先检查文件是否已存在：

    ```swift
    if (!File.Exists (databaseFile))

    ```

1.  然后，我们可以继续创建数据库。我们首先使用`SqliteConnection.CreateFile(string)`静态方法创建文件，如下所示：

    ```swift
    SqliteConnection.CreateFile (databaseFile);

    ```

1.  我们通过初始化一个`SqliteConnection`对象并调用其`Open()`方法来连接新创建的文件。SQLite 数据库的连接字符串为`Data Source =`，后跟数据库的文件名：

    ```swift
    using (SqliteConnection sqlCon = new SqliteConnection (String.Format ("Data Source = {0};", databaseFile)))
    sqlCon.Open();

    ```

1.  在数据库中创建一个表时，会初始化一个`SqliteCommand`对象。我们将一个标准的 SQL 字符串传递给其`CommandText`属性，并调用`ExecuteNonQuery()`方法来执行 SQL：

    ```swift
    sqlCom.CommandText = "CREATE TABLE Customers (ID INTEGER PRIMARY KEY, FirstName VARCHAR(20), LastName VARCHAR(20))";
    sqlCom.ExecuteNonQuery ();

    ```

#### 更多内容...

注意 try-catch 块的使用。它用于在数据库创建过程中出现错误时向用户显示消息。

##### SQL 表创建

在这个任务中，我们为我们的数据库创建了一个简单的表，名为`Customers`。它包含三个字段。`FirstName`和`LastName`字段类型为`VARCHAR(20)`，而`ID`字段类型为`INTEGER`，同时也是表的`PRIMARY KEY`。

除了使用 SQL 命令创建表之外，我们还可以使用各种商业或免费的 GUI 工具创建 SQLite 数据库。在互联网上简单搜索就会得到各种结果。

#### 参见

在本章中：

+   *查询 SQLite 数据库*

+   *插入和更新数据*

+   *使用现有的数据库*

# 插入和更新数据

在这个菜谱中，我们将学习如何将数据写入数据库。

## 准备工作

对于这个任务，我们将扩展我们在上一个任务中创建的`CreateSQLiteApp`项目。

## 如何操作...

1.  在视图中添加两个额外的按钮。

1.  在`CreateSQLiteAppViewController`类内部，创建两个方法，这些方法将使用上一个任务中的代码连接到数据库文件。这里的区别在于`SqliteCommand`对象的使用：

    ```swift
    using (SqliteCommand sqlCom = new SqliteCommand (sqlCon)){
    // INSERT statement
    sqlCom.CommandText = "INSERT INTO Customers (FirstName, LastName) VALUES (@firstName, @lastName)";

    sqlCom.Parameters.Add (new SqliteParameter ("@firstName", "John"));
    sqlCom.Parameters.Add (new SqliteParameter ("@lastName", "Smith"));
    //UPDATE statement
    //sqlCom.CommandText = "UPDATE Customers SET FirstName = 'James' WHERE LastName = @lastName";

    //sqlCom.Parameters.Add (new SqliteParameter ("@lastName", "Smith"));
    sqlCom.ExecuteNonQuery ();
    }

    ```

### 它是如何工作的...

要在 SQLite 表中插入和更新数据，我们分别使用常见的`INSERT`和`UPDATE`语句。代码中高亮的部分表示了 SQLite 参数的使用。这两个语句都在`sqlCom.ExecuteNonQuery()`行上对数据库进行执行。`ExecuteNonQuery`的返回值类型为`int`，表示受影响的表中的行数。所以，如果我们像以下示例中那样调用该方法，我们会得到输出`1`，表示影响了一行：

```swift
Console.WriteLine(sqlCom.ExecuteNonQuery());

```

### 更多内容...

由于我们已经使用了上一个任务中的项目，其中我们提供了创建数据库文件的代码，因此我们应该在我们的每个执行数据操作的方法的开始处添加以下代码：

```swift
if (!File.Exists (databaseFile)) {
this.lblMessage.Text = "Database file does not exist. Tap the appropriate button to create it.";
return;
}

```

这是为了确保当用户在没有任何数据库文件的情况下点击 **插入** 或 **更新** 按钮时，不会出现异常。

#### SQLite 性能

虽然 SQLite 提供了出色的性能和可移植性，但它在很大程度上依赖于其宿主文件系统，无论它存储在哪个平台上。如果您想执行多个并发 `INSERT` 或 `UPDATE` 语句，请考虑使用 `SqliteTransaction`。除了性能上的好处外，通过将多个语句批处理在一起，事务提供了一种在出现问题时回滚操作的方法。

### 参见

在本章中：

+   *创建文件*

+   *创建 SQLite 数据库*

+   *查询 SQLite 数据库*

# 查询 SQLite 数据库

在本配方中，我们将学习如何从 SQLite 数据库中检索数据。

## 准备工作

再次使用 `CreateSQLiteApp` 项目。完成此任务后，该项目将是一个完整的 SQLite 管理应用程序。

## 如何操作...

1.  在视图中添加另一个按钮。

1.  在 `CreateSQLiteAppViewController` 类内部，添加一个处理查询的方法。执行查询的部分如下：

    ```swift
    using (SqliteCommand sqlCom = new SqliteCommand (sqlCon)){
    sqlCom.CommandText = "SELECT * FROM Customers WHERE LastName = @lastName";
    sqlCom.Parameters.Add (new SqliteParameter ("@lastName", "Smith"));
    using (SqliteDataReader dbReader = sqlCom.ExecuteReader ()){
    if (dbReader.HasRows){
    while (dbReader.Read ()){
    this.lblMessage.Text += String.Format ("ID: {0}\n", Convert.ToString (dbReader["ID"]));
    this.lblMessage.Text += String.Format ("First name: {0}\n", Convert.ToString (dbReader["FirstName"]));
    this.lblMessage.Text += String.Format ("Last name: {0}\n", Convert.ToString (dbReader["LastName"]));
    }
    }
    }
    }

    ```

### 它是如何工作的...

要在 SQLite 数据库上执行查询，我们创建一个简单的 `SELECT` 语句并将其分配给 `SqliteCommand` 对象的 `CommandText` 属性：

```swift
sqlCom.CommandText = "SELECT * FROM Customers WHERE LastName = @lastName";

```

`SELECT` 关键字后面的星号 (*) 表示我们想要检索表中的所有字段。执行 SQL 查询的最简单方法是通过使用 `SqliteDataReader` 类：

```swift
using (SqliteDataReader dbReader = sqlCom.ExecuteReader ())

```

`SqliteCommand.ExecuteReader()` 方法在表上执行查询并创建一个 `SqliteDataReader` 对象。现在我们有了对象，我们首先检查表是否包含任何行：

```swift
if (dbReader.HasRows)

```

如果上述返回 `true`，我们开始通过传递 `SqliteDataReader.Read()` 方法在 `while` 循环中逐行前进：

```swift
while (dbReader.Read ())

```

我们现在可以通过传递每个字段的名称作为 `SqliteDataReader` 实例的索引来检索每个字段的值：

```swift
this.lblMessage.Text += String.Format ("ID: {0}\n", Convert.ToString (dbReader["ID"]));
//...

```

由于索引的 `dbReader` 变量返回的是 `System.Object` 类型的对象，我们使用 `Convert.ToString(object)` 静态方法将其转换为字符串。

### 更多内容...

SQLite 数据库不是线程安全的。如果我们想在执行查询的同时，另一个线程可能正在执行 `INSERT` 或 `UPDATE` 语句，那么提供某种同步机制会更好。

#### 查询性能

虽然运行 iOS 平台的设备在性能上优于一些较老的桌面计算机，但它们的资源仍然有限。在查询大型数据库中的数据时，请考虑使用 SQL `WHERE` 语句缩小结果，以获取特定时间所需的数据。此外，如果不需要排序，请避免使用 `ORDER BY` 语句。

### 参见

在本章中：

+   *创建 SQLite 数据库*

+   *插入和更新数据*

+   *使用现有的 SQLite 数据库*

# 使用现有的 SQLite 数据库

在这个菜谱中，我们将讨论如何将现有的 SQLite 数据库文件包含到我们的项目中。

## 准备工作

使用某种前端创建数据库总是更容易。在这个任务中，我们将了解如何将现有的 SQLite 数据库与 iPhone 项目集成。创建一个新的 **iPhone 单视图应用程序** 项目，并将其命名为 `SqliteIntegrationApp`。

## 如何做到这一点...

1.  在 Interface Builder 中的视图上添加一个 `UIButton` 和一个 `UILabel`。将它们与适当的出口连接起来。确保标签足够高，可以显示六行，并在 **检查器** 选项卡中设置其 **# Lines** 字段。

1.  在 MonoDevelop 中，在 **解决方案** 面板中右键单击项目，然后点击 **添加文件...**。

1.  在显示的对话框中，导航到数据库文件所在的路径并选择它。文件将被添加到项目中。

1.  右键单击它，然后选择 **构建操作 | 内容**。

1.  在 `SqliteIntegrationAppViewController` 类中输入以下方法：

    ```swift
    private string CopyDatabase (){
    string docPath = Environment.GetFolderPath (Environment.SpecialFolder.Personal);
    string databaseFile = "CustomersDB.db3";
    string databasePath = Path.Combine (docPath, databaseFile);
    if (!File.Exists (databasePath)){
    File.Copy (databaseFile, databasePath);
    }
    return databasePath;
    }
    private void QueryData (object sender, EventArgs e)
    { //... }

    ```

1.  `QueryData` 方法的功能与之前任务中执行查询的方法相同。主要区别在于，在这个项目中，它将作为按钮的 TouchUpInside 事件处理程序。

1.  添加一个类型为 `string` 的字段，用于存储数据库路径：

    ```swift
    private string database;

    ```

1.  最后，在 `ViewDidLoad` 方法中：

    ```swift
    this.database = this.CopyDatabase ();
    this.buttonQuery.TouchUpInside += this.QueryData;

    ```

1.  编译并运行应用程序。点击按钮，并观察数据库内容在标签中显示。

## 工作原理...

我们在这里所做的是将之前任务中讨论的各种实践相结合。当我们想在项目中添加各种文件时，我们设置它们的 **构建操作** 为 **内容** 非常重要。任何标记为 **内容** 的文件都会在应用程序包中按原样复制。在这种情况下，文件是一个 SQLite 数据库，因此我们需要在运行时对它有写入权限。我们需要将其复制到应用程序的 **Documents** 文件夹中。这就是 `CopyDatabase()` 方法在这一点上所做的事情：

```swift
if (!File.Exists (databasePath)){
File.Copy (databaseFile, databasePath);
}

```

检查文件是否已经存在非常重要，这样它就不会在应用程序下次执行时被覆盖。

## 还有更多...

如果遇到 `SqliteException`，首先检查是否由于某种原因数据库文件没有被复制到正确的文件夹。

## 参见

在本章中：

+   *创建文件*

+   *创建 SQLite 数据库*

# 使用序列化存储数据

在这个菜谱中，我们将讨论使用 .NET 序列化来存储 C# 对象。

## 准备工作

在 MonoDevelop 中创建一个新的 **iPhone 单视图应用程序** 项目。将其命名为 `SerializationApp`。

## 如何做到这一点...

1.  在视图上添加两个按钮和一个标签，在 Interface Builder 中。在 `SerializationAppViewController.cs` 文件中添加以下使用指令：

    ```swift
    using System.IO;
    using System.Runtime.Serialization.Formatters.Binary;

    ```

1.  在 `AppDelegate` 类中添加以下方法：

    ```swift
    private void Serialize (){

    CustomerData custData = new CustomerData ();
    custData.ID = 1;
    custData.FirstName = "John";custData.LastName = "Smith";
    using (MemoryStream ms = new MemoryStream ()){
    BinaryFormatter bf = new BinaryFormatter ();
    bf.Serialize (ms, custData);
    ms.Seek (0, SeekOrigin.Begin);
    this.objBuffer = new byte[ms.Length];
    ms.Read (this.objBuffer, 0, this.objBuffer.Length);
    }
    this.labelOutput.Text = "Customer data serialized.\n";
    }
    private void DeserializeAndDisplay (){
    if (null != this.objBuffer){
    CustomerData custData = null;
    using (MemoryStream ms = new MemoryStream (this.objBuffer)){
    ms.Seek (0, SeekOrigin.Begin);
    BinaryFormatter bf = new BinaryFormatter ();
    custData = (CustomerData)bf.Deserialize (ms);
    }
    this.labelOutput.Text += String.Format ("ID: {0}\n \tFirst name: {1}\n\tLast name: {2}", custData.ID, custData.FirstName, custData.LastName);
    } else {
    this.labelOutput.Text = "Buffer is null!";
    }
    }

    ```

完整代码可以在 `SerializationApp` 项目中找到。

### 工作原理...

使用 MonoTouch 进行序列化与桌面 C#应用程序中的操作相同。在突出显示的代码中指示的`CustomerData`类是一个简单的对象，它包含一个整数和两个字符串属性。

1.  要序列化对象，我们首先初始化一个`MemoryStream`：

    ```swift
    using (MemoryStream ms = new MemoryStream ())

    ```

1.  然后我们使用`BinaryFormatter`类来序列化对象并将其存储到流中：

    ```swift
    BinaryFormatter bf = new BinaryFormatter ();
    bf.Serialize (ms, custData);

    ```

1.  序列化后，我们将流的位置重置为其开始处，并将数据从流中读取到字节数组中：

    ```swift
    ms.Seek (0, SeekOrigin.Begin);
    this.objBuffer = new byte[ms.Length];
    ms.Read (this.objBuffer, 0, this.objBuffer.Length);

    ```

1.  要从缓冲区反序列化对象，过程类似，但顺序相反。在`MemoryStream`初始化后，我们将流的位置重置为其开始处，并使用`BinaryFormatter`的`Deserialize`方法。它返回一个`System.Object`类型的对象，因此我们需要将其转换为我们的对象类型：

    ```swift
    ms.Seek (0, SeekOrigin.Begin);
    BinaryFormatter bf = new BinaryFormatter ();
    custData = (CustomerData)bf.Deserialize (ms);

    ```

#### 还有更多...

在使用 MonoTouch 序列化对象时，有一件重要的事情要记住，那就是用`PreserveAttribute`装饰将要序列化的对象。此属性指示链接器避免删除未使用的成员，保持对象完整。在这个任务中声明的`CustomerData`类如下：

```swift
[Preserve()]
[Serializable()]
public class CustomerData

```

##### 可序列化对象

这是一个简单的示例，用于展示在 iOS 应用程序中使用二进制序列化的用法。可以通过继承`ISerializable` C#接口来定制用于序列化的对象。

当创建用于序列化的对象时，不要忘记用`SerializableAttribute`标记它。如果尝试序列化未标记此属性的对象，将会发生异常。

#### 相关内容

在本章中：

+   *使用 XML 存储数据*

在这本书中：

第一章,开发工具：

+   *编译*

## 使用 XML 存储数据

在这个菜谱中，我们将讨论如何使用 XML 序列化存储数据。

### 准备工作

在 MonoDevelop 中创建一个新的**iPhone 单视图应用程序**项目，并将其命名为`XMLDataApp`。

### 如何操作...

1.  在视图中添加一个`UIButton`和一个`UILabel`。将上一个任务中的`CustomerData`类添加到项目中。在`XMLDataAppViewController.cs`文件中添加以下`using`指令：

    ```swift
    using System.Xml.Serialization;
    using System.Xml;
    using System.Text;

    ```

1.  在`XMLDataAppViewController`类中添加以下方法：

    ```swift
    private void CreateXML (){
    CustomerData custData = new CustomerData ();
    custData.ID = 1;
    custData.FirstName = "John";
    custData.LastName = "Smith";
    this.sb = new StringBuilder ();
    XmlSerializer xmlSer = new XmlSerializer (typeof(CustomerData));
    xmlSer.Serialize (XmlWriter.Create (sb), custData);
    this.labelOutput.Text = sb.ToString ();
    }

    ```

完整的解决方案可以在`XMLDataApp`项目中找到。

### 工作原理...

在这个例子中，我们使用`XmlSerializer`类将对象序列化为 XML。`StringBuilder`对象将保存 XML 数据：

```swift
this.sb = new StringBuilder ();
XmlSerializer xmlSer = new XmlSerializer (typeof(CustomerData));

```

要初始化`XmlSerializer`，我们使用它的`XmlSerializer(System.Type)`构造函数，将我们想要序列化的对象的类型作为参数传递：`(typeof(CustomerData))`。然后我们调用它的`Serialize(XmlWriter, object)`方法，该方法将执行实际的序列化并将输出 XML 存储到`StringBuilder`中，通过`XmlWriter`类，如下所示：

```swift
xmlSer.Serialize (XmlWriter.Create (sb), custData);

```

当你编译并运行应用程序时，你将在标签中看到输出 XML。

### 还有更多...

使用 XML 存储数据有两个主要优势：

1.  输出是纯文本，与二进制序列化不同，这使得它易于阅读。

1.  数据可以传输或接收来自不同类型的应用程序、网站等，这些应用程序和网站不一定是用 C#编写的。

#### 反序列化

要从 XML 反序列化我们的对象，我们将使用以下行：

```swift
custData = (CustomerData)xmlSer.Deserialize (new StringReader (sb.ToString ()));

```

### 相关内容

在本章中：

+   *使用序列化存储数据*

+   *使用 LINQ to XML 管理 XML 数据*

## 使用 LINQ to XML 管理 XML 数据

在这个配方中，我们将学习如何使用**语言集成查询（LINQ）**来管理 XML 数据。

### 准备工作

在这个任务中，我们将使用上一个任务中的项目`XMLDataApp`。在 MonoDevelop 中打开它。

### 如何操作...

1.  在视图中添加另一个`UIButton`。在`XMLDataAppViewController.cs`文件中添加对`System.Xml.Linq`程序集的引用和以下`using`指令：

    ```swift
    using System.Linq.Xml;

    ```

1.  在`XMLDataAppViewController`类中输入以下方法：

    ```swift
    private void ReadXML (){
    if (null != sb){
    XDocument xDoc = XDocument.Parse (this.sb.ToString ());
    var query = from s in xDoc.Descendants()
    select new CustomerData() {
    ID = Convert.ToInt32(s.Element("ID").Value),
    FirstName = s.Element("FirstName").Value,
    LastName = s.Element("LastName").Value
    };
    CustomerData custData = query.FirstOrDefault();
    if (null != custData){
    this.labelOutput.Text = String.Format("ID: {0}\n\t FirstName: {1}\n\tLastName: {2}", custData.ID, custData.FirstName, custData.LastName);
    }
    }
    }

    ```

#### 它是如何工作的...

LINQ 在查询 XML 数据方面非常灵活且直观。

1.  我们首先从我们的 XML 数据创建一个`XDocument`：

    ```swift
    XDocument xDoc = XDocument.Parse (this.sb.ToString ());

    ```

1.  然后，我们对其`Descendants()`方法创建一个查询，该方法返回一个`IEnumerable<XElement>`对象。每个`XElement`对象对应一个 XML 元素。

1.  使用返回的每个`XElement`的信息，我们在`select`语句中构建我们的`CustomerData`对象：

    ```swift
    var query = from s in xDoc.Descendants()
    select new CustomerData() {
    ID = Convert.ToInt32(s.Element("ID").Value),
    FirstName = s.Element("FirstName").Value,
    LastName = s.Element("LastName").Value
    };

    ```

1.  要检索创建的对象，我们在查询上调用扩展方法`FirstOrDefault()`：

    ```swift
    CustomerData custData = query.FirstOrDefault();

    ```

##### 更多内容...

在这个例子中，XML 只包含一个类型为`CustomerData`的对象。如果有更多，我们可以通过提供一个`where`语句来缩小结果：

```swift
var query = from s in xDoc.Descendants() where s.Element("FirstName").Value == "John" select new CustomerData() {
ID = Convert.ToInt32(s.Element("ID").Value),
FirstName = s.Element("FirstName").Value,
LastName = s.Element("LastName").Value
};

```

###### LINQ 中的匿名类型

在 MonoTouch 中，我们还可以在查询中使用 C#的强大功能——匿名类型，例如，`select new { ID = … }`

##### 相关内容

在本章中：

+   *使用序列化存储数据*

+   *存储数据*
