# 第九章：Android 中的并发

在本章中，我们将解释 Android 中的并发。我们将给出例子和建议，并将并发应用到我们的 Journaler 应用程序中。我们已经通过演示`AsyncTask`类的使用来介绍了一些基础知识，但现在我们将深入探讨。

在本章中，我们将涵盖以下主题：

+   处理程序和线程

+   `AsyncTask`

+   Android Looper

+   延迟执行

# 介绍 Android 并发

我们的应用程序的默认执行是在主应用程序线程上执行的。这个执行必须是高效的！如果发生某些操作花费太长时间，那么我们会得到 ANR--Android 应用程序无响应的消息。为了避免 ANR，我们在后台运行我们的代码。Android 提供了机制，让我们可以高效地这样做。异步运行操作不仅可以提供良好的性能，还可以提供良好的用户体验。

# 主线程

所有用户界面更新都是从一个线程执行的。这就是主线程。所有事件都被收集在一个队列中，并由`Looper`类实例处理。

以下图片解释了涉及的类之间的关系：

![](img/1b186908-463f-4a05-9454-c76fdb4bdde7.png)

重要的是要注意，主线程更新是你看到的所有 UI。但它也可以从其他线程执行。直接从其他线程执行这些操作会导致异常，你的应用程序可能会崩溃。为了避免这种情况，通过从当前活动上下文调用`runOnUiThread()`方法在主线程上执行所有与线程相关的代码。

# 处理程序和线程

在 Android 中，可以通过使用线程来执行线程。不建议只是随意启动线程而没有任何控制。因此，为此目的，可以使用`ThreadPools`和`Executor`类。

为了演示这一点，我们将更新我们的应用程序。创建一个名为`execution`的新包，并在其中创建一个名为`TaskExecutor`的类。确保它看起来像这样：

```kt
     package com.journaler.execution 

     import java.util.concurrent.BlockingQueue 
     import java.util.concurrent.LinkedBlockingQueue 
     import java.util.concurrent.ThreadPoolExecutor 
     import java.util.concurrent.TimeUnit 

     class TaskExecutor private constructor( 
        corePoolSize: Int, 
        maximumPoolSize: Int, 
        workQueue: BlockingQueue<Runnable>? 

    ) : ThreadPoolExecutor( 
        corePoolSize, 
        maximumPoolSize, 
        0L, 
        TimeUnit.MILLISECONDS, 
        workQueue 
    ) { 

    companion object { 
        fun getInstance(capacity: Int): TaskExecutor { 
            return TaskExecutor( 
                    capacity, 
                    capacity * 2, 
                    LinkedBlockingQueue<Runnable>() 
            ) 
        } 
    } }
```

我们扩展了`ThreadPoolExecutor`类和`companion`对象，并为执行器实例化添加了成员方法。让我们将其应用到我们现有的代码中。我们将从我们使用的`AsyncTask`类切换到`TaskExecutor`。打开`NoteActivity`类并按照以下方式更新它：

```kt
     class NoteActivity : ItemActivity() { 
       ... 
       private val executor = TaskExecutor.getInstance(1) 
       ... 
       private val locationListener = object : LocationListener { 
         override fun onLocationChanged(p0: Location?) { 
            p0?.let { 
                LocationProvider.unsubscribe(this) 
                location = p0 
                val title = getNoteTitle() 
                val content = getNoteContent() 
                note = Note(title, content, p0) 
                executor.execute { 
                  val param = note 
                  var result = false 
                  param?.let { 
                      result = Db.insert(param) 
                  } 
                  if (result) { 
                      Log.i(tag, "Note inserted.") 
                  } else { 
                      Log.e(tag, "Note not inserted.") 
                  } 
               } 

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
           executor.execute { 
             val param = note 
             var result = false 
             param?.let { 
                result = Db.update(param) 
             } 
             if (result) { 
                Log.i(tag, "Note updated.") 
             } else { 
                Log.e(tag, "Note not updated.") 
             } 
           } 
        } 
       } 
  ... }
```

