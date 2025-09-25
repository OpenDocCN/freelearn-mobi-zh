# 3

# 备份您的 WhatsPackt 消息

在任何聊天应用中，数据处理都是一个重要的问题——我们需要确保发送和接收的消息被正确存储，在需要时能够快速检索，并且对潜在的损失具有弹性，如设备故障或意外删除等不可预见的情况。这需要一个强大的数据持久化策略。我们还需要考虑性能和用户体验，这需要有效的缓存机制，并确保在数据丢失或用户更换设备时我们有备份。

在本章中，我们将首先向您介绍 Room，这是一个持久化库，它提供了一个在 SQLite 之上的抽象层，使得在 Android 中处理数据库变得更加容易。您将了解其架构和组件，以及如何使用它来存储和检索聊天对话和消息。

接下来，我们将处理创建一个缓存机制，协调 Room 在本地使用以及使用 API 从后端获取数据。

接下来，我们将让您熟悉 Firebase 存储。您将学习如何设置它，了解其优势，以及如何确保存储在其中的数据安全。然后我们将使用 Firebase 存储来创建聊天对话的备份，这对于任何聊天应用来说都是一个基本功能。

最后，我们将探讨如何使用`WorkManager` API，它使得即使在应用退出或设备重启的情况下也能轻松安排可延迟的异步任务。您将了解它如何用于安排聊天备份，以及如何将这些备份上传到**Amazon Simple Storage Service**（**Amazon S3**），确保数据安全。

因此，在本章中，我们将涵盖以下主题：

+   理解 Room

+   在 WhatsPackt 中实现 Room

+   了解 Firebase 存储

+   安排**WorkManager**发送备份

+   使用 Amazon S3 进行存储

# 技术要求

如前一章所述，您需要已安装 Android Studio（或您偏好的其他编辑器）。

