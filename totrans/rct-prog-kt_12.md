# 反应式 Kotlin 和 Android

因此，我们关于 Kotlin 中反应式编程的学习几乎已经完成。我们已经到达了本书的最后一章，但可能是最重要的一章。Android 可能是 Kotlin 最大的平台。在最近的 Google IO—Google IO 17 上，Google 宣布了对 Kotlin 的官方支持，并将 Kotlin 添加为 Android 应用开发的第一个公民。现在，Kotlin 是除了 Java 之外唯一官方支持的 Android 应用开发语言。

反应式编程已经在 Android 中存在——Android 中大多数顶级库都支持反应性。因此，在名为 *Reactive Programming in Kotlin* 的书中，我们必须涵盖 Android。

从零开始教授 Android 开发超出了本书的范围，因为这是一个庞大的主题。如果您想从头学习 Android 开发，可以找到很多书籍。本书假设您对 Android 应用开发有一些基本知识，并且可以与 `RecyclerView`、`Adapter`、`Activity`、Fragment、CardView、AsyncTask 等一起工作。如果您不熟悉所提到的任何主题，可以阅读 *Prajyot Mainkar* 的 *Expert Android Programming*。

那么，您想知道本章为您准备了什么吗？请查看以下我们将涵盖的主题列表：

+   在 Android Studio 2.3.3 和 3.0 中设置 Kotlin

+   在 Android 和 Kotlin 中开始 `ToDoApp` 的开发

+   使用 Retrofit 2 进行 API 调用

+   设置 RxAndroid 和 RxKotlin

+   使用 RxKotlin 与 Retrofit 2

+   开发我们的应用

+   RxBinding 简介简要

那么，让我们开始设置 Android Studio 中的 Kotlin。

# 在 Android Studio 中设置 Kotlin

我们强烈建议您使用 Android Studio 3.0 进行 Android 开发，无论您是否使用 Kotlin。Android Studio 3.0 是 Android Studio 的最新版本，包含许多错误修复、新功能和改进的 Gradle 构建时间。

对于 Android Studio 3.0，您在创建新项目时只需选择“包含 Kotlin 支持”即可，无需进行任何设置。以下是供您参考的截图：

![](img/522fe92a-f72c-4e55-a816-7b254c3fed99.jpg)

我们已经突出了 Android Studio—创建 Android 项目对话框中的“包含 Kotlin 支持”部分。

然而，如果您正在使用 Android Studio 2.3.3，请按照以下步骤操作：

1.  前往 Android Studio | 设置 | 插件。

1.  搜索 `Kotlin`（查看以下截图）并按照以下步骤安装该插件：

![](img/deb76a4a-bd0c-427a-afd6-19311956e6b4.jpg)

1.  开始一个新的 Android 项目。

1.  要将 Kotlin 插件应用到项目中，打开项目级别的 `build.gradle` 并修改内容，如下所示：

![](img/68e6de34-8c0a-43a0-8f44-b674e9e0a53d.jpg)

1.  打开您模块中的 `build.gradle` 文件（或者我们也可以说是应用级别的 `build.gradle` 文件）并添加以下 `dependencies`：

```kt
      compile "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version" 
```

您现在已准备好开始在 Android Studio 中编写 Kotlin 代码。

然而，在开始编写 Kotlin 代码之前，让我们首先回顾一下我们的`build.gradle`。我之前为 Android Studio 2.3.3 展示的代码对 Android Studio 3.0 同样有效，你只需要不需要手动添加，因为 Android Studio 3.0 会自动为你添加。然而，这些行的目的是什么？让我们检查一下。

在项目级别的`build.gradle`文件中，`ext.kotlin_version = "1.1.51"`这一行在 Gradle 中创建了一个名为`kotlin_version`的变量；这个变量将持有`String`类型的值，`1.1.51`（这是撰写本书时的最新版本）。我们在变量中写入这个版本，因为这个版本在项目级别和应用程序级别的`build.gradle`文件中的多个地方都需要。如果我们只声明一次并在多个地方使用它，那么将会有一致性，并且不会有人为错误的机会。

然后，在同一个文件（项目级别的`build.gradle`文件）中，我们将添加`classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"`。这将定义 Gradle 用于在添加它们作为依赖项时搜索`kotlin-jre`的类路径。

在应用级别的`build.gradle`文件中，我们将编写`implementation "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"`。

