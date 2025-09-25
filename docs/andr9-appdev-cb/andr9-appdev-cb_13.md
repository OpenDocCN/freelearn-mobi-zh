# 第十三章：电信、网络和互联网

在本章中，我们将涵盖以下主题：

+   如何拨打电话

+   监控电话呼叫事件

+   如何发送短信（文本）消息

+   接收短信消息

+   在您的应用中显示网页

+   检查在线状态和连接类型

+   电话号码屏蔽 API

# 简介

我们将首先通过查看“如何拨打电话”来探讨电信功能。在探索了如何拨打电话之后，我们将查看如何通过监控电话呼叫事件来监控电话呼叫。然后，在“如何发送短信消息”部分，我们将介绍短信消息的接收。

然后，我们将探索`WebView`以向您的应用添加浏览器功能。在基本层面上，`WebView`是一个基本的 HTML 查看器。我们将展示如何扩展`WebViewClient`类并通过`WebSettings`修改设置来创建完整的浏览器功能，包括 JavaScript 和缩放功能。

本章的最后一个食谱将探讨一个新 API（在 Android 7.0 Nougat 中添加），该 API 可以在操作系统级别屏蔽电话号码。

# 如何拨打电话

如前文所述，我们可以通过使用 Intent 简单地调用默认应用。对于电话呼叫，有两个 Intent：

+   `ACTION_DIAL`: 使用默认的电话应用拨打电话（无需权限）

+   `CALL_PHONE`: 跳过 UI 直接拨打电话（需要权限）

下面是设置并调用 Intent 以使用默认电话应用的代码：

```java
Intent intent = new Intent(Intent.ACTION_DIAL); 
intent.setData(Uri.parse("tel:" + number)); 
startActivity(intent); 
```

由于您的应用不执行拨号，并且用户必须按下拨号按钮，因此您的应用不需要任何拨号权限。接下来的食谱将向您展示如何直接拨打电话，绕过拨号器应用。

# 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为`DialPhone`。使用默认的 Phone & Tablet 选项，并在提示活动类型时选择 Empty Activity。

# 如何做...

首先，我们需要添加适当的权限来拨打电话。然后，我们需要添加一个按钮来调用我们的拨号方法。首先打开 Android Manifest，并按照以下步骤操作：

1.  添加以下权限：

```java
<uses-permission android:name="android.permission.CALL_PHONE"/>
```

1.  打开`activity_main.xml`并将现有的`TextView`替换为以下按钮：

```java
<Button
    android:id="@+id/button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Dial"
    android:onClick="dialPhone"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

1.  添加此方法，它将检查您的应用是否已被授予`CALL_PHONE`权限：

```java
private boolean checkPermission(String permission) {
    int permissionCheck = ContextCompat.checkSelfPermission(this, permission);
    return (permissionCheck == PackageManager.PERMISSION_GRANTED);
}
```

1.  添加拨打电话的代码：

```java
public void dialPhone(View view){
    if (checkPermission(Manifest.permission.CALL_PHONE)) {
        Intent intent = new Intent(Intent.ACTION_CALL);
        intent.setData(Uri.parse("tel:0123456789"));
        startActivity(intent);
    } else {
        ActivityCompat.requestPermissions(this, new String[] {Manifest.permission.CALL_PHONE},1);
    }
}
```

1.  在您的设备上运行之前，请务必将 0123456789 替换为有效的号码。

# 它是如何工作的...

如介绍中所述，使用`CALL_PHONE` Intent 需要适当的权限。我们在第一步中将所需的权限添加到清单中，并在第四步实际调用 Intent 之前，在第三步中使用该方法来验证权限。从 Android 6.0 Marshmallow（API 23）开始，权限不再在安装期间授予。因此，我们在尝试拨打电话之前检查应用程序是否有权限。

# 参见

+   参考第十五章中的*The Android 6.0 Runtime Permission Model*配方，*为 Play 商店准备您的应用程序*，以获取有关新运行时权限的更多信息。

# 监控电话呼叫事件

在之前的配方中，我们演示了如何进行电话呼叫，无论是通过 Intent 调用默认应用程序，还是直接拨打电话而不显示 UI。

如果您想在电话结束时收到通知怎么办？这会变得稍微复杂一些，因为您需要监控 Telephony 事件并跟踪电话状态。在这个配方中，我们将演示如何创建一个`PhoneStateListener`来读取电话状态事件。

# 准备工作

在 Android Studio 中创建一个新的项目，命名为`PhoneStateListener`。使用默认的“电话和平板”选项，并在“添加到移动”对话框中选择“Empty Activity”。

虽然不是必需的，但您可以使用之前的配方来发起电话。否则，使用默认拨号器并/或监视来电事件。

# 如何操作...

我们只需要在布局中添加一个`TextView`来显示事件信息。打开`activity_main.xml`文件并按照以下步骤操作：

1.  添加或修改`TextView`如下：

```java
<TextView
    android:id="@+id/textView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