我们还假设您已经跟随了前一章的内容。您可以从这里下载本章的完整代码：[`github.com/PacktPublishing/Thriving-in-Android-Development-using-Kotlin/tree/main/Chapter-3`](https://github.com/PacktPublishing/Thriving-in-Android-Development-using-Kotlin/tree/main/Chapter-3)。

# 理解 Room

在 Android 开发中，最基本的一项任务就是管理您应用的数据在本地数据库中的存储。Android Jetpack 的一部分，**Room**持久化库，是在 Android 中自带的一个流行数据库 SQLite 之上的抽象层。Room 提供了更健壮的数据库访问，同时利用了 SQLite 的全部功能。

## Room 的关键特性

在 Room 之前，开发者主要直接使用**SQLite**或其他**对象关系映射（ORM**）库。虽然 SQLite 功能强大，但使用起来可能很繁琐，因为它需要编写大量的样板代码。此外，SQL 查询中的错误通常直到运行时才会被发现，这可能导致崩溃。

Room 通过提供一个比标准 SQLite 更简单、更健壮的 API 来解决这些问题，用于管理本地数据存储。以下是它的一些关键特性：

+   **SQL 查询的编译时验证**：Room 在编译时验证您的 SQL 查询，而不是在运行时。这意味着如果您的查询中存在错误，您将在编译应用时立即知道，而不是在将应用发布给用户之后。这导致更健壮和可靠的代码。

+   **减少样板代码**：使用 Room，您不需要编写太多代码来执行简单的数据库操作。这导致代码更清晰、更易于阅读。

+   **与其他架构组件的集成**：Room 被设计成与其他**Android 架构组件（AAC**）库组件无缝集成，例如**LiveData**和**ViewModel**。这意味着您可以创建一个结构良好、健壮的应用，遵循 Android 开发的最佳实践。

+   **易于迁移路径**：Room 提供了强大的迁移支持，包括迁移路径和测试。随着您的应用数据需求的发展，Room 使您能够轻松地调整数据库结构以满足这些需求。

+   **支持复杂查询**：尽管简化了与 SQLite 的交互，但 Room 仍然允许您在需要更多灵活性和功能时执行复杂的 SQL 查询。

如您所见，Room 提供了一种高效且简化的方法来管理您应用的本地区域数据。它是一个强大的工具，可以使您的 Android 开发体验更加愉快和高效。

## Room 的架构和组件

房间架构基于三个主要组件：

+   **数据库**

+   **实体**

+   **数据访问对象（DAO**）

在这里，您可以看到每个 Room 组件如何与整个应用交互：

![图 3.1：Room 架构图](img/B19443_03_001.jpg)

图 3.1：Room 架构图

理解这些组件对于有效地使用 Room 至关重要，因此让我们更深入地探讨它们。

### 数据库

Room 中的`Database`类是一个高级类，作为您应用持久数据的主体访问点。它是一个抽象类，您在其中为应用中的每个`@Dao`注解定义一个抽象方法。当您创建`Database`类的实例时，Room 会生成这些 DAO 方法的实现代码（DAO 将在稍后详细介绍）。

`Database`类使用`@Database`注解，指定其包含的实体和数据库版本。如果您修改数据库模式，您需要更新版本号并定义一个迁移策略，如下例所示：

```kt
@Database(entities = [Message::class, Conversation::class],
    version = 1)
abstract class ChatAppDatabase : RoomDatabase() {
    abstract fun messageDao(): MessageDao
    abstract fun conversationDao(): ConversationDao
}
```

在这里，我们定义了一个包含两个实体`Message`和`Conversation`的`ChatAppDatabase` Room `Database`类。我们还定义了访问我们的 DAO 的抽象方法 - `messageDao()`和`conversationDao()`。`@Database`注解中的`entities`参数接受数据库中所有实体的数组，而`version`参数用于数据库迁移目的。

### 实体

**实体**在 Room 中表示数据库中的表。每个实体对应一个表，每个实体的实例代表表中的一行。Room 使用实体中的类字段来定义表中的列。

您可以通过在数据类上标注`@Entity`来声明一个实体。每个`@Entity`类代表数据库中的一个表，并且您可以定义表名。如果您没有定义表名，Room 将使用类名作为表名，如下例所示：

```kt
@Entity(tableName = "messages")
data class Message(
    @PrimaryKey val id: String,
    @ColumnInfo(name = "conversation_id") val
        conversationId: String,
    // ...
)
```

在这里，`Message`是一个实体，代表我们数据库中的`"messages"`表。`Message`的每个实例将代表`"messages"`表中的一行。`Message`类中的每个属性代表表中的一列。`@PrimaryKey`注解用于表示主键，而`@ColumnInfo`注解用于指定数据库中的列名。如果没有指定，Room 将使用变量名作为列名。

### DAO

DAO 是定义您想要执行的所有数据库操作的接口。对于每个 DAO，您可以定义不同的操作方法，例如插入、删除和查询。

您应该使用`@Dao`注解一个接口，然后使用相应的操作注解每个方法，例如`@Insert`、`@Delete`、`@Update`或`@Query`用于自定义查询。然后，Room 将在编译时自动生成执行这些操作所需的代码。以下是一个示例：

```kt
@Dao
interface MessageDao {
    @Insert
    fun insert(message: Message)
    @Query("SELECT * FROM messages WHERE conversation_id =
        :conversationId")
    fun getMessagesForConversation(conversationId: String):
        List<Message>
}
```

在这个`MessageDao`接口中，我们定义了两个方法 - `insert()`用于将`Message`对象插入到我们的数据库中，以及`getMessagesForConversation()`从我们的数据库中检索与特定对话相关的所有消息。`@Insert`注解是一个方便的注解，用于将实体插入到表中。`@Query`注解允许我们编写 SQL 查询以执行复杂的读写操作。

理解这些组件将使我们能够有效地利用 Room 的强大功能。以下几节将指导您在 WhatsPackt 应用中实现 Room 的过程，从在 Android Studio 中设置它开始，到创建实体和 DAO。

# 在 WhatsPackt 中实现 Room

在本节中，您将指导我们在我们的聊天应用中实现 Room 的实际步骤。我们将从在 Android Studio 中设置 Room 开始，然后创建实体和 DAO，最终使用这些组件与我们的数据库进行交互。

## 添加依赖项

要开始使用 Room，我们首先需要在项目中包含必要的依赖项。打开你的 `build.gradle` 文件，并在 `dependencies` 下添加以下依赖项：

```kt
dependencies {
    implementation "androidx.room:room-runtime:2.3.0"
    kapt "androidx.room:room-compiler:2.3.0"
    implementation "androidx.room:room-ktx:2.3.0"
    // optional - Test helpers
    testImplementation "androidx.room:room-testing:2.3.0"
}
```

`room-runtime` 依赖项包括 Room 的核心库，而 `room-compiler` 依赖项是 Room 注解处理能力的必需品。Room 的 Kotlin 扩展和协程支持由 `room-ktx` 提供，而 `room-testing` 提供了用于测试 Room 设置的有用类。

添加这些行后，同步你的项目。你可以通过 `build.gradle` 文件来完成：

![图 3.2：当 Android Studio 检测到 Gradle 文件有任何更改时出现的“同步现在”选项](img/B19443_03_002.jpg)

图 3.2：当 Android Studio 检测到 Gradle 文件有任何更改时出现的“同步现在”选项

我们现在准备好创建我们的数据库。

## 创建数据库

如前所述，`Database` 组件是我们应用程序数据的主要访问点。因此，让我们创建一个 `ChatAppDatabase` 类：

```kt
@Database(entities = [Message::class, Conversation::class],
version = 1)
abstract class ChatAppDatabase : RoomDatabase() {
    abstract fun messageDao(): MessageDao
    abstract fun conversationDao(): ConversationDao
    companion object {
        @Volatile
        private var INSTANCE: ChatAppDatabase? = null
        fun getDatabase(context: Context): ChatAppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    ChatAppDatabase::class.java,
                    "chat_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

`@Database` 注解将这个类标记为 Room 数据库。它接受两个参数：

+   **entities** 是一个被 **@Entity** 注解的类数组，代表数据库中的表。在这种情况下，**Message** 和 **Conversation** 类是 **ChatAppDatabase** 的实体。

+   **version** 是数据库版本。如果你对数据库模式进行了更改，你需要增加这个版本号并定义一个迁移策略。

接下来，`abstract fun messageDao(): MessageDao` 和 `abstract fun conversationDao(): ConversationDao` 是返回相应 DAO 的抽象方法。它们没有方法体，因为 Room 会生成它们的实现。

然后，我们通过使用 `@Volatile` 注解声明一个伴随对象来持有 `ChatAppDatabase` 的单例实例。这个注解意味着 `INSTANCE` 可以被多个线程同时访问，但总是处于一致的状态，这意味着一个线程对 `INSTANCE` 的更改会立即对所有其他线程可见。`INSTANCE` 被标记为可空，因为它可能不会立即初始化。

在 `getDatabase()` 函数中，我们正在实现创建类单例实例的常见模式，以线程安全的方式。这个模式确保 `ChatAppDatabase` 只会创建一个实例。

我们使用 `?:` 操作符来检查 `INSTANCE` 是否不是 `null`，如果是，则进入同步块。这个块确保一次只有一个线程可以进入这段代码，防止在多个线程从多个线程并发调用该函数时创建多个 `ChatAppDatabase` 实例。

在同步块内，我们调用 `Room.databaseBuilder()` 来创建 `ChatAppDatabase` 的新实例。我们提供应用程序上下文以避免内存泄漏，数据库的类以及数据库的名称。

最后，我们调用 `build()` 来创建 `ChatAppDatabase` 实例。

在创建新实例后，我们将其分配给`INSTANCE`以缓存它，然后返回实例。下次调用`getDatabase`时，它将返回缓存的数据库实例而不是创建一个新的实例。这很重要，因为创建 Room 数据库实例是一个昂贵的操作，拥有多个实例将会浪费资源。

这种结构对于创建一个数据库实例至关重要，这将使我们能够存储消息和对话。

下一步是创建实体类。

## 创建实体类

我们将要创建的第一个实体类是`Message`类：

```kt
@Entity(
    tableName = "messages",
    foreignKeys = [
        ForeignKey(
            entity = Conversation::class,
            parentColumns = arrayOf("id"),
            childColumns = arrayOf("conversation_id"),
            onDelete = ForeignKey.CASCADE
        )
    ],
    indices = [
        Index(value = ["conversation_id"])
    ]
)
data class Message(
    @PrimaryKey(name = "id") val id: Int,
    @ColumnInfo(name = "conversation_id") val
        conversationId: Int,
    @ColumnInfo(name = "sender") val sender: String,
    @ColumnInfo(name = "content") val content: String,
    @ColumnInfo(name = "timestamp") val timestamp: Long
)
```

在这段代码中，我们在注解中包含了相当多的指令，所以让我们逐一解释它们。

`@Entity`注解告诉 Room 将这个类作为数据库中的表处理。它带有可选参数，其中一些在这里使用：

+   **tableName**: 这设置了表在数据库中的名称。在这种情况下，我们的表将命名为**"messages"**。

+   **foreignKeys**: 这设置了与另一个表的外键关系。一个**ForeignKey**实例接受四个主要参数：

    +   **entity**: 这表示与该实体有关系的父表类。在这种情况下，它是**Conversation::class**。

    +   **parentColumns**: 这指定了父实体中外键引用的列。在这里，它是**Conversation**的**id**字段。

    +   **childColumns**: 这指定了子实体中包含外键的列。在这里，它是**Message**中的**conversation_id**字段。

    +   **onDelete**: 这表示如果父表中的引用行被删除，将采取的操作。在这里，使用**ForeignKey.CASCADE**，这意味着如果删除**Conversation**实例，所有具有**conversation_id**值引用对话 ID 的消息也将被删除。

+   **indices**: 这用于在**conversation_id**上创建索引以加快查询速度。索引以额外的磁盘空间和较慢的写入速度为代价加快数据检索速度。索引在这里特别有用，因为我们经常会执行与特定对话相关的操作，对**conversation_id**进行索引将使这些操作更高效。

然后，我们还添加了注解到类的属性中：

+   **@PrimaryKey**: 这个注解表示**id**字段是**Message**表的唯一键。主键唯一标识表中的每一行。这里我们可以使用**autoGenerate = true**，这意味着这个字段将为每一行新行自动填充一个递增的整数。

+   **@ColumnInfo(name = "column_name")**: 这个注解允许你在数据库中指定自定义列名。如果没有指定，Room 将使用变量名作为列名。

现在，让我们创建一个`Conversation`实体：

```kt
@Entity(
    tableName = "conversations",
)
class Conversation(
    @PrimaryKey
    @ColumnInfo(name = "id") val id: String,
    @ColumnInfo(name = "last_message_time") val
        lastMessageTime: Long
)
```

`Conversation`实体非常简单——我们只需在数据库中存储`Conversation` ID 和对话中最后一条消息的时间。

现在我们已经创建并定义了我们的实体，是时候创建 DAO 以获取和更新数据了。

## 创建 DAO

DAO 是一个接口，它作为应用程序代码和数据库之间的通信层。它定义了我们在数据库中的实体可能执行的所有操作的函数。

让我们从`Message`实体的 DAO 开始：

```kt
@Dao
interface MessageDao {
    @Query("SELECT * FROM messages WHERE conversation_id =
        :conversationId ORDER BY timestamp ASC")
    fun getMessagesInConversation(conversationId: Int):
        Flow<List<Message>>
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertMessage(message: Message): Long
    @Delete
    suspend fun deleteMessage(message: Message)
}
```

分析代码，我们有以下内容：

+   **@Dao**：这个注解用于标识接口作为 DAO。

+   **@Query**：这个注解用于指定复杂数据检索任务的 SQL 语句。

+   **@Insert**：这个注解用于定义一个方法，将它的参数插入到数据库中。**OnConflictStrategy.REPLACE**意味着如果已经存在具有相同主键的消息，它将被新的消息替换。

+   **@Delete**：这个注解用于定义一个方法，从数据库中删除它的参数。

现在，让我们为`Conversation`实体创建一个 DAO：

```kt
@Dao
interface ConversationDao {
    @Query("SELECT * FROM conversations ORDER BY
        last_message_time DESC")
    fun getAllConversations(): Flow<List<Conversation>>
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertConversation(conversation:
        Conversation): Long
    @Delete
    suspend fun deleteConversation(conversation:
        Conversation)
}
```

注解在这里的作用与在`MessageDao`中一样。在这里，我们正在按最后一条消息的时间顺序检索所有对话，并且我们有插入和删除对话的方法。

现在，我们需要为其他应用组件提供这些 DAO，以便它们可以被注入。考虑到这一点，我们将创建以下模块：

```kt
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext appContext:
    Context): ChatAppDatabase {
        return ChatAppDatabase.getDatabase(appContext)
    }
    @Provides
    fun provideMessageDao(database: ChatAppDatabase):
    MessageDao {
        return database.messageDao()
    }
    @Provides
    fun provideConversationDao(database: ChatAppDatabase):
    ConversationDao {
        return database.conversationDao()
    }
}
```

由于我们已经在之前的章节中介绍了 Hilt 模块的创建，我们不会再次遍历所有代码。相反，以下是代码的关键部分：

+   我们使用**@Singleton**来表示应该只创建一个对象实例，并将其作为依赖项提供。

+   我们想要使用的`Context`可能会让我们感到困惑或提供一个不适合这种情况的`Context`。使用`@ApplicationContext`限定符将确保注入的`Context`将是预期的（在这种情况下是`Application Context`）。

现在，就像我们在上一章中为 API 或 WebSocket 所做的那样，我们将创建一个数据源来连接数据库：`LocalMessagesDataSource`。

## 创建一个 LocalMessagesDataSource 数据源

我们需要创建一个`LocalMessagesDataSource`数据源，它将封装 DAO 并暴露我们应用需要的特定数据库操作。这样，如果我们将来决定更改数据库，我们只需在这里更改（而不是在其他消费者中更改）。这个类将在更高层次的抽象中作为 DAO，简化我们应用其余部分的 API，并使在测试中模拟数据库变得更加容易。

在下面的代码中，我们只是调用我们在 DAO 中已经定义的函数：

```kt
class MessagesLocalDataSource @Inject constructor(private
val messageDao: MessageDao) {
    fun getMessagesInConversation(conversationId: Int):
    Flow<List<Message>> {
        return
            messageDao.getMessagesInConversation(
                conversationId)
    }
    suspend fun insertMessage(message: Message): Long {
        return messageDao.insertMessage(message)
    }
    suspend fun deleteMessage(message: Message) {
        messageDao.deleteMessage(message)
    }
}
```

正如我们之前所说的，我们将使用这个数据源来封装数据库并提供额外的抽象层。

现在，是时候将这个本地数据源与远程数据源结合起来。这将迫使我们考虑缓存策略。

## 在 MessagesRepository 组件中处理两个数据源

到目前为止，我们只有一个数据源（WebSocket 数据源），但我们希望用户即使在短时间内没有连接也能检索到他们的消息。这就是我们刚刚创建数据库并使其准备就绪的原因。

由于我们的用例是为 WebSocket 提供后备，以便用户可以继续检查他们的消息，我们将遵循一种策略，其中主要的事实来源将继续是 WebSocket，但我们将在应用数据库中存储消息的副本。此外，我们不想让这个数据库的记录无限增长，所以我们为每场对话设置了 `100` 条消息的上限。

负责组合两个数据源的组件是 `MessagesRepository`，我们在上一章中已经实现，使其连接到 `WebsocketDataSource`。现在让我们修改它以包含两个数据源，并协调数据检索和本地存储：

```kt
class MessagesRepository @Inject constructor(
    private val dataSource: MessagesSocketDataSource,
    private val localDataSource: DatabaseDataSource
): IMessagesRepository {
```

接下来，我们将修改 `getMessages()` 方法，以包含将来自 `MessagesSocketDataSource`（远程数据源）的信息存储到 `DatabaseDataSource`（本地数据源）的逻辑：

```kt
override suspend fun getMessages(chatId: String, userId:
String): Flow<Message> {
        return flow {
            try {
                dataSource.connect().collect { message ->
                    localDataSource.insertMessage(message)
                    emit(message)
                    manageDatabaseSize()
                }
            } catch (e: Exception) {
                localDataSource.getMessagesInConversation(
                chatId.toInt()).collect {
                    it.forEach { message -> emit(message) }
                }
            }
        }
    }
```

如所见，我们已经连接到套接字数据源，但我们把这个动作包裹在一个 `try`-`catch` 块中。所以，如果一切顺利，我们将把每条新消息存储到我们的数据库中，然后将其在流程中发出。

同时，我们调用 `manageDatabaseSize()`，这将检查并保持数据库的大小在我们设定的限制之下（每场对话最多 100 条消息）。如果套接字失败，我们将直接从数据库中检索消息。

现在，我们还将修改 `sendMessage` 方法，其中我们将存储每条新发送的消息：

```kt
    override suspend fun sendMessage(chatId: String,
    message: Message) {
        dataSource.sendMessage(message)
        localDataSource.insertMessage(message)
    }
```

断开连接将与之前保持一致，因为我们不需要做任何与新的数据源相关的事情：

```kt
    override suspend fun disconnect() {
        dataSource.disconnect()
    }
```

最后，这是我们将实施的机制，以保持数据库的大小在商定的每场对话消息数量之下：

```kt
    private suspend fun manageDatabaseSize() {
        val messages =
            localDataSource.getMessagesInConversation(
                chatId.toInt()).first()
        if (messages.size > 100) {
            // Delete the oldest messages until we have 100
               left
            messages.sortedBy { it.timestamp
            }.take(messages.size - 100).forEach {
                localDataSource.deleteMessage(it)
            }
        }
    }
}
```

我们将获取与对话相关的所有消息，并检查其大小是否超过 100。然后，我们将根据它们的时间戳对它们进行排序，并移除最旧的那些。

现在，我们已经将 Room 数据库集成到我们的应用中。即使我们失去连接，我们最后的消息也将可用。在下一节中，让我们看看我们如何也将这些消息的备份发送到云端存储。为此，我们将使用 Firebase 存储。

# 了解 Firebase 存储

**Firebase 存储**，也称为 Firebase 云存储，是一个为 Google 规模构建的强大对象存储服务。它使开发者能够存储和检索用户生成的内容，如照片、视频或其他形式用户数据。Firebase 存储由 **Google Cloud Storage**（**GCS**）支持，使其对任何大小的数据都稳健且可扩展，从小型文本文件到大型视频文件。

这里是 Firebase 存储的一些关键特性和功能：

+   **用户生成内容**：Firebase Storage 允许你的用户直接从他们的设备上传自己的内容。这可能包括从个人资料图片到博客文章的任何内容。

+   **与 Firebase 和 Google Cloud 的集成**：Firebase Storage 与 Firebase 生态系统的其余部分无缝集成，包括 Firebase 身份验证和 Firebase 安全规则。它也是更大 Google Cloud 生态系统的一部分，这为使用 Google Cloud 的高级功能，如 Cloud Functions，提供了可能性。

+   **安全性**：Firebase Storage 提供了强大的安全功能。使用 Firebase 安全规则，你可以控制谁可以访问哪些数据。你可以根据用户的认证状态、身份和声明、数据模式和元数据来限制访问。

+   **可扩展性**：Firebase Storage 被设计用来处理大量的上传、下载和数据存储。它能够根据你的用户基础和流量自动扩展，这意味着你不需要担心容量规划。

+   **离线功能**：Firebase Cloud Storage 的**软件开发工具包**（**SDKs**）为你的 Firebase 应用添加了 Google 安全，无论网络质量如何，都可以用于文件的上传和下载。你可以用它来暂停、恢复和取消传输。

+   **丰富媒体**：Firebase Storage 支持丰富媒体内容。这意味着你可以用它来存储图片、音频、视频，甚至是其他二进制数据。

+   **强一致性**：Firebase Storage 保证强一致性，这意味着一旦上传或下载完成，数据将立即从所有 Google Cloud Storage 位置可用，并且后续的读取将返回最新更新的数据。

在我们的上下文中，一个消息应用，Firebase Storage 可以用来存储和检索消息历史或备份、共享文件，甚至在对话中存储多媒体内容。这可以作为可靠的备份解决方案或跨多个设备同步聊天历史的方法。然而，你需要确保你处理隐私和安全问题，特别是由于聊天对话可能包含敏感数据。

## 如何使用 Firebase Storage

在 Firebase Storage 中，数据以对象的形式存储在分层结构中。Firebase Storage 中对象的完整路径包括项目 ID 和对象在存储桶中的位置。

注意

在云存储的上下文中，一个**桶**是一个基本容器，用于存储数据。它是数据组织层次结构中的主要父容器。云存储中的所有数据都存储在桶中。桶的概念被许多云存储系统使用，包括 GCS、Amazon S3 和 Firebase Storage。这些系统通常允许你在存储空间中创建一个或多个桶，然后将数据作为对象或文件上传到这些桶中。每个桶在云存储系统中都有一个唯一的名称，它包含数据对象或文件，每个对象或文件都由一个键或名称标识。

对象的位置由您指定的路径定义。此路径类似于文件系统路径，包括目录和文件名。例如，在 `images/profiles/user123.jpg` 路径中，`images` 和 `profiles` 是目录，而 `user123.jpg` 是文件名。

当您将文件上传到 Firebase 存储时，您创建一个指向您将要存储文件的位置的引用。此引用由一个 `StorageReference` 对象表示，您通过在指向您的 Firebase 存储 bucket 的引用上调用 `child()` 方法并传递路径作为参数来创建它，如下例所示：

```kt
val storageRef = Firebase.storage.reference
val fileRef =
storageRef.child("images/profiles/user123.jpg")
```

在这里，`fileRef` 是 `images` 目录中 `profiles` 目录内 `user123.jpg` 文件的引用。

您可以使用此引用执行各种操作，例如上传文件、下载文件或获取文件的 URL。这些操作中的每一个都返回一个 `Task` 对象，您可以使用它来监控操作进度或获取其结果。

Firebase 存储中的路径是灵活的，您可以根据应用程序的需要对其进行结构化。例如，在消息传递应用程序中，您可能将对话记录存储在 `chat_logs` 目录中，每个日志的文件名是聊天 ID。聊天日志的路径可能如下所示：`chat_logs/chat123.txt`。

最后，值得注意的是，Firebase 存储使用规则来控制谁可以读取和写入您的存储 bucket。默认情况下，只有经过身份验证的用户可以读取和写入数据。您可以根据应用程序的需求自定义这些规则。

让我们开始在我们的项目中设置 Firebase 存储。

## 设置 Firebase 存储

要开始使用 Firebase 存储，我们首先需要将 Firebase Cloud Storage Android 库添加到我们的应用程序中。这可以通过将以下行添加到我们的模块的 `build.gradle` 文件中完成：

```kt
implementation 'com.google.firebase:firebase-storage-ktx'
```

关于聊天消息，一种方法是将聊天记录保存为 Firebase Storage 中的文本文件。每个对话可以有自己的文本文件，每条消息都是该文件中的一行。因此，我们将创建一个数据源来上传这些文件：

```kt
class StorageDataSource @Inject constructor(private val
firebaseStorage: FirebaseStorage) {
    suspend fun uploadFile(localFile: File, remotePath:
    String) {
        val storageRef =
            firebaseStorage.reference.child(remotePath)
        storageRef.putFile(localFile.toUri()).await()
    }
    suspend fun downloadFile(remotePath: String, localFile:
    File) {
        val storageRef =
            firebaseStorage.reference.child(remotePath)
        storageRef.getFile(localFile).await()
    }
}
```

在这里，我们将 Firebase 存储实例作为构造函数的参数添加，允许在通过 Hilt 实例化类时注入。`uploadFile` 和 `downloadFile` 方法使用 `await()` 扩展函数挂起协程，直到上传或下载操作完成。

为了能够使用 Firebase 存储实例，我们需要提供 `FirebaseStorage` 依赖项。为此，我们需要创建以下模块，以便 Hilt 了解如何获取它：

```kt
@Module
@InstallIn(SingletonComponent::class)
object StorageModule {
    @Singleton
    @Provides
    fun provideFirebaseStorage(): FirebaseStorage =
        FirebaseStorage.getInstance()
}
```

现在，我们需要创建这些文件，然后使用此数据源上传。我们将在新创建的存储库中完成此操作：`BackupRepository`。

`BackupRepository`仓库将在不同数据源（如通过 DAO 的本地数据库和远程数据源，如 Firebase 存储）与应用程序的其余部分之间充当中介。它从源检索数据，如有必要，对其进行处理，并以方便的形式将其提供给调用代码。

这是此存储库的代码：

```kt
class BackupRepository @Inject constructor(
    private val messageDao: MessageDao,
    private val conversationDao: ConversationDao,
    private val storageDataSource: StorageDataSource
) {
    private val gson = Gson()
    suspend fun backupAllConversations() {
        // Get all the conversations
        val conversations =
            conversationDao.getAllConversations()
        // Backup each conversation
        for (conversation in conversations) {
            val messages =
                messageDao.getMessagesForConversation(
                    conversation.conversationId)
            // create a JSON representation of the messages
            val messagesJson = gson.toJson(messages)
            // create a temporary file and write the JSON
               to it
            val tempFile = createTempFile("messages",
                ".json")
            tempFile.writeText(messagesJson)
            // upload the file to Firebase Storage
            val remotePath =
               "conversations/${conversation.conversationId
               }/messages.json"
            storageDataSource.uploadFile(tempFile,
               remotePath)
            // delete the local file
            tempFile.delete()
        }
    }
    private fun createTempFile(prefix: String, suffix:
    String): File {
        // specify the directory where the temporary file
           will be created
        val tempDir =
            File(System.getProperty("java.io.tmpdir"))
        // create a temporary file with the specified
           prefix and suffix
        return File.createTempFile(prefix, suffix, tempDir)
    }
}
```

如代码所示，它使用`ConversationDao`从本地数据库中检索所有对话。每个对话代表一个独特的聊天线程。

然后，对于每个对话，它使用`MessageDao`检索相关的消息，使用 Gson 库将消息转换为 JSON 字符串，将此 JSON 字符串写入临时文件，然后通过`StorageDataSource`将文件上传到 Firebase 存储。

一旦 Firebase 存储的上传完成，它将删除本地临时文件以清理设备上的存储空间。

`BackupRepository`处理所有数据检索、处理和存储的细节。应用程序的其他部分不需要知道数据是如何存储或检索的。它们只需与`BackupRepository`交互，该仓库为这些操作提供了一个简单的接口。这使得代码更容易维护、理解和测试。

最后，我们将创建`UploadMessagesUseCase`，它将负责执行上传操作。

## 创建`UploadMessagesUseCase`

`UploadMessagesUseCase`的责任将是使用`BackupRepository`执行备份。由于大部分逻辑已经在仓库中，代码将更简单，看起来像这样：

```kt
class UploadMessagesUseCase @Inject constructor(
    private val backupRepository: BackupRepository
) {
    suspend operator fun invoke() {
        backupRepository.backupAllConversations()
    }
}
```

现在，我们已经准备好检索和上传这些备份。由于这可能是一个耗时且资源消耗的任务，所以想法是定期执行，每周或每天一次。这就是`WorkManager`派上用场的地方。

# 安排`WorkManager`发送备份

`WorkManager`是用于需要保证和高效执行的任务的推荐工具。

`WorkManager`使用基于以下标准的底层作业调度服务：

+   它使用**JobScheduler**为 API 23 及以上的设备

+   对于 API 14 至 22 的设备，它使用**BroadcastReceiver**（用于系统广播）和**AlarmManager**的组合。

+   如果应用程序包含可选的**WorkManager**依赖项 Firebase **JobDispatcher**，并且设备上可用的 Google Play 服务，**WorkManager**将使用 Firebase **JobDispatcher**

`WorkManager`根据设备 API 级别和包含的依赖项选择安排后台任务的最佳方式。要使用`WorkManager`，我们首先需要了解如何创建`Worker`和`WorkRequest`实例。

## 介绍`Worker`类

创建一个`Worker`类（如果你使用 Kotlin 协程，则为`CoroutineWorker`），并重写`doWork()`方法来定义任务应该执行的操作。

`doWork()` 方法是放置需要在后台执行代码的地方。这是你定义需要执行的操作的地方，例如从服务器获取数据、上传文件、处理图像等等。

每个 `Worker` 实例有最多 10 分钟的时间来完成其执行并返回一个 `Result` 实例。`Result` 实例可以是三种类型之一：

+   **Result.success()**: 表示工作成功完成。你可以选择返回一个 **Data** 对象，该对象可以用作此工作的输出数据。

+   **Result.failure()**: 表示工作失败。你可以选择返回一个 **Data** 对象来描述失败。

+   **Result.retry()**: 表示工作失败，应该根据其重试策略在另一时间尝试。

`Worker` 的一个独特特性是，当 `Worker` 实例正在运行且应用进入后台时，`Worker` 实例可以继续运行，而如果设备在 `Worker` 实例运行时重启，任务可以在设备恢复后继续。这确保了即使在你的应用进程不存在的情况下，工作也会在创建 `WorkRequest` 实例时指定的约束下执行。

下面是一个基本的 `Worker` 类的示例：

```kt
class ExampleWorker(appContext: Context, workerParams:
WorkerParameters)
    : Worker(appContext, workerParams) {
    override fun doWork(): Result {
        // Code to execute in the background
        return Result.success()
    }
}
```

在示例中，我们扩展了 `Worker` 类并重写了 `doWork()` 方法来指定要执行的任务。在这种情况下，我们只是返回成功的结果，但实际工作的代码将放在 `// Code to execute in the background` 注释所在的位置。

