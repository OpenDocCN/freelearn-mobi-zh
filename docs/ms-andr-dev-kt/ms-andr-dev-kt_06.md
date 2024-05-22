# 第六章：权限

你好！你能相信这本书的一个重要部分已经在我们身后了吗？我们已经完成了用户界面，现在，我们正在进入这本书更复杂的部分——系统。

在本章以及接下来的章节中，我们将深入了解 Android 系统的结构。您将学习有关权限、数据库处理、首选项、并发、服务、消息传递、后端、API 和高性能的知识。

然而，不要被愚弄；这本书及其内容并未涵盖整个框架。那是不可能的；Android 是一个如此庞大的框架，完全掌握它可能需要数年时间。在这里，我们只是深入了解 Android 和 Kotlin 的世界。

然而，不要灰心！在这本书中，我们将为您提供掌握 Kotlin 和 Android 所需的知识和技能。在本章中，我们将讨论 Android 中的权限。您将学习权限是什么，它们用于什么，最重要的是，为什么我们需要（强调需要）使用它们。

在本章中，我们将涵盖以下主题：

+   来自 Android 清单的权限

+   请求权限

+   以 Kotlin 方式处理权限

# 来自 Android 清单的权限

Android 应用在它们自己的进程中运行，并且与操作系统的其余部分分离。因此，为了执行一些特定于系统的操作，需要请求它们。这样的权限请求的一个例子是请求使用蓝牙、检索当前 GPS 位置、发送短信，或者读取或写入文件系统。权限授予对各种设备功能的访问。处理权限有几种方法。我们将从使用清单开始。

首先，我们必须确定需要哪些权限。在安装过程中，用户可能决定不安装应用程序，因为权限太多。例如，用户可能会问为什么一个应用程序需要发送短信功能，当应用程序本身只是一个简单的图库应用程序。

对于我们在本书中开发的 Journaler 应用程序，我们将需要以下权限：

+   读取 GPS 坐标，因为我们希望我们创建的每个笔记都有相关联的坐标

+   我们需要访问互联网，这样我们就可以稍后执行 API 调用

+   启动完成事件，我们需要它，这样应用程序服务可以在每次重新启动手机时与后端进行同步

+   读取和写入外部存储，以便我们可以读取数据或存储数据

+   访问网络状态，以便我们知道是否有可用的互联网连接

+   使用振动，这样我们就可以在从后端接收到东西时振动

打开 `AndroidManifest.xml` 文件，并使用以下权限进行更新：

```kt
    <manifest xmlns:android=
     "http://schemas.android.com/apk/res/android" 
     package="com.journaler"> 

      <uses-permission android:name="android.permission.INTERNET" /> 
      <uses-permission android:name=
       "android.permission.RECEIVE_BOOT_COMPLETED" /> 
      <uses-permission android:name=
       "android.permission.READ_EXTERNAL_STORAGE" /> 
      <uses-permission android:name=
       "android.permission.WRITE_EXTERNAL_STORAGE" /> 
      <uses-permission android:name=
       "android.permission.ACCESS_NETWORK_STATE" /> 
      <uses-permission android:name=
       "android.permission.ACCESS_FINE_LOCATION" /> 
      <uses-permission android:name=
       "android.permission.ACCESS_COARSE_LOCATION" /> 
      <uses-permission android:name="android.permission.VIBRATE" /> 
       <application ... > 
         ... 
       </application 

       ... 

     </manifest>  
```

我们刚刚请求的权限的名称基本上是不言自明的，并且它们涵盖了我们提到的所有要点。除了这些权限，您还可以请求一些其他权限。看一下每个权限的名称，您会惊讶于您实际上可以请求到什么：

