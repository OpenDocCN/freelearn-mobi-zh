# 第十一章：消息

在本章中，我们将使用 Android 广播，并将其用作接收和发送消息的机制。我们将分步理解它。首先，我们将解释底层机制以及如何使用 Android 广播消息。然后，我们将监听一些最常见的消息。因为仅仅监听是不够的，我们将创建新的消息并进行广播。最后，我们将了解启动、关闭和网络广播消息，以便我们的应用程序意识到这一重要的系统事件。

在本章中，我们将涵盖以下主题：

+   Android 广播

+   监听广播

+   创建广播

+   监听网络事件

# 理解 Android 广播

Android 应用程序可以发送或接收消息。消息可以是系统相关事件，也可以是我们定义的自定义事件。感兴趣的各方通过定义适当的意图过滤器和广播接收器来注册特定的消息。当广播消息时，所有感兴趣的各方都会收到通知。重要的是要注意，一旦你订阅了广播消息（特别是从`Activity`类），你必须在某个时候取消订阅。我们什么时候可以使用广播消息？我们在应用程序需要跨应用程序的消息系统时使用广播消息。例如，想象一下你在后台启动了一个长时间运行的进程。在某个时候，你想通知多个上下文处理结果。广播消息是这个问题的完美解决方案。

# 系统广播

系统广播是在发生各种系统事件时由 Android 系统发送的广播。我们发送和最终接收的每条消息都包装在包含有关特定事件信息的`Intent`类中。每个`Intent`必须设置适当的操作。例如--`android.intent.action.ACTION_POWER_CONNECTED`。有关事件的信息用捆绑的额外数据表示。例如，我们可能捆绑了一个额外的字符串字段，表示与我们感兴趣的事件相关的特定数据。让我们考虑一个充电和电池信息的例子。每当电池状态发生变化时，感兴趣的各方将收到通知，并接收包含电池电量信息的广播消息：

```kt
    val intentFilter = IntentFilter(Intent.ACTION_BATTERY_CHANGED) 
    val batteryStatus = registerReceiver(null, intentFilter) 

    val status = batteryStatus.getIntExtra(BatteryManager.
      EXTRA_STATUS, -1) 

    val isCharging = 
                status == BatteryManager.BATTERY_STATUS_CHARGING || 
                status == BatteryManager.BATTERY_STATUS_FULL 

    val chargePlug =    batteryStatus.getIntExtra(BatteryManager.
      EXTRA_PLUGGED, -1) 
    val usbCharge = chargePlug == BatteryManager.
      BATTERY_PLUGGED_USB 
    val acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC
```

在这个例子中，我们注册了电池信息的意图过滤器。然而，我们没有传递广播接收器的实例。为什么？因为电池数据是粘性的。**粘性意图**是在广播执行后一段时间内保持存在的意图。注册到这些数据将立即返回包含最新数据的意图。我们也可以传递广播接收器的实例。让我们来做一下：

```kt
    val receiver = object : BroadcastReceiver() { 
      override fun onReceive(p0: Context?, batteryStatus: Intent?) { 
      val status = batteryStatus?.getIntExtra
      (BatteryManager.EXTRA_STATUS, -1) 
                 val isCharging = 
                        status == 
                        BatteryManager.BATTERY_STATUS_CHARGING || 
                        status == BatteryManager.BATTERY_STATUS_FULL 
                        val chargePlug = batteryStatus?.getIntExtra
                       (BatteryManager.EXTRA_PLUGGED, -1) 
                        val usbCharge = chargePlug ==
                       BatteryManager.BATTERY_PLUGGED_USB 
                       val acCharge = chargePlug == 
                       BatteryManager.BATTERY_PLUGGED_AC 
        } 
    } 

    val intentFilter = IntentFilter(Intent.ACTION_BATTERY_CHANGED) 
    registerReceiver(receiver, intentFilter)
```

每当电池信息发生变化时，接收器将执行我们在其实现中定义的代码；我们也可以在 Android 清单中定义我们的接收器：

```kt
    <receiver android:name=".OurPowerReceiver"> 
      <intent-filter> 
        <action android:name="android.intent.action.
        ACTION_POWER_CONNECTED"/> 
        <action android:name="android.intent.action.
        ACTION_POWER_DISCONNECTED"/> 
      </intent-filter> 
    </receiver>
```