要使我们的 `Worker` 实例工作，我们需要另一个组件：`WorkRequest`。让我们看看我们如何配置和使用它。

## 配置 WorkRequest 组件

`WorkRequest` 是一个定义单个工作单元的类。它封装了你的 `Worker` 类，以及工作运行必须满足的任何约束以及它需要的任何输入数据。

有两种具体的 `WorkRequest` 实现你可以使用：

+   **OneTimeWorkRequest**: 如其名所示，这代表一个一次性作业。它只会执行一次。

+   **PeriodicWorkRequest**: 这用于执行周期性任务的重复作业。可以定义的最小重复间隔是 15 分钟。这个限制在官方文档中有进一步的讨论：[`developer.android.com/reference/androidx/work/PeriodicWorkRequest`](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest).

`WorkRequest` 有几个选项可以设置工作执行的条件以及安排多个工作按特定顺序运行：

+   **约束**：**WorkRequest**实例可以设置**Constraints**对象，这允许您指定工作必须满足的条件才能有资格运行。例如，您可能需要设备处于空闲或充电状态，或者具有某种类型的网络连接。我们将在接下来的几段中详细了解这些条件。

+   **输入数据**：您可以使用**setInputData()**方法将输入数据附加到**WorkRequest**实例，为您的**Worker**实例提供完成工作所需的所有信息。