```kt
     <uses-permission android:name=
     "android.permission.ACCESS_CHECKIN_PROPERTIES" /> 
     <uses-permission  android:name=
     "android.permission.ACCESS_LOCATION_EXTRA_COMMANDS" /> 
     <uses-permission android:name=
     "android.permission.ACCESS_MOCK_LOCATION" /> 
     <uses-permission android:name=
     "android.permission.ACCESS_SURFACE_FLINGER" /> 
     <uses-permission android:name=
     "android.permission.ACCESS_WIFI_STATE" /> 
     <uses-permission android:name=
     "android.permission.ACCOUNT_MANAGER" /> 
     <uses-permission android:name=
     "android.permission.AUTHENTICATE_ACCOUNTS" /> 
     <uses-permission android:name=
     "android.permission.BATTERY_STATS" /> 
     <uses-permission android:name=
     "android.permission.BIND_APPWIDGET" /> 
     <uses-permission android:name=
     "android.permission.BIND_DEVICE_ADMIN" /> 
     <uses-permission android:name=
     "android.permission.BIND_INPUT_METHOD" /> 
     <uses-permission android:name=
     "android.permission.BIND_REMOTEVIEWS" /> 
     <uses-permission android:name=
     "android.permission.BIND_WALLPAPER" /> 
     <uses-permission android:name=
     "android.permission.BLUETOOTH" /> 
     <uses-permission android:name=
     "android.permission.BLUETOOTH_ADMIN" /> 
     <uses-permission android:name=
     "android.permission.BRICK" /> 
     <uses-permission android:name=
     "android.permission.BROADCAST_PACKAGE_REMOVED" /> 
     <uses-permission android:name=
     "android.permission.BROADCAST_SMS" /> 
     <uses-permission android:name=
     "android.permission.BROADCAST_STICKY" /> 
     <uses-permission android:name=
      "android.permission.BROADCAST_WAP_PUSH" /> 
     <uses-permission android:name=
      "android.permission.CALL_PHONE"/> 
     <uses-permission android:name=
      "android.permission.CALL_PRIVILEGED" /> 
     <uses-permission android:name=
      "android.permission.CAMERA"/> 
     <uses-permission android:name=
      "android.permission.CHANGE_COMPONENT_ENABLED_STATE" /> 
     <uses-permission android:name=
     "android.permission.CHANGE_CONFIGURATION" /> 
     <uses-permission android:name=
     "android.permission.CHANGE_NETWORK_STATE" /> 
     <uses-permission android:name=
     "android.permission.CHANGE_WIFI_MULTICAST_STATE" /> 
     <uses-permission android:name=
     "android.permission.CHANGE_WIFI_STATE" /> 
     <uses-permission android:name=
     "android.permission.CLEAR_APP_CACHE" /> 
     <uses-permission android:name=
     "android.permission.CLEAR_APP_USER_DATA" /> 
     <uses-permission android:name=
     "android.permission.CONTROL_LOCATION_UPDATES" /> 
     <uses-permission android:name=
     "android.permission.DELETE_CACHE_FILES" /> 
     <uses-permission android:name=
     "android.permission.DELETE_PACKAGES" /> 
     <uses-permission android:name=
     "android.permission.DEVICE_POWER" /> 
     <uses-permission android:name=
     "android.permission.DIAGNOSTIC" /> 
     <uses-permission android:name=
     "android.permission.DISABLE_KEYGUARD" /> 
     <uses-permission android:name=
     "android.permission.DUMP" /> 
     <uses-permission android:name=
     "android.permission.EXPAND_STATUS_BAR" /> 
     <uses-permission android:name="
     android.permission.FACTORY_TEST" /> 
     <uses-permission android:name=
     "android.permission.FLASHLIGHT" /> 
     <uses-permission android:name=
     "android.permission.FORCE_BACK" /> 
     <uses-permission android:name=
     "android.permission.GET_ACCOUNTS" /> 
     <uses-permission android:name=
     "android.permission.GET_PACKAGE_SIZE" /> 
     <uses-permission android:name=
     "android.permission.GET_TASKS" /> 
     <uses-permission android:name=
     "android.permission.GLOBAL_SEARCH" /> 
     <uses-permission android:name=
     "android.permission.HARDWARE_TEST" /> 
     <uses-permission android:name=
     "android.permission.INJECT_EVENTS" /> 
     <uses-permission android:name=
     "android.permission.INSTALL_LOCATION_PROVIDER" /> 
     <uses-permission android:name=
     "android.permission.INSTALL_PACKAGES" /> 
     <uses-permission android:name=
     "android.permission.INTERNAL_SYSTEM_WINDOW" /> 
     <uses-permission android:name=
     "android.permission.KILL_BACKGROUND_PROCESSES" /> 
     <uses-permission android:name=
     "android.permission.MANAGE_ACCOUNTS" /> 
     <uses-permission android:name=
     "android.permission.MANAGE_APP_TOKENS" /> 
     <uses-permission android:name=
     "android.permission.MASTER_CLEAR" /> 
     <uses-permission android:name=
     "android.permission.MODIFY_AUDIO_SETTINGS" /> 
     <uses-permission android:name=
     "android.permission.MODIFY_PHONE_STATE" /> 
     <uses-permission android:name=
     "android.permission.MOUNT_FORMAT_FILESYSTEMS" /> 
     <uses-permission android:name=
     "android.permission.MOUNT_UNMOUNT_FILESYSTEMS" /> 
     <uses-permission android:name=
     "android.permission.NFC" /> 
     <uses-permission android:name=
     "android.permission.PROCESS_OUTGOING_CALLS" /> 
     <uses-permission android:name=
     "android.permission.READ_CALENDAR" /> 
    <uses-permission android:name=
     "android.permission.READ_CONTACTS" /> 
    <uses-permission android:name=
    "android.permission.READ_FRAME_BUFFER" /> 
    <uses-permission android:name=
    "android.permission.READ_HISTORY_BOOKMARKS" /> 
    <uses-permission android:name=
    "android.permission.READ_INPUT_STATE" /> 
    <uses-permission android:name=
    "android.permission.READ_LOGS" /> 
    <uses-permission android:name=
    "android.permission.READ_PHONE_STATE" /> 
    <uses-permission android:name=
    "android.permission.READ_SMS" /> 
    <uses-permission android:name=
    "android.permission.READ_SYNC_SETTINGS" /> 
    <uses-permission android:name=
    "android.permission.READ_SYNC_STATS" /> 
    <uses-permission android:name=
    "android.permission.REBOOT" /> 
    <uses-permission android:name=
    "android.permission.RECEIVE_MMS" /> 
    <uses-permission android:name=
    "android.permission.RECEIVE_SMS" /> 
    <uses-permission android:name=
    "android.permission.RECEIVE_WAP_PUSH" /> 
    <uses-permission android:name=
    "android.permission.RECORD_AUDIO" /> 
    <uses-permission android:name=
    "android.permission.REORDER_TASKS" /> 
    <uses-permission android:name=
    "android.permission.RESTART_PACKAGES" /> 
    <uses-permission android:name=
    "android.permission.SEND_SMS" /> 
    <uses-permission android:name=
    "android.permission.SET_ACTIVITY_WATCHER" /> 
    <uses-permission android:name=
     "android.permission.SET_ALARM" /> 
    <uses-permission android:name=
     "android.permission.SET_ALWAYS_FINISH" /> 
    <uses-permission android:name=
     "android.permission.SET_ANIMATION_SCALE" /> 
    <uses-permission android:name=
     "android.permission.SET_DEBUG_APP" /> 
    <uses-permission android:name=
     "android.permission.SET_ORIENTATION" /> 
    <uses-permission android:name=
     "android.permission.SET_POINTER_SPEED" /> 
    <uses-permission android:name=
     "android.permission.SET_PROCESS_LIMIT" /> 
    <uses-permission android:name=
     "android.permission.SET_TIME" /> 
    <uses-permission android:name=
     "android.permission.SET_TIME_ZONE" /> 
    <uses-permission android:name=
     "android.permission.SET_WALLPAPER" /> 
    <uses-permission android:name=
     "android.permission.SET_WALLPAPER_HINTS" /> 
    <uses-permission android:name=
     "android.permission.SIGNAL_PERSISTENT_PROCESSES" /> 
    <uses-permission android:name=
     "android.permission.STATUS_BAR" /> 
    <uses-permission android:name=
     "android.permission.SUBSCRIBED_FEEDS_READ" /> 
    <uses-permission android:name=
     "android.permission.SUBSCRIBED_FEEDS_WRITE" /> 
    <uses-permission android:name=
     "android.permission.SYSTEM_ALERT_WINDOW" /> 
    <uses-permission android:name=
     "android.permission.UPDATE_DEVICE_STATS" /> 
    <uses-permission android:name=
     "android.permission.USE_CREDENTIALS" /> 
    <uses-permission android:name=
     "android.permission.USE_SIP" /> 
    <uses-permission android:name=
     "android.permission.WAKE_LOCK" /> 
    <uses-permission android:name=
     "android.permission.WRITE_APN_SETTINGS" /> 
    <uses-permission android:name=
     "android.permission.WRITE_CALENDAR" /> 
    <uses-permission android:name=
     "android.permission.WRITE_CONTACTS" /> 
    <uses-permission android:name=
     "android.permission.WRITE_GSERVICES" /> 
    <uses-permission android:name=
     "android.permission.WRITE_HISTORY_BOOKMARKS" /> 
    <uses-permission android:name=
     "android.permission.WRITE_SECURE_SETTINGS" /> 
    <uses-permission android:name=
     "android.permission.WRITE_SETTINGS" /> 
    <uses-permission android:name=
     "android.permission.WRITE_SMS" /> 
    <uses-permission android:name=
     "android.permission.WRITE_SYNC_SETTINGS" /> 
    <uses-permission android:name=
     "android.permission.BIND_ACCESSIBILITY_SERVICE"/> 
    <uses-permission android:name=
     "android.permission.BIND_TEXT_SERVICE"/> 
    <uses-permission android:name=
     "android.permission.BIND_VPN_SERVICE"/> 
    <uses-permission android:name=
     "android.permission.PERSISTENT_ACTIVITY"/> 
    <uses-permission android:name=
     "android.permission.READ_CALL_LOG"/> 
    <uses-permission android:name=
     "com.android.browser.permission.READ_HISTORY_BOOKMARKS"/> 
    <uses-permission android:name=
     "android.permission.READ_PROFILE"/> 
    <uses-permission android:name=
     "android.permission.READ_SOCIAL_STREAM"/> 
    <uses-permission android:name=
     "android.permission.READ_USER_DICTIONARY"/> 
    <uses-permission android:name=
     "com.android.alarm.permission.SET_ALARM"/> 
    <uses-permission android:name=
     "android.permission.SET_PREFERRED_APPLICATIONS"/> 
    <uses-permission android:name=
     "android.permission.WRITE_CALL_LOG"/> 
    <uses-permission android:name=
     "com.android.browser.permission.WRITE_HISTORY_BOOKMARKS"/> 
    <uses-permission android:name=
     "android.permission.WRITE_PROFILE"/> 
    <uses-permission android:name=
     "android.permission.WRITE_SOCIAL_STREAM"/> 
    <uses-permission android:name=
     "android.permission.WRITE_USER_DICTIONARY"/>  
```

