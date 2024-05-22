# 第十章：Android 服务

在上一章中，我们开始使用 Android 中的并发机制。我们取得了很大的进展。然而，我们对 Android 并发机制的旅程还没有结束。我们必须介绍 Android 框架中可能是最重要的部分--Android 服务。在本章中，我们将解释服务是什么，何时以及如何使用它们。

在本章中，我们将涵盖以下主题：

+   服务分类

+   Android 服务的基础知识

+   定义主要应用程序服务

+   定义意图服务

# 服务分类

在我们定义 Android 服务分类并深入研究每种类型之前，我们必须回答 Android 服务到底是什么。嗯，**Android 服务**是 Android 框架提供的一种机制，通过它我们可以将长时间运行的任务移至后台。Android 服务提供了一些很好的附加功能，可以使开发人员的工作更加灵活和简单。为了解释它如何使我们的开发更容易，我们将通过扩展我们的 Journaler 应用程序来创建一个服务。

Android 服务是一个没有任何 UI 的应用程序组件。它可以被任何 Android 应用程序组件启动，并在需要时继续运行，即使我们离开我们的应用程序或杀死它。

Android 服务有三种主要类型：

+   前台

+   背景

+   绑定

# 前台 Android 服务

前台服务执行的任务对最终用户是可见的。这些服务必须显示状态栏图标。即使没有与应用程序的交互，它们也会继续运行。

# 后台 Android 服务

与前台服务不同，后台服务执行的任务不会被最终用户注意到。例如，我们将与后端实例进行同步。用户不需要知道我们的进度。我们决定不去打扰用户。一切都将在我们应用程序的后台默默执行。

# 绑定 Android 服务

我们的应用程序组件可以绑定到一个服务并触发不同的任务来执行。在 Android 中与服务交互非常简单。只要有至少一个这样的组件，服务就会继续运行。当没有组件绑定到服务时，服务就会被销毁。

可以创建一个在后台运行并具有绑定能力的后台服务。

# Android 服务基础知识

要定义 Android 服务，您必须扩展`Service`类。我们必须重写以下一些方法，以便服务能够正常运行：

+   `onStartCommand()`: 当`startService()`方法被某个 Android 组件触发时执行此方法。方法执行后，Android 服务就会启动并可以在后台无限期运行。要停止这个服务，必须执行`stopService()`方法，它与`startService()`方法的功能相反。

+   `onBind()`: 要从另一个 Android 组件绑定到服务，请使用`bindService()`方法。绑定后，将执行`onBind()`方法。在此方法的服务实现中，您必须提供一个接口，客户端通过返回一个`Ibinder`类实例与服务通信。实现此方法是不可选的，但如果您不打算绑定到服务，只需返回`null`即可。

+   `onCreate()`: 当服务被创建时执行此方法。如果服务已经在运行，则不会执行此方法。

+   `onDestroy()`: 当服务被销毁时执行此方法。重写此方法并在此处执行所有清理任务。

+   `onUnbind()`: 当我们从服务解绑时执行此方法。

# 声明您的服务

要声明您的服务，您需要将其类添加到 Android 清单中。以下代码片段解释了 Android 清单中服务定义应该是什么样子的：

```kt
    <manifest xmlns:android=
     "http://schemas.android.com/apk/res/android"   
      package="com.journaler"> 
      ... 
      <application ... > 
        <service 
          android:name=".service.MainService" 
          android:exported="false" /> 
          ... 

      </application> 
     </manifest>

```

正如你所看到的，我们定义了扩展`Service`类的`MainService`类，并且它位于`service`包下。导出标志设置为`false`，这意味着`service`将在与我们的应用程序相同的进程中运行。要在一个单独的进程中运行你的`service`，将这个标志设置为`true`。

重要的是要注意，`Service`类不是你唯一可以扩展的类。`IntentService`类也是可用的。那么，当我们扩展它时，我们会得到什么？`IntentService`代表从`Service`类派生的类。`IntentService`使用工作线程逐个处理请求。我们必须实现`onHandleIntent()`方法来实现这个目的。当`IntentService`类被扩展时，它看起来像这样：