+   **回退标准**：您可以设置**WorkRequest**实例的回退标准，以控制工作失败时的重试时间。

+   **标签**：您还可以向您的**WorkRequest**实例添加标签，这将使跟踪、观察或取消特定工作组的任务变得更加容易。

+   **工作链式化**：**WorkManager**允许您创建依赖的工作链。这意味着您可以确保某些工作以特定的顺序执行。您可以创建复杂的链，以特定的顺序运行一系列**WorkRequest**对象。

`WorkManager`提供了几种类型的约束，您可以在`WorkRequest`对象上设置这些约束，以指定任务应在何时运行。这是通过使用`Constraints.Builder`类来完成的。以下是您可以设置的可用约束：

+   **网络类型**（**setRequiredNetworkType**）：此约束指定必须可用的网络类型，以便工作可以运行。选项包括**NetworkType.NOT_REQUIRED**、**NetworkType.CONNECTED**、**NetworkType.UNMETERED**、**NetworkType.NOT_ROAMING**和**NetworkType.METERED**。

+   **电池电量不高**（**setRequiresBatteryNotLow**）：如果此约束设置为**true**，则工作仅在电池电量不高时运行。

+   **设备空闲**（**setRequiresDeviceIdle**）：如果此约束设置为**true**，则工作仅在设备处于空闲模式时运行。这通常是在用户一段时间内未与设备交互时。