如你所见，我们用执行器替换了`AsyncTask`。我们的执行器一次只处理一个线程。

除了标准的线程方法，Android 还提供了处理程序作为开发人员的选择之一。处理程序不是线程的替代品，而是一种补充！处理程序实例会在其父线程中注册自己。它代表了向特定线程发送数据的机制。我们可以发送`Message`或`Runnable`类的实例。让我们通过一个例子来说明它的用法。我们将使用指示器更新笔记屏幕，如果一切都执行正确，指示器将是绿色。如果数据库持久化失败，它将是红色。它的默认颜色将是灰色。打开`activity_note.xml`文件并扩展它以包含指示器。指示器将是普通视图，如下所示：

```kt
     <?xml version="1.0" encoding="utf-8"?> 
     <ScrollView xmlns:android=
      "http://schemas.android.com/apk/res/android" 
     android:layout_width="match_parent" 
     android:layout_height="match_parent" 
     android:fillViewport="true"> 

     <LinearLayout 
        android:layout_width="match_parent" 
        android:layout_height="wrap_content" 
        android:background="@color/black_transparent_40" 
        android:orientation="vertical"> 

        ... 

        <RelativeLayout 
            android:layout_width="match_parent" 
            android:layout_height="wrap_content"> 

            <View 
                android:id="@+id/indicator" 
                android:layout_width="40dp" 
                android:layout_height="40dp" 
                android:layout_alignParentEnd="true" 
                android:layout_centerVertical="true" 
                android:layout_margin="10dp" 
                android:background="@android:color/darker_gray" /> 

            <EditText 
                android:id="@+id/note_title" 
                style="@style/edit_text_transparent" 
                android:layout_width="match_parent" 
                android:layout_height="wrap_content" 
                android:hint="@string/title" 
                android:padding="@dimen/form_padding" /> 

        </RelativeLayout>         
         ...      
      </LinearLayout> 

    </ScrollView> 
```

现在，当我们添加指示器时，它将根据数据库插入结果改变颜色。像这样更新你的`NoteActivity`类源代码：

```kt
     class NoteActivity : ItemActivity() { 
      ... 
      private var handler: Handler? = null 
      .... 
      override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 
        handler = Handler(Looper.getMainLooper()) 
        ... 
      } 
      ... 
      private val locationListener = object : LocationListener { 
        override fun onLocationChanged(p0: Location?) { 
            p0?.let { 
                ... 
                executor.execute { 
                    ... 
                    handler?.post { 
                        var color = R.color.vermilion 
                        if (result) { 
                            color = R.color.green 
                        } 
                        indicator.setBackgroundColor( 
                                ContextCompat.getColor( 
                                        this@NoteActivity, 
                                        color 
                                ) 
                        ) 
                    } 
                } 
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
            ... 
        } else { 
            ... 
            executor.execute { 
                ... 
                handler?.post { 
                    var color = R.color.vermilion 
                    if (result) { 
                        color = R.color.green 
                    } 
                    indicator.setBackgroundColor
                    (ContextCompat.getColor( 
                        this@NoteActivity, 
                        color 
                    )) 
                 } 
               } 
            } 
        } }
```

构建你的应用程序并运行它。创建一个新的笔记。你会注意到，在输入标题和消息内容后，指示器的颜色变成了绿色。

我们将进行一些更改，并对`Message`类实例执行相同的操作。根据这个示例更新你的代码：