# 监听广播

如前面的例子所示，我们可以以以下两种方式之一接收广播：

+   通过 Android 清单注册广播接收器

+   使用`registerBroadcast()`方法在上下文中注册广播

通过清单声明需要以下内容：

+   带有`android:name`和`android:exported`参数的`<receiver>`元素。

+   接收器必须包含我们订阅的操作的`intent`过滤器。看看下面的例子：

```kt
        <receiver android:name=".OurBootReceiver"
          android:exported="true"> 
          <intent-filter> 
            <action android:name=
            "android.intent.action.BOOT_COMPLETED"/> 
            ... 
            <action android:name="..."/> 
            <action android:name="..."/> 
            <action android:name="..."/> 

         </intent-filter> 

       </receiver> 
```

如你所见，`name`属性代表我们广播接收器类的名称。导出意味着应用程序可以或不能接收来自接收器应用程序外部的消息。

如果你子类化`BroadcastReceiver`，它应该像这个例子一样：

```kt
     val receiver = object : BroadcastReceiver() { 
        override fun onReceive(ctx: Context?, intent: Intent?) { 
            // Handle your received code. 

        } 
    }
```

注意，在`onReceive()`方法实现中执行的操作不应该花费太多时间。否则会发生 ANR！

# 从上下文注册

现在我们将向您展示一个从 Android 上下文注册广播接收器的示例。要注册接收器，你需要它的一个实例。假设我们的实例是`myReceiver`：

```kt
    val myReceiver = object : BroadcastReceiver(){ 

      ... 

     }We need intent filter prepared: 
     val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
     registerReceiver(myReceiver, filter)
```

这个例子将注册一个接收者，用于监听连接信息。由于此接收者是从上下文中注册的，只要我们注册的上下文有效，它就是有效的。您还可以使用`LocalBroadcastManager`类。`LocalBroadcastManager`的目的是注册并向您进程中的本地对象发送意图的广播。这是例子：

```kt
    LocalBroadcastManager 
      .getInstance(applicationContext) 
      .registerReceiver(myReceiver, intentFilter)
```

要取消注册，请执行以下代码片段：

```kt
    LocalBroadcastManager 
      .getInstance(applicationContext) 
      .unregisterReceiver(myReceiver)
```

对于上下文订阅的接收者，重点要注意取消注册。例如，如果我们在活动的`onCreate()`方法中注册接收者，我们必须在`onDestroy()`方法中取消注册。如果我们不这样做，就会出现接收者泄漏！同样，如果我们在活动的`onResume()`中注册，我们必须在`onPause()`中取消注册。如果我们不这样做，就会多次注册！

# 接收者执行

我们在`onReceive()`实现中执行的代码被视为前台进程。广播接收器在我们从该方法返回之前是活动的。系统将始终运行您在实现中定义的代码，除非发生极端的内存压力。正如我们提到的，您应该只执行短操作！否则，可能会发生 ANR！当接收到消息时执行长时间运行的操作的一个很好的例子是启动`AsyncTask`并在那里执行所有工作。接下来，我们将向您展示一个演示这一点的例子：

```kt
     class AsyncReceiver : BroadcastReceiver() { 
       override fun onReceive(p0: Context?, p1: Intent?) { 
         val pending = goAsync() 
         val async = object : AsyncTask<Unit, Unit, Unit>() { 
           override fun doInBackground(vararg p0: Unit?) { 
             // Do some intensive work here... 
             pending.finish() 
           } 
       } 
       async.execute() 
      }
    } 
```

在这个例子中，我们介绍了`goAsync()`方法的使用。它是做什么的？该方法返回一个`PendingResult`类型的对象，表示调用`API`方法的待定结果。Android 系统认为接收者在调用此实例的`finish()`方法之前是活动的。使用这种机制，可以在广播接收器中进行异步处理。在完成繁重的工作后，我们调用`finish()`来指示 Android 系统可以回收此组件。

# 发送广播

Android 有以下三种发送广播消息的方式：