```kt
     public class MainIntentService extends IntentService { 
       /** 
       * A constructor is mandatory! 
       */ 
       public MainIntentService() { 
         super("MainIntentService"); 
       } 

       /** 
       * All important work is performed here. 
       */ 
       @Override 
       protected void onHandleIntent(Intent intent) { 
         // Your implementation for handling received intents. 

       } 
     } 
```

让我们回到扩展`Service`类并专注于它。我们将重写`onStartCommand()`方法，使其看起来像这样：

```kt
    override fun onStartCommand(intent: Intent?, flags: Int, startId:  
    Int): Int { 

      return Service.START_STICKY 
    }
```

那么，`START_STICKY`返回的结果是什么意思？如果我们的服务被系统杀死，或者我们杀死了服务所属的应用程序，它将重新启动。相反的是`START_NOT_STICKY`；在这种情况下，服务将不会被重新创建和重新启动。

# 启动服务

要启动服务，我们需要定义代表它的意图。这是服务可以启动的一个例子：

```kt
    val startServiceIntent = Intent(ctx, MainService::class.java) 
    ctx.startService(startServiceIntent) 
```

这里，`ctx`代表 Android `Context`类的任何有效实例。

# 停止服务

要停止服务，执行 Android `Context`类的`stopService()`方法，就像这样：

```kt
     val stopServiceIntent = Intent(ctx, MainService::class.java)
     ctx.stopService(startServiceIntent) 
```

# 绑定到 Android 服务

**绑定服务**是允许 Android 组件绑定到它的服务。要执行绑定，我们必须调用`bindService()`方法。当你想要从活动或其他 Android 组件与服务交互时，服务绑定是必要的。为了使绑定工作，你必须实现`onBind()`方法并返回一个`IBinder`实例。如果没有人感兴趣了，所有人都解绑了，Android 就会销毁服务。对于这种类型的服务，你不需要执行停止例程。

# 停止服务

我们已经提到`stopService`将停止我们的服务。无论如何，我们可以通过在我们的服务实现中调用`stopSelf()`来实现相同的效果。

# 服务生命周期

我们涵盖并解释了在 Android 服务的生命周期中执行的所有重要方法。服务像所有其他 Android 组件一样有自己的生命周期。到目前为止我们提到的一切都在下面的截图中表示出来：

![](img/38a59665-0d7e-433a-8e7a-5188c64ad0cd.png)

现在，我们对 Android 服务有了基本的了解，我们将创建我们自己的服务并扩展 Journaler 应用程序。这个服务将在后面的章节中被重复扩展更多的代码。所以，请注意每一行，因为它可能是至关重要的。

# 定义主应用程序服务

正如你已经知道的，我们的应用程序处理笔记和待办事项。当前的应用程序实现将我们的数据保存在 SQLite 数据库中。这些数据将与运行在某个远程服务器上的后端实例进行同步。所有与同步相关的操作将在我们应用程序的后台默默执行。所有的责任将交给我们将要定义的服务。创建一个名为`service`的新包和一个名为`MainService`的新类，它将扩展 Android `service`类。确保你的实现看起来像这样：

```kt
    class MainService : Service(), DataSynchronization { 

      private val tag = "Main service" 
      private var binder = getServiceBinder() 
      private var executor = TaskExecutor.getInstance(1) 

      override fun onCreate() { 
        super.onCreate() 
        Log.v(tag, "[ ON CREATE ]") 
      } 

      override fun onStartCommand(intent: Intent?, flags: Int, startId:
      Int): Int { 
        Log.v(tag, "[ ON START COMMAND ]") 
        synchronize() 
        return Service.START_STICKY 
      } 

      override fun onBind(p0: Intent?): IBinder { 
        Log.v(tag, "[ ON BIND ]") 
        return binder 
      } 

      override fun onUnbind(intent: Intent?): Boolean { 
        val result = super.onUnbind(intent) 
        Log.v(tag, "[ ON UNBIND ]") 
        return result 
      } 

      override fun onDestroy() { 
        synchronize() 
        super.onDestroy() 
        Log.v(tag, "[ ON DESTROY ]") 
      } 

      override fun onLowMemory() { 
        super.onLowMemory() 
        Log.w(tag, "[ ON LOW MEMORY ]") 
      } 

      override fun synchronize() { 
        executor.execute { 
            Log.i(tag, "Synchronizing data [ START ]") 
            // For now we will only simulate this operation! 
            Thread.sleep(3000) 
            Log.i(tag, "Synchronizing data [ END ]") 
        } 
      } 

      private fun getServiceBinder(): MainServiceBinder = 
      MainServiceBinder() 

      inner class MainServiceBinder : Binder() { 
        fun getService(): MainService = this@MainService 
      } 
    }
```