```kt
     class NoteActivity : ItemActivity() { 
      ... 
      override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 
        handler = object : Handler(Looper.getMainLooper()) { 
            override fun handleMessage(msg: Message?) { 
                msg?.let { 
                    var color = R.color.vermilion 
                    if (msg.arg1 > 0) { 
                        color = R.color.green 
                    } 
                    indicator.setBackgroundColor
                    (ContextCompat.getColor( 
                       this@NoteActivity, 
                       color 
                    )) 
                  } 
                 super.handleMessage(msg) 
               } 
             } 
            ... 
          } 
        ... 
        private val locationListener = object : LocationListener { 
        override fun onLocationChanged(p0: Location?) { 
            p0?.let { 
                ... 
                executor.execute { 
                    ... 
                    sendMessage(result) 
                } 
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
            ... 
        } else { 
            ... 
            executor.execute { 
                ... 
                sendMessage(result) 
            } 
        } 
      } 
     ... 
     private fun sendMessage(result: Boolean) { 
        val msg = handler?.obtainMessage() 
        if (result) { 
            msg?.arg1 = 1 
        } else { 
            msg?.arg1 = 0 
        } 
        handler?.sendMessage(msg) 
     } 
     ... 
    } 
```

注意处理程序的实例化和`sendMessage()`方法。我们使用`Handler`类的`obtainMessage()`方法获取了`Message`实例。作为消息参数，我们传递了一个整数数据类型。根据它的值，我们将更新指示器的颜色。

# AsyncTask

你可能已经注意到，我们已经在我们的应用程序中使用了`AsyncTask`类。现在，我们将进一步运用它--我们将在执行器上运行它。为什么我们要这样做呢？

首先，默认情况下，所有的`AsyncTasks`都是按顺序在 Android 中执行的。要并行执行它，我们需要在执行器上执行它。

等等！现在，当我们并行执行任务时，想象一下你执行了一些任务。比如说我们从两个开始。这很好。它们将执行它们的操作并在完成时向我们报告。然后，想象一下我们同时运行了四个任务。它们也会工作，在大多数情况下，如果它们执行的操作不太繁重。然而，在某些时候，我们同时运行了五十个`AsyncTasks`。

然后，你的应用程序变慢了！一切都会变慢，因为对任务的执行没有控制。我们必须管理任务，以保持性能。所以，让我们来做吧！我们将继续更新到目前为止更新的同一个类。按照以下方式更改你的`NoteActivity`：

```kt
    class NoteActivity : ItemActivity() { 
      ... 
      private val threadPoolExecutor = ThreadPoolExecutor( 
            3, 3, 1, TimeUnit.SECONDS, LinkedBlockingQueue<Runnable>() 
    ) 

    private class TryAsync(val identifier: String) : AsyncTask<Unit,
    Int, Unit>() { 
        private val tag = "TryAsync" 

        override fun onPreExecute() { 
            Log.i(tag, "onPreExecute [ $identifier ]") 
            super.onPreExecute() 
      } 

      override fun doInBackground(vararg p0: Unit?): Unit { 
         Log.i(tag, "doInBackground [ $identifier ][ START ]") 
         Thread.sleep(5000) 
         Log.i(tag, "doInBackground [ $identifier ][ END ]") 
         return Unit 
       } 

       override fun onCancelled(result: Unit?) { 
         Log.i(tag, "onCancelled [ $identifier ][ END ]") 
         super.onCancelled(result) 
        } 

       override fun onProgressUpdate(vararg values: Int?) { 
         val progress = values.first() 
         progress?.let { 
           Log.i(tag, "onProgressUpdate [ $identifier ][ $progress ]") 
         } 
          super.onProgressUpdate(*values) 
        } 

        override fun onPostExecute(result: Unit?) { 
          Log.i(tag, "onPostExecute [ $identifier ]") 
          super.onPostExecute(result) 
        } 
      } 
      ... 
      private val textWatcher = object : TextWatcher { 
        override fun afterTextChanged(p0: Editable?) { 
            ... 
        } 

      override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2:
      Int, p3: Int) {} 

      override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int,
      p3: Int) { 
            p0?.let {  
                tryAsync(p0.toString())  
            } 
        } 
     } 
     ... 
     private fun tryAsync(identifier: String) { 
        val tryAsync = TryAsync(identifier) 
        tryAsync.executeOnExecutor(threadPoolExecutor) 
     } 
    } 
```

由于这实际上不是我们将在 Journaler 应用程序中保留的内容，请不要提交此代码。如果你愿意，可以将其创建为一个单独的分支。我们创建了一个`ThreadPoolExecutor`的新实例。构造函数接受几个参数，如下所示：