+   使用`sendOrderedBroadcast(Intent, String)`方法一次向一个接收者发送消息。由于接收者按顺序执行，可以将结果传播给下一个接收者。此外，还可以中止广播，使其不会传递给其他接收者。我们可以控制接收者执行的顺序。我们可以使用匹配意图过滤器的`android:priority`属性来设置优先级。

+   使用`sendBroadcast(Intent)`方法向所有接收者发送广播消息。发送是无序的。

+   使用`LocalBroadcastManager.sendBroadcast(Intent)`方法向与发送方相同应用程序中的接收者发送广播。

让我们看一个向所有感兴趣的方发送广播消息的例子：

```kt
    val intent = Intent() 
    intent.action = "com.journaler.broadcast.TODO_CREATED" 
    intent.putExtra("title", "Go, buy some lunch.") 
    intent.putExtra("message", "For lunch we have chicken.")
    sendBroadcast(intent)
```

我们创建了一个包含有关我们创建的`note`（标题和消息）的额外数据的广播消息。所有感兴趣的方都需要一个适当的`IntentFilter`实例来执行操作：

```kt
    com.journaler.broadcast.TODO_CREATED
```

不要混淆启动活动和发送广播消息。`Intent`类只是用作我们信息的包装器。这两个操作完全不同！您可以使用本地广播机制来实现相同的功能：

```kt
    val ctx = ... 
    val broadcastManager = LocalBroadcastManager.getInstance(ctx) 
    val intent = Intent() 
    intent.action = "com.journaler.broadcast.TODO_CREATED" 
    intent.putExtra("title", "Go, buy some lunch.") 
    intent.putExtra("message", "For lunch we have chicken.") 
    broadcastManager.sendBroadcast(intent)
```

现在，当我们向您展示了广播消息的最重要方面后，我们将继续扩展我们的应用程序。Journaler 将发送和接收包含数据的自定义广播消息，并与系统广播进行交互，例如系统启动、关闭和网络。

# 创建自己的广播消息

正如你可能记得的，我们对`NoteActivity`类进行了代码重构。让我们展示我们在重要部分的最后状态，以便进行进一步的演示：

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

        override fun onStatusChanged(p0: String?, p1: Int, p2: Bundle?)
        {} 
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

如果您再次查看这个，您会注意到我们在执行时向我们的服务发送了`intent`，但由于我们没有得到返回值，我们只是用布尔值`true`作为其参数执行了`sendMessage()`方法。在这里，我们期望一个代表 CRUD 操作结果的值，即成功或失败。我们将使用广播消息将我们的服务与`NoteActivity`连接起来。每次插入或更新`note`广播时，都会触发一个消息。我们在`NoteActivity`中定义的监听器将对此消息做出响应并触发`sendMessage()`方法。让我们更新我们的代码！打开`Crud`接口并使用包含操作和 CRUD 操作结果常量的`companion`对象进行扩展：

```kt
    interface Crud<T> { 
      companion object { 
        val BROADCAST_ACTION = "com.journaler.broadcast.crud" 
        val BROADCAST_EXTRAS_KEY_CRUD_OPERATION_RESULT = "crud_result" 
      } 
       ... 
    }
```

现在，打开`DatabaseService`并扩展它，负责在执行 CRUD 操作时发送广播消息的方法：

```kt
     class DatabaseService : IntentService("DatabaseService") { 
       ... 
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
                        broadcastResult(result) 
                    } 
                    MODE.EDIT.mode -> { 
                        val result = Db.update(note) 
                        if (result) { 
                            Log.i(tag, "Note updated.") 
                        } else { 
                            Log.e(tag, "Note not updated.") 
                        } 
                        broadcastResult(result) 
                    } 
                    else -> { 
                        Log.w(tag, "Unknown mode [ $operation ]") 
                    } 
                 } 
             } 
           } 
        } 
        ... 
        private fun broadcastResult(result: Boolean) { 
          val intent = Intent() 
          intent.putExtra( 
                Crud.BROADCAST_EXTRAS_KEY_CRUD_OPERATION_RESULT, 
                if (result) { 
                    1 
                } else { 
                    0 
                } 
          ) 
        } }
```

我们引入了一个新方法。其他一切都一样。我们将获取 CRUD 操作结果并将其作为消息广播。`NoteActivity`将监听它：