让我们解释一下我们的主要服务。正如你们已经知道的，我们将扩展 Android 的`Service`类以获得所有的服务功能。我们还实现了`DataSynchronization`接口，它将描述我们服务的主要功能，即同步。请参考以下代码：

```kt
    package com.journaler.service 
    interface DataSynchronization { 

     fun synchronize() 
    }
```

所以，我们定义了`synchronize()`方法的实现，它实际上将模拟真正的同步。稍后，我们将更新这段代码以执行真正的后端通信。

所有重要的生命周期方法都被重写。注意`bind()`方法！此方法将通过调用`getServiceBinder()`方法返回一个由`MainServiceBinder`类生成的绑定器实例。由于`MainServiceBinder`类，我们将向最终用户公开我们的`service`实例，最终用户将能够在需要时触发同步机制。

同步不仅仅是由最终用户触发的，还会被服务自动触发。当服务启动和销毁时，我们会触发同步。

对我们来说，`MainService`的启动和停止是下一个重要的点。打开代表您的应用程序的`Journaler`类，并应用此更新：

```kt
     class Journaler : Application() { 

       companion object { 
         val tag = "Journaler" 
         var ctx: Context? = null 
       } 

       override fun onCreate() { 
         super.onCreate() 
         ctx = applicationContext 
         Log.v(tag, "[ ON CREATE ]") 
         startService() 
       } 

       override fun onLowMemory() { 
         super.onLowMemory() 
         Log.w(tag, "[ ON LOW MEMORY ]") 
         // If we get low on memory we will stop service if running. 
         stopService() 
       } 

       override fun onTrimMemory(level: Int) { 
         super.onTrimMemory(level) 
         Log.d(tag, "[ ON TRIM MEMORY ]: $level") 
       } 

       private fun startService() { 
         val serviceIntent = Intent(this, MainService::class.java) 
         startService(serviceIntent) 
       } 

       private fun stopService() { 
        val serviceIntent = Intent(this, MainService::class.java) 
        stopService(serviceIntent) 
       } 

     } 
```

当 Journaler 应用程序被创建时，`MainService`将被启动。我们还将添加一个小的优化。如果我们的应用程序内存不足，我们将停止我们的`MainService`类。由于服务是粘性启动的，如果我们明确杀死我们的应用程序，服务将重新启动。

到目前为止，我们已经涵盖了服务的启动和停止以及其实现。您可能还记得我们的模拟，在我们的应用程序抽屉底部，我们计划放置一个额外的项目。我们计划放置同步按钮。触发此按钮将与后端进行同步。

我们将添加该菜单项并将其与我们的服务连接起来。首先让我们做一些准备工作。打开`NavigationDrawerItem`类并按以下方式更新它：

```kt
    data class NavigationDrawerItem( 
      val title: String, 
      val onClick: Runnable, 
      var enabled: Boolean = true 
    ) 
```

我们引入了`enabled`参数。这样，如果需要，我们的应用程序抽屉中的一些项目可以被禁用。我们的同步按钮将默认禁用，并在绑定到`main`服务时启用。这些更改也必须影响`NavigationDrawerAdapter`。请参考以下代码：

```kt
    class NavigationDrawerAdapter( 
      val ctx: Context, 
      val items: List<NavigationDrawerItem> 
      ) : BaseAdapter() { 

        private val tag = "Nav. drw. adptr." 

        override fun getView(position: Int, v: View?, group: 
        ViewGroup?): View { 
          ... 
          val item = items[position] 
          val title = view.findViewById<Button>(R.id.drawer_item) 
          ... 
          title.setOnClickListener { 
            if (item.enabled) { 
                item.onClick.run() 
            } else { 
                Log.w(tag, "Item is disabled: $item") 
            } 
          } 

          return view 
       } 
        ... 
    }
```