1.  将以下权限添加到 AndroidManifest.xml 中：

```java
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
```

1.  打开`MainActivity.java`并将以下`PhoneStateListener`类添加到`MainActivity`类中：

```java
PhoneStateListener mPhoneStateListener = new PhoneStateListener() {
    @Override
    public void onCallStateChanged(int state, String number) {
        String phoneState = number;
        switch (state) {
            case TelephonyManager.CALL_STATE_IDLE:
                phoneState += "CALL_STATE_IDLE\n";
                break;
            case TelephonyManager.CALL_STATE_RINGING:
                phoneState += "CALL_STATE_RINGING\n";
                break;
            case TelephonyManager.CALL_STATE_OFFHOOK:
                phoneState += "CALL_STATE_OFFHOOK\n";
                break;
        }
        TextView textView = findViewById(R.id.textView);
        textView.append(phoneState);
    }
};
```

1.  修改`onCreate()`以设置监听器：

```java
final TelephonyManager telephonyManager = 
        (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);
telephonyManager.listen(mPhoneStateListener, PhoneStateListener.LISTEN_CALL_STATE);
```

1.  在设备上运行应用程序并发起和/或接收电话。返回此应用程序后，您将看到事件列表。

# 它是如何工作的...

为了演示使用监听器，我们在`onCreate()`方法中创建 Telephony 监听器，使用以下代码：

```java
final TelephonyManager telephonyManager =
        (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);
telephonyManager.listen(mPhoneStateListener, PhoneStateListener.LISTEN_CALL_STATE);
```

当发生`PhoneState`事件时，它会被发送到我们的`PhoneStateListener`类。

# 更多内容...

在本配方中，我们正在监控呼叫状态事件，如常量`LISTEN_CALL_STATE`所示。其他有趣的选项包括以下内容：

+   `LISTEN_CALL_FORWARDING_INDICATOR`

+   `LISTEN_DATA_CONNECTION_STATE`

+   `LISTEN_SIGNAL_STRENGTHS`

查看*参见*中的`PhoneStateListener`链接，获取完整列表。

当我们完成对事件的监听后，调用`listen()`方法并传递`LISTEN_NONE`，如下所示：

```java
telephonyManager.listen(mPhoneStateListener,PhoneStateListener.LISTEN_NONE); 
```

# 参见