+   **存储空间不高**（**setRequiresStorageNotLow**）：如果设置为**true**，则工作仅在存储空间不高时运行。

+   **设备充电**（**setRequiresCharging**）：如果设置为**true**，则工作仅在设备充电时运行。

下面是一个配置`WorkRequest`实例的示例：

```kt
val constraints = Constraints.Builder()
    .setRequiresCharging(true)
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .setRequiresBatteryNotLow(true)
    .build()
val workRequest = OneTimeWorkRequestBuilder<MyWorker>()
    .setConstraints(constraints)
    .addTag("myWorkTag")
    .build()
```

在此示例中，`MyWorker`仅在设备充电、连接到网络且电池电量不高时运行。它还将有一个标签，这将使我们能够轻松识别它。

这是一个流程图，展示了执行`Worker`和`WorkRequest`实例时遵循的流程：

![图 3.3：执行 WorkRequest 实例的 WorkManager 流程图](img/B19443_03_003.jpg)

图 3.3：执行 WorkRequest 实例的 WorkManager 流程图

我们现在有了构建自己的`Worker`实例和配置`WorkRequest`实例以检索和上传备份的工具。那么，让我们实际创建它们。

## 创建我们的 Worker 实例

首先，为了支持`WorkManager` API，我们需要在我们的代码中包含相关的依赖项：