# 请求权限

在 Android SDK 版本 23 之后，需要在运行时请求权限（并非所有权限）。这意味着我们也需要从代码中请求它们。我们将演示如何从我们的应用程序中执行此操作。我们将在用户打开应用程序时请求获取 GPS 位置所需的权限。如果没有获得批准，用户将收到一个对话框以批准权限。打开您的 `BaseActivity` 类，并将其扩展如下：

```kt
    abstract class BaseActivity : AppCompatActivity() {
      companion object { 
      val REQUEST_GPS = 0 
      ... }
      ... 
      override fun onCreate(savedInstanceState: Bundle?) {   
        super.onCreate(savedInstanceState)
        ...
        requestGpsPermissions() } 
     ...
     private fun requestGpsPermissions() {   
       ActivityCompat.requestPermissions( 
         this@BaseActivity,
         arrayOf( 
           Manifest.permission.ACCESS_FINE_LOCATION,
           Manifest.permission.ACCESS_COARSE_LOCATION ),
           REQUEST_GPS ) }
            ... 
      override fun onRequestPermissionsResult(
        requestCode:
         Int, permissions: Array<String>, grantResults: IntArray ) {
           if (requestCode == REQUEST_GPS) { 
            for (grantResult in grantResults) 
            { if (grantResult == PackageManager.PERMISSION_GRANTED)
             { Log.i( tag, String.format( Locale.ENGLISH, "Permission 
              granted [ %d ]", requestCode ) ) 
             } 
             else {
               Log.e( tag, String.format( Locale.ENGLISH, "Permission
               not granted [ %d ]", requestCode ) )
             } } } } }

```