+   `corePoolSize`：这代表了池中保持的最小线程数。

+   `maximumPoolSize`：这代表了池中允许的最大线程数。

+   `keepAliveTime`：如果线程数大于核心数，非核心线程将等待新任务，如果在这个参数定义的时间内没有得到新任务，它们将终止。

+   `Unit`：这代表了`keepAliveTime`的时间单位。

+   `WorkQueue`：这代表了将用于保存任务的队列实例。

+   我们将在这个执行器上运行我们的任务。`AsyncTask`具体化将记录其生命周期中的所有事件。在`main`方法中，我们将等待 5 秒。运行应用程序，尝试添加一个标题为`Android`的新笔记。观察你的 Logcat 输出：

```kt
08-04 14:56:59.283 21953-21953 ... I/TryAsync: onPreExecute [ A ] 
08-04 14:56:59.284 21953-23233 ... I/TryAsync: doInBackground [ A ][ START ] 
08-04 14:57:00.202 21953-21953 ... I/TryAsync: onPreExecute [ An ] 
08-04 14:57:00.204 21953-23250 ... I/TryAsync: doInBackground [ An ][ START ] 
08-04 14:57:00.783 21953-21953 ... I/TryAsync: onPreExecute [ And ] 
08-04 14:57:00.784 21953-23281 ... I/TryAsync: doInBackground [ And ][ START ] 
08-04 14:57:01.001 21953-21953 ... I/TryAsync: onPreExecute [ Andr ] 
08-04 14:57:01.669 21953-21953 ... I/TryAsync: onPreExecute [ Andro ] 
08-04 14:57:01.934 21953-21953 ... I/TryAsync: onPreExecute [ Androi ] 
08-04 14:57:02.314 21953-2195 ... I/TryAsync: onPreExecute [ Android ] 
08-04 14:57:04.285 21953-23233 ... I/TryAsync: doInBackground [ A ][ END ] 
08-04 14:57:04.286 21953-23233 ... I/TryAsync: doInBackground [ Andr ][ START ] 
08-04 14:57:04.286 21953-21953 ... I/TryAsync: onPostExecute [ A ] 
08-04 14:57:05.204 21953-23250 ... I/TryAsync: doInBackground [ An ][ END ] 
08-04 14:57:05.204 21953-21953 ... I/TryAsync: onPostExecute [ An ] 
08-04 14:57:05.205 21953-23250 ... I/TryAsync: doInBackground [ Andro ][ START ] 
08-04 14:57:05.784 21953-23281 ... I/TryAsync: doInBackground [ And ][ END ] 
08-04 14:57:05.785 21953-23281 ... I/TryAsync: doInBackground [ Androi ][ START ] 
08-04 14:57:05.786 21953-21953 ... I/TryAsync: onPostExecute [ And ] 
08-04 14:57:09.286 21953-23233 ... I/TryAsync: doInBackground [ Andr ][ END ] 
08-04 14:57:09.287 21953-21953 ... I/TryAsync: onPostExecute [ Andr ] 
08-04 14:57:09.287 21953-23233 ... I/TryAsync: doInBackground [ Android ][ START ] 
08-04 14:57:10.205 21953-23250 ... I/TryAsync: doInBackground [ Andro ][ END ] 
08-04 14:57:10.206 21953-21953 ... I/TryAsync: onPostExecute [ Andro ] 
08-04 14:57:10.786 21953-23281 ... I/TryAsync: doInBackground [ Androi ][ END ] 
08-04 14:57:10.787 21953-2195 ... I/TryAsync: onPostExecute [ Androi ] 
08-04 14:57:14.288 21953-23233 ... I/TryAsync: doInBackground [ Android ][ END ] 
08-04 14:57:14.290 21953-2195 ... I/TryAsync: onPostExecute [ Android ] 
```

让我们通过我们在任务中执行的方法来过滤日志。首先让我们看一下`onPreExecute`方法的过滤器：

