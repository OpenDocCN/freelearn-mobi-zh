# 第四章：Android Wear

本章将向您介绍 Android Wear 以及它如何作为手表和其他设备实现的现象。

在本章中，您将学习以下内容：

+   可穿戴设备

+   全屏可穿戴应用

+   表盘

+   通知

# 可穿戴设备

Android Wear 是许多可穿戴设备运行的系统。您可能自己有一块智能手表。可穿戴设备会成为继手机、平板电脑之后的下一个热潮吗？还是智能手表会成为更大事物的一部分，比如**物联网**（**IoT**）？

Android Wear 是 Android SDK 的一个特殊版本，专门用于通常在硬件和可用传感器方面更受限制、屏幕更小的可穿戴设备。可穿戴设备可能出现为手表、眼镜，或者将来可能会出现为隐形眼镜、纹身或服装。

目前，我们看到可穿戴设备主要出现在手表上，但您可以想到还有许多其他可穿戴设备。然而，人们需要一些时间来接受这项新技术。例如，想想谷歌眼镜项目。这是一个很棒的解决方案，但主要是因为内置摄像头，人们对它有严重的反对意见。在旧金山，他们甚至为此创造了一个词：glass hole。嗯。这真的不太讨人喜欢，对吧？让我们看看以下设备：

![可穿戴设备](img/B04299_04_01.jpg)

设备不一定要是可穿戴的。当讨论 IOT 时，项目 Brillo 就会浮现在脑海中。它将 Android 平台扩展到您能想到的任何连接设备上。未来，Brillo 和 Android Wear 甚至可能会合并。

想象一个炎热的夏日；冰箱通知我们即将用完气泡水（还是啤酒？）。酷！学习型恒温器在您回家前一小时将温度设定为 18°C。更酷！客厅的灯光自动调暗，因为现在是晚上；您正在播放一些浪漫的音乐，系统知道您刚刚打开了一瓶葡萄酒-嗯。奇怪。这是一个完全不同的故事，Brillo 现在也是如此。

相反，让我们找出我们可以为智能手表构建哪些应用，比如全新的表盘或健康应用程序，不时显示通知。在接下来的步骤中，我们将看到为此需要做些什么。

首先，让我们看看我们是否可以在可穿戴设备上运行起来。在前两个步骤中，您不需要拥有真正的智能手表。我们将在第一个步骤中创建一个虚拟的智能手表。

# 全屏可穿戴应用

可穿戴全屏应用程序确实有手机（或其他手持设备）和可穿戴组件。用户在手机上安装手持应用程序，可穿戴组件会自动推送到配对的可穿戴设备上。

这是探索为可穿戴设备开发应用程序的有趣世界的一个很好的开始，因为它们基本上与 Android 手机应用程序相同。然而，谷歌鼓励您将应用程序与 Android Wear 的上下文流集成在一起。这个上下文流包含各种有趣的信息。可以将它们视为收件箱中的新邮件、天气、今天走的步数或心率。我们将在有关通知的食谱中了解更多信息。

## 准备就绪

要完成这个食谱，您需要确保 Android Studio 已经运行起来。还要确保您已安装了最新的 SDK，包括 Android Wear SDK。当您打开 SDK 管理器时，可以检查是否已经安装了这些（导航到**工具**菜单，**Android SDK 管理器**），如下截图所示： 

![准备就绪](img/B04299_04_02.jpg)

## 如何做...

让我们看看如何创建我们自己的可穿戴应用，并通过以下步骤在虚拟设备上运行它：

1.  开始一个新的 Android Studio 项目。将应用命名为`WatchApp`，并在**公司域**字段中输入`packtpub.com`。然后，点击**下一步**按钮。

1.  在下一个对话框中，勾选**手机和平板电脑**。还要勾选**可穿戴设备**选项。

1.  对于这两个选项，选择**API 21**或更高版本，然后点击**下一步**按钮。

