# 第5章。公开C API

在本章中，你将了解有关**C API**（**C应用程序编程接口**）的所有内容，并通过代码在你的应用程序中使用它。通过拥有超过200个API调用到这个轻量级、小型且不断扩展的API集，SQLite将让你惊讶于它如何实现你想要的移动和灵活的数据库。

你将查看一些语法和用法，并了解如何通过使用其庞大的API调用库来扩展SQLite的功能。

# SQLite C组件的功能

SQLite是用C语言编写的，其创建者已经使它能够公开，并通过提供一个API（C API）增强了其功能。一般来说，SQLite有许多不同的API调用，例如，大约有200个API用于不同的功能。作为程序员，你可能觉得难以接受，但API是为特定任务设计的，例如，`sqlite3_reset()`函数清除与SQLite预编译语句关联的对象，并将其重置到其原始状态和值。

首先，两个核心对象是数据库连接和预编译语句对象。函数的顺序和类型可以给你一个如何编写SQL事务连接数据库、创建表和索引以及用`insert`语句填充它的概念。这些函数构成了SQL-C接口函数的主要元素，允许数据在代码和SQLite之间连接和传递。

## sqlite3_open()

`sqlite3_open(const char *filename, sqlite ** Db_name)`将打开位于您选择位置的SQLite数据库文件，并返回数据库连接对象，该对象将被其他SQLite组件或函数使用。

在进行任何其他操作之前，需要`sqlite3_open()`函数来建立连接。它将使其他操作得以进行。

如果在`sqlite3_open`函数中，文件名不包含`NULL`，则函数将使用提供的值，或者如果不存在数据库文件，SQLite将尝试使用该名称打开一个新的数据库。一旦通过`sqlite_open()`函数建立了数据库连接，就可以执行`sqlite_prepare()`等命令。以下是`sqlite3_open()`的一个示例：

![sqlite3_open()](img/B04725_05_01.jpg)

## sqlite3_prepare()

使用新的数据库连接，我们将获得一个指针地址，该地址将作为`sqlite_prepare()`命令的输入。语句将编译源SQL语句到目标代码。`sqlite3_prepare()`函数的功能是绑定并设置相关参数，将查询字符串作为数据处理的一部分链接起来。以下是`sqlite3_prepare()`过程的简要示例：

![sqlite3_prepare()](img/B04725_05_02.jpg)

## sqlite3_step()

`sqlite3_step()` 语句将对之前 `sqlite3_prepare()` 语句生成的输出对象代码进行分析、检查和评估。它将执行一个 `prepare` 语句并返回一个 SQLite 状态码。如果有数据，则返回 `SQLITE_ROW` 状态码。当语句执行完毕时，将返回 `SQLITE_DONE` 状态码。任何其他返回值都将被视为错误。`SQLite3_step()` 必须重置后才能再次使用。

此方法主要用于 `SELECT` 语句。其他语句，如 `DELETE`、`UPDATE` 或 `INSERT`，将从第一条记录执行到最后一项完成：

![sqlite3_step()](img/B04725_05_03.jpg)

## sqlite3_column()

随着 `SQLITE3_prepare` 语句的评估，`SQLITE3_column` 语句将显示结果集中的一个单独的列。`SQLITE3_column` 在 `SQLITE` API 中执行占位符功能，并且是其他各种函数的中心点，例如 `SQLite_column_count()`。

有关更多信息，请参阅以下内容：

![sqlite3_column()](img/B04725_05_04.jpg)

## sqlite3_finalize()

如其名所示，此语句将最终化并密封所有预准备语句。一旦执行了 `sqlite3_finalize` 语句，任何内存都将被释放，内部进程资源也将被释放。一旦完成，该语句就不能再被重用，并且在内部也不有效。有关更多信息，请参阅以下命令：

![sqlite3_finalize()](img/B04725_05_05.jpg)

## sqlite3_close()

这是最后要执行的部分，即 `sqlite3_close` 命令，它将使用数据库连接的指针和引用关闭数据库，并且必须在关闭连接之前完成之前创建的预准备语句。