```kt
08-04 14:56:59.283 21953-21953 ... I/TryAsync: onPreExecute [ A ] 
08-04 14:57:00.202 21953-21953 ... I/TryAsync: onPreExecute [ An ] 
08-04 14:57:00.783 21953-21953 ... I/TryAsync: onPreExecute [ And ] 
08-04 14:57:01.001 21953-21953 ... I/TryAsync: onPreExecute [ Andr ] 
08-04 14:57:01.669 21953-21953 ... I/TryAsync: onPreExecute [ Andro ] 
08-04 14:57:01.934 21953-21953 ... I/TryAsync: onPreExecute [ Androi ] 
08-04 14:57:02.314 21953-21953 ... I/TryAsync: onPreExecute [ Android ] 
```

对每个方法都做同样的事情，并关注方法执行的时间。为了给你的代码更多的挑战，将`doInBackground()`方法的实现改为做一些更严肃和密集的工作。然后，通过输入一个更长的标题来触发更多的任务，例如整个句子。过滤和分析你的日志。

# 理解 Android Looper

让我们解释一下`Looper`类。我们在之前的例子中用过它，但我们没有详细解释过它。

`Looper`代表了一个用于在队列中执行`messages`或`runnable`实例的类。普通线程没有像`Looper`类那样的队列。

我们在哪里可以使用`Looper`类？对于执行多个`messages`或`runnable`实例，需要`Looper`！一个使用的例子可以是在添加新任务到队列的同时，任务处理操作正在运行。

# 准备 Looper

要使用`Looper`类，我们必须首先调用`prepare()`方法。当`Looper`准备好后，我们可以使用`loop()`方法。这个方法用于在当前线程中创建一个`message`循环。我们将给你一个简短的例子：

```kt
    class LooperHandler : Handler() { 
      override fun handleMessage(message: Message) { 
            ... 
      } 
    } 

    class LooperThread : Thread() { 
      var handler: Handler? = null 

      override fun run() { 
         Looper.prepare() 
         handler = LooperHandler() 
         Looper.loop() 
      } 
    } 
```

在这个例子中，我们演示了编程`Looper`类的基本步骤。不要忘记`prepare()`你的`Looper`类，否则你会得到一个异常，你的应用程序可能会崩溃！

# 延迟执行

本章还有一件重要的事情要向你展示。我们将向你展示在 Android 中的延迟执行。我们将给你一些延迟操作应用到我们的 UI 的例子。打开你的`ItemsFragment`并做出以下更改：

```kt
     class ItemsFragment : BaseFragment() { 
      ... 
       override fun onResume() { 
         super.onResume() 
         ... 
         val items = view?.findViewById<ListView>(R.id.items) 
         items?.let { 
            items.postDelayed({ 
              if (!activity.isFinishing) { 
                items.setBackgroundColor(R.color.grey_text_middle) 
              } 
            }, 3000) 
         } 
      } 
       ... 
     } 
```

三秒后，如果我们不关闭这个屏幕，背景颜色将变成稍微深一点的灰色。运行你的应用程序，亲自看看。现在，让我们用另一种方式做同样的事情：

```kt
     class ItemsFragment : BaseFragment() { 
      ... 
      override fun onResume() { 
        super.onResume() 
        ... 
        val items = view?.findViewById<ListView>(R.id.items) 
        items?.let { 
            Handler().postDelayed({ 
                if (!activity.isFinishing) { 
                    items.setBackgroundColor(R.color.grey_text_middle) 
                } 
            }, 3000) 
         } 
        } 
       } 
       ...
     }
```

这一次，我们使用了`Handler`类来执行延迟修改。

# 总结

在本章中，我们向您介绍了 Android 并发性。我们为每个部分进行了解释，并为您提供了示例。在深入了解 Android 服务之前，这是一个很好的介绍。Android 服务是 Android 提供的最强大的并发特性，正如您将看到的，它可以被用作应用程序的大脑。