最后，我们将更新我们的`MainActivity`类如下，以便同步按钮可以触发同步：

```kt
    class MainActivity : BaseActivity() { 
      ... 
      private var service: MainService? = null 

      private val synchronize: NavigationDrawerItem by lazy { 
        NavigationDrawerItem( 
          getString(R.string.synchronize), 
          Runnable { service?.synchronize() }, 
          false 
        ) 
     } 

     private val serviceConnection = object : ServiceConnection { 
        override fun onServiceDisconnected(p0: ComponentName?) { 
            service = null 
            synchronize.enabled = false 
        } 

        override fun onServiceConnected(p0: ComponentName?, binder: 
        IBinder?) { 
          if (binder is MainService.MainServiceBinder) { 
            service = binder.getService() 
            service?.let { 
              synchronize.enabled = true 
            } 
           } 
        } 
     } 

      override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 
        ... 
        val menuItems = mutableListOf<NavigationDrawerItem>() 
        ... 
        menuItems.add(synchronize) 
        ... 
      } 

      override fun onResume() { 
        super.onResume() 
        val intent = Intent(this, MainService::class.java) 
        bindService(intent, serviceConnection, 
        android.content.Context.BIND_AUTO_CREATE) 
     } 

     override fun onPause() { 
        super.onPause() 
        unbindService(serviceConnection) 
     } 

     ... 
    } 
```

我们将根据我们的主活动状态是否活动来绑定或解绑`main`服务。为了执行绑定，我们需要`ServiceConnection`实现，因为它将根据绑定状态启用或禁用同步按钮。此外，我们将根据绑定状态维护`main`服务实例。同步按钮将访问`service`实例，并在点击时触发`synchronize()`方法。

# 定义`intent`服务

我们的`main`服务正在运行并且责任已定义。现在，我们将通过引入另一个服务来对我们的应用程序进行更多改进。这一次，我们将定义`intent`服务。`intent`服务将接管数据库 CRUD 操作的执行责任。基本上，我们将定义我们的`intent`服务并对我们已有的代码进行重构。

首先，我们将在`service`包内创建一个名为`DatabaseService`的新类。在我们放置整个实现之前，我们将在 Android 清单中注册它如下：

```kt
    <manifest xmlns:android=
      "http://schemas.android.com/apk/res/android" 
       package="com.journaler"> 
       ... 
      <application ... > 
      <service 
        android:name=".service.MainService" 
        android:exported="false" /> 

      <service 
        android:name=".service.DatabaseService" 
        android:exported="false" /> 
        ... 
      </application> 
    </manifest> 

    Define DatabaseService like this: 
    class DatabaseService :
     IntentService("DatabaseService") { 

       companion object { 
         val EXTRA_ENTRY = "entry" 
         val EXTRA_OPERATION = "operation" 
       } 

       private val tag = "Database service" 

       override fun onCreate() { 
         super.onCreate() 
         Log.v(tag, "[ ON CREATE ]") 
       } 

       override fun onLowMemory() { 
         super.onLowMemory() 
         Log.w(tag, "[ ON LOW MEMORY ]") 
       } 

       override fun onDestroy() { 
         super.onDestroy() 
         Log.v(tag, "[ ON DESTROY ]") 
       } 

       override fun onHandleIntent(p0: Intent?) { 
         p0?.let { 
            val note = p0.getParcelableExtra<Note>(EXTRA_ENTRY) 
            note?.let { 
               val operation = p0.getIntExtra(EXTRA_OPERATION, -1) 
               when (operation) { 
                 MODE.CREATE.mode -> { 
                   val result = Db.insert(note) 
                   if (result) { 
                      Log.i(tag, "Note inserted.") 
                   } else { 
                      Log.e(tag, "Note not inserted.") 
                      } 
                   } 
                   MODE.EDIT.mode -> { 
                     val result = Db.update(note) 
                     if (result) { 
                       Log.i(tag, "Note updated.") 
                     } else { 
                       Log.e(tag, "Note not updated.") 
                      } 
                    } 
                    else -> { 
                        Log.w(tag, "Unknown mode [ $operation ]") 
                    } 

                  } 

                } 

             } 

         } 

     } 
```