如前所述，为了在 SQLite 或任何其他数据库中调用或运行 SQL 语句，您必须连接到数据库，并在完成工作后断开连接。

之前的 `SQLite_open()` 语句是直接使用 C API 而不涉及 Swift 语言实现的一种方式。以下是在 C API 和 Swift 中使用打开数据库语句的两种方法。有两种类型的处理方式：

+   使用 **C API** 和打开数据库语句

+   使用 Swift 和打开数据库语句

### 使用 C API 和打开数据库语句

看看以下代码：

[PRE0]

定义了一个名为 `db1` 的变量来调用 `SQLiteDatabase()` 函数。然后使用括号内的数据使用 `db1.open()` 方法指向 `database1` 数据库，如前述代码所示。

### 使用 Swift 和打开数据库语句

使用 Swift 打开数据库的另一种方法是：

[PRE1]

请记住，对于 Swift，您必须导入 `sqlite3.h` 文件并将 `libsqlite3.0.dylib` SQLite 库添加到您的项目中。

要将 `libsqlite3.0.dylib` 添加到您的项目中，请按照以下步骤操作：

1.  在项目编辑器中选择目标和框架。

1.  在编辑器的顶部点击**构建阶段**，并打开包含库部分的链接。

1.  点击**+**以添加框架，或者在我们的情况下，添加`libsqlite3.0.dylib` SQLite库。![使用Swift与开放数据库语句](img/B04725_05_06.jpg)

1.  使用`NSSearchPathForDirectoriesinDomains`搜索来设置`datadocuments`变量，然后设置`databasepath`变量作为`sqlite`文件的存放位置。

1.  设置一个名为`databasedb`的变量，并执行检查以查看带有输入参数的`sqlite3_open()`函数是否实际打开数据库，否则显示错误信息。

SQLite的美丽之处在于其灵活性和产品的扩展性，包括可扩展的SQL功能，如排序序列，以及使你的应用程序与众不同和独特的SQL函数。改变的范围和扩展应用程序的能力正在增长。

你构建的扩展可以链接到你的应用程序中，你可以使用`SQLite_extension_init`函数作为指针或地址来确保名称不冲突。

SQLite扩展被归类为**DLL**（动态链接库——一组在需要时由大型应用程序使用的程序）。

根据定义，DLL需要入口或起始点来与程序交互。这是进程附加到DLL并建立连接或交换信息和使用功能的地方。入口点函数用于在交互时执行清理任务或初始化。当进程使用入口点函数时，它可以用于分配内存或虚拟地址空间。

SQLite可以使用加载扩展，这些扩展是在SQLite外部编写的，根据需要测试和部署。一旦开发完成，这些扩展可以轻松地链接到SQLite。如果SQLite没有某些功能，第三方可以开发它，并将其提供给特定行业的潜在客户或用户，例如。

当为SQLite创建扩展时，每个操作系统的扩展都不同：

+   一些Unix系统使用`.so`文件扩展名

+   Windows系统使用`.dll`扩展

+   OS X（Mac）系统使用`.Dylib`扩展

这显示了该软件的巨大灵活性，可以满足各种操作系统，因此允许SQLite数据库应用于不同的系统和技术。

## load_extension()

`load_extension(X,Y)`函数是另一个功能强大的函数，允许加载函数/扩展。其方法与`sqlite3_load_extension()` C接口类似。这两种方法都使用入口点，需要一个名称作为标识符。可以传递空指针作为输入参数。

有一些更改数据库但不返回任何结果的命令，例如`Update`语句。执行此任务的函数是`sqlite3_exec()`。这种方法更快，且学习或执行都不困难。

## sqlite3_exec()

`sqlite3_exec()` 过程包含一个指向打开数据库的指针，一个或多个 SQL 语句，以及一个使用指向回调函数的指针作为其功能的一部分。它将使用一个或多个 SQL 语句与一个以 null 结尾的字符串结合进行处理。查询的每一行都将有一个指向回调函数的指针。作为回调函数的第一个参数的一部分，还将发送一个指针。包括一个指向错误字符串变量的指针。