那么，让我们开始编写 Kotlin 代码。正如我们在上一章中提到的，我们将创建一个`ToDoApp`。将会有三个屏幕，一个用于`ToDo List`，一个用于创建`ToDo`，另一个用于编辑/删除`ToDo`。

# 在 Android 上开始使用 ToDoApp

如前所述，我们在这个项目中使用的是 Android Studio 3.0（稳定版）。下面的截图展示了我们使用的项目结构：

![图片](img/71882bb0-c011-4d63-b1db-f39e50862a2e.jpg)

在这个项目中，我们使用按功能划分的包，我确实更喜欢在 Android 开发中使用按功能划分的包，主要是因为其可扩展性和可维护性。此外，请注意，在 Android 中使用按功能划分的包是一种最佳实践；尽管如此，你显然可以使用你喜欢的模型。你可以在[`hackernoon.com/package-by-features-not-layers-2d076df1964d`](https://hackernoon.com/package-by-features-not-layers-2d076df1964d)了解更多关于按功能划分的包的信息。

现在，让我们了解在这个应用程序中使用的包结构。这里的根包是`com.rivuchk.todoapplication`，它是应用程序的包，与`applicationId`相同。根包包含两个类——`ToDoApp`和`BaseActivity`。`ToDoApp`类扩展了`android.app.Application`，这样我们就可以有自己的`Application`类实现。现在，什么是`BaseActivity`？`BaseActivity`是在这个项目中创建的一个抽象类，这个项目中的所有活动都应该扩展`BaseActivity`；因此，如果我们想在项目中的所有活动中实现某些功能，我们可以在`BaseActivity`中编写代码，并且可以放心，所有活动现在都会实现相同的代码。

接下来，我们有一个`apis`包，用于与 API 调用相关的类和文件（我们将使用 Retrofit），以及`datamodels`用于模型（POJO）类。

我们有`Utils`包用于`CommonFunctions`和`Constants`（一个单例`Object`，用于存储如`BASE_URL`等常量变量）。

`addtodo`、`tododetails`和`todolist`是三个基于功能的包。`todolist`包包含用于显示待办事项列表的`Activity`和`Adapter`。`tododetails`包包含负责显示待办事项详细信息的`Activity`。我们也将使用相同的`Activity`来编辑。`addtodo`包包含用于实现添加待办事项功能的`Activity`。

在开始活动布局之前，我希望你先看看`BaseActivity`和`ToDoApp`的内部结构，所以这里是`ToDoApp.kt`文件中的代码：

```kt
    class ToDoApp:Application() { 
      override fun onCreate() { 
        super.onCreate() 
        instance = this 
      } 

      companion object { 
        var instance:ToDoApp? = null 
      } 
    } 
```

确实是一个小类；它只包含一个`companion object`来为我们提供实例。随着我们继续本章的学习，这个类将会逐渐增长。我们在清单文件中声明了`ToDoApp`为这个项目的`application`类，如下所示：

```kt
    <application 
      android:allowBackup="true" 
      android:icon="@mipmap/ic_launcher" 
      android:label="@string/app_name" 
      android:roundIcon="@mipmap/ic_launcher_round" 
      android:supportsRtl="true" 
      android:theme="@style/AppTheme" 
 android:name=".ToDoApp"> 
      .... 
    </application> 
```

`BaseActivity`现在也很小。与`ToDoApp`一样，它也会在本章的进程中逐渐增长：

```kt
    abstract class BaseActivity : AppCompatActivity() { 
       final override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 
        onCreateBaseActivity(savedInstanceState) 
       } 
 abstract fun onCreateBaseActivity(savedInstanceState: Bundle?) 
    } 
```

目前，`BaseActivity`只隐藏了`Activity`类的`onCreate`方法，并提供了一个新的抽象方法——`onCreateBaseActivity`。这个类还强制要求我们在子类中重写`onCreateBaseActivity`，这样我们就可以在所有活动的`onCreate`方法中实现任何需要的功能，而无需在其他地方实现。

那么，让我们开始处理`todolist`。这个包包含了显示待办事项列表所需的所有源代码。如果你仔细查看前面的截图，你应该会注意到这个包包含两个类——`TodoListActivity`和`ToDoAdapter`。

那么，让我们从`TodoListActivity`的设计开始；当完成时，这个`Activity`应该看起来像下面的截图：

![图片](img/ae96f7fb-8770-4d9a-b0c0-0af6aac2131e.jpg)

如截图所示，我们需要一个`FloatingActionButton`和一个`RecyclerView`来构建这个`Activity`，所以这里是这个示例的 XML 布局文件——`activity_todo_list.xml`：

```kt
    <?xml version="1.0" encoding="utf-8"?>
    <android.support.design.widget.CoordinatorLayout 

     android:layout_width="match_parent" 
     android:layout_height="match_parent" 
     tools:context="com.rivuchk.todoapplication.
     todolist.TodoListActivity"> 

     <android.support.design.widget.AppBarLayout 
       android:layout_width="match_parent" 
       android:layout_height="wrap_content" 
       android:theme="@style/AppTheme.AppBarOverlay"> 

     <android.support.v7.widget.Toolbar 
       android:id="@+id/toolbar" 
       android:layout_width="match_parent" 
       android:layout_height="?attr/actionBarSize" 
       android:background="?attr/colorPrimary" 
       app:popupTheme="@style/AppTheme.PopupOverlay" /> 

     </android.support.design.widget.AppBarLayout> 

 <android.support.v7.widget.RecyclerView android:id="@+id/rvToDoList" 
       android:layout_width="match_parent" 
       android:layout_height="match_parent" 
       app:layoutManager="LinearLayoutManager" 
       android:orientation="vertical" 
       app:layout_behavior="@string/appbar_scrolling_view_behavior"/> 

 <android.support.design.widget.FloatingActionButton android:id="@+id/fabAddTodo" 
       android:layout_width="wrap_content" 
       android:layout_height="wrap_content" 
       android:layout_gravity="bottom|end" 
       android:layout_margin="@dimen/fab_margin" 
       app:srcCompat="@drawable/ic_add" /> 

    </android.support.design.widget.CoordinatorLayout> 
```

看一下前面的布局。在`RecyclerView`的声明中，我们将其`layoutManager`设置为`LinearLayoutManager`，并将方向设置为从布局本身开始的垂直，所以我们就不需要在代码中设置它了。

我们使用了一个`FloatingActionButton`来添加新的待办事项。我们还使用了`AppBarLayout`作为操作栏。

是时候继续前进，看看`TodoListActivity`的`onCreateBaseActivity`方法了，如下所示：

```kt
    lateinit var adapter: ToDoAdapter 

    private val INTENT_EDIT_TODO: Int = 100 

    private val INTENT_ADD_TODO: Int = 101 

    override fun onCreateBaseActivity(savedInstanceState: Bundle?) { 
      setContentView(R.layout.activity_todo_list) 
      setSupportActionBar(toolbar) 

      fabAddTodo.setOnClickListener { _ -> 
         startActivityForResult(intentFor<AddTodoActivity>
         (),INTENT_ADD_TODO) 
      } 

      adapter = ToDoAdapter(this,{ 
       todoItem-> 
       startActivityForResult(intentFor<TodoDetailsActivity>
       (Pair(Constants.INTENT_TODOITEM,todoItem)),INTENT_EDIT_TODO) 
      }) 
      rvToDoList.adapter = adapter 

      fetchTodoList() 
    } 
```

在前面的程序中，我们创建了一个 `ToDoAdapter` 实例，将其设置为 `rvToDoList` 的适配器，`rvToDoList` 是我们将显示待办事项列表的 `RecyclerView`。在创建 `ToDoAdapter` 实例时，我们传递了一个 lambda；当点击 `rvToDoList` 中的项目时，应该调用这个 lambda。

我们还在 `onCreateBaseActivity` 方法的末尾调用了一个 `fetchTodoList()` 函数。正如其名称所示，它负责从 REST API 获取待办事项列表。我们将在稍后查看该方法的定义和细节，但现在，让我们看看 `Adapter`：

```kt
    class ToDoAdapter( 
 private val context:Context, //(1) val onItemClick:(ToDoModel?)->Unit = {}//(2) 
      ):RecyclerView.Adapter<ToDoAdapter.ToDoViewHolder>() { 
 private val inflater:LayoutInflater =  
      LayoutInflater.from(context)//(3) private val   
      todoList:ArrayList<ToDoModel> = arrayListOf()//(4) fun
      setDataset(list:List<ToDoModel>) {//(5) 
        todoList.clear() 
        todoList.addAll(list) 
        notifyDataSetChanged() 
      } 

      override fun getItemCount(): Int = todoList.size 

      override fun onBindViewHolder(holder: ToDoViewHolder?,
      position: Int) { 
        holder?.bindView(todoList[position]) 
      } 

      override fun onCreateViewHolder
      (parent: ViewGroup?, viewType: Int): ToDoViewHolder { 
        return ToDoViewHolder
        (inflater.inflate(R.layout.item_todo,parent,false)) 
      } 

      inner class ToDoViewHolder(itemView:View):
      RecyclerView.ViewHolder(itemView) { 
        fun bindView(todoItem:ToDoModel?) { 
 with(itemView) {//(6) 
             txtID.text = todoItem?.id?.toString() 
             txtDesc.text = todoItem?.todoDescription 
             txtStatus.text = todoItem?.status 
             txtDate.text = todoItem?.todoTargetDate 

             onClick { 
 this@ToDoAdapter.onItemClick(todoItem)//(7) 
             } 
          } 
        } 
      } 
    } 
```

仔细研究前面的代码。这是完整的 `ToDoAdapter` 类。我们取了一个 `context` 实例作为注释 `(1)` 构造函数参数。我们使用那个 `context` 来获取一个 `Inflater` 实例，然后在该实例中用于 `onCreateViewHolder` 方法中的布局填充。我们创建了一个空的 `ToDoModel` `ArrayList`。我们使用那个列表来获取适配器的 `getItemCount()` 函数的项目数，并在 `onBindViewHolder` 函数中将其传递给 `ViewHolder` 实例。

我们还在 `ToDoAdapter` 构造函数内部将 lambda 作为 `val` 参数——`onItemClick`（注释 `(2)`）。这个 lambda 应该接收一个 `ToDoModel` 实例作为参数，并返回 unit。

我们在 `ToDoViewHolder` 的 `bindView` 中使用了那个 lambda，在 `itemView` 的 `onClick`（注释 `(7)`）中。所以，每次我们点击一个项目时，都会调用 `onItemClick` lambda，它是由 `TodoListActivity` 传递的。

现在，关注注释 `(5)` 中的 `setDataset()` 方法。此方法用于将一个新的列表分配给适配器。它将清除 `ArrayList`—`TodoList` 并将传递的列表中的所有项目添加到其中。这个 `setDataset` 方法应该在 `TodoListActivity` 中的 `fetchTodoList()` 方法中调用。那个 `fetchTodoList()` 方法负责从 REST API 获取列表，并将该列表传递给适配器。

我们将在稍后查看 `fetchTodoList()` 方法，但让我们集中关注 REST API 和 Retrofit 2 用于 API 调用。

# Retrofit 2 用于 API 调用

Retrofit by Square 是 Android 中最著名和最广泛使用的 REST 客户端之一。它内部使用 OkHTTP 进行 HTTP 和网络调用。REST 客户端这个词使其与其他 Android 网络库不同。虽然大多数网络库（Volley、OkHTTP 等）专注于同步/异步请求、优先级、有序请求、并发/并行请求、缓存等，但 Retrofit 更注重使网络调用和解析数据更像方法调用。它简单地将你的 HTTP API 转换为 Java 接口。而且它甚至不尝试自己解决网络问题，而是将此委托给内部的 OkHTTP。

那么，它是如何将 HTTP API 转换为 Java 接口的呢？Retrofit 简单地使用一个转换器将 POJO（**普通的 Java 对象**）类序列化和反序列化为 JSON 或 XML。现在，什么是转换器？转换器是那些为你解析 JSON/XML 的辅助类。转换器通常使用`Serializable`接口内部进行 JSON/XML 和 POJO 类（在 Kotlin 中的数据类）之间的转换。它具有可插拔性，为你提供了许多转换器的选择，如下所示：

+   Gson

+   Jackson

+   Guava

+   Moshi

+   Java 8 转换器

+   Wire

+   Protobuf

+   SimpleXML

我们将使用 Gson 来处理我们的书籍。要使用 Retrofit，你需要以下三个类：

+   一个`Model`类（POJO 或数据类）

+   一个类，通过`Retrofit.Builder()`提供 Retrofit 客户端实例

+   一个`Interface`，定义可能的 HTTP 操作，包括请求类型（GET 或 POST）、参数/请求体/查询字符串，以及最终的响应类型

那么，让我们从`Model`类开始。

在创建类之前，我们首先需要了解 JSON 响应的结构。我们在上一章中看到了 JSON 响应，但为了快速回顾，以下是`GET_TODO_LIST` API 的 JSON 响应：

```kt
    { 
      "error_code": 0, 
      "error_message": "", 
      "data": [ 
       { 
         "id": 1, 
         "todoDescription": "Lorem ipsum dolor sit amet, consectetur 
          adipiscing elit. Integer tincidunt quis lorem id rhoncus. Sed 
          tristique arcu non sapien consequat commodo. Nulla dolor 
          tellus, molestie nec ipsum at, eleifend bibendum quam.", 
          "todoTargetDate": "2017/11/18", 
          "status": "complete" 
        } 
       ] 
    } 
```

`error_code`表示是否存在错误。如果`error_code`是非零值，则必须存在错误。如果是零，则没有错误，你可以继续解析数据。

如果有错误，`error_message`将包含信息。如果`error_code`为零，则`error_message`将为空。

`data`键将包含待办事项列表的 JSON 数组。

这里需要注意的一点是，`error_code`和`error_message`将与我们项目中的所有 API 保持一致，因此如果我们为所有 API 创建一个基类，然后在需要时扩展该类会更好。

这是我们`BaseAPIResponse`类：

```kt
    open class BaseAPIResponse ( 
      @SerializedName("error_code") 
      val errorCode:Int, 
      @SerializedName("error_message") 
      val errorMessage:String): Serializable 
```

在这个类中，我们有两个`val`属性——`errorCode`和`errorMessage`；注意`@SerializedName`注解。这个注解由 Gson 用来声明属性的序列化名称；序列化名称应该与 JSON 响应相同。如果你有与 JSON 响应相同的变量名，你可以轻松地避免这个注解。如果变量名不同，序列化名称用于匹配 JSON 响应。

现在，让我们继续进行`GetToDoListAPIResponse`；以下是这个类的定义：

```kt
    open class GetToDoListAPIResponse( 
      errorCode:Int, 
      errorMessage:String, 
      val data:ArrayList<ToDoModel> 
    ):BaseAPIResponse(errorCode,errorMessage) 
```

这里，我们跳过了`@SerializedName`注解的`data`，因为我们使用的是与 JSON 响应相同的名称。剩余的两个属性是由`BaseAPIResponse`类声明的。

对于数据，我们使用`ToDoModel`的`ArrayList`；Gson 将负责将 JSON 数组转换为`ArrayList`。

现在，让我们看一下`ToDoModel`类：

```kt
    data class ToDoModel ( 
      val id:Int, 
      var todoDescription:String, 
      var todoTargetDate:String, 
      var status:String 
    ):Serializable 
```

Retrofit 的`builder`类很简单，如下所示：

```kt
    class APIClient { 
      private var retrofit: Retrofit? = null 
      fun getClient(): Retrofit { 

        if(null == retrofit) { 

          val client = OkHttpClient.Builder().connectTimeout(3,
          TimeUnit.MINUTES) 
          .writeTimeout(3, TimeUnit.MINUTES) 
          .readTimeout(3,
          TimeUnit.MINUTES).addInterceptor(interceptor).build() 

           retrofit = Retrofit.Builder() 
             .baseUrl(Constants.BASE_URL) 
             .addConverterFactory(GsonConverterFactory.create()) 
             .client(client) 
             .build() 
          } 

          return retrofit!! 
         } 

         fun getAPIService() =
         getClient().create(APIService::class.java) 
    } 
```

`getClient()`函数负责创建并提供 Retrofit 客户端。`getAPIService()`函数帮助你将 Retrofit 客户端与定义的 HTTP 操作配对，并创建接口的实例。

我们使用`OkHttpClient`和`Retrofit.Builder()`来创建`Retrofit`实例。如果你不熟悉它们，你可以访问[`www.vogella.com/tutorials/Retrofit/article.html`](http://www.vogella.com/tutorials/Retrofit/article.html)。

现在，让我们创建 HTTP 操作的接口——`APIService`——如下所示：

```kt
    interface APIService { 
      @POST(Constants.GET_TODO_LIST) 
      fun getToDoList(): Call<GetToDoListAPIResponse> 

      @FormUrlEncoded 
      @POST(Constants.EDIT_TODO) 
      fun editTodo( 
            @Field("todo_id") todoID:String, 
            @Field("todo") todo:String 
      ): Call<BaseAPIResponse> 

      @FormUrlEncoded 
      @POST(Constants.ADD_TODO) 
      fun addTodo(@Field("newtodo") todo:String): Call<BaseAPIResponse> 
     } 
```

我们为所有 API 创建了 API 接口。注意函数的返回类型。它们返回一个封装实际预期响应的`Call`实例。

那么，`Call`实例是什么？使用它的目的是什么？

`Call`实例是对 Retrofit 方法的一个调用，它向 web 服务器发送请求并返回响应。每个调用都产生它自己的 HTTP 请求和响应对。我们该如何处理`Call<T>`实例？我们必须用`Callback<T>`实例`enqueue`它。

因此，同样的拉取机制，同样的回调地狱。然而，我们应该是响应式的，不是吗？让我们这么做。

# RxKotlin 与 Retrofit

在 Android 中，我们可以使用 RxAndroid 以及 RxKotlin 来增加 Android 风味和好处，并且 Retrofit 也支持它们。

那么，让我们首先修改我们的`build.gradle`以支持 ReactiveX。将以下依赖项添加到应用的`build.gradle`级别：

```kt
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.3.0 ' 
    implementation 'io.reactivex.rxjava2:rxandroid:2.0.1' 
    implementation 'io.reactivex.rxjava2:rxkotlin:2.1.0' 
```

第一个提供了 Retrofit 2 适配器用于 RxJava 2，而接下来的两个添加了 RxAndroid 和 RxKotlin 到项目中。

注意，RxKotlin 是 RxJava 的包装器，所以 RxJava 2 的适配器将与 RxKotlin 2 完美兼容。

现在我们已经添加了依赖项，让我们继续修改我们的代码，使其与`Observable`/`Flowable`而不是`Call`一起工作。

这是修改后的`APIClient.kt`文件：

```kt
    class APIClient { 
      private var retrofit: Retrofit? = null 
      enum class LogLevel { 
        LOG_NOT_NEEDED, 
        LOG_REQ_RES, 
        LOG_REQ_RES_BODY_HEADERS, 
        LOG_REQ_RES_HEADERS_ONLY 
      }  
      /** 
       * Returns Retrofit builder to create 
       * @param logLevel - to print the log of Request-Response 
       * @return retrofit 
       */ 
      fun getClient(logLevel: Int): Retrofit { 

        val interceptor = HttpLoggingInterceptor() 
        when(logLevel) { 
            LogLevel.LOG_NOT_NEEDED -> 
                interceptor.level = HttpLoggingInterceptor.Level.NONE 
            LogLevel.LOG_REQ_RES -> 
                interceptor.level = HttpLoggingInterceptor.Level.BASIC 
            LogLevel.LOG_REQ_RES_BODY_HEADERS -> 
                interceptor.level = HttpLoggingInterceptor.Level.BODY 
            LogLevel.LOG_REQ_RES_HEADERS_ONLY -> 
                interceptor.level =
            HttpLoggingInterceptor.Level.HEADERS 
         }  

        val client = OkHttpClient.Builder().connectTimeout(3, 
        TimeUnit.MINUTES) 
          .writeTimeout(3, TimeUnit.MINUTES) 
          .readTimeout(3,
          TimeUnit.MINUTES).addInterceptor(interceptor).build() 

          if(null == retrofit) { 
            retrofit = Retrofit.Builder() 
              .baseUrl(Constants.BASE_URL) 
              .addConverterFactory(GsonConverterFactory.create()) 
 .addCallAdapterFactory
              (RxJava2CallAdapterFactory.create()) 
              .client(client) 
              .build() 
           } 

           return retrofit!! 
         } 

         fun getAPIService(logLevel: LogLevel =   
         LogLevel.LOG_REQ_RES_BODY_HEADERS) = 
         getClient(logLevel).create(APIService::class.java)  
    } 
```

这次，我们添加了一个 OkHttp Logging 拦截器（`HttpLoggingInterceptor`）以及一个 RxJava 适配器。这个 OkHttp Logging 拦截器将帮助我们记录请求和响应。回到 RxJava 适配器，看看高亮代码——我们添加了`RxJava2CallAdapterFactory`作为 Retrofit 客户端的`CallAdapterFactory`。

我们还需要修改`APIService.kt`文件，以便让函数返回`Observable`而不是`Call`，如下所示：

```kt
    interface APIService { 
      @POST(Constants.GET_TODO_LIST) 
      fun getToDoList(): Observable<GetToDoListAPIResponse> 

      @POST(Constants.EDIT_TODO) 
      fun editTodo( 
            @Body todo:String 
      ): Observable<BaseAPIResponse> 

      @POST(Constants.ADD_TODO) 
      fun addTodo(@Body todo:String): Observable<BaseAPIResponse> 
    }  
```

现在所有的 API 都返回`Observable`而不是`Call`。最后，我们一切都准备好了，可以查看`TodoListActivity`中的`fetchTodoList()`函数。

```kt
    private fun fetchTodoList() { 
      APIClient() 
      .getAPIService() 
      .getToDoList() 
 .subscribeOn(Schedulers.computation()) .observeOn(AndroidSchedulers.mainThread()) 
      .subscribeBy( 
         onNext = { response -> 
                    adapter.setDataset(response.data) 
                  }, 
         onError = { 
                     e-> e.printStackTrace() 
                   } 
       ) 
    } 
```

这个函数执行一个简单的任务；它订阅 API（来自 API 的`Observable`），并在数据到达时将其设置到适配器中。你应该考虑在这里添加逻辑来检查错误代码，但到目前为止它工作得相当好。这个活动的截图已经在本章的开头显示过了，所以我们在这里省略它。

# 使 Android 事件响应式

我们已经使我们的 API 调用响应式了，但我们的事件呢？记住 `ToDoAdapter`；我们在 `TodoListActivity` 中创建并传递了一个 lambda，并在 `ToDoViewHolder` 内部使用了它。这相当混乱。这些事件也应该响应式，不是吗？所以，让我们使事件也响应式。

`Subject` 在使事件响应式方面发挥着极好的作用。由于 `Subject` 是 `Observable` 和 `Observer` 的绝佳组合，我们可以在 `Adapter` 内部将其用作 `Observer`，并在 `Activity` 内部将其用作 `Observable`，从而使得传递事件变得容易。

因此，让我们按照以下方式修改 `ToDoAdapter`：

```kt
    class ToDoAdapter( 
      private val context:Context, //(1) 
      val onClickTodoSubject:Subject<Pair<View,ToDoModel?>>//(2) 
      ):RecyclerView.Adapter<ToDoAdapter.ToDoViewHolder>() { 
      private val inflater:LayoutInflater = 
      LayoutInflater.from(context)//(3) 
      private val todoList:ArrayList<ToDoModel> = arrayListOf()//(4) 

      fun setDataset(list:List<ToDoModel>) {//(5) 
        todoList.clear() 
        todoList.addAll(list) 
        notifyDataSetChanged() 
      } 

      override fun getItemCount(): Int = todoList.size 

      override fun onBindViewHolder(holder: ToDoViewHolder?,
      position: Int) { 
        holder?.bindView(todoList[position]) 
      } 

      override fun onCreateViewHolder
      (parent: ViewGroup?, viewType: Int): ToDoViewHolder { 
        return ToDoViewHolder(inflater.inflate
        (R.layout.item_todo,parent,false)) 
      } 

     inner class ToDoViewHolder(itemView:View):
     RecyclerView.ViewHolder(itemView) { 
       fun bindView(todoItem:ToDoModel?) { 
         with(itemView) {//(6) 
           txtID.text = todoItem?.id?.toString() 
           txtDesc.text = todoItem?.todoDescription 
           txtStatus.text = todoItem?.status 
           txtDate.text = todoItem?.todoTargetDate 

           onClick { 
             onClickTodoSubject.onNext(Pair
            (itemView,todoItem))//(7) 
          } 
         } 
       } 
     } 
   } 
```

现在适配器看起来更简洁了。我们在构造函数中有一个 `Subject` 实例，当 `itemView` 被点击时，我们调用 `Subject` 的 `onNext` 事件，并通过 `Pair` 将 `itemView` 和 `ToDoModel` 实例传递。

然而，它仍然看起来像缺少了什么。`onClick` 方法仍然是一个回调；我们能不能也使其响应式呢？让我们这么做。

# 在 Android 中引入 RxBinding

为了帮助 Android 开发者，Jake Wharton 创建了 RxBinding 库，它可以帮助您以响应式的方式获取 Android 事件。您可以在 [`github.com/JakeWharton/RxBinding`](https://github.com/JakeWharton/RxBinding) 找到它们。让我们通过将其添加到项目中开始吧。

将以下依赖项添加到应用级别的 `build.gradle` 文件中：

```kt
    implementation 'com.jakewharton.rxbinding2:rxbinding-kotlin:2.0.0' 
```

然后，我们可以用以下代码行替换 `ToDoViewHolder` 内部的 `onClick`：

```kt
    itemView.clicks() 
    .subscribeBy { 
       onClickTodoSubject.onNext(Pair(itemView,todoItem)) 
    }
```

这很简单。然而，你可能正在想，使它们响应式有什么好处呢？这里的实现足够简单，但想想你有很多逻辑的情况。你可以轻松地将逻辑划分为操作符，特别是 `map` 和 `filter` 可以为你提供极大的帮助。不仅如此，RxBindings 还为您提供了一致性。例如，当我们需要观察 `EditText` 上的文本变化时，我们通常会在 `TextWatcher` 实例中编写大量的代码，但如果你使用 RxBindings，它将让你这样做：

```kt
    textview.textChanges().subscribeBy {  
      changedText->Log.d("Text Changed",changedText) 
    } 
```

是的，这真的很简单，也很容易。RxBinding 还为您提供了更多的好处。您可以查看 [`speakerdeck.com/lmller/kotlin-plus-rxbinding-equals`](https://speakerdeck.com/lmller/kotlin-plus-rxbinding-equals) 和 [`adavis.info/2017/07/using-rxbinding-with-kotlin-and-rxjava2.html`](http://adavis.info/2017/07/using-rxbinding-with-kotlin-and-rxjava2.html)。

因此，现在，多亏了 Jake Wharton，我们也可以使我们的视图和事件响应式。

# Kotlin 扩展

在本章结束之际，我想向您介绍 Kotlin 扩展。不，确切地说，不是 Kotlin 扩展函数，尽管它们与 Kotlin 扩展函数非常相关。Kotlin 扩展是一份精心挑选的、在 Android 中最常用的扩展函数列表。

例如，如果您想要一个扩展函数，可以从`View`/`ViewGroup`实例创建位图（在添加 MapFragment 中的标记时特别有用），您可以从那里复制并粘贴以下扩展函数：

```kt
    fun View.getBitmap(): Bitmap { 
      val bmp = Bitmap.createBitmap(width, height,
      Bitmap.Config.ARGB_8888) 
      val canvas = Canvas(bmp) 
      draw(canvas) 
      canvas.save() 
      return bmp 
    } 
```

或者，更常见的情况，当您需要隐藏键盘时，以下扩展函数将帮助您：

```kt
    fun Activity.hideSoftKeyboard() { 
      if (currentFocus != null) { 
        val inputMethodManager = getSystemService(Context 
          .INPUT_METHOD_SERVICE) as InputMethodManager 
        inputMethodManager.hideSoftInputFromWindow
        (currentFocus!!.windowToken, 0) 
      } 
    } 
```

这个在线列表为您提供了更多扩展函数，由 Ravindra Kumar 维护（Twitter，GitHub—`@ravidsrk`）。所以，下次您需要扩展函数时，在编写自己的函数之前，先看看 [`kotlinextensions.com/`](http://kotlinextensions.com/)。

# 摘要

我们已经完成了这本书的最后一章。在这一章中，我们学习了如何为 RxKotlin 和 RxAndroid 配置 Retrofit，以及如何使我们的 Android 视图和事件以及自定义视图变得响应式。

我们学习了如何使用 RxJava2Adapter 和 Retrofit，以及如何使用`Subject`进行事件传递。我们还学习了如何使用 RxBindings。

在整本书中，我们试图深入探讨响应式编程，涵盖所有可能的概念，并尝试使所有代码都变得响应式。

如果您对此书有任何疑问，或者对此书有任何顾虑，请随时发送电子邮件至 rivu.chakraborty6174@gmail.com，并在邮件主题行中提及 `Book Query - Reactive Programming in Kotlin`。您也可以查看 Rivu Chakraborty 的网站 ([`www.rivuchk.com`](http://www.rivuchk.com))，他在那里定期发布关于 Kotlin、Google 开发者小组加尔各答和 Kotlin 加尔各答用户组聚会的文章。他还在那里撰写教程和博客，以及介绍他自己开发的 Android 插件的简介。此外，当他撰写其他地方的博客和文章时，他会在自己的网站上发布它们的 URL。

感谢您阅读这本书。祝您在 Kotlin 中进行愉快的响应式编程。