```kt
dependencies {
    implementation "androidx.work:work-runtime-ktx:$2.9.0"
    // Hilt AndroidX WorkManager integration
    implementation 'androidx.hilt:hilt-work:$2.44
    ...
}
```

正如我们之前看到的，`Worker`类将在应用未被使用时执行后台可运行的任务。换句话说，它是一个可以在特定条件下安排运行的工作单元。在我们的案例中，我们刚刚创建了相应的逻辑（在`UploadMessagesUseCase`中），因此我们的`Worker`类需要访问这个类。

这也是为什么我们将开始添加`HiltWorker`，这是由 Hilt 的`androidx.hilt`扩展库提供的注解。这个注解告诉 Hilt 它应该创建一个可注入的`Worker`实例（即，Hilt 应该管理这个`Worker`实例的依赖项）。

这里是我们`Worker`类的完整代码：

```kt
@HiltWorker
class UploadMessagesWorker @AssistedInject constructor(
    @Assisted appContext: Context,
    @Assisted workerParams: WorkerParameters,
    private val uploadMessagesUseCase:
        UploadMessagesUseCase
) : CoroutineWorker(appContext, workerParams) {
    override suspend fun doWork(): Result = coroutineScope
    {
        try {
            uploadMessagesUseCase.execute()
            Result.success()
        } catch (e: Exception) {
            if (runAttemptCount < MAX_RETRIES) {
                Result.retry()
            } else {
                Result.failure()
            }
        }
    }
    companion object {
        private const val MAX_RETRIES = 3
    }
}
```

我们还使用了一个新的注解：`AssistedInject`。现在，`AssistedInject`是 Dagger Hilt 的一个特性，它有助于处理需要注入一些依赖项但同时在运行时还需要提供一些参数的场景。在这里，构造函数的`appContext`和`workerParams`参数在运行时提供（当`Worker`实例由`WorkManager`创建时），而`uploadMessagesUseCase`是一个应该被注入的依赖项。

`doWork()`函数是定义这个`Worker`实例应该执行的工作的地方。这个函数是一个挂起函数，并在协程作用域内运行。这意味着它可以执行长时间运行的操作，如网络请求或数据库操作，而不会阻塞主线程。

在`doWork()`中，调用`uploadMessagesUseCase.execute()`来执行上传消息的实际工作。如果此操作成功，则返回`Result.success()`。如果抛出`Exception`错误，并且`runAttemptCount`小于`MAX_RETRIES`，则返回`Result.retry()`，这意味着应该重试工作。如果`runAttemptCount`等于或超过`MAX_RETRIES`，则返回`Result.failure()`，这意味着不应该重试工作。

由于我们希望它只重试三次，我们使用了`runAttemptCount`，这是由`ListenableWorker`（`CoroutineWorker`的父类）提供的属性，它跟踪工作尝试了多少次。

最后，`MAX_RETRIES`是一个定义最大重试次数的常量。在这个例子中，它被设置为`3`。

总结来说，这个`Worker`实例通过调用`uploadMessagesUseCase.execute()`来上传消息，并且在失败的情况下可以重试操作最多三次。这个`Worker`实例的实际依赖项（`UploadMessagesUseCase`）是通过`WorkRequest`类提供的。

## 设置`WorkRequest`类

在`WorkRequest`类的情况下，我们需要考虑我们希望消息备份的频率；例如，我们可以每周备份一次。此外，我们将配置`WorkRequest`类，使其仅在用户有 Wi-Fi 连接时被调用。以下是我们的做法：