## sqlite3_config()

`sqlite3_config()` 函数的功能允许在全局范围内进行更改。必须在打开数据库之前调用 `sqlite3_config()`。`sqlite3_config()` 接口将允许调整 SQLite 的内存分配，并为整个过程生成错误日志。它设置并配置 `SQLite` 库，控制内存分配和相关资源的许多方面。

为了扩展 SQLite，需要进一步研究并使用 `sqlite3_create_collation()`、`sqlite3_create_function()`、`sqlite3_create_module()` 和 `sqlite3_vfs_register()` 等函数和例程，根据需要作为改进产品功能的一部分。这些系统的维护将限于那些了解和使用该技术的人。

以下是一些使用 `select`、`update`、`delete` 和 `insert` 命令结合 Apple 的新语言 Swift 使用数据库函数的示例。

如前所述，存在不同的 SQLite 包装器，其中一些专门为 Swift 编写（不多），但最常见的是 **FMDB**，它已被用于不同的应用程序测试：

1.  为了将 Objective-C 引入 Swift，需要一个“桥接头”，即 `sqlite3.h`。要使用此头文件，请使用以下命令：

    [PRE2]

1.  如前所述，将 `libsqlite3.0.dylib` SQLite 库添加到您的项目中。

1.  将 `libsqlite3.0.dylib` 库添加到项目后，下一个任务是创建数据库。

1.  接下来，使用 `sqlite3_exec` 功能执行 `create table...` SQL 语句，例如，在使用 Swift 的过程中：

    [PRE3]

1.  下一个要使用的语句是 `Insert` 语句，用于将数据输入到新创建的 `test` 表中。以下信息将展示如何准备、绑定和执行 SQL 语句。将使用 `sqlite3_prepare_v2` 函数准备 SQL，使用占位符 `?` 来绑定所需值：

    [PRE4]

1.  可以使用一个常量 `SQLITE_TRANSIENT` 作为以下过程的组成部分：

    [PRE5]

这是使用这些变量的标准方式。有时，如果它们没有作为 `.h` 文件的一部分或在下述部分中定义，则这些变量可能不起作用。由于“不安全的指针转换”，它们在 Swift 中不受支持。

## 准备语句

作为 `prepare` 语句的一部分，使用 `sqlite3_prepare_v2` 的功能与 SQL 语句一起使用，使用问号 (`?`) 作为占位符来绑定输入值。以下示例展示了这一点：

[PRE6]

标准是使用 `SQLITE_STATIC` 和 `SQLITE_TRANSIENT` 作为设置，如下所示：

[PRE7]

在 Swift 2 中，代码可能会有所改变，如下所示：

[PRE8]

接下来，让我们使用一个 `NULL` 值执行一个 `insert` 语句来证明 SQL 确实是有效的：

[PRE9]

如前所述，SQLite 可以通过允许每个 SQL 语句只准备一次、评估、执行然后销毁来工作，但它也提供了使用如 `sqlite3_reset()` 和 `sqlite3_bind()` 函数之类的例程来准备相同的系统并多次评估不同时间的功能。SQLite 是一个良好且功能齐全的数据库，无需任何调整即可适用于不同的应用程序。

以下代码用于在完成工作后关闭数据库：

[PRE10]

这是具有内置 SQL 功能的语言的优势。

本章中的信息主要集中在 C API 的功能上。今天的开发者将根据需要扩展并使用不同类型的功能，为他们的应用程序嵌入 Swift、Objective-C、Java 或其他语言的使用。

# 摘要

在本章中，你学习了如何扩展 C API 并生成代码，这些代码可以用来构建一些有趣、激动人心、新颖和智能的数据驱动应用程序，并促进 SQLite 的使用。所使用的语言是 Swift。在下一章中，你将简要了解如何使用 Swift 与 IOS 和 SQLite，以及如何安装 Xcode 并与 Swift 和 SQLite 库一起工作。