1.  在**添加到 wear 的活动**对话框中，选择**空白 wear 活动**，然后点击**下一步**按钮。

1.  选择**空白活动**，然后点击**下一步**按钮。

1.  将您的新活动命名为`PhoneActivity`，然后点击**下一步**按钮。

1.  选择**空白 wear 活动**，然后点击**下一步**按钮，如下一个截图所示：![如何做...](img/B04299_04_03.jpg)

1.  将您的新 wear 活动命名为`WatchActivity`，然后点击**完成**按钮。

1.  Android Studio 将创建两个模块：`mobile`和`wear`。移动模块在智能手机（或平板电脑或平板电脑）上运行。wear 应用程序将被推送到配对的可穿戴设备，例如您的智能手表。项目视图现在看起来像这样：![如何做...](img/B04299_04_04.jpg)

1.  让我们看看默认情况下它在智能手机上的样子。为此，我们将创建一个可穿戴虚拟设备。从**工具**菜单中，选择**Android**选项，然后选择**AVD Manager**选项。

1.  然后，点击**创建虚拟设备**按钮。

1.  在弹出的对话框中，在**类别**列表中选择**Wear**。在旁边的列表中选择**Android Wear Round**设备，然后点击**下一步**按钮，如下一个截图所示：![如何做...](img/B04299_04_05.jpg)

1.  在下一个对话框中，选择一个系统镜像，例如**棒棒糖**，**API 级别 21**，**x86**（如果可用，也可以选择更高级别。您可能需要先点击**下载**链接）。然后，点击**下一步**按钮继续。

1.  给您的虚拟设备起一个好听的名字，然后点击**完成**按钮。您的新 Android wear 设备现在将出现在列表中，如下一个截图所示：![如何做...](img/B04299_04_06.jpg)

1.  点击播放图标启动设备。

1.  虚拟设备启动后，将配置更改为**wear**，然后点击工具栏旁边的**运行**按钮。![如何做...](img/B04299_04_07.jpg)

应用程序安装完成后，将如下所示：

![如何做...](img/B04299_04_08.jpg)

如果**Hello Round World!**消息没有立即出现，那么该应用程序可能已安装，但可能尚不可见。多次滑动屏幕以检查是否存在。

如果您的应用程序已经运行，那么现在是时候探索更有趣的东西了。让我们在下一个教程中创建一个表盘。

## 还有更多...

在撰写本文时，Genymotion 尚不支持可穿戴设备。这就是为什么在本教程中我们使用默认的模拟器。

