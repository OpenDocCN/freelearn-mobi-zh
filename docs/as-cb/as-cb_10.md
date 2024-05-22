# 第十章：测试您的应用程序

您已经尽力确保应用程序的质量和性能。现在是时候将应用程序发布到测试版用户，看看他们对此的看法了。

### 提示

在发布应用程序之前，您应该先查看 Crashlytics。您可以在[`try.crashlytics.com`](https://try.crashlytics.com)找到它。

Crashlytics 可以为您提供实时崩溃报告信息，不仅在测试版测试期间，还在您的应用程序发布到 Play 商店后。迟早，您的应用程序会在您没有测试过的设备上运行，并在其上崩溃。Crashlytics 可以帮助您找到这一原因。

只需下载他们的 SDK，向您的应用程序添加几行代码，然后您就可以开始了。

在将应用程序发布到 Play 商店上向大众公开之前，先分发您的应用程序并进行测试。从他们的反馈中学习并改进您的应用程序。

最后，您可以将这个标志放在您的网站上：

![测试您的应用程序](img/B04299_10_01.jpg)

在本章中，您将学习以下内容：

+   构建变体

+   运行时权限

+   Play 商店测试版分发

# 介绍

典型的软件发布周期是这样的，尽管不一定必须经过每个阶段：

Alpha -> 封闭测试版 -> 公开测试版 -> 发布。

您可以直接在 Google Play 商店上发布您的应用程序，但至少进行一轮测试是明智的。收集反馈并进行进一步改进可以使您的应用程序变得更好。

我们将看看如何为您的应用程序设置多个不同的风味，以及如何为其定义不同的构建类型。例如，您的发布应用程序很可能会使用不同的 API 端点，而不是您用于调试和测试的端点，至少我希望如此。

您选择的最低 API 级别、所需功能和所请求的权限将影响您的应用程序在 Play 商店中可用的设备数量。此外，我们将预览 Android Marshmallow 提供的运行时权限需要不同的方法。

最后，我们将找出在 Google Play 商店上分发应用程序的测试版或 Alpha 版本需要做什么。

# 构建变体

Android Studio 支持应用程序的不同配置。例如，您的应用程序可能会在调试时使用不同的 API 端点。为此，我们将使用构建类型。

除此之外，您可能会有不同版本的应用程序。一个项目可以有多个定制版本的应用程序。如果这些变化很小，例如只是改变了应用程序的外观，那么使用风味是一个好方法。

构建变体是构建类型和特定风味的组合。接下来的教程将演示如何使用这些。

## 准备工作

对于这个教程，您只需要一个最新版本的 Android Studio。

## 如何做...

我们将构建一个简单的消息应用程序，该应用程序使用不同的构建类型和构建风味：

1.  在 Android Studio 中创建一个新项目，命名为`WhiteLabelMessenger`，在**公司域**字段中输入公司名称，然后单击**确定**按钮。

1.  接下来，选择**手机和平板电脑**，然后单击**下一步**按钮。

1.  选择**空白活动**，然后单击**下一步**按钮。

1.  接受建议的值，然后单击**完成**按钮。

1.  打开`strings.xml`文件并添加一些额外的字符串。它们应该看起来像这样：

```kt
<resources>
    <string name="app_name">WhiteLabelMessenger</string>
    <string name="hello_world">Hello world!</string>
    <string name="action_settings">Settings</string>
    <string name="button_send">SEND YEAH!</string>
    <string name="phone_number">Your phone number</string>
    <string name="yeah">Y-E-A-H</string>
    <string name="really_send_sms">YES</string>
</resources>
```

1.  在`res/drawable`文件夹中创建一个`icon.xml`和一个`background.xml`资源文件。

1.  在`res/drawable`文件夹中，创建一个名为`icon.xml`的新文件。它将绘制一个蓝色的圆圈：

```kt
<?xml version="1.0" encoding="utf-8"?>
<shape    
    android:shape="oval">
    <solid
        android:color="@android:color/holo_blue_bright"/>
    <size
        android:width="120dp"
        android:height="120dp"/>
</shape>
```

1.  在`res/drawable`文件夹中，创建一个名为`background.xml`的新文件。它定义了一个渐变蓝色背景：

```kt
<?xml version="1.0" encoding="utf-8"?>
<selector >
    <item>
        <shape>
            <gradient
                android:angle="90"
                android:startColor="@android:color/holo_blue_light"android:endColor="@android:color/holo_blue_bright"android:type="linear" />
        </shape>
    </item>
</selector>
```

1.  打开`activity_main.xml`文件并修改它，使其看起来像这样：

```kt
<FrameLayout xmlns:android=
  "http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
     android:layout_height="match_parent"    android:paddingLeft="@dimen/activity_horizontal_margin"       
     android:paddingRight="@dimen/activity_horizontal_margin"android:paddingTop="@dimen/activity_vertical_margin"android:background="@drawable/background"       
     android:paddingBottom= "@dimen/activity_vertical_margin" 
     tools:context=".MainActivity">
    <EditText
        android:id="@+id/main_edit_phone_number"
        android:layout_marginTop="38dp"
        android:textSize="32sp"
        android:gravity="center"
        android:hint="@string/phone_number"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <Button
        android:id="@+id/main_button_send"android:background="@drawable/icon"android:layout_gravity="center"android:layout_width="200dp"android:layout_height="200dp" />
    <TextView
        android:text="@string/button_send"android:textSize="32sp"android:gravity="center"android:layout_gravity="bottom"android:textColor="@android:color/white"android:layout_width="match_parent"android:layout_height="wrap_content" />
</FrameLayout>
```

1.  打开`androidmanifest.xml`文件并添加一个发送短信的权限：

```kt
<uses-permission 
 android:name="android.permission.SEND_SMS"/>
```

1.  修改`MainActivity`文件的`onCreate`方法。您可以按两次*Shift*键来显示搜索面板。在搜索面板上输入`onCreate`，并选择`MainActivity`类的`onCreate`方法：

```kt
findViewById(R.id.main_button_send).setOnClickListener(this);
```

1.  在`MainActivity`类上添加一个点击监听器，并实现`onClick`方法：

```kt
public class MainActivity extends Activity implements View.OnClickListener{
@Override
public void onClick(View v) {
    String phoneNumber = ((EditText)findViewById( 
     R.id.main_edit_phone_number)).getText().toString();
    SmsManager sms = SmsManager.getDefault();
    String message = getString(R.string.yeah);
    if (getString(R.string.really_send_sms)  == "YES"){
     Toast.makeText(this, String.format(
      "TEST Send %s to %s", message, phoneNumber), Toast.LENGTH_SHORT).show();
    }
    else {
      sms.sendTextMessage(phoneNumber, null, message, null, 
       null);

      Toast.makeText(this, String.format(
       "Send %s to %s", message, phoneNumber), Toast.LENGTH_SHORT).show();
    }
}
```

1.  选择`app`文件夹。然后，从**构建**菜单中选择**编辑风味**。

1.  列表中只包含一个 defaultConfig。单击**+**按钮添加一个新的风味。将其命名为`blueFlavor`，并与**defaultConfig**相同的值作为`min sdk version`和`target sdk version`。

1.  对于**application id**字段，使用包名**+**扩展名`.blue`。

1.  为该风味输入**版本代码**和**版本名称**，然后单击**确定**按钮。

1.  为另一个风味重复步骤 14 到 16。将该风味命名为`greenFlavor`。

1.  现在您的`build.gradle`文件应该包含如下风味：

```kt
productFlavors {
    blueFlavor {
        minSdkVersion 21
        applicationId 'packt.com.whitelabelmessenger.blue'targetSdkVersion 21
        versionCode 1
        versionName '1.0'
    }
    greenFlavor {
        minSdkVersion 21
        applicationId 'packt.com.whitelabelmessenger.green'targetSdkVersion 21versionCode 1
        versionName '1.0'
    }
}
```

1.  在**项目**面板中，选择`app`文件夹下的`src`文件夹。然后，创建一个新文件夹，并命名为`blueFlavor`。在该文件夹中，您可以保持与`main`文件夹相同的结构。对于本教程，只需添加一个`res`文件夹，在该文件夹中再添加一个名为`drawable`的文件夹即可。

1.  对`greenFlavor`构建的风味执行相同的操作。项目结构现在如下所示：![如何操作...](img/B04299_10_02.jpg)

1.  从`/main/res/drawable`文件夹中复制`background.xml`和`icon.xml`文件，并将它们粘贴到`blueFlavor/res/drawable`文件夹中。

1.  为`greenFlavor`重复此操作，并在`greenFlavor/res/drawable`文件夹中打开`background.xml`文件。修改其内容。对于绿色风味，我们将使用渐变绿色：

```kt
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android=
  "http://schemas.android.com/apk/res/android">
    <item>
        <shape>
            <gradient
            android:angle="90"
            android:startColor= 
             "@android:color/holo_green_light"                   
            android:endColor=  
             "@android:color/holo_green_dark"
            android:type="linear" />
        </shape>
    </item>
</selector>
```

1.  现在，在同一文件夹中，打开`icon.xml`文件，并将`drawable`文件夹也显示为绿色：

```kt
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android=
   "http://schemas.android.com/apk/res/android"
     android:shape="oval">
    <solidandroid:color="@android:color/holo_green_dark"/>
    <size
        android:width="120dp"android:height="120dp"/>
</shape>
```

1.  可以使用相同的方法来为调试和发布构建类型使用不同的值（或类或布局）。在`app/src`文件夹中创建一个`debug`文件夹。

1.  在该文件夹中，创建一个`res`文件夹，然后在其中创建一个`values`文件夹。

1.  将`strings.xml`文件从`main/res/values`文件夹复制并粘贴到`debug/res/values`文件夹中。

1.  打开`strings.xml`文件，并修改`really_send_sms`字符串资源：

```kt
<string name="really_send_sms">NO</string>
```

### 提示

当然，为了简单起见，我们将修改字符串资源，而更好的方法当然是使用一个定义不同值的常量类。

### 构建变体

选择`app`文件夹，并从**构建**菜单中选择**选择构建变体**。它将显示如下截图所示的**构建变体**面板：

![构建变体](img/B04299_10_03.jpg)

在**构建变体**中按照以下步骤进行：

1.  选择**greenFlavorDebug**构建变体，并运行应用程序。

1.  如果一切顺利，应用程序将呈现绿色外观，并且表现得好像正在进行调试。

1.  现在将构建变体更改为**blueFlavorDebug**，然后再次运行应用程序。确实，现在它看起来是蓝色的。

### 构建类型

调试和发布构建类型也基本相同；但是，这次不是外观，而是行为或数据（或者端点）发生了变化。

### 注意

发布应用程序需要签名，这是我们将在将应用程序分发到 Play 商店时执行的操作，这在上一篇教程中已经描述过了。

![构建类型](img/B04299_10_04.jpg)

这基本上就是构建变体的全部内容。大多数理想的构建类型和风味只包含少量修改。如果您的应用程序的各种风味之间的差异不仅仅是在布局、可绘制对象或常量值上进行一些微调，那么您将不得不考虑采用不同的方法。

## 还有更多...

Android Studio 还提供了一些其他很棒的功能来完成您的应用程序。其中之一是自动生成技术文档。只需向类或方法添加一些注释，就像这样：

```kt
/*** This is the main activity where all things are happening*/
public class MainActivity extends Activity implements View.OnClickListener{
```

现在，如果您从**工具**菜单中选择**生成 JavaDoc**，并在出现的对话框中定义**输出目录**字段的路径，您只需要点击**确定**按钮，所有文档都将被生成为 HTML 文件。结果将显示在您的浏览器中，如下所示：

![更多内容...](img/B04299_10_05.jpg)

### 注意

**Android Studio 提示**

您经常需要返回到代码中的特定位置吗？使用*Cmd* + *F3*（对于 Windows：*F11*）快捷键创建书签。

要显示书签列表并从中选择，请使用快捷键*Cmd* + *F3*（对于 Windows：*Shift* + *F11*）。

# 运行时权限

您的应用程序将针对不同类型的设备取决于功能要求（需要权限）和您所针对的市场（通过明确选择特定国家或提供特定语言的应用程序）的数量。

例如，如果您的应用程序需要前置摄像头和后置摄像头，那么您将针对较少数量的设备，就像您只需要后置摄像头一样。

通常，在安装应用程序时，用户会被要求接受（或拒绝）所有所需的权限，就像在应用程序的`AndroidManifest`文件中定义的那样。

随着 Android 6（Marshmallow）的推出，用户被要求特定权限的方式发生了变化。只有在需要某种类型的权限时，用户才会被提示，以便他可以允许或拒绝该权限。

有了这个机会，应用程序可以解释为什么需要这个权限。之后，整个过程对用户来说就更有意义了。这些所谓的运行时权限需要一种稍微不同的开发方法。

对于这个示例，我们将修改之前发送短信的应用程序。现在，我们需要在用户点击按钮后请求用户的权限，以便发送短信。

## 准备工作

要测试运行时权限，您需要有一个运行 Android 6.0 或更高版本的设备，或者您需要有一个运行 Android Marshmallow 或更高版本的虚拟设备。

还要确保您已经下载了 Android 6.x SDK（API 级别 23 或更高）。

## 操作步骤...

那么，这些运行时权限是什么样的，我们如何处理它们？可以通过以下步骤来检查：

1.  从上一个示例中打开项目。

1.  打开`AndroidManifest`文件，并添加权限（根据新模型）以发送短信：

```kt
<uses-permission-sdk- 
 android:name="android.permission.SEND_SMS"/>
```

1.  在`app`文件夹中打开`build.gradle`文件，并将`compileSdkVersion`的值设置为最新可用版本。还要将每个`minSdkVersion`和`targetSdkVersion`的值更改为`23`或更高。

1.  修改`onClick`方法：

```kt
@Override
public void onClick(View v) {
    String phoneNumber = ((EditText) findViewById( 
     R.id.main_edit_phone_number)).getText().toString();
    String message = getString(R.string.yeah);
    if (Constants.isTestSMS) {
      Toast.makeText(this, String.format(
       "TEST Send %s to %s", message, phoneNumber), 
       Toast.LENGTH_SHORT).show();
    } 
    else {
      if (checkSelfPermission(Manifest.permission.SEND_SMS)   
       != PackageManager.PERMISSION_GRANTED) {
            requestPermissions(new String[]{  
              Manifest.permission.SEND_SMS},
                 REQUEST_PERMISSION_SEND_SMS);
        }
    }
}
```

1.  添加一个常量值，以便以后我们将知道权限请求的权限结果是指哪个权限请求：

```kt
private final int REQUEST_PERMISSION_SEND_SMS = 1;
```

1.  实现`sendSms`方法。我们将使用`SmsManager`方法将`Y-E-A-H`文本发送到用户输入的电话号码。一旦消息发送成功，将显示一个 toast：

```kt
private void sendSms(){
    String phoneNumber = ((EditText) findViewById( 
     R.id.main_edit_phone_number)).getText().toString();
    String message = getString(R.string.yeah);
    SmsManager sms = SmsManager.getDefault();
    sms.sendTextMessage(phoneNumber, null, 
     getString(R.string.yeah), null, null);
    Toast.makeText(this, String.format("Send %s to %s", getString(R.string.yeah), phoneNumber), Toast.LENGTH_SHORT).show();
}
```

1.  最后，实现`onRequestPermissionsResult`方法。如果授予的权限是短信权限，则调用`sendSms`方法。如果权限被拒绝，则会显示一个 toast，并且**发送**按钮和输入电话号码的编辑文本将被禁用：

```kt
@Override
public void onRequestPermissionsResult(int requestCode,  String permissions[], int[] grantResults) {
    switch (requestCode) {
        case REQUEST_PERMISSION_SEND_SMS: {
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                sendSms();
            }
            else {                
              findViewById(
               R.id.main_edit_phone_number).setEnabled(false); 
              findViewById(  
               R.id.main_button_send).setEnabled(false);
                Toast.makeText(this, 
                 getString(R.string.no_sms_permission), Toast.LENGTH_SHORT).show();
            }
            return;
        }
    }
}
```

1.  运行您的应用程序。使用运行 Android 6.0 或更高版本的设备，或者创建一个运行 API 级别 23 或更高版本的虚拟设备。

1.  现在，发送短信的权限不会被事先要求（也就是说，如果用户安装了应用程序）。相反，一旦您点击**发送**按钮，就会弹出一个请求权限的对话框。

1.  如果您同意请求权限，短信将被发送。如果您拒绝了请求的权限，编辑框和按钮将被禁用，并且将显示一个 toast 以提供反馈：![操作步骤...](img/B04299_10_06.jpg)

这个示例演示了运行时权限的基本概念。

## 更多内容...

要了解何时以及如何请求权限，或者何时以及如何提供有关不可用特定功能的反馈意见，您可以在[`www.google.com/design/spec/patterns/permissions.html`](https://www.google.com/design/spec/patterns/permissions.html)上查看 Google 的指南。

### 注意

**Android Studio 提示**

您可以轻松地从变得太大的方法中提取代码。只需标记您想要移动的代码，然后使用快捷键*Cmd* + *Alt* + *M*（对于 Windows：*Ctrl* + *Alt* + *M*）。

# Play 商店 beta 分发

好了，我们将把我们的应用程序上传到 Play 商店作为 beta 分发。很激动人心，不是吗？

## 准备工作

对于这个食谱，我们将使用第一个食谱中的应用程序；尽管如此，任何您认为已准备好进行 beta 发布的应用程序都可以。

确保您也有一些艺术作品，例如图标和截图。别担心，对于这个食谱，您也可以从<[www.packtpub.com](http://www.packtpub.com)>下载这些项目。此外，考虑您应用程序的元数据，例如标题、描述和类别。

最重要的是您必须拥有开发者帐户，并且可以访问 Google Play 开发者控制台。如果您没有帐户，您需要首先通过[`developer.android.com/distribute/googleplay/start.html`](http://developer.android.com/distribute/googleplay/start.html)注册。

## 如何做...

将您的应用程序放入 Play 商店并不难。只是需要一些时间来正确设置事物：

1.  登录到您的**Google Play 开发者控制台**网页，或者如果需要的话，首先注册。

1.  在仪表板上，点击**添加新应用程序**按钮。

1.  在对话框中，输入应用程序的**标题**“蓝色信使”，然后点击**立即上传 APK**按钮。

1.  您会注意到**production**、**beta**和**alpha**选项卡。理想情况下，您应该从 alpha 测试开始，但出于演示目的，我们将立即选择**beta**选项卡。在那里，将显示**将第一个 APK 上传到 beta**按钮。点击该按钮。

1.  在 Android Studio 中，打开我们为第一个（或第二个）食谱创建的应用程序，然后从**构建**菜单中选择**生成已签名的 APK**选项。

1.  选择`app`模块，然后点击**下一步**按钮。

1.  输入**密钥库的路径**。如果没有，请点击**创建新...**按钮，找到一个适合您的密钥库文件（带有`.jks`扩展名）的好地方。为其输入一个**密码**，重复密码，并输入**名字**的合适值。然后，点击**确定**按钮。

1.  输入**密钥库密码**，创建一个新的**密钥别名**，并将其命名为`whitelabelmessenger`。为密钥输入一个**密码**，然后点击**下一步**按钮。

1.  如果需要，输入**主密码**，然后点击**确定**按钮。

1.  如果需要，修改**目标路径**，然后选择**构建类型**和**风味**。选择**发布**和**blueFlavor**，然后点击**确定**按钮。

1.  一个新的对话框通知我们，如果一切顺利，已成功创建了一个新的已签名 APK。点击**在 Finder 中显示**（或者在 Windows 中使用 Windows 资源管理器找到）按钮，以查看刚刚创建的 APK 文件。

1.  在浏览器中上传此 APK 文件。一旦 APK 文件上传完成，版本将显示在**beta**选项卡上；您可以选择测试方法并查看受支持设备的数量，这将取决于您选择的 API 级别以及带有短信权限的必需功能（例如，这将立即排除许多平板电脑）。

1.  对于测试方法，点击**设置封闭式 beta 测试**按钮。

1.  点击**创建列表**按钮创建一个列表。给列表取一个名字，例如**内部测试**，然后添加测试人员的电子邮件地址（或者只是为了练习，输入您自己的）。完成后，点击**保存**按钮。

1.  将您自己的电子邮件地址输入为**反馈渠道**，然后点击**保存草稿**按钮。

1.  尽管我们尚未在商店上发布任何内容，但您需要为**商店列表**部分输入一些值，这是您可以从网页左侧的菜单中选择的选项：![如何做…](img/B04299_10_07.jpg)

1.  输入标题、简短和长描述。还要添加两张截图、一个高分辨率图标和一个特色图像。您可以从<[www.packtpub.com](http://www.packtpub.com)>下载这些资源，或者您可以通过从您的应用中截取截图并使用某种绘图程序进行一些有趣的操作，以使它们具有正确的宽度和高度。

1.  在**分类**中，选择**应用程序**作为**应用程序类型**，并选择**社交**或**通讯**作为**类别**。

1.  输入您的**联系方式**，并选择**目前不提交隐私政策**（除非您确实希望这样做）。

1.  点击**保存草稿**按钮，然后从屏幕左侧的菜单中选择**内容评级**部分，继续进行。

### 为您的应用评分

点击**继续**按钮，输入您的**电子邮件地址**，并回答有关您的应用是否具有任何暴力、色情或其他潜在危险内容或功能的问题。最后，点击**保存问卷**按钮：

1.  现在，您可以点击**计算评级**按钮。之后将显示您的评级。点击**应用评级**按钮，然后您就完成了。

1.  接下来是**定价和分发**部分。从页面左侧的菜单中选择此选项。

1.  通过点击**免费**按钮，使其成为免费应用，并**选择所有国家**（或者如果您愿意，可以指定特定国家）。之后，点击**保存草稿**按钮。

1.  到目前为止，**发布应用**按钮应该已经启用。点击它。如果它没有启用，您可以点击**我无法发布？**链接，找出缺少哪些信息。

1.  在这里，“发布”这个词有点令人困惑。实际上，在这种情况下，它意味着该应用将被发布给您刚刚创建的测试用户名单上的用户。不用担心。在您将应用程序推广到生产环境之前，Play 商店中将不会有任何内容，尽管“发布”这个词似乎暗示了这一点。

1.  当您的应用状态显示为**待发布**时，您可以调查一些其他选项，比如您的应用支持的设备列表、所需功能和权限以及用于分析目的的选项，包括功能分割测试（A/B 测试）。

### 休息一下

**待发布**状态可能需要几个小时（甚至更长时间），因为自 2015 年 4 月以来，谷歌宣布将事先审查应用程序（以半手动半自动的方式），即使是 alpha 和 beta 版本的分发也是如此。

1.  吃一个棉花糖，喝点咖啡，或者在公园里散散步。几个小时后回来检查您的应用状态是否已更改为**已发布**。可能需要一些时间，但会成功的。

### 注意

您的测试人员可能需要更改其（安全）设置，以**允许在 Google Play 商店之外安装应用程序**。

1.  还有一些其他看起来令人困惑的事情。在包名称后面，会有一个链接，上面写着**在 Play 商店中查看…**，还有一个提示说 alpha 和 beta 应用程序不会在 Play 商店中列出。

1.  在网页左侧的菜单中点击**APK**项目。通过链接，您将在**Beta**选项卡上找到**Opt In Url**，您的测试用户可以通过该链接下载并安装 beta 应用程序：![休息一下](img/B04299_10_08.jpg)

太棒了！您的第一个 beta 分发已经准备好进行测试。您可能需要多次迭代才能做到完美，或者也许只需要一个 beta 版本就足以发现您的应用已经准备好进入**Play 商店**。

要在 Play 商店上发布你的应用，点击**推广到生产**按钮，如果你敢的话…

就到这里吧。还有很多关于 Android 开发的东西要讲和学习，比如服务、Android Pay、**近场通讯**（NFC）和蓝牙等等；然而，通过阅读这本书，你已经看到了 Android Studio IDE 的大部分元素，这也是我们的目标。

就是这样了。谢谢你的阅读，祝你编码愉快！

## 还有更多…

你应该意识到，除了技术，方法论同样重要。开发一个不仅在技术上完美，而且有很多用户对你的应用和其流程、可用性和外观都非常满意，给你应得的五星评价的应用是很难的。

我假设你不想花几个月甚至几年的时间开发一个应用，最后发现其实没有人在乎。在早期阶段找出是什么让人们真正想使用你的应用，你应该考虑精益创业方法论来开发你的应用。

**构建-测量-学习**

**精益创业**方法论是一种开发企业和产品（或服务）的方法。其理念是基于假设的实验、验证学习和迭代产品发布会导致更短的产品开发周期。

精益创业方法论的最重要的关键元素是：

+   **最小可行产品**（MVP）

+   分割测试和可操作指标

+   持续部署

简而言之，MVP 是产品的一个版本，需要最小的努力来测试特定的假设。

要了解更多关于精益创业方法论的信息，可以查看网站[`theleanstartup.com`](http://theleanstartup.com)，阅读 Eric Ries 的书，或者从[`www.leanstartupcircle.com`](http://www.leanstartupcircle.com)找到一个靠近你的精益创业活动。

**Play 商店开发者控制台**提供了分割测试和测量应用程序使用情况的选项。谷歌分析可以帮助你做到这一点，因为这是获得可操作指标的最简单方法，你需要收集这些指标以便通过学习改进你的应用程序。

**持续部署**很好地融入了精益创业方法论。它可以提高应用程序开发的质量和速度。

你可能会想知道持续部署是什么。完全解释这个概念需要另一本书，但这里是对持续集成和持续交付的简要介绍，如果结合起来，就是持续部署的内容。

**持续集成**（CI）是开发人员提交他们的更改并将结果合并到源代码存储库的过程。构建服务器观察代码存储库的更改，拉取和编译代码。服务器还运行自动化测试。

**持续交付**是自动创建可部署版本的过程，例如，通过在 Play 商店发布 alpha 或 beta 应用。因此，提交和验证的代码始终处于可部署状态是很重要的。

设置持续部署需要一些前期工作，但最终会导致更小更快的开发周期。

对于 Android 应用程序的持续部署，`Jenkins`和`TeamCity`都是合适的。`Teamcity`经常被推荐，并且使用插件可以与 Android Studio 集成。

要了解如何设置`TeamCity`服务器或找到更多信息，你可以查看 Packt Publishing 的网站，那里有一些很好的书来解释持续集成和`TeamCity`的概念。