+   开发者文档：`PhoneStateListener`在[`developer.android.com/reference/android/telephony/PhoneStateListener.html`](https://developer.android.com/reference/android/telephony/PhoneStateListener.html)

# 如何发送短信（文本）消息

由于您可能已经熟悉短信（或文本）消息，我们不会花时间解释它们是什么或为什么它们很重要。（如果您不熟悉短信或需要更多信息，请参阅本配方“参见”部分中提供的链接。）本配方将演示如何发送短信消息。（下一个配方将演示如何接收新消息的通知以及如何读取现有消息。）

# 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为`SendSMS`。使用默认的 Phone & Tablet 选项，并在“添加一个活动到移动”对话框中选择 Empty Activity。

# 如何做到这一点...

首先，我们需要添加发送短信所需的必要权限。然后，我们将创建一个包含电话号码和消息字段以及发送按钮的布局。当点击发送按钮时，我们将创建并发送短信。以下是步骤：

1.  打开 AndroidManifest 并添加以下权限：

```java
<uses-permission android:name="android.permission.SEND_SMS"/>
```

1.  打开`activity_main.xml`并用以下 XML 替换现有的布局：

```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <EditText
        android:id="@+id/editTextNumber"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="number"
        android:ems="10"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"
        android:hint="Number"/>
    <EditText
        android:id="@+id/editTextMsg"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/editTextNumber"
        android:layout_centerHorizontal="true"
        android:hint="Message"/>
    <Button
        android:id="@+id/buttonSend"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Send"
        android:layout_below="@+id/editTextMsg"
        android:layout_centerHorizontal="true"
        android:onClick="send"/>
</RelativeLayout>
```

1.  打开`MainActivity.java`并添加以下全局变量：

```java
final int SEND_SMS_PERMISSION_REQUEST_CODE=1; 
Button mButtonSend; 
```

1.  将以下代码添加到现有的`onCreate()`回调中：

```java
mButtonSend = findViewById(R.id.buttonSend);
mButtonSend.setEnabled(false);

if (checkPermission(Manifest.permission.SEND_SMS)) {
    mButtonSend.setEnabled(true);
} else {
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.SEND_SMS}, 
            SEND_SMS_PERMISSION_REQUEST_CODE);
}
```

1.  添加以下方法来检查权限：

```java
private boolean checkPermission(String permission) {
    int permissionCheck = ContextCompat.checkSelfPermission(this,permission);
    return (permissionCheck == PackageManager.PERMISSION_GRANTED);
}
```

1.  重写`onRequestPermissionsResult()`来处理权限

    请求响应：

```java
@Override
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
    switch (requestCode) {
        case SEND_SMS_PERMISSION_REQUEST_CODE: {
            if (grantResults.length > 0
                    && grantResults[0] ==
                    PackageManager.PERMISSION_GRANTED) {
                mButtonSend.setEnabled(true);
            }
            return;
        }
    }
}
```

1.  最后，添加实际发送短信的方法：

```java
public void send(View view) {
    String phoneNumber = ((EditText)findViewById(R.id.editTextNumber)).getText().toString();
    String msg = ((EditText)findViewById(R.id.editTextMsg)).getText().toString();

    if (phoneNumber==null || phoneNumber.length()==0 || msg==null || msg.length()==0 ) {
        return;
    }

    if (checkPermission(Manifest.permission.SEND_SMS)) {
        SmsManager smsManager = SmsManager.getDefault();
        smsManager.sendTextMessage(phoneNumber, null, msg, null, null);
    } else {
        Toast.makeText(MainActivity.this, "No Permission", Toast.LENGTH_SHORT).show();
    }
}
```

1.  你现在可以运行应用程序在设备或模拟器上。（在发送到另一个模拟器时使用模拟器设备号码。第一个模拟器是 5554；第二个是 5556，每个额外的模拟器递增 2。）

# 它是如何工作的...

发送短信的代码只有两行，如下所示：

```java
SmsManager smsManager = SmsManager.getDefault(); 
smsManager.sendTextMessage(phoneNumber, null, msg, null, null); 
```

`sendTextMessage()`方法执行实际的发送。这个菜谱的大多数代码是用来检查和获取所需的权限。

# 还有更多...

虽然发送短信消息很简单，但我们还有一些其他选项。

# 多部分消息

尽管这取决于运营商，但通常每个短信允许的最大字符数是 160 个。你可以修改前面的代码来检查消息是否超过 160 个字符，如果是的话，你可以调用 SMSManager 的 divideMessage()方法。该方法返回`ArrayList`，你可以将其发送到`sendMultipartTextMessage()`。以下是一个示例：

```java
ArrayList<String> messages=smsManager.divideMessage(msg);
smsManager.sendMultipartTextMessage(phoneNumber, null, messages, null, null);
```

注意，使用`sendMultipartTextMessage()`发送的消息在模拟器上可能无法正确工作，因此请确保在真实设备上进行测试。

# 投递状态通知

如果你希望被通知消息的状态，有两个可选字段你可以使用。以下是 SMSManager 文档中定义的`sendTextMessage()`方法：

```java
sendTextMessage(String destinationAddress, String scAddress, String text, 
        PendingIntent sentIntent, PendingIntent deliveryIntent)
```

你可以包含一个待处理的 Intent 来通知发送状态和/或投递状态。在你收到待处理的 Intent 时，它将包含一个结果代码，要么是 Activity 的`RESULT_OK`，如果发送成功，或者是一个在 SMSManager 文档中定义的错误代码（见以下链接）：

+   `RESULT_ERROR_GENERIC_FAILURE`：通用失败原因

+   `RESULT_ERROR_NO_SERVICE`：由于服务当前不可用而失败

+   `RESULT_ERROR_NULL_PDU`：由于没有提供 PDU 而失败

+   `RESULT_ERROR_RADIO_OFF`：由于无线电被明确关闭而失败

# 参见

+   维基百科上的短信息服务[`en.wikipedia.org/wiki/Short_Message_Service`](https://en.wikipedia.org/wiki/Short_Message_Service)

+   开发者文档：SMSManager 在 [`developer.android.com/reference/android/telephony/SmsManager.html`](https://developer.android.com/reference/android/telephony/SmsManager.html)

# 接收短信消息

这个食谱将演示如何设置一个广播接收器来通知你新的短信消息。值得注意的是，你的应用不需要运行就可以接收短信意图。Android 将启动你的服务来处理短信。

# 准备工作

在 Android Studio 中创建一个新的项目，命名为 `ReceiveSMS`。使用默认的 Phone & Tablet 选项，并在“添加一个活动到移动”对话框中选择 Empty Activity。

# 如何操作...

在这个演示中，我们不会使用布局，因为所有的工作都将放在广播接收器中。我们将使用 Toast 显示传入的短信消息。打开 AndroidManifest 并按照以下步骤操作：

1.  添加以下权限：

```java
<uses-permission android:name="android.permission.RECEIVE_SMS" />
```

1.  将以下广播接收器声明添加到应用程序元素中：

```java
<receiver android:name=".SMSBroadcastReceiver">
    <intent-filter>
        <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
    </intent-filter>
</receiver>
```

1.  打开 `MainActivity.java` 并添加以下方法：

```java
private boolean checkPermission(String permission) {
    int permissionCheck = ContextCompat.checkSelfPermission(this, permission);
    return (permissionCheck == PackageManager.PERMISSION_GRANTED);
}
```

1.  修改现有的 `onCreate()` 回调以检查权限：

```java
if (!checkPermission(Manifest.permission.RECEIVE_SMS)) {
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.RECEIVE_SMS}, 0);
}
```

1.  使用以下代码在项目中添加一个新的 Java 类，命名为 `SMSBroadcastReceiver`：

```java
public class SMSBroadcastReceiver extends BroadcastReceiver {
    final String SMS_RECEIVED = "android.provider.Telephony.SMS_RECEIVED";

    @Override
    public void onReceive(Context context, Intent intent) {
        if (SMS_RECEIVED.equals(intent.getAction())) {
            Bundle bundle = intent.getExtras();
            if (bundle != null) {
                Object[] pdus = (Object[]) bundle.get("pdus");
                String format = bundle.getString("format");
                final SmsMessage[] messages = new SmsMessage[pdus.length];
                for (int i = 0; i < pdus.length; i++) {
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                        messages[i] = SmsMessage.createFromPdu((byte[]) pdus[i], format);
                    } else {
                        messages[i] = SmsMessage.createFromPdu((byte[]) pdus[i]);
                    }
                    Toast.makeText(context, messages[0].getMessageBody(), Toast.LENGTH_SHORT)
                            .show();
                }
            }
        }
    }
}
```

6. 你现在可以在设备或模拟器上运行应用程序了。

# 它是如何工作的...

就像在之前的发送短信食谱中一样，我们首先需要检查应用是否有权限。（在 Android 6.0 之前的设备上，清单声明将自动提供权限，但对于 Marshmallow 及以后的版本，我们需要像这里一样提示用户。）

如你所见，广播接收器接收新短信消息的通知。我们通过在 AndroidManifest 中使用以下代码告诉系统我们想要接收新的短信接收广播：

```java
<receiver android:name=".SMSBroadcastReceiver">
    <intent-filter>
        <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
    </intent-filter>
</receiver>
```

通知通过标准的 `onReceive()` 回调传入，因此我们使用以下代码检查操作：

```java
if (SMS_RECEIVED.equals(intent.getAction())) {} 
```

这可能是这个食谱中最复杂的一行代码：

```java
messages[i] = SmsMessage.createFromPdu((byte[]) pdus[i]); 
```

基本上，它调用 SmsMessage 库从 PDU 创建一个 SMSMessage 对象。（PDU，即协议数据单元，是短信消息的二进制数据格式。）如果你不熟悉 PDU 结构，你不需要了解。SmsMessage 库会为你处理并返回一个 SMSMessage 对象。

如果你的应用没有接收短信广播消息，可能是因为其他现有应用阻止了你的应用。你可以尝试增加 intent-filter 中的优先级值，如所示，或者禁用/卸载其他应用：

```java
<intent-filter   android:priority="100">   
    <action android:name=
           "android.provider.Telephony.SMS_RECEIVED" />   
</intent-filter>   
```

# 更多内容...

这个食谱演示了如何显示接收到的短信消息，但关于读取现有消息怎么办？

# 读取现有短信消息

首先，为了读取现有消息，你需要以下权限：

```java
<uses-permission android:name="android.permission.READ_SMS" /> 
```

下面是使用 SMS 内容提供程序获取游标的示例：

```java
Cursor cursor = getContentResolver().query(
        Uri.parse("content://sms/"), null, null, null, null);
while (cursor.moveToNext()) {
    textView.append("From :" + cursor.getString(1) + " : " + cursor.getString(11)+"\n");
}
```

在撰写本文时，SMS 内容提供程序有超过 30 列。以下是前 12 列，它们是最有用的（记住，列数从零开始）：

1.  `_id`

1.  `thread_id`

1.  `address`

1.  `person`

1.  `date`

1.  `protocol`

1.  `read`

1.  `status`

1.  `type`

1.  `reply_path_present`

1.  `subject`

1.  `body`

# 相关内容

+   开发者文档：`SmsManager` 在 [`developer.android.com/reference/android/telephony/SmsManager.html`](https://developer.android.com/reference/android/telephony/SmsManager.html)

+   **协议数据单元** (**PDU**) 在 [`en.wikipedia.org/wiki/Protocol_data_unit`](https://en.wikipedia.org/wiki/Protocol_data_unit)

+   开发者文档：`Telephony.Sms.Intents` 在 [`developer.android.com/reference/android/provider/Telephony.Sms.Intents.html`](https://developer.android.com/reference/android/provider/Telephony.Sms.Intents.html)

# 在你的应用程序中显示网页

当你想显示一个网页时，你有两个选择：调用默认浏览器或在你的应用程序中显示内容。如果你只想调用默认浏览器，使用以下 Intent：

```java
Uri uri = Uri.parse("https://www.packtpub.com/"); 
Intent intent = new Intent(Intent.ACTION_VIEW, uri); 
startActivity(intent); 
```

如果你需要在你的应用程序中显示内容，你可以使用 `WebView`。这个示例将展示如何在你的应用程序中显示一个网页，如截图所示：

![](img/bac4a050-ddd1-431b-adfa-0c824a1b451c.png)

# 准备工作

在 Android Studio 中创建一个新的项目，命名为 `WebView`。使用默认的 Phone & Tablet 选项，并在“添加一个活动到移动”对话框中选择 Empty Activity。

# 如何操作...

我们将通过代码创建 WebView，因此不会修改布局。我们首先打开 Android Manifest 并按照以下步骤进行：

1.  添加以下权限：

```java
<uses-permission android:name="android.permission.INTERNET"/>
```

1.  修改现有的 `onCreate()` 方法以包含以下代码：

```java
WebView webview = new WebView(this);
setContentView(webview);
webview.loadUrl("https://www.packtpub.com/");
```

1.  你现在可以运行应用程序在设备或模拟器上。

# 它是如何工作的...

我们创建一个 WebView 作为布局，并使用 `loadUrl()` 加载我们的网页。前面的代码是有效的，但在这一级别，它非常基础，只显示第一页。如果你点击任何链接，默认浏览器将处理请求。

# 更多内容...

如果你想要全功能的网页浏览功能，即用户点击的任何链接都仍在你的 `WebView` 中加载，创建 `WebViewClient` 如下代码所示：

```java
webview.setWebViewClient(new WebViewClient()); 
```

# 控制页面导航

如果你想要对页面导航有更多控制，你可以创建自己的 `WebViewClient` 类。如果你只想允许链接在你的网站内，则覆盖 `shouldOverrideUrlLoading()` 回调，如下所示：

```java
private class mWebViewClient extends WebViewClient {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if (Uri.parse(url).getHost().equals("www.packtpub.com")) {
            return false;  //Don't override since it's the same host 
        } else {
            return true; //Stop the navigation since it's a different 
            //site 
        }
    }
}
```

然后，使用以下代码设置客户端：

```java
webview.setWebViewClient(new mWebViewClient());
```

# 如何启用 JavaScript

我们可以通过 WebSetting 来自定义许多其他的 WebView 选项。如果你想要启用 JavaScript，从 WebView 中获取 `webSettings` 并调用 `setJavaScriptEnabled()`，如下所示：

```java
WebSettings webSettings = webview.getSettings(); 
webSettings.setJavaScriptEnabled(true); 
```

# 启用内置缩放

另一个 `webSettings` 选项是 `setBuiltInZoomControls()`。从前面的代码继续，只需添加以下内容：

```java
webSettings.setBuiltInZoomControls(true); 
```

在下一节中查看 webSettings 链接，以获取大量其他选项。

# 相关内容

+   开发者文档：`WebView` 在 [`developer.android.com/reference/android/webkit/WebView.html`](https://developer.android.com/reference/android/webkit/WebView.html)

+   开发者文档：`webSettings` 在 [`developer.android.com/reference/android/webkit/WebSettings.html`](https://developer.android.com/reference/android/webkit/WebSettings.html)

+   开发者文档：`android.webkit` 在 [`developer.android.com/reference/android/webkit/package-summary.html`](https://developer.android.com/reference/android/webkit/package-summary.html)

# 检查在线状态和连接类型

这是一个简单的配方，但也是一个非常常见的配方，可能被包含在你构建的每一个互联网应用中：检查在线状态。在检查在线状态时，我们还可以检查连接类型：`WIFI` 或 `MOBILE`。

# 准备工作

在 Android Studio 中创建一个新的项目，命名为 `isOnline`。使用默认的 `Phone & Tablet` 选项，并在“添加活动到移动”对话框中选择 Empty Activity。

# 如何做...

首先，我们需要添加必要的权限来访问网络。然后，我们将创建一个简单的布局，包含 `Button` 和 `TextView`。要开始，打开 Android Manifest 并按照以下步骤操作：

1.  添加以下权限：

```java
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

1.  打开 `activity_main.xml` 文件，并用以下内容替换现有的布局：

```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Check"
        android:layout_centerInParent="true"
        android:onClick="getStatus"/>
</RelativeLayout>
```

1.  将此方法添加到检查连接状态：

```java
private boolean isOnline() {
    ConnectivityManager connectivityManager = (ConnectivityManager)
                    getSystemService(Context.CONNECTIVITY_SERVICE);
    NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
    return (networkInfo != null && networkInfo.isConnected());
}
```

1.  添加以下方法来处理按钮点击：

```java
public void getStatus(View view) {
    TextView textView = findViewById(R.id.textView);
    if (isOnline()) {
        ConnectivityManager connectivityManager =
                (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
        textView.setText(networkInfo.getTypeName());
    } else {
        textView.setText("Offline");
    }
}
```

1.  你现在可以运行应用在设备或模拟器上。

# 它是如何工作的...

我们创建了 `isOnline()` 方法，使其易于重用此代码。

要检查状态，我们通过调用 `getType()` 来获取 `ConnectivityManager` 的实例，读取 `NetworkInfo` 状态。如果它报告我们已连接，我们将通过调用 `getType()` 获取活动网络的名称，它返回以下常量之一：

+   `TYPE_MOBILE`

+   `TYPE_WIFI`

+   `TYPE_WIMAX`

+   `TYPE_ETHERNET`

+   `TYPE_BLUETOOTH`

还可以查看后面的 `ConnectivityManager` 链接以获取其他常量。为了显示目的，我们调用 `getTypeName()`。我们也可以调用 `getType()` 来获取一个数字常量。

# 还有更多...

我们还可以设置应用在网络状态变化时被通知。

# 监控网络状态变化

如果你的应用需要响应网络状态的变化，请查看 `ConnectivityManager` 中的 `CONNECTIVITY_ACTION`。设置过滤器以通知连接状态变化事件有两种方式：

+   通过 Android Manifest

+   通过代码

下面是一个如何在 Android Manifest 中通过接收器意图过滤器包含动作的示例：

```java
<receiver android:name=".MyBroadcastReceiver"> 
    <intent-filter> 
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" /> 
    </intent-filter> 
</receiver> 
```

使用 Android Manifest 时要小心，因为它会在网络状态改变时通知你的应用，即使你的应用没有被使用。这可能导致不必要的电池消耗。

针对 Android 7.0 及以后的版本的应用将不再在 Manifest 中接收 `CONNECTIVITY_CHANGE`。（这是为了防止不必要的电池消耗）。相反，通过代码注册意图过滤器，如下所示。

更好的解决方案（并且对于 Android 7.0 及以后的版本是必需的）是通过代码注册你的意图过滤器。以下是一个示例：

```java
registerReceiver(mReceiver, new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION));
```

查看文件下载中的食谱示例，了解如何记录`CONNECTIVITY_CHANGE`事件。

# 参见

+   开发者文档：`ConnectivityManager`在[`developer.android.com/reference/android/net/ConnectivityManager.html`](https://developer.android.com/reference/android/net/ConnectivityManager.html)

+   开发者文档：`NetworkInfo`在[`developer.android.com/reference/android/net/NetworkInfo.html`](https://developer.android.com/reference/android/net/NetworkInfo.html)

# 电话号码阻止 API

Android Nougat（API 24）引入的新功能是在操作系统级别处理阻止电话号码的能力。这为用户在多个设备上提供了一致的用户体验：

+   被阻止的号码会阻止所有来电和短信

+   被阻止的号码可以使用备份和还原功能进行备份

+   设备上的所有应用程序共享相同的被阻止号码列表

在本食谱中，我们将查看如何添加一个号码到阻止列表、移除号码以及检查号码是否已被阻止的代码。

# 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为`BlockedCallList`。在`Target Android Devices`对话框中，选择`Phone & Tablet`选项，并将最小 SDK 选择为 API 24：Android 7.0 Nougat（或更高）。在`Add an Activity to Mobile`对话框中选择`Empty Activity`。

# 如何操作...

我们将首先创建一个带有`EditText`输入电话号码和三个按钮（`Block`、`Unblock`和`isBlocked`）的用户界面。首先，打开`activity_main.xml`并按照以下步骤操作：

1.  用以下 XML 代码替换现有的布局：

```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <EditText
        android:id="@+id/editTextNumber"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:inputType="phone"
        android:ems="10"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="36dp" />
    <Button
        android:id="@+id/buttonblock"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Block"
        android:layout_above="@+id/buttonUnblock"
        android:layout_centerHorizontal="true"
        android:onClick="onClickBlock"/>
    <Button
        android:id="@+id/buttonUnblock"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Block"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true"
        android:onClick="onClickUnblock"/>
    <Button
        android:id="@+id/buttonIsBlocked"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="isBlocked"
        android:layout_below="@+id/buttonUnblock"
        android:layout_centerHorizontal="true"
        android:onClick="onClickIsBlocked"/>
</RelativeLayout>
```

1.  打开`MainActivity.java`并在类声明中添加以下代码：

```java
private EditText mEditTextNumber;
```

1.  将以下代码行添加到`onCreate()`方法的末尾：

```java
mEditTextNumber=findViewById(R.id.editTextNumber);
```

1.  添加处理按钮点击的三个方法：

```java
public void onClickBlock(View view) {
    String number = mEditTextNumber.getText().toString();
    if (number!=null && number.length()>0) {
        blockNumber(number);
    }
}
public void onClickUnblock(View view) {
    String number = mEditTextNumber.getText().toString();
    if (number!=null && number.length()>0) {
        unblockNumber(number);
    }
}
public void onClickIsBlocked(View view) {
    String number = mEditTextNumber.getText().toString();
    if (number!=null && number.length()>0) {
        isBlocked(number);
    }
}
```

1.  添加以下函数以阻止号码：

```java
private void blockNumber(String number) {
    if (BlockedNumberContract.canCurrentUserBlockNumbers(this)) {
        ContentValues values = new ContentValues();
        values.put(BlockedNumberContract.BlockedNumbers.COLUMN_ORIGINAL_NUMBER, number);
        getContentResolver().insert(BlockedNumberContract.BlockedNumbers.CONTENT_URI, values);
    }
}
```

1.  添加以下函数以取消阻止号码：

```java
private void unblockNumber(String number) {
    if (BlockedNumberContract.canCurrentUserBlockNumbers(this)) {
        ContentValues values = new ContentValues();
        values.put(BlockedNumberContract.BlockedNumbers.COLUMN_ORIGINAL_NUMBER, number);
        Uri uri = getContentResolver()
                .insert(BlockedNumberContract.BlockedNumbers.CONTENT_URI, values);
        getContentResolver().delete(uri, null, null);
    }
}
```

1.  添加以下函数以检查号码是否被阻止：

```java
public void isBlocked(String number) {
    if (BlockedNumberContract.canCurrentUserBlockNumbers(this)) {
        boolean blocked = BlockedNumberContract.isBlocked(this,number);
        Toast.makeText(MainActivity.this, number + "blocked: " + blocked, 
                Toast.LENGTH_SHORT).show();
    } else {
        Toast.makeText(MainActivity.this, "User cannot perform this operation", 
                Toast.LENGTH_SHORT).show();
    }
}
```

1.  你已经准备好在至少运行 Android 7.0 的设备或模拟器上运行应用程序了。

# 它是如何工作的...

在我们调用`BlockedNumberContract` API 之前，我们通过调用`canCurrentUserBlockNumbers()`来检查我们是否有权限，就像以下代码所示：

```java
if (BlockedNumberContract.canCurrentUserBlockNumbers(this)) {
```

如果为真，我们进行实际的 API 调用。

重要：只有以下应用程序可以读取和写入`BlockedNumber`提供者：默认短信应用程序、默认电话应用程序和运营商应用程序。用户可以选择他们的默认短信和电话应用程序。

从`BlockedNumber`列表中添加和删除号码使用标准的 Service Provider 格式。

更新方法不受支持；请使用`Add`和`Delete`方法代替。

要检查一个号码是否已在阻止列表中，调用`isBlocked()`方法，传入当前上下文和要检查的号码，就像我们在以下代码中所做的那样：

```java
boolean blocked = BlockedNumberContract.isBlocked(this,number);
```

# 更多内容...

要获取所有当前被阻止号码的列表，使用以下代码创建一个带有列表的游标：

```java
Cursor cursor = getContentResolver().query(
        BlockedNumberContract.BlockedNumbers.CONTENT_URI,
        new String[]{BlockedNumberContract.BlockedNumbers.COLUMN_ID,
                BlockedNumberContract.BlockedNumbers.COLUMN_ORIGINAL_NUMBER,
                BlockedNumberContract.BlockedNumbers.COLUMN_E164_NUMBER},
        null, null, null);
```

# 参见

如需更多信息，请参阅`BlockedNumberContract`参考文档：[`developer.android.com/reference/android/provider/BlockedNumberContract`](https://developer.android.com/reference/android/provider/BlockedNumberContract)
