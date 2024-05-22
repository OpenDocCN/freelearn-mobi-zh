# 第十三章：调整以获得高性能

我们刚刚掌握了与后端和 API 的工作。我们接近旅程的尽头，但还没有结束！我们必须涵盖一些非常重要的要点！其中之一是性能优化。我们将指导你通过一些在实现这一目标时的良好实践。思考到目前为止我们已经开发的代码，以及如何应用这些建议。

在本章中，我们将涵盖以下主题：

+   布局优化

+   优化以保护电池寿命

+   优化以获得最大的响应性

# 优化布局

为了实现最佳的 UI 性能，请遵循以下几点：

+   **优化你的布局层次结构**：避免嵌套布局，因为它可能会影响性能！例如，你可以有多个嵌套的`LinearLayout`视图。与此相反，切换到`RelativeLayout`。这可以显著提高性能！嵌套布局需要更多的处理能力用于计算和绘制。

+   **尽可能重用布局**：Android 提供了`<include />`来实现这一点。

看一个例子：

```kt
    to_be_included.xml: 
    <RelativeLayout xmlns:android=
    "http://schemas.android.com/apk/res/android" 

      android:layout_width="match_parent" 
      android:layout_height="wrap_content" 
      android:background="@color/main_bg" 
      tools:showIn="@layout/includes" > 

      <TextView  
       android:id="@+id/title" 
       android:layout_width="wrap_content" 
       android:layout_height="wrap_content" 
     /> 

    </RelativeLayout>

    includes.xml 
      <LinearLayout xmlns:android=
      "http://schemas.android.com/apk/res/android" 
       android:orientation="vertical" 
       android:layout_width="match_parent" 
       android:layout_height="match_parent" 
       android:background="@color/main_bg" 
      > 
       ... 
      <include layout="@layout/to_be_included"/> 
      ... 
    </LinearLayout> 
```

+   此外，可以使用`<merge>`。当在一个布局中包含另一个布局时，合并可以消除视图层次结构中多余的`view groups`。让我们看一个例子：

```kt
    to_merge.xml 
    <merge > 

     <ImageView 
        android:id="@+id/first" 
        android:layout_width="fill_parent" 
        android:layout_height="wrap_content" 
        android:src="img/first"/> 

     <ImageView 
        android:id="@+id/second" 
        android:layout_width="fill_parent" 
        android:layout_height="wrap_content" 
        android:src="img/second"/> 

   </merge> 
```

当我们在另一个布局中使用 include 包含`to_merge.xml`时，就像我们在之前的例子中所做的那样，Android 会忽略`<merge>`元素，并直接将我们的视图添加到`<include />`所放置的容器中：

+   只在需要时将布局包含到屏幕中--如果你暂时不需要视图，将其可见性设置为`Gone`而不是`Invisible`。`Invisible`仍会创建视图的实例。使用`Gone`时，Android 只有在可见性更改为`Visible`时才会实例化视图。

+   使用`ListView`或`GridView`等容器来保存你的数据组。我们已经在前一章中解释了为什么你应该使用它们。

# 优化电池寿命

有很多方法可以耗尽你的电池。其中一个例子就是在应用程序中做太多的工作。过多的处理会影响电池寿命。然而，我们将指出你可以节省电池的方法以及你必须避免的事情。遵循这些要点，并在开发应用程序时时刻牢记。

为了保持电池处于最佳状态，请遵循以下建议：

+   尽量减少网络通信。频繁的网络调用会影响电池寿命。因此，尽量使其达到最佳状态。

+   确定你的手机是否在充电。这可能是启动应用程序可能需要执行的密集和性能要求的操作的好时机。

+   监控连接状态，并且只有在连接状态正常时执行与连接相关的操作。

+   合理利用广播消息。频繁和不必要地发送广播消息会影响性能。考虑发送频率，并在不需要接收消息时注销接收器。

+   注意 GPS 的使用强度。频繁的位置请求会显著影响电池寿命。

# 保持你的应用程序响应

你有多少次使用某个 Android 应用程序时收到应用程序无响应的消息？为什么会发生这种情况？我们会解释！注意以下几点，以免发生同样的情况：

+   确保没有任何东西阻塞你的输入（任何密集的操作，特别是网络流量）。

+   不要在主应用程序线程上执行长时间的任务。

+   不要在广播接收器的`onReceive()`方法中执行长时间运行的操作。

+   尽量使用`AsyncTask`类。考虑使用`ThreadPoolExecutor`。

+   尽可能使用内容加载器。

+   避免同时执行太多线程。

+   如果要写入文件系统，请使用单独的线程。

如果您的应用程序仍然出现 ANR，或者应用程序表现迟缓，请使用诸如 systrace 和 Traceview 之类的工具来跟踪问题的根源。

# 摘要

在这一简短但重要的章节中，我们强调了关于维护和实现良好应用性能和响应能力的重要要点。这些建议在应用程序优化中至关重要。因此，如果您的应用程序不遵循这些规则，您必须相应地进行优化。通过完成这一章，我们涵盖了您开发 Android 应用程序所需的一切。在下一章中，我们将对其进行测试。准备好编写一些单元测试和仪器测试！