那么这段代码到底是在做什么呢？我们将从上到下解释所有行。

在`companion`对象中，我们定义了我们请求的 ID。我们将等待该 ID 的结果。在`onCreate()`方法中，我们调用了`requestGpsPermissions()`方法，实际上是在我们定义的 ID 下进行权限请求。权限请求的结果将在`onRequestPermissionsResult()`重写方法中可用。如你所见，我们正在记录权限请求的结果。应用现在可以检索 GPS 数据。

对于所有其他安卓权限，原则是相同的。构建你的应用并运行它。将会询问你权限，如下截图所示：

![](img/ae6af33a-4ea1-4a16-ab34-4801233b8808.png)

# 用 Kotlin 的方式来做

如果我们的应用程序需要通过代码处理很多权限，会发生什么？这意味着我们有很多处理不同权限请求的代码。幸运的是，我们正在使用 Kotlin。Kotlin 将是我们简化事情的工具！

创建一个名为`permission`的新包。然后创建两个新的 Kotlin 文件如下：

`PermissionCompatActivity`和`PermissionRequestCallback`。

让我们定义权限请求回调如下：

```kt
     package com.journaler.permission 

     interface PermissionRequestCallback { 
       fun onPermissionGranted(permissions: List<String>) 
       fun onPermissionDenied(permissions: List<String>) 
     } 
```

