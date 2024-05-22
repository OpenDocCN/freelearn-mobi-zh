# 前言

Android 可能是本十年的热词。在短短的时间内，它已经占据了大部分手机市场。Android 计划在今年秋天通过 Android L 版本接管可穿戴设备、我们的电视房间以及我们的汽车。随着 Android 的快速增长，开发人员也需要提升自己的技能。面向数据库的应用程序开发是每个开发人员都应该具备的关键技能之一。应用程序中的 SQLite 数据库是数据中心产品的核心，也是构建优秀产品的关键。理解 SQLite 并实现 Android 数据库对一些人来说可能是一个陡峭的学习曲线。诸如内容提供程序和加载程序之类的概念更加复杂，需要更多的理解和实现。*Android SQLite Essentials*以简单的方式为开发人员提供了构建基于数据库的 Android 应用程序的工具。它是根据当前行业的需求和最佳实践编写的。让我们开始我们的旅程。

# 本书涵盖内容

第一章, *进入 SQLite*，提供了对 SQLite 架构、SQLite 基础知识及其与 Android 的连接的深入了解。

第二章, *连接点*，介绍了如何将数据库连接到 Android 视图。它还涵盖了构建以数据库为中心/启用的应用程序应遵循的一些最佳实践。

第三章, *分享就是关怀*，将反映如何通过内容提供程序访问和共享 Android 中的数据，以及如何构建内容提供程序。

第四章, *小心处理线程*，将指导您如何使用加载程序并确保数据库和数据的安全。它还将为您提供探索在 Android 应用程序中构建和使用数据库的替代方法的提示。

# 本书所需内容

为了有效地使用本书，您需要一个预装有 Windows、Ubuntu 或 Mac OS 的工作系统。下载并设置 Java 环境；我们需要这个环境来运行我们选择的 IDE Eclipse。从 Android 开发者网站下载 Android SDK 和 Eclipse 的 Android ADT 插件。或者，您可以下载包含 Eclipse SDK 和 ADT 插件的 Eclipse ADT 捆绑包。您也可以尝试 Android Studio；这个 IDE 刚刚转入 beta 版，也可以在开发者网站上找到。确保您的操作系统、JDK 和 IDE 都是 32 位或 64 位中的一种。

# 本书适合对象

*Android SQLite Essentials*是一本面向 Android 程序员的指南，他们想要探索基于 SQLite 数据库的 Android 应用程序。读者应该具有一些 Android 基本构建块的实际经验，以及 IDE 和 Android 工具的知识。

# 约定

在本书中，您将找到一些文本样式，用于区分不同类型的信息。以下是这些样式的一些示例以及它们的含义解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“要关闭`Cursor`对象，将使用`close()`方法调用。”

代码块设置如下：

```kt
ContentValues cv = new ContentValues();
cv.put(COL_NAME, "john doe");
cv.put(COL_NUMBER, "12345000");
dataBase.insert(TABLE_CONTACTS, null, cv);
```

任何命令行输入或输出都以以下形式书写：

```kt
adb shell SQLite3 --version
SQLite 3.7.11: API 16 - 19
SQLite 3.7.4: API 11 - 15
SQLite 3.6.22: API 8 - 10
SQLite 3.5.9: API 3 - 7

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词、菜单或对话框中的单词等，都会以这种方式出现在文本中：“从**Windows**菜单中转到**Android 虚拟设备管理器**以启动模拟器。”

### 注意

警告或重要提示以以下方式显示在一个框中。

### 提示

提示和技巧以这种方式出现。