```kt
   class NoteActivity : ItemActivity() { 
     ... 
     private val crudOperationListener = object : BroadcastReceiver() { 
        override fun onReceive(ctx: Context?, intent: Intent?) { 
            intent?.let { 
                val crudResultValue =
                intent.getIntExtra(MODE.EXTRAS_KEY, 0) 
                sendMessage(crudResultValue == 1) 
            } 
        } 
      } 
      ... 
      override fun onCreate(savedInstanceState: Bundle?) { 
        .... 
        registerReceiver(crudOperationListener, intentFiler) 
      } 

      override fun onDestroy() { 
        unregisterReceiver(crudOperationListener) 
        super.onDestroy() 
      } 
      ... 
      private fun sendMessage(result: Boolean) { 
         Log.v(tag, "Crud operation result [ $result ]") 
         val msg = handler?.obtainMessage() 
         if (result) { 
            msg?.arg1 = 1 
         } else { 
            msg?.arg1 = 0 
         } 
         handler?.sendMessage(msg) 
     } }
```

这很简单！我们重新连接了原始的`sendMessage()`方法与 CRUD 操作结果。在接下来的章节中，我们将考虑应用程序可以通过监听启动、关机和网络广播消息来进行一些重大改进。

# 使用启动和关闭广播

有时，服务在应用程序启动时运行至关重要。有时，在终止之前进行一些清理工作也很重要。在下面的示例中，我们将扩展 Journaler 应用程序以监听这些广播消息并进行一些工作。我们要做的第一件事是创建两个扩展`BroadcastReceiver`类的类：

+   `BootReceiver`：这是处理系统启动事件的

+   `ShutdownReceiver`：这是处理系统关机事件的

在`manifest`文件中注册它们如下：

```kt
    <manifest  
     ... 
    > 
    ... 
    <receiver 
        android:name=".receiver.BootReceiver" 
        android:enabled="true" 
        android:exported="false"> 
        <intent-filter> 
          <action android:name=
           "android.intent.action.BOOT_COMPLETED" /> 
        </intent-filter> 

        <intent-filter> 
          <action android:name=
          "android.intent.action.PACKAGE_REPLACED" /> 
          data android:scheme="package" /> 
        </intent-filter> 

        <intent-filter> 
          <action android:name=
          "android.intent.action.PACKAGE_ADDED" /> 
          <data android:scheme="package" /> 
        </intent-filter> 
     </receiver> 

    <receiver android:name=".receiver.ShutdownReceiver"> 

      <intent-filter> 
        <action android:name=
        "android.intent.action.ACTION_SHUTDOWN" /> 
        <action android:name=
        "android.intent.action.QUICKBOOT_POWEROFF" /> 
      </intent-filter> 
    </receiver> 
     ...
    </manifest> 
```

`BootReceiver`类将在启动或替换应用程序时触发。关闭将在关闭设备时触发。让我们创建适当的实现。打开`BootReceiver`类并定义如下：

```kt
     package com.journaler.receiver 

     import android.content.BroadcastReceiver 
     import android.content.Context 
     import android.content.Intent 
     import android.util.Log 

     class BootReceiver : BroadcastReceiver() { 

       val tag = "Boot receiver" 

       override fun onReceive(p0: Context?, p1: Intent?) { 
         Log.i(tag, "Boot completed.") 
         // Perform your on boot stuff here. 
       } 

     } 
```

如您所见，我们为这两个类定义了`receiver`包。对于`ShutdownReceiver`，请像这样定义类：

```kt
    package com.journaler.receiver 

    import android.content.BroadcastReceiver 
    import android.content.Context 
    import android.content.Intent 
    import android.util.Log 

    class ShutdownReceiver : BroadcastReceiver() { 

      val tag = "Shutdown receiver" 

      override fun onReceive(p0: Context?, p1: Intent?) { 
        Log.i(tag, "Shutting down.") 
        // Perform your on cleanup stuff here.   
      } }
```

为了使其工作，我们需要进行一次更改；否则，应用程序将崩溃。从`Application`类中启动`main`服务移动到主活动`onCreate()`方法中。这是`Journaler`类的第一个更新：