但那个太慢了！您可能会说。这是真的，但通过安装 HAXM，您可以使它快一点。关于这个主题有一些有趣的信息，可以在[`developer.android.com/tools/devices/emulator.html`](http://developer.android.com/tools/devices/emulator.html)找到。

如果您确实有真实设备，当然也可以在智能手表上部署您的应用程序。如果要这样做，您还需要在手持设备上安装 Android wear 配套应用程序，因为您无法直接在其上安装和测试可穿戴应用程序。

您可以从 Google Play 获取这个配套应用。下载应用程序，安装它，并通过 USB 连接您的手持设备。

## 另请参阅

+   参考第一章中的*使用名为 Genymotion 的模拟器测试您的应用程序*部分，*欢迎使用 Android Studio*

# 表盘

您的 Android 智能手表默认配备了各种表盘，还有许多其他表盘可供下载。它们以任何形状或类型提供：方形和圆形，模拟和数字。实际上，甚至还有另一种形状 - 所谓的平坦轮胎形状 - 就像 Moto 360 设备上看到的那样。

有许多自定义选项，但所有的表盘都是为了以简单的方式显示时间和日期信息。这首先是手表的用途，不是吗？

他们应该注意到即将到来的通知，还需要为系统指示器腾出空间，例如电池寿命图标和**Ok Google**文本。有关更多信息，请访问[`developer.android.com/design/wear/watchfaces.html`](https://developer.android.com/design/wear/watchfaces.html)。

在即将创建的示例中，我们将创建一个手表表盘，告诉你时间，例如**七点半**或**十点五分钟**。

## 准备工作

要完成本示例，您需要运行 Android Studio。还要确保已安装了最新的 SDK，包括 Android Wear SDK。您可以通过打开 SDK 管理器来检查是否已安装，该管理器可在**工具**菜单下的**Android**中找到，该菜单项位于**SDK Manager**下。

## 操作步骤

让我们按以下步骤创建一个手表表盘应用的新 Android 项目：

1.  创建一个新的 Android Studio 项目。

1.  将应用命名为`HelloTime`，或者您想要的应用名称。在**公司域**字段中输入`packtpub.com`，然后单击**下一步**按钮。

1.  在下一个对话框中，勾选**手机和平板**。还要勾选**Wear**选项。

1.  对于这两个选项，选择**API 21**或更高版本，然后单击**下一步**按钮。

1.  选择**空白活动**，并单击**下一步**按钮。

1.  将新的活动命名为`PhoneActivity`，并单击**下一步**按钮。

1.  选择**表盘**，并单击**下一步**按钮。

1.  将表盘命名为`HelloTimeWatchFace`，并选择**数字**作为**样式**。之后，点击**完成**按钮。

1.  Android Studio 将为手机或平板和可穿戴设备创建必要的模块。

1.  在项目视图中，打开`wear`模块的`HelloTimeWatchFace`类。

1.  在`wear`模块的`res/values`文件夹中打开`strings.xml`文件，并将`my_digital_name`的字符串更改为`Hello Time!`

1.  让我们看看我们到目前为止得到了什么。启动虚拟（或真实的）可穿戴设备。如果你不知道如何创建虚拟可穿戴设备，请参考上一个示例。

1.  虚拟设备启动后，将配置更改为**Wear**，并单击工具栏旁边的**运行**按钮，如下图所示：![操作步骤…](img/B04299_04_09.jpg)

1.  在可穿戴设备上，滑动查看**设置**图标并点击它。

1.  向下滑动到**更改表盘**，并点击它。

1.  向右滑动，直到看到**Hello Time!**表盘，然后点击它。

1.  您现在将看到 Android Studio 为您创建的数字表盘。

让我们稍微检查一下这段代码。为你创建的`HelloTimeWatchFace`类扩展了`CanvasWatchFaceService`，并添加了一个内部的`Engine`类。引擎有一个处理程序，以便可以更新时间。它还有一个广播接收器，将处理用户在旅行时移动到另一个时区的情况。

`Engine`类有一些有趣的方法。`onCreate`方法分配了两个`Paint`对象：一个用于背景，一个用于前景（文本）。`onVisibilityChanged`方法将在用户显示或隐藏表盘时调用。`onApplyWindowInSets`方法用于确定应用是否在圆形或方形屏幕上运行。

接下来是`onPropertiesChanged`方法，一旦可穿戴设备的硬件属性已知，例如是否支持低位环境模式，就会调用该方法。`onAmbientModeChanged`方法非常重要，因为它可以节省电池。它还可以用于应用防烧屏保护。在这里，您可能想要更改背景或前景的颜色。

让我们改变时间的显示方式：

1.  添加一个以口语语言返回当前时间的方法，类似于这样：

```kt
private String[] getFullTextTime(){
    String time = "";Calendar cal = Calendar.getInstance();
   int minute = cal.get(Calendar.MINUTE);
   int hour = cal.get(Calendar.HOUR);
   if (minute<=7){
        time = String.format("%s o'clock",   getTextDigit(hour));
    }
   else if (minute<=15){
        time = String.format("ten past %s",    getTextDigit(hour));
    }
   else if (minute<=25){
       time = String.format("Quarter past %s", getTextDigit(hour));
    }
   else if (minute<=40){
       time = String.format("Half past %s", getTextDigit(hour));
   }
  else if (minute<53){
       time = String.format("Quarter to %s",  
       getTextDigit(hour));
  }
  else {
       time = String.format("Almost %d o'clock", (hour<=11)? hour+1: 1);
  }
  return time.split(" ");
}
```

1.  添加此方法以将数字转换为文本：

```kt
private String getTextDigit(int digit){
    String[] texts ={ "twelve", "one", "two", "three",  
     "four", "five", "six", "seven", "eight", "nine",       
       "eleven"};
     return texts[digit];
```

1.  在`onDraw`方法中，用这里显示的行替换`canvas.DrawText`部分。此方法显示口语语言中当前时间的多行：

```kt
String[] timeTextArray = getFullTextTime();
float y = mYOffset;
for (String timeText : timeTextArray){
    canvas.drawText(timeText, mXOffset, y, mTextPaint);
    y+=65;
}
```

### 注意

**魔术并不总是很酷...**

等等！在上一步中那个魔术数字是在做什么？65 实际上并没有意义。这是什么意思？它是做什么的？在您的类中的某个地方创建一个常量值，并使用该变量名称（在这里最好将值放在尺寸资源文件中，但我们稍后会看到这一点，所以现在让我们暂时忘记它）：

```kt
private static final int ROW_HEIGHT  = 65;
y+= ROW_HEIGHT;
```

1.  转到`onCreate`方法，并添加此行以使文本以漂亮的绿色显示（是的，`GREEN`也是一个常量）：

```kt
mTextPaint.setColor(Color.GREEN);
```

再次运行您的应用程序。它会看起来像这样：

![如何做...](img/B04299_04_10.jpg)

为了以后准备好 Play 商店的手表表盘，您需要在完成后拍摄屏幕截图。您需要为方形和圆形手表提供屏幕截图。在`res/drawable`文件夹中，您会找到 Android Studio 为您创建的默认预览图像。

目前，您只是以最基本的形式创建了您的第一个手表表盘应用程序。在下一个食谱中，我们将看到通知到来时会发生什么。

## 还有更多...

本食谱中的手表表盘应用程序远非完美。文本未对齐；它没有正确响应环境模式的更改，您可能希望将其本地化以以您自己的语言显示时间。

要了解这可能会发展成什么样，您可以查看 Play 商店中已经可用的许多手表表盘。

# 通知

Android Wear 与在手机或平板上运行的应用程序有所不同。 Android Wear 使用卡片，而不是图标和列表，这是我们在介绍材料设计基本概念的食谱中已经看到的东西。

根据上下文，并且只在相关时刻，一旦新通知到达，就会向卡片流中添加一张卡片。这被称为上下文流，其中包含各种有趣的信息。将它们视为收件箱中的电子邮件，天气，今天走的步数，心率，以及其他事件或提醒。

还记得上一章的饮水应用程序吗？例如，我们可以创建一个提醒我们更频繁饮水并为其添加新卡片的通知。这将是一个很好的功能。

## 准备工作

此食谱需要安装 Android Studio 和最新的 SDK，包括 wear SDK。有关更多信息，请查看上一个食谱。

您还需要一台运行 Android Lollipop 或更高版本的手持设备，该设备已安装`Android Wear`应用程序，并且通过蓝牙连接到您的手持设备的可穿戴设备。

## 如何做...

让我们看看如何触发通知以及如何在智能手表上漂亮地显示它们：

1.  在 Android Studio 中创建一个新项目。将其命名为`WaterNowNotification`，然后单击**下一步**按钮。

1.  选择**手机和平板电脑**作为智能手表平台。不要选择**Wear**选项。然后单击**下一步**按钮。

1.  选择**空白活动**，然后单击**下一步**按钮。

1.  将您的活动命名为`WaterNowActivity`，然后单击**完成**按钮。

1.  在您的应用中打开`build.gradle`文件。将其添加到依赖项部分并应用适当的版本：

```kt
compile 'com.android.support:support-v4:22.0+'
```

1.  单击工具栏上可以找到的**与 Gradle 文件同步项目**按钮。

1.  打开`activity_water_now.xml`文件，并使用 Android Studio 底部的选项卡将其更改为**文本**模式。

1.  创建一个带有按钮的布局，我们将用它来发送测试通知：

```kt
<LinearLayout

android:layout_width="match_parent"
android:layout_height="match_parent"
android:orientation="vertical"
tools:context=".WaterNowActivity">
<Button
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:text="Drink water now!"
android:id="@+id/water_now_button"
android:layout_gravity="center" />
</LinearLayout>
```

1.  在`WaterNowActivity`类的`onCreate`方法中，添加一个`onClick`处理程序，用于刚刚创建的按钮。根据需要使用*Alt* + *Enter*快捷键添加导入语句：

```kt
Button waterNowButton = (Button)findViewById(R.id.water_now_button);
waterNowButton.setOnClickListener(new View.OnClickListener() {
@Override
        public void onClick(View v) {
        sendNotification();   }
});
```

1.  创建`sendNotification`方法：

```kt
private void sendNotification(){
    NotificationCompat.Builder notificationBuilder =
    new NotificationCompat.Builder(   
      WaterNowActivity.this)
      .setContentTitle("Water app!")
      .setSmallIcon(R.drawable.icon)
      .setContentText("Hey there! Drink water now!");
    NotificationManagerCompat notificationManager =NotificationManagerCompat.from(  
      WaterNowActivity.this);
    notificationManager.notify(1 ,   
     notificationBuilder.build());
}
```

1.  通知确实需要一个图标，所以在`res/drawable`文件夹中创建一个。创建一个 drawable `icon.xml`文件，并添加实现以创建一个漂亮的蓝色圆圈：

```kt
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android= "http://schemas.android.com/apk/res/android"android:shape="oval">
<corners android:radius="10dip"/>
<stroke android:color="#0000FF" android:width="15dip"/>
<solid android:color="#0000FF"/>
</shape>
```

1.  连接你的手持设备；确保可穿戴设备已连接（使用`Android wear`应用来检查），然后运行应用。你会看到类似以下截图的输出：![操作步骤...](img/B04299_04_11.jpg)

1.  点击应用内的**现在喝水**按钮。

1.  手机上会显示类似以下截图的通知。如果通知没有立刻出现，屏幕顶部会有一些指示。在这种情况下，打开通知中心查看。![操作步骤...](img/B04299_04_12.jpg)

1.  如果一切正常并且配置正确，同样的通知会出现在可穿戴设备上，如下所示：![操作步骤...](img/B04299_04_13.jpg)

1.  如果通知在你的手机上显示，但在你的可穿戴设备上没有出现，那么请验证**通知访问**设置。打开**设置**应用，选择**声音和消息**。接下来，选择**通知访问**，并检查**Android Wear**选项是否已被选中。

对于其他 Android 版本或特定品牌（定制的 Android 版本），你要找的设置可能在其他地方，或者可能有不同的名称。

## 还有更多...

接下来怎么办？你可以将这个通知配方与第三章中的 Water 应用配方相结合，创造出更酷的东西，或者你可以检查是否可以找到一种自定义通知的方法。

智能手表、手机、平板手机和平板电脑都配备了各种尺寸和形状的屏幕。我们如何从更大的屏幕中受益，或者如何为较小的屏幕提供智能导航，并在一个应用中保持相同的功能和代码？

不同 Android 版本的不同布局？多个布局与多个片段的结合正是我们需要的。这就是下一章中的配方发挥作用的地方。

## 另请参阅

+   参考第三章中的*RecyclerView*和*CardView*部分，*材料设计*

+   参考第五章, *尺寸很重要*