这将是在解决权限时触发的`callback`。然后，定义我们的权限`compat`活动：

```kt
     package com.journaler.permission 

     import android.content.pm.PackageManager 
     import android.support.v4.app.ActivityCompat 
     import android.support.v7.app.AppCompatActivity 
     import android.util.Log 
     import java.util.concurrent.ConcurrentHashMap 
     import java.util.concurrent.atomic.AtomicInteger 

     abstract class PermissionCompatActivity : AppCompatActivity() { 

       private val tag = "Permissions extension" 
       private val latestPermissionRequest = AtomicInteger() 
       private val permissionRequests = ConcurrentHashMap<Int,
       List<String>>() 
       private val permissionCallbacks =  
        ConcurrentHashMap<List<String>, PermissionRequestCallback>() 

       private val defaultPermissionCallback = object :  
       PermissionRequestCallback { 
         override fun onPermissionGranted(permissions: List<String>) { 
            Log.i(tag, "Permission granted [ $permissions ]") 
         } 
         override fun onPermissionDenied(permissions: List<String>) { 
            Log.e(tag, "Permission denied [ $permissions ]") 
         } 
      } 

     fun requestPermissions( 
        vararg permissions: String,  
        callback: PermissionRequestCallback = defaultPermissionCallback 
     ) { 
        val id = latestPermissionRequest.incrementAndGet() 
        val items = mutableListOf<String>() 
        items.addAll(permissions) 
        permissionRequests[id] = items 
        permissionCallbacks[items] = callback 
        ActivityCompat.requestPermissions(this, permissions, id) 
     } 

     override fun onRequestPermissionsResult( 
        requestCode: Int,  
        permissions: Array<String>,  
        grantResults: IntArray 
     ) { 
        val items = permissionRequests[requestCode] 
        items?.let { 
           val callback = permissionCallbacks[items] 
           callback?.let { 
             var success = true 
              for (x in 0..grantResults.lastIndex) { 
                  val result = grantResults[x] 
                  if (result != PackageManager.PERMISSION_GRANTED) { 
                      success = false 
                      break 
                  } 
              } 
              if (success) { 
                 callback.onPermissionGranted(items) 
              } else { 
                  callback.onPermissionDenied(items) 
              } 
             } 
           } 
         } 
     }
```

这个类的理念是--我们向终端用户公开了`requestPermissions()`方法，该方法接受表示我们感兴趣的权限的可变数量的参数。我们可以传递（我们刚刚定义的）可选的`callback`（接口）。如果我们不传递自己的`callback`，将使用默认的`callback`。在权限解决后，我们触发`callback`。只有当所有权限都被授予时，我们才认为权限解决成功。

让我们更新我们的`BaseActivity`类如下：

```kt
     abstract class BaseActivity : PermissionCompatActivity() { 
     ... 
     override fun onCreate(savedInstanceState: Bundle?) { 
         ... 
         requestPermissions( 
            Manifest.permission.ACCESS_FINE_LOCATION, 
            Manifest.permission.ACCESS_COARSE_LOCATION 
         ) 
     } 
     ... 
    } 
```

如你所见，我们从`BaseActivity`类中删除了所有先前与权限相关的代码，并用一个`requestPermission()`调用替换了它。

# 总结

本章可能很短，但你学到的信息非常宝贵。每个安卓应用都需要权限。它们存在是为了保护用户和开发者。正如你所见，根据你的需求，有很多不同的权限可以使用。

在下一章中，我们将继续讲解系统部分，你将学习数据库处理。