```kt
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED)
    .build()
val uploadMessagesRequest =
PeriodicWorkRequestBuilder<UploadMessagesWorker>(7,
TimeUnit.DAYS)
    .setConstraints(constraints)
    .setBackoffCriteria(BackoffPolicy.LINEAR,
        PeriodicWorkRequest.MIN_PERIODIC_INTERVAL_MILLIS,
        TimeUnit.MILLISECONDS)
    .build()
WorkManager.getInstance(this).enqueue(
    uploadMessagesRequest)
```

我们使用`PeriodicWorkRequestBuilder`创建一个`WorkRequest`实例，该实例每周运行一次`UploadMessagesWorker`。`WorkRequest`实例有一个约束条件，要求有一个非计费的网络连接（Wi-Fi）。它还指定了一个线性退避策略用于重试——这意味着每次重试尝试都会延迟固定的时间，并且随着后续重试的进行而线性增加。

`enqueue()`方法安排`WorkRequest`实例运行。如果满足约束条件，并且队列中没有其他工作在其前面，它将立即开始运行。否则，它将等待直到满足约束条件，并且它是队列中的`WorkRequest`实例的轮次。

请注意，由于操作系统限制，`PeriodicWorkRequest`实例可能不会在周期结束时正好运行；它可能会有一些延迟，但至少会在该时间段内运行一次。

我们可以从应用中的任何地方调用此代码并将`WorkRequest`实例入队，但要确保它被安排，最方便的地方是在我们启动应用时，在`WhatsPacktApplication.onCreate`方法中：

```kt
@HiltAndroidApp
class WhatsPacktApp: Application() {
    override fun onCreate() {
        super.onCreate()
        //Include WorkRequest initialization here
}
}
```

在所有这些准备就绪之后，我们的应用可以定期备份消息，我们的工作可能就已经完成了。然而，为了探索不同的方法，让我们看看如果我们需要集成另一个存储提供商会发生什么——例如，Amazon S3。

# 使用 Amazon S3 进行存储

**Amazon S3**是一个可扩展、高速、基于网络的云存储服务，旨在为**Amazon Web Services**（**AWS**）上的数据和应用提供在线备份和存档。它是 Firebase Storage 的一个知名替代品。

下面是 Amazon S3 一些关键功能和能力的简要概述：

+   **存储**：Amazon S3 可以存储任何数量的数据，并可以从网络上的任何地方访问它。它提供了几乎无限的存储空间。

+   **持久性和可用性**：Amazon S3 被设计为 99.999999999%（11 个 9）的持久性，并在多个地理上分离的数据中心存储数据的冗余副本。它还提供在给定一年内的 99.99%对象可用性。

+   **安全性**：Amazon S3 提供了高级安全功能，如静态数据和传输中的加密，以及使用 AWS **身份和访问管理**（**IAM**）、**访问控制列表**（**ACLs**）和存储桶策略对资源的细粒度访问控制。

+   **可扩展性**：Amazon S3 被设计用于扩展存储、请求和用户，以支持无限数量的网络规模应用程序。

+   **性能**：AWS 存储确保在您添加或删除文件时，您可以立即读取文件的最新版本。如果您覆盖文件或删除它，这些更改可能需要一些时间才能在所有地方完全更新。

+   **集成**：Amazon S3 与其他 AWS 服务很好地集成，例如 AWS CloudTrail 用于日志记录和监控、Amazon CloudFront 用于内容分发、AWS Lambda 用于无服务器计算等。

+   **管理功能**：S3 提供了用于管理任务的功能，例如组织数据和配置精细的访问控制以满足特定的业务、组织和合规性要求。

+   **数据传输**：S3 传输加速允许在您的客户端和 Amazon S3 存储桶之间快速、轻松且安全地在长距离上传输文件。

+   **存储类别**：Amazon S3 为不同类型的存储需求提供了几种存储类别，例如 S3 Standard 用于频繁访问数据的通用存储，S3 Intelligent-Tiering 用于访问模式未知或变化的存储，S3 Standard-IA 用于长期但很少访问的数据，以及 S3 Glacier 用于长期归档和数字保存。

+   **本地查询功能**：S3 Select 允许应用程序通过使用简单的 SQL 表达式从对象中检索仅部分数据。

这些功能使 Amazon S3 成为各种用例的强大且灵活的选择，从 Web 应用程序到备份和恢复、归档、企业应用程序、物联网设备和大数据分析。

要实现基于 Amazon S3 的存储解决方案，我们首先需要将 AWS SDK 集成到我们的应用程序中。

## 集成 AWS S3 SDK

我们可以通过在`build.gradle`文件中添加以下依赖项将 AWS S3 SDK 集成到我们的 Android 项目中：

```kt
implementation 'com.amazonaws:aws-android-sdk-s3:
$latest_version'
implementation 'com.amazonaws:aws-android-sdk-
cognitoidentityprovider:$latest_version'
```

我们在这里添加了 AWS SDK 和用于使用 Amazon Cognito 的依赖项。

我们还需要向 SDK 提供我们的 AWS 凭证（访问密钥 ID 和秘密访问密钥）。对于移动应用程序，建议使用 Amazon Cognito 进行凭证管理。

## 设置 Amazon Cognito

**Amazon Cognito**是一项提供用户注册和登录服务以及移动和 Web 应用程序访问控制的服务。当您为用户池使用 Amazon Cognito 时，您可以选择在 AWS 服务（如用于文件存储的 Amazon S3）中安全地存储您的数据，而无需在应用程序代码中嵌入 AWS 密钥，这是一个重大的安全风险。

下面是在我们的 Android 应用程序中设置 Amazon Cognito 的说明：