```kt
    class Journaler : Application() { 
      ... 
      override fun onCreate() { // We removed start service method
        execution. 
        super.onCreate() 
        ctx = applicationContext 
        Log.v(tag, "[ ON CREATE ]") 
     } 
     // We removed startService() method implementation. 
     ... 
    }
```

然后通过在`onCreate()`方法的末尾添加行来扩展`MainActivity`类：

```kt
    class MainActivity : BaseActivity() { 
      ... 
      override fun onCreate(savedInstanceState: Bundle?) { 
        ... 
        val serviceIntent = Intent(this, MainService::class.java) 
        startService(serviceIntent) 
     } 
    ... } }
```

构建并运行您的应用程序。首先关闭手机，然后再次开机。过滤您的 Logcat，使其仅显示应用程序的日志。您应该有以下输出：

```kt
... I/Shutdown receiver: Shutting down. 
... I/Boot receiver: Boot completed.
```

请记住，有时需要超过两分钟才能接收到启动事件！

# 监听网络事件

我们想要的最后一个改进是在建立连接时使我们的应用程序能够执行同步。在相同的`NetworkReceiver`包中创建一个名为的新类。确保您有以下实现：

```kt
    class NetworkReceiver : BroadcastReceiver() {
      private val tag = "Network receiver"
      private var service: MainService? = null

      private val serviceConnection = object : ServiceConnection { 
        override fun onServiceDisconnected(p0: ComponentName?) { 
          service = null 
        } 

        override fun onServiceConnected(p0: ComponentName?, binder:
        IBinder?) { 
            if (binder is MainService.MainServiceBinder) { 
                service = binder.getService() 
                service?.synchronize() 
            } 
        } 
       } 

       override fun onReceive(context: Context?, p1: Intent?) { 
       context?.let { 

            val cm = context.getSystemService
           (Context.CONNECTIVITY_SERVICE) as ConnectivityManager 

            val activeNetwork = cm.activeNetworkInfo 
            val isConnected = activeNetwork != null &&
            activeNetwork.isConnectedOrConnecting 
            if (isConnected) { 
                Log.v(tag, "Connectivity [ AVAILABLE ]") 
                if (service == null) { 
                    val intent = Intent(context,
                    MainService::class.java) 
                    context.bindService( 
                        intent, serviceConnection,
                        android.content.Context.BIND_AUTO_CREATE 
                    ) 
                } else { 
                    service?.synchronize() 
                } 
            } else { 
                Log.w(tag, "Connectivity [ UNAVAILABLE ]") 
                context.unbindService(serviceConnection) 
            } 
          } 
        } 
    }
```

当发生连接事件时，此接收器将接收消息。每当我们有上下文和连接时，我们将绑定到服务并触发同步。不要担心频繁的同步触发，因为在接下来的章节中，我们将在同步方法的实现中保护自己免受它的影响。通过更新 Journaler 应用程序类来注册您的监听器如下：

```kt
     class Journaler : Application() { 
       ... 
       override fun onCreate() { 
          super.onCreate() 
          ctx = applicationContext 
          Log.v(tag, "[ ON CREATE ]") 
          val filter =  
          IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION) 
          registerReceiver(networkReceiver, filter) 
      } 
      ... 
    } 
```

构建并运行您的应用程序。关闭您的连接（Wi-Fi、移动数据）然后再次打开。观察以下 Logcat 输出：

```kt
... V/Network receiver: Connectivity [ AVAILABLE ] 
... V/Network receiver: Connectivity [ AVAILABLE ] 
... V/Network receiver: Connectivity [ AVAILABLE ] 
... W/Network receiver: Connectivity [ UNAVAILABLE ] 
... V/Network receiver: Connectivity [ AVAILABLE ] 
... V/Network receiver: Connectivity [ AVAILABLE ] 
... V/Network receiver: Connectivity [ AVAILABLE ] 
```

# 总结

在本章中，我们学习了如何使用广播消息。我们还学习了如何监听系统广播消息以及我们自己创建的广播消息。Journaler 应用程序得到了显著改进，变得更加灵活。我们不会止步于此，而是将通过学习新知识和扩展我们的代码来继续在 Android 框架中取得进展。