服务将接收意图，获取操作，并从中获取实例。根据操作，将触发适当的 CRUD 操作。为了将`Note`实例传递给`Intent`，我们必须实现`Parcelable`，以便数据传递效率高。例如，与`Serializable`相比，`Parcelable`要快得多。为此目的，代码已经进行了大量优化。我们将执行显式序列化，而不使用反射。打开您的`Note`类并按以下方式更新它：

```kt
    package com.journaler.model 
    import android.location.Location 
    import android.os.Parcel 
    import android.os.Parcelable 

    class Note( 
      title: String, 
      message: String, 
      location: Location 
    ) : Entry( 
      title, 
      message, 
      location 
    ), Parcelable { 

      override var id = 0L 

      constructor(parcel: Parcel) : this( 
        parcel.readString(), 
        parcel.readString(), 
        parcel.readParcelable(Location::class.java.classLoader) 
      ) { 
         id = parcel.readLong() 
        } 

       override fun writeToParcel(parcel: Parcel, flags: Int) { 
         parcel.writeString(title) 
         parcel.writeString(message) 
         parcel.writeParcelable(location, 0) 
         parcel.writeLong(id) 
       } 

       override fun describeContents(): Int { 
         return 0 
       } 

       companion object CREATOR : Parcelable.Creator<Note> { 
         override fun createFromParcel(parcel: Parcel): Note { 
            return Note(parcel) 
        } 

         override fun newArray(size: Int): Array<Note?> { 
            return arrayOfNulls(size) 
        } 
      } 

    } 
```

当通过`intent`传递到`DatabaseService`时，`Note`类将被高效地序列化和反序列化。

最后一块拼图是更改当前执行 CRUD 操作的代码。我们将创建`intent`并将其发送，以便我们的服务为我们处理其余工作。打开`NoteActivity`类并按以下方式更新代码：

```kt
    class NoteActivity : ItemActivity() { 
      ... 
      private val locationListener = object : LocationListener { 
        override fun onLocationChanged(p0: Location?) { 
          p0?.let { 
            LocationProvider.unsubscribe(this) 
            location = p0 
            val title = getNoteTitle() 
            val content = getNoteContent() 
            note = Note(title, content, p0) 

            // Switching to intent service. 
            val dbIntent = Intent(this@NoteActivity, 
            DatabaseService::class.java) 
            dbIntent.putExtra(DatabaseService.EXTRA_ENTRY, note) 
            dbIntent.putExtra(DatabaseService.EXTRA_OPERATION, 
            MODE.CREATE.mode) 
            startService(dbIntent) 
            sendMessage(true) 
          } 
      } 

     override fun onStatusChanged(p0: String?, p1: Int, p2: Bundle?) {} 
     override fun onProviderEnabled(p0: String?) {} 
     override fun onProviderDisabled(p0: String?) {} 
   } 
    ... 
    private fun updateNote() { 
      if (note == null) { 
        if (!TextUtils.isEmpty(getNoteTitle()) && 
        !TextUtils.isEmpty(getNoteContent())) { 
           LocationProvider.subscribe(locationListener) 
        } 
        } else { 
            note?.title = getNoteTitle() 
            note?.message = getNoteContent() 

            // Switching to intent service. 
            val dbIntent = Intent(this@NoteActivity, 
            DatabaseService::class.java) 
            dbIntent.putExtra(DatabaseService.EXTRA_ENTRY, note) 
            dbIntent.putExtra(DatabaseService.EXTRA_OPERATION,
            MODE.EDIT.mode) 
            startService(dbIntent) 
            sendMessage(true) 
        } 
      } 
      ... 
    } 
```

正如你所看到的，改变真的很简单。构建你的应用程序并运行它。当你创建或更新你的`Note`类时，你会注意到我们执行的数据库操作的日志。此外，你还会注意到`DatabaseService`的生命周期方法被记录下来。

# 总结

恭喜！你掌握了 Android 服务并显著改进了应用程序！在本章中，我们解释了什么是 Android 服务。我们还解释了每种类型的 Android 服务，并举例说明了它们的用途。现在，当你完成这些实现时，我们鼓励你至少考虑一个可以接管应用程序的某个现有部分或引入全新内容的服务。玩转这些服务，并尝试思考它们能给你带来的好处。