1.  首先，转到 Amazon Cognito 控制台：[`console.aws.amazon.com/cognito/home`](https://console.aws.amazon.com/cognito/home)。

1.  从那里，点击**身份池**，然后点击**创建新的** **身份池**。

1.  在**认证**部分下检查**访客访问**，然后点击**下一步**。

1.  选择**创建新的 IAM 角色**，为其创建一个名称，然后点击**下一步**。

1.  然后为身份池创建一个新的名称并点击**下一步**。

1.  查看摘要（如*图 3.4*），然后点击**创建** **身份池**：

![图 3.4：新的身份池配置](img/B19443_03_004.jpg)

图 3.4：新的身份池配置

注意

在这里，我们正在启用对未认证身份的访问。您还可以选择仅对认证身份提供访问权限，但请注意，您将不得不在 Amazon Cognito 中创建每个用户。尽管如此，这种方法比使用 S3 SDK 在我们的应用代码中存储密钥更安全。

1.  接下来，在我们的应用中，我们需要获取 AWS 凭证提供者。为此，我们将使用我们的 **IdentityPoolId** 类初始化 **CognitoCachingCredentialsProvider**，在配置的区域：

    ```kt
    val credentialsProvider =
    CognitoCachingCredentialsProvider(
        applicationContext,
        "IdentityPoolId", // Identity Pool ID
        Regions.US_EAST_1 // Region
    )
    ```

1.  现在，我们可以在创建 AWS 服务客户端时使用凭证提供者实例。例如，要与其配合使用 Amazon S3，请使用以下代码：

    ```kt
    val s3 = AmazonS3Client(credentialsProvider)
    ```

现在，是时候创建一个新的存储提供者了。

## 创建 AWS S3 存储提供者并将其集成到我们的代码中

现在，我们需要做与 Firebase Storage 相同的事情，但使用 AWS SDK：创建一个提供者。这个提供者将是 `AWSS3Provider`，并用于处理文件上传到 AWS S3。它将接受一个 `Context` 对象和一个 `CognitoCachingCredentialsProvider` 对象作为构造函数参数。

这就是我们可以这样实现的方式：

```kt
class AWSS3Provider(
    private val context: Context,
    private val credentialsProvider:
        CognitoCachingCredentialsProvider
) {
    suspend fun uploadFile(bucketName: String, objectKey:
    String, filePath: String) {
        withContext(Dispatchers.IO) {
            val transferUtility = TransferUtility.builder()
                .context(context)
                .awsConfiguration(AWSMobileClient
                    .getInstance().configuration)
                .s3Client(AmazonS3Client(
                    credentialsProvider))
                .build()
            val uploadObserver = transferUtility.upload(
                bucketName,
                objectKey,
                File(filePath)
            )
            uploadObserver.setTransferListener(object :
            TransferListener {
                override fun onStateChanged(id: Int, state:
                TransferState) {
                    if (TransferState.COMPLETED == state) {
                        // The file has been uploaded
                           successfully
                    }
                }
                override fun onProgressChanged(id: Int,
                bytesCurrent: Long, bytesTotal: Long) {
                    val progress = (bytesCurrent.toDouble()
                        / bytesTotal.toDouble() * 100.0)
                    Log.d("Upload Progress", "$progress%")
                }
                override fun onError(id: Int, ex:
                Exception) {
                    throw ex
                }
            })
        }
    }
}
```

`uploadFile` 函数是一个挂起函数，这意味着它可以从任何协程作用域中调用。`withContext(Dispatchers.IO)` 函数用于将协程上下文切换到 I/O 分派器，该分派器针对 I/O 相关任务进行了优化，例如网络调用或磁盘操作。

让我们深入探讨 `uploadFile` 函数，这是这个类的核心。

`TransferUtility` 类简化了将文件上传到/从 Amazon S3 的过程。在这里，我们正在构建一个 `TransferUtility` 实例，向其提供 Android 上下文、AWS 配置以及使用提供的 `CognitoCachingCredentialsProvider` 类初始化的 `AmazonS3Client` 实例。

使用 `transferUtility.upload()` 方法将文件上传到 S3 中指定的存储桶。我们提供存储桶的名称（`bucketName`）、存储新对象的键（`objectKey`）以及我们想要上传的文件（`File(filePath)`）。此函数返回一个 `UploadObserver` 实例。

`UploadObserver` 用于监控上传进度。

我们将 `TransferListener`附加到观察者，以便在上传状态改变、上传进度或发生错误时获取回调。

当传输状态改变时，会调用 `onStateChanged()` 方法。如果状态是 `TransferState.COMPLETED`，这意味着文件已成功上传。

当传输的字节数增加时，会调用 `onProgressChanged()` 方法。在这里，我们计算进度作为百分比并记录它。

如果在传输过程中发生错误，将调用 `onError()` 方法。当发生这种情况时，我们将抛出一个错误，由消费者或此提供者处理。

`uploadFile` 函数是在协程内部调用的，由于实际的上传操作是一个网络 I/O 操作，因此它被包装在 `withContext(Dispatchers.IO)` 中。这确保了操作不会阻塞主线程，因为 I/O 分派器使用一个针对磁盘和网络 I/O 优化的单独线程池。

现在，我们需要创建一个数据源来将我们的 `BackupRepository` 实例连接到这个新的提供者。最佳做法是通过实现 `IStorageDataSource`，这是一个适用于两个数据源的通用接口。这样，你就可以在不改变其余代码的情况下交换底层实现（Firebase Storage、AWS S3 等）。（这是 **依赖倒置原则**（**DIP**）的应用，它是 **面向对象**（**OO**）设计的 SOLID 原则之一，有助于使你的代码更加灵活且易于维护。）

这就是我们实现 `S3StorageDataSource` 的方法：

```kt
class S3StorageDataSource @Inject constructor(
    private val awsS3Provider: AWSS3Provider
) : IStorageDataSource {
    override suspend fun uploadFile(remotePath: String,
    file: File) {
        awsS3Provider.uploadFile(BUCKET_NAME, remotePath,
        file.absolutePath)
    }
    companion object {
        private const val BUCKET_NAME = "our-bucket-name"
    }
}
```

在此代码中，我们正在实现 `uploadFile` 函数，调用 `awsProvider.uploadFile` 函数，该函数将文件上传到名为 `our-bucket-name` 的存储桶。

这个新的 `S3StorageDataSource` 类可以通过 Hilt 以类似的方式提供，就像之前的 `FirebaseStorageDataSource` 类一样：

```kt
@Module
@InstallIn(SingletonComponent::class)
object StorageModule {
    @Provides
    @Singleton
    fun provideStorageDataSource(awsS3Provider:
    AWSS3Provider): IStorageDataSource {
        return S3StorageDataSource(awsS3Provider)
    }
}
```

在这里，我们创建一个 `@Module` 注解，它包括一个 `@Provides` 或 `@Binds` 方法用于 `IStorageDataSource`，Hilt 将根据你的配置注入正确的实现。如果你想从 Firebase Storage 切换到 AWS S3，你需要修改这个模块以提供 `S3StorageDataSource` 而不是 `FirebaseStorageDataSource`。

最后，我们需要将其集成到我们的 `BackupRepository` 类中。这就像替换 `StorageDataSource` 依赖项为 `IStorageDataSource` 依赖项一样简单：

```kt
class BackupRepository @Inject constructor(
    private val messageDao: MessageDao,
    private val conversationDao: ConversationDao,
    private val storageDataSource: IStorageDataSource
) {
    // The rest of the class as it was before
...
}
```

就这样。根据我们在 Hilt 模块中提供的什么来满足 `IStorageDataSource` 依赖项，它将使用 Firebase Storage 或 AWS S3。

随着这个变化，我们完成了本章，也完成了在 WhatsPackt 应用中的工作！

![图 3.5：WhatsPackt 的最终外观](img/B19443_03_005.jpg)

图 3.5：WhatsPackt 的最终外观

# 摘要

在本章中，我们致力于为我们的用户创建良好的离线体验（使用 Room 在本地数据库中存储消息）并提供一种机制来存储消息备份，以防出现故障。我们还学习了如何使用不同的提供者将我们的文件存储在云端，使用 Firebase Firestore 和 AWS S3。

现在，我们已经完成了在 WhatsPackt 应用中的工作。在下一章中，我们将开始构建一个新的应用：Packtagram。它将是一个与朋友分享照片和视频的应用，在创建过程中将提供新的和不同的挑战，例如捕获视频。这些是我们将学会克服的挑战。
