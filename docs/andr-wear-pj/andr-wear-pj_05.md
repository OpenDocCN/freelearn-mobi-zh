# 第五章：测量您的健康状态并同步收集的传感器数据

在上一章中，我们构建了一个提醒我们喝水的 Wear 应用，并通过内置的 Wear 传感器检查步数和心率。Wear 和移动应用的理念是提高浏览的便捷性，不错过任何重要信息，而 Upbeat 项目看似功能不多，但在用户的腕上和口袋中占有一席之地。目前 upbeat Wear 应用的功能仅限于显示从传感器接收到的数据。在本章中，我们将通过 Wear 和移动应用的互操作性来增强这个应用。我们将通过`RealmDB`持久化所有传输的数据。我们将从移动设备向 Wear 发送通知，启动应用以检查心率。我们将在 Wear 应用中拥有健康提示和食物卡路里卡片列表。

在本章中，我们将探讨以下内容：

+   收集 Wear 传感器的数据

+   处理接收到的数据以查找卡路里和距离

+   `WearableListenerService`和消息传递 API

+   从移动应用向 Wear 应用发送数据

+   `RealmDB`集成

+   使用`CardView`的`WearableRecyclerview`

# 收集 Wear 传感器的数据

从 Wear 设备收集传感器数据需要一个通信机制，而 Wearable DataLayer API 作为 Google Play 服务的一部分，在通信过程中扮演着重要角色。我们将在后续课程中深入探讨通信过程，但本章我们需要在移动应用中接收传感器数据。我们已经创建了一个项目，其中已经包含移动模块和简单的“Hello World”样板代码。当我们从 Wear 应用设置数据发送机制后，我们将处理移动模块。让我们从 Wear 模块的服务包中的步数传感器开始，进入`WearStepService`类。我们已经构建了这个服务，用于发送通知并监听步数计数器数据。现在，在`GoogleApiClient`和 Wear 消息传递 API 的帮助下，我们需要将数据发送到移动应用。

在`WearStepService`类中，在类的全局范围内实例化`GoogleApiClient`：

```java
GoogleApiClient mGoogleApiClient;

```

在`onStartCommand`中，调用初始化`mGoogleApiClient`的方法：

```java
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    Log.d(TAG, "onStartCommand");

    getSensorManager();
    getCountSensor();
 getGoogleClient();

    return super.onStartCommand(intent, flags, startId);
}

```

为了初始化`GoogleClient`，我们将使用`GoogleClient`的构建器模式，并需要添加`Wearable.API`。然后，我们可以使用`connect()`方法后跟构建器的`build()`方法连接`GoogleClient`：

```java
private void getGoogleClient() {
    if (null != mGoogleApiClient)
        return;

    Log.d(TAG, "getGoogleClient");
    mGoogleApiClient = new GoogleApiClient.Builder(this)
            .addApi(Wearable.API)
            .build();
    mGoogleApiClient.connect();
}

```

在`WearStepService`类中，我们将重写属于`IBinder`接口的`onBind`方法。我们可以通过以下方式使用它进行远程服务的客户端交互：

```java
@Override
public IBinder onBind(Intent intent) {
    return null;
}

```

我们返回 null，因为发送数据后我们不希望有任何返回。如果我们希望返回某些信息，那么我们可以按以下方式返回`IBinder`实例：

```java
private final IBinder mBinder = new LocalBinder();

@Override
public IBinder onBind(Intent intent) {
 return mBinder;
}

```

为了能够向移动设备发送数据，我们需要两个 DataLayer API 机制，即可穿戴节点和消息 API。我们将使用 Node API 获取连接的节点。使用消息 API，我们将数据发送到特定路径，而在接收端，我们应该监听该路径以获取数据。

在 Node API 中，我们将有`Resultcallback`类，它返回`ConnectedNodes`的列表，我们必须实现具有返回连接节点列表能力的`onResult`方法。我们可以向所有连接的节点发送消息，或者只向已连接的节点发送。我们可以使用节点类的`getDisplayname`获取连接节点的名称，如下所示：

```java
node.getDisplayName();

```

目前，我们将使用 Node 和 Message API 并将数据发送到连接的节点：

```java
private void sendData(){

    if (mGoogleApiClient == null)
        return;

    // use the api client to send the heartbeat value to our handheld
    final PendingResult<NodeApi.GetConnectedNodesResult> nodes = 
    Wearable.NodeApi.getConnectedNodes(mGoogleApiClient);
    nodes.setResultCallback(new 
    ResultCallback<NodeApi.GetConnectedNodesResult>() {
        @Override
        public void onResult(NodeApi.GetConnectedNodesResult result) {
            final List<Node> nodes = result.getNodes();
            final String path = "/stepcount";
            String Message = StepsTaken.getSteps()+"";

            for (Node node : nodes) {
                Log.d(TAG, "SEND MESSAGE TO HANDHELD: " + Message);
                node.getDisplayName();
                byte[] data = Message.getBytes(StandardCharsets.UTF_8);
                Wearable.MessageApi.sendMessage(mGoogleApiClient, 
                node.getId(), path, data);
            }
        }
    });
}

```

在上一个方法中，我们将使用`PendingResults`类来获取连接节点的结果。在我们收到连接节点的列表后，我们可以使用`wearableMessageApi`类发送消息。别忘了发送和接收数据到相同的路径。

# 完成的 WearStepService 类

完整的`WearStepService`类代码如下：

```java
public class WearStepService extends Service implements SensorEventListener {

    public static final String TAG = "WearStepService";
    private static final long THREE_MINUTES = 3 * 60 * 1000;
    private static final String STEP_COUNT_PATH = "/step-count";
    private static final String STEP_COUNT_KEY = "step-count";
    private SensorManager sensorManager;
    private Sensor countSensor;

    GoogleApiClient mGoogleApiClient;

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate");
        setAlarm();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand");

        getSensorManager();
        getCountSensor();
        getGoogleClient();

        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    private void getGoogleClient() {
        if (null != mGoogleApiClient)
            return;

        Log.d(TAG, "getGoogleClient");
        mGoogleApiClient = new GoogleApiClient.Builder(this)
                .addApi(Wearable.API)
                .build();
        mGoogleApiClient.connect();
    }

    /**
     * if the countSensor is null, try initializing it, and try               
     registering it with sensorManager
     */
    private void getCountSensor() {
        if (null != countSensor)
            return;

        Log.d(TAG, "getCountSensor");
        countSensor = 
        sensorManager.getDefaultSensor(Sensor.TYPE_STEP_COUNTER);
        registerCountSensor();
    }

    /**
     * if the countSensor exists, then try registering
     */
    private void registerCountSensor() {
        if (countSensor == null)
            return;

        Log.d(TAG, "sensorManager.registerListener");
        sensorManager.registerListener(this, countSensor, 
        SensorManager.SENSOR_DELAY_UI);
    }

    /**
     * if the sensorManager is null, initialize it, and try registering 
     the countSensor
     */
    private void getSensorManager() {
        if (null != sensorManager)
            return;

        Log.d(TAG, "getSensorManager");
        sensorManager = (SensorManager) 
        getSystemService(Context.SENSOR_SERVICE);
        registerCountSensor();
    }

    private void setAlarm() {
        Log.d(TAG, "setAlarm");

        Intent intent = new Intent(this, AlarmNotification.class);
        PendingIntent pendingIntent = 
        PendingIntent.getBroadcast(this.getApplicationContext(), 
        234324243, intent, 0);
        AlarmManager alarmManager = (AlarmManager) 
        getSystemService(ALARM_SERVICE);
        long firstRun = System.currentTimeMillis() + THREE_MINUTES;
        alarmManager.setInexactRepeating(AlarmManager.RTC_WAKEUP, 
        firstRun, THREE_MINUTES, pendingIntent);
    }

    @Override
    public void onSensorChanged(SensorEvent event) {
        if (event.sensor.getType() == Sensor.TYPE_STEP_COUNTER)
            StepsTaken.updateSteps(event.values.length);
        Log.d(TAG, "onSensorChanged: steps count is" + 
        event.values.length);
//        sendToPhone();
        sendData();
        updateNotification();
    }

    private void sendData(){

        if (mGoogleApiClient == null)
            return;

        // use the api client to send the heartbeat value to our 
        handheld
        final PendingResult<NodeApi.GetConnectedNodesResult> nodes = 
        Wearable.NodeApi.getConnectedNodes(mGoogleApiClient);
        nodes.setResultCallback(new 
        ResultCallback<NodeApi.GetConnectedNodesResult>() {
            @Override
            public void onResult(NodeApi.GetConnectedNodesResult 
            result) {
                final List<Node> nodes = result.getNodes();
                final String path = "/stepcount";
                String Message = StepsTaken.getSteps()+"";

                for (Node node : nodes) {
                    Log.d(TAG, "SEND MESSAGE TO HANDHELD: " + Message);
                    node.getDisplayName();
                    byte[] data = 
                    Message.getBytes(StandardCharsets.UTF_8);
                    Wearable.MessageApi.sendMessage(mGoogleApiClient, 
                    node.getId(), path, data);
                }
            }
        });
    }

    private void updateNotification() {
        // Create a notification builder that's compatible with 
        platforms >= version 4
        NotificationCompat.Builder builder =
                new NotificationCompat.Builder
                (getApplicationContext());

        // Set the title, text, and icon
        builder.setContentTitle(getString(R.string.app_name))
                .setSmallIcon(R.drawable.ic_step_icon);

        builder.setContentText("steps: " + StepsTaken.getSteps());

        // Get an instance of the Notification Manager
        NotificationManager notifyManager = (NotificationManager)
                getSystemService(Context.NOTIFICATION_SERVICE);

        // Build the notification and post it
        notifyManager.notify(0, builder.build());
    }

    @Override
    public void onAccuracyChanged(Sensor sensor, int accuracy) {
        // drop these messages
        updateNotification();

    }
}

```

我们成功地向提到的路径发送了消息。现在，让我们看看如何从 Wear 设备接收消息。在移动模块内部，为代码可读性创建额外的包。我们将把包命名为 models、services 和 utils。

是时候创建一个扩展了`WearableListenerService`并覆盖了`onMessageReceived`方法的类了。创建一个名为`StepListener`的类，并让它扩展`WearableListenerService`；代码如下：

```java
public class StepListner extends WearableListenerService {

    private static final String TAG = "StepListner";

    @Override
    public void onMessageReceived(MessageEvent messageEvent) {
        if (messageEvent.getPath().equals("/stepcount")) {
            final String message = new String(messageEvent.getData());
            Log.v(TAG, "Message path received from wear is: " + 
            messageEvent.getPath());
            Log.v(TAG, "Message received on watch is: " + message);

            // Broadcast message to wearable activity for display
            Intent messageIntent = new Intent();
            messageIntent.setAction(Intent.ACTION_SEND);
            messageIntent.putExtra("message", message);
            LocalBroadcastManager.getInstance(this)
            .sendBroadcast(messageIntent);
        }
        else {
            super.onMessageReceived(messageEvent);
        }
    }
}

```

在清单文件中使用与发送数据相同的路径注册之前的服务类，以下代码说明了可穿戴设备的`DATA_CHANGED`和`MESSAGE_RECEIVED`动作以及数据路径：

```java
<service android:name=".services.StepListner">
    <intent-filter>
        <action android:name=
        "com.google.android.gms.wearable.DATA_CHANGED" />
        <action android:name=
        "com.google.android.gms.wearable.MESSAGE_RECEIVED" />

        <data
            android:host="*"
            android:pathPrefix="/stepcount"
            android:scheme="wear" />
    </intent-filter>
</service>

```

`Steplistner`类已完成；我们可以使用这个类进一步处理数据。在`steplistener`类中，我们注册了`localbroadcast`接收器类，以在广播接收器的作用域内发送接收到的数据。在我们构建用户界面之前，我们将在移动应用程序的`MainActivity`中接收所有数据。编写一个内部类以读取收到的步骤：

```java
public class StepReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            String message = intent.getStringExtra("message");
            Log.v("steps", "Main activity received message: " + 
            message);

        // Shows the step counts received by the wearlistener //service        
            mSteps.setText("Steps:"+ message);
            int value = Integer.valueOf(message);
      }
    }

```

使用以下代码在`oncreate`方法中注册该类：

```java
// Register the local broadcast receiver
IntentFilter StepFilter = new IntentFilter(Intent.ACTION_SEND);
StepReceiver StepReceiver = new StepReceiver();
LocalBroadcastManager.getInstance(this).registerReceiver(StepReceiver, StepFilter);

```

我们成功收集了步数计数器数据。让我们用同样的过程来收集脉搏速率。我们将持久化步数计数，稍后，我们将实时传输连接节点的心跳脉搏速率的实时流。

# 切换回 Wear 模块

将项目范围切换到 Wear 模块，并选择`HeartRateFragment`以实例化`GoogleClient`对象：

```java
private GoogleApiClient mGoogleApiClient;

```

在`oncreate`方法中初始化`GoogleClient`实例，如下所示：

```java
mGoogleApiClient = new GoogleApiClient.Builder(getActivity()).
addApi(Wearable.API).
build();

mGoogleApiClient.connect();

```

编写一个将脉搏速率计数发送到连接节点的方法，就像我们之前为步数计数器所做的那样：

```java
private void sendMessageToHandheld(final String message) {

    if (mGoogleApiClient == null)
        return;

    // use the api client to send the heartbeat value to our handheld
    final PendingResult<NodeApi.GetConnectedNodesResult> nodes = 
    Wearable.NodeApi.getConnectedNodes(mGoogleApiClient);
    nodes.setResultCallback(new 
    ResultCallback<NodeApi.GetConnectedNodesResult>() {
        @Override
        public void onResult(NodeApi.GetConnectedNodesResult result) {
            final List<Node> nodes = result.getNodes();
            final String path = "/heartRate";

            for (Node node : nodes) {
                Log.d(TAG, "SEND MESSAGE TO HANDHELD: " + message);

                byte[] data = message.getBytes(StandardCharsets.UTF_8);
                Wearable.MessageApi.sendMessage(mGoogleApiClient, 
                node.getId(), path, data);
            }
        }
    });
}

```

在`onSensorchanged`回调内部调用方法，并从传感器事件触发中接收 BPM 计数：

```java
sendMessageToHandheld(currentValue.toString());

```

切换到移动项目范围。我们需要一个`WearableListenerService`类来与心率数据进行通信：

```java
public class HeartListener extends WearableListenerService {

    @Override
    public void onMessageReceived(MessageEvent messageEvent) {

    }

}

```

在`onMessageReceived`回调中注册一个`localbroadcast`事件，以在活动中接收数据。完整的监听器类代码如下：

```java
public class HeartListener extends WearableListenerService {

    @Override
    public void onMessageReceived(MessageEvent messageEvent) {
        if (messageEvent.getPath().equals("/heartRate")) {
            final String message = new String(messageEvent.getData());
            Log.v("pactchat", "Message path received on watch is: " + 
            messageEvent.getPath());
            Log.v("packtchat", "Message received on watch is: " + 
            message);

            // Broadcast message to wearable activity for display
            Intent messageIntent = new Intent();
            messageIntent.setAction(Intent.ACTION_SEND);
            messageIntent.putExtra("heart", message);
            LocalBroadcastManager.getInstance(this)
            .sendBroadcast(messageIntent);
        }
        else {
            super.onMessageReceived(messageEvent);
        }
    }
}

```

在 Manifest 中注册`service`类如下：

```java
<service android:name=".services.HeartListener">
    <intent-filter>
        <action android:name=
        "com.google.android.gms.wearable.DATA_CHANGED" />
        <action android:name=
        "com.google.android.gms.wearable.MESSAGE_RECEIVED" />

        <data
            android:host="*"
            android:pathPrefix="/heartRate"
            android:scheme="wear" />
    </intent-filter>
</service>

```

在`MainActivity`中，我们将编写另一个广播接收器。我们称之为`HeartRateReceiver`：

```java
 public class HeartRateReciver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            String data = intent.getStringExtra("heart");
            Log.v("heart", "Main activity received message: " + 
            message);

            mHeart.setText(message);

        }
    }

```

在`oncreate`方法中如下注册`BroadcastReceiver`：

```java
// Register the local broadcast receiver
IntentFilter DataFilter = new IntentFilter(Intent.ACTION_SEND);
HeartRateReciver DataReceiver = new HeartRateReciver();
LocalBroadcastManager.getInstance(this).registerReceiver(DataReceiver, DataFilter);

```

我们已经成功从`HeartListener`直接接收心率数据到`broadcastreceiver`。现在，让我们处理移动项目的用户界面。我们需要保持 UI 简单而强大；以下设计涉及到与 Wear 应用的互操作性以及距离和卡路里消耗预测。

# 概念化应用程序

Upbeat 移动应用程序应显示步数和心率。Upbeat 需要向 Wear 应用发送心率请求。历史记录显示从数据库中消耗的距离和卡路里。重置将清除数据库。

着陆页：当用户打开应用时，他将看到类似于以下设计的内容：

![图片](img/00066.jpeg)

在开始设计之前，我们需要确定一些事情，比如颜色、背景等。在`res/values`目录中，打开`colors.xml`文件，并添加以下颜色值：

```java
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#607d8b</color>
    <color name="colorPrimaryDark">#34515e</color>
    <color name="colorAccent">#FFF</color>
    <color name="grey">#afaeae</color>
    <color name="white">#fff</color>
</resources>

```

创建一个名为`button_bg.xml`的`drawable`资源文件，并添加以下选择器代码以选择背景：

```java
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@color/colorPrimaryDark" 
    android:state_pressed="true"/>
    <item android:drawable="@color/grey" android:state_focused="true"/>
    <item android:drawable="@color/colorPrimary"/>
</selector>

```

在`activity_main.xml`中，根据设计和计划的功能，我们需要三个按钮和三个文本视图。我们将使用相对布局作为根容器，以下代码解释了如何操作：

```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.packt.upbeat.MainActivity">

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginBottom="102dp"
        android:layout_above="@+id/calory"
        android:layout_centerHorizontal="true">

        <TextView
            android:id="@+id/steps"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignBottom="@+id/linearLayout"
            android:layout_toStartOf="@+id/calory"
            android:layout_weight="1"
            android:text="Steps!"
            android:textColor="@color/colorPrimaryDark"
            android:textSize="30sp"
            android:textStyle="bold"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <LinearLayout
            android:layout_weight="1"
            android:id="@+id/linearLayout"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="center">

            <ImageView
                android:id="@+id/heartbeat"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignStart="@+id/heart"
                android:layout_below="@+id/heart"/>

            <TextView
                android:id="@+id/heart"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignBaseline="@+id/steps"
                android:layout_alignBottom="@+id/steps"
                android:layout_alignParentEnd="true"
                android:layout_marginEnd="25dp"
                android:text="Heart!"
                android:textColor="@color/colorPrimaryDark"
                android:textSize="30sp"
                android:textStyle="bold" />
        </LinearLayout>

    </LinearLayout>

    <TextView
        android:id="@+id/calory"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="Calories!"
        android:textColor="@color/colorPrimary"
        android:textSize="20sp"
        android:textStyle="bold"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentStart="true"
        android:orientation="horizontal">

        <android.support.v7.widget.AppCompatButton
            android:id="@+id/reset"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@drawable/button_background"
            android:elevation="5dp"
            android:gravity="center"
            android:text="Reset"
            android:textAllCaps="true"
            android:textColor="@color/white"
            android:textStyle="bold" />

        <android.support.v7.widget.AppCompatButton
            android:id="@+id/history"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@drawable/button_background"
            android:elevation="5dp"
            android:gravity="center"
            android:text="History"
            android:textAllCaps="true"
            android:textColor="@color/white"
            android:textStyle="bold" />

        <android.support.v7.widget.AppCompatButton
            android:id="@+id/pulseRequest"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@drawable/button_background"
            android:elevation="5dp"
            android:gravity="center"
            android:text="Request pulse"
            android:textAllCaps="true"
            android:textColor="@color/white"
            android:textStyle="bold" />
    </LinearLayout>
</RelativeLayout>

```

为了显示心率，我们有一个带有`Imageview`和`Textview`的`LinearLayour`，其中`imageview`将保持静态。相反，用我们在 Wear 模块中创建的`HeartBeatView`替换`imageview`，以实现心形自定义动画。让我们最后一次创建它。

在`res/values`文件夹内，添加`heartbeatview_attrs.xml`文件，并添加以下代码：

```java
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <declare-styleable name="HeartBeatView">
        <attr name="scaleFactor" format="float" />
        <attr name="duration" format="integer" />
    </declare-styleable>

</resources>

```

在 drawables 中，创建一个矢量图形 XML 文件，并在其中添加以下代码以实现心形：

```java
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportHeight="24.0"
    android:viewportWidth="24.0">
    <path
        android:fillColor="#FFFF0000"
        android:pathData="M12,21.35l-1.45,-1.32C5.4,15.36 2,12.28 2,8.5 
        2,5.42 4.42,3 7.5,3c1.74,0 3.41,0.81 4.5,2.09C13.09,3.81 
        14.76,3 16.5,3 19.58,3 22,5.42 22,8.5c0,3.78 -3.4,6.86 
       -8.55,11.54L12,21.35z"/>
</vector>

```

我们可以在 utils 包内创建一个名为`HeartBearView`的类，并添加以下代码以处理所有动画和自定义视图逻辑。关于实现的更多细节，可以参考前一章中的 Wear 模块`HeartBeatView`类：

```java
public class HeartBeatView extends AppCompatImageView {

    private static final String TAG = "HeartBeatView";

    private static final float DEFAULT_SCALE_FACTOR = 0.2f;
    private static final int DEFAULT_DURATION = 50;
    private Drawable heartDrawable;

    private boolean heartBeating = false;

    float scaleFactor = DEFAULT_SCALE_FACTOR;
    float reductionScaleFactor = -scaleFactor;
    int duration = DEFAULT_DURATION;

    public HeartBeatView(Context context) {
        super(context);
        init();
    }

    public HeartBeatView(Context context, AttributeSet attrs) {
        super(context, attrs);
        populateFromAttributes(context, attrs);
        init();
    }

    public HeartBeatView(Context context, AttributeSet attrs, int 
    defStyleAttr) {
        super(context, attrs, defStyleAttr);
        populateFromAttributes(context, attrs);
        init();
    }

    private void init() {
        //make this not mandatory
        heartDrawable = ContextCompat.getDrawable(getContext(), 
        R.drawable.ic_heart_red_24dp);
        setImageDrawable(heartDrawable);

    }

    private void populateFromAttributes(Context context, AttributeSet 
    attrs) {
        TypedArray a = context.getTheme().obtainStyledAttributes(
                attrs,
                R.styleable.HeartBeatView,
                0, 0
        );
        try {
            scaleFactor = a.getFloat(R.styleable
            .HeartBeatView_scaleFactor, DEFAULT_SCALE_FACTOR);
            reductionScaleFactor = -scaleFactor;
            duration = a.getInteger(R.styleable.HeartBeatView_duration, 
            DEFAULT_DURATION);

        } finally {
            a.recycle();
        }

    }

    /**
     * toggles current heat beat state
     */
    public void toggle() {
        if (heartBeating) {
            stop();
        } else {
            start();
        }
    }

    /**
     * Starts the heat beat/pump animation
     */
    public void start() {
        heartBeating = true;
        animate().scaleXBy(scaleFactor)
        .scaleYBy(scaleFactor).setDuration(duration)
        .setListener(scaleUpListener);
    }

    /**
     * Stops the heat beat/pump animation
     */
    public void stop() {
        heartBeating = false;
        clearAnimation();
    }

    /**
     * is the heart currently beating
     *
     * @return
     */
    public boolean isHeartBeating() {
        return heartBeating;
    }

    public int getDuration() {
        return duration;
    }

    private static final int milliInMinute = 60000;

    /**
     * set the duration of the beat based on the beats per minute
     *
     * @param bpm (positive int above 0)
     */
    public void setDurationBasedOnBPM(int bpm) {
        if (bpm > 0) {
            duration = Math.round((milliInMinute / bpm) / 3f);
        }
    }

    public void setDuration(int duration) {
        this.duration = duration;
    }

    public float getScaleFactor() {
        return scaleFactor;
    }

    public void setScaleFactor(float scaleFactor) {
        this.scaleFactor = scaleFactor;
        reductionScaleFactor = -scaleFactor;
    }

    private final Animator.AnimatorListener scaleUpListener = new 
    Animator.AnimatorListener() {

        @Override
        public void onAnimationStart(Animator animation) {
        }

        @Override
        public void onAnimationRepeat(Animator animation) {

        }

        @Override
        public void onAnimationEnd(Animator animation) {
            //we ignore heartBeating as we want to ensure the heart is 
            reduced back to original size
            animate().scaleXBy(reductionScaleFactor)
            .scaleYBy(reductionScaleFactor).setDuration(duration)
            .setListener(scaleDownListener);
        }

        @Override
        public void onAnimationCancel(Animator animation) {

        }
    };

    private final Animator.AnimatorListener scaleDownListener = new 
    Animator.AnimatorListener() {

        @Override
        public void onAnimationStart(Animator animation) {
        }

        @Override
        public void onAnimationRepeat(Animator animation) {
        }

        @Override
        public void onAnimationEnd(Animator animation) {
            if (heartBeating) {
                //duration twice as long for the upscale
                animate().scaleXBy(scaleFactor).scaleYBy(scaleFactor)
                .setDuration
                (duration * 2).setListener(scaleUpListener);
            }
        }

        @Override
        public void onAnimationCancel(Animator animation) {
        }
    };
}

```

在`activity_main.xml`文件中，用项目中创建的自定义视图替换`ImageView`代码：

```java
<com.packt.upbeat.utils.HeartBeatView
    android:id="@+id/heartbeat"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_alignStart="@+id/heart"
    android:layout_below="@+id/heart"/>

```

既然我们的着陆页用户界面已经准备好了，我们可以开始处理`MainActivity`。

在`MainActivity`中，让我们实例化我们在布局中使用的所有 UI 组件：

```java
private AppCompatButton mReset, mHistory, mHeartPulse;
private TextView mSteps, mHeart, mCalory;
private HeartBeatView heartbeat;

```

在`oncreate`方法中使用`findviewbyid`方法将组件与其 ID 进行映射：

```java
heartbeat = (HeartBeatView)findViewById(R.id.heartbeat);
mSteps = (TextView) findViewById(R.id.steps);
mHeart = (TextView) findViewById(R.id.heart);
mCalory = (TextView) findViewById(R.id.calory);
mReset = (AppCompatButton) findViewById(R.id.reset);
mHistory = (AppCompatButton) findViewById(R.id.history);
mHeartPulse = (AppCompatButton) findViewById(R.id.pulseRequest);

```

在`HeartRateReceiver`类中，获取数据，将数据转换为整数，并在 UI 中显示。以下代码说明如何使用从 Wear 应用接收的数据激活`HeartBeatAnimation`：

```java
 @Override
        public void onReceive(Context context, Intent intent) {
            String data = intent.getStringExtra("heart");
            Log.v("heart", "Main activity received message: " + data);

            mHeart.setText(data);
            heartbeat.setDurationBasedOnBPM(Integer.valueOf(data));
            heartbeat.toggle();
        }

```

在`StepReceiver`中，我们将数据设置为标记为`mSteps`的步数`textview`：

```java
mSteps.setText("Steps:"+ message);

```

我们已经完成了接收脉搏数和步数并在手机的 UI 中显示。现在，我们需要显示这些步骤消耗的卡路里。

根据您的身体质量指数等，可以通过多种不同方法从步数计算消耗的卡路里。关于计步器步数到卡路里的研究引入了一个转换因子，如下所示：

*转换因子 = 每英里 99.75 卡路里 / 每英里 2,200 步 = 每步 0.045 卡路里*

因此，使用这个值，我们可以通过简单地将这个值与步数相乘来确定卡路里。

*消耗的卡路里 = 7,000 步 x 每步 0.045 卡路里 = 318 卡路里*

在`StepReceiver`类中，在`onReceive`方法内，添加以下代码：

```java
int value = Integer.valueOf(message);
mCalory.setText(String.valueOf((int)(value * 0.045)) + "/ncalories" + "/nburnt");

```

通过手机完成的卡路里消耗和脉搏率检查。在`MainActivity`中我们还有更多工作要做。我们需要将步数计数器的数据持久化，以显示卡路里和距离的历史记录。让我们使用第一章尝试过的`RealmDB`。

将以下 classpath 添加到项目级别的 gradle 文件中：

```java
classpath "io.realm:realm-gradle-plugin:2.2.1"

```

在 gradle 移动模块中应用上一个插件：

```java
apply plugin: 'realm-android'

```

项目中的 Realm 已准备就绪。现在，我们需要为步数数据设置 setters 和 getters。将以下类添加到 models 包中：

```java
public class StepCounts extends RealmObject {

    private String ReceivedDateTime;
    private String Data;

    public String getReceivedDateTime() {
        return ReceivedDateTime;
    }

    public void setReceivedDateTime(String receivedDateTime) {
        ReceivedDateTime = receivedDateTime;
    }

    public String getData() {
        return Data;
    }

    public void setData(String data) {
        Data = data;
    }
}

```

在`MainActivity`中，实例化 Realm 并在`onCreate`方法中初始化，如下所示：

```java
private Realm realm;

@Override
protected void onCreate(Bundle savedInstanceState) {
...
Realm.init(this);
realm = Realm.getDefaultInstance();

}

```

当接收到步数时，将数据添加到`RealmDB`中。在`StepReceiver`内部类的`onReceive`方法内添加以下代码：

```java
realm.beginTransaction();
StepCounts Steps = realm.createObject(StepCounts.class);
Steps.setData(message);
String TimeStamp =  DateFormat.getDateTimeInstance().format(System.currentTimeMillis());
Steps.setReceivedDateTime(TimeStamp);
realm.commitTransaction();

```

为了在 UI 中显示最后一个值，在`onCreate`方法中添加以下代码：

```java
RealmResults<StepCounts> results = realm.where(StepCounts.class).findAll();

if(results.size() == 0){
    mSteps.setText("Steps: ");
}else{
    mSteps.setText("Steps: "+results.get(results.size()-1).getData());
    int value = Integer.valueOf(results
    .get(results.size()-1).getData());
    mCalory.setText(String.valueOf((int)(value * 0.045)) 
    + "/ncalories" + "/nburnt");
}

```

对于按钮，现在将点击监听器附加到`onCreate`方法中：

```java
mHistory.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {

    }
});

mHeartPulse.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {

    }
});

mReset.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {

    }
});

```

让我们创建另一个 Activity，并将其命名为`HistoryActivity`，它将显示接收到的数据列表。在`activity_history.xml`文件中，添加以下代码：

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.packt.upbeat.HistoryActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_margin="5dp" />

</LinearLayout>

```

现在，我们需要为`recyclerview`中的每个项目创建`row_layout`，布局如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:card_view="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <android.support.v7.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="0dp"
        android:layout_marginLeft="5dp"
        android:layout_marginRight="5dp"
        android:layout_marginTop="9dp"
        card_view:cardCornerRadius="3dp"
        card_view:cardElevation="0.01dp">

        <LinearLayout
            android:layout_margin="10dp"
            android:orientation="vertical"
            android:id="@+id/top_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <TextView
                android:layout_margin="10dp"
                android:id="@+id/steps"
                android:text="Steps"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />
            <TextView
                android:layout_margin="10dp"
                android:id="@+id/calories"
                android:text="calory"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />
            <TextView
                android:layout_margin="10dp"
                android:id="@+id/distance"
                android:text="distance"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />

            <TextView
                android:layout_margin="10dp"
                android:id="@+id/date"
                android:layout_width="match_parent"
                android:layout_height="40dp"
                android:layout_gravity="bottom"
                android:background="#ff444444"
                android:gravity="center"
                android:text="Timestamp"
                android:textColor="#fff"
                android:textSize="20dp" />

        </LinearLayout>
    </android.support.v7.widget.CardView>
</LinearLayout>

```

请记住，在使用`cardview`和`recyclerview`之前，我们需要将依赖项添加到我们的 gradle 模块中：

`compile 'com.android.support:cardview-v7:25.1.1'`

`compile 'com.android.support:recyclerview-v7:25.1.1'`

# Recyclerview 适配器

我们将不得不创建一个`adapter`类，它从 Realm 获取数据并适配到创建的`row_layout`：

```java
public class HistoryAdapter extends RecyclerView.Adapter<HistoryAdapter.ViewHolder> {

    public List<StepCounts> steps;
    public Context mContext;

    public HistoryAdapter(List<StepCounts> steps, Context mContext) {
        this.steps = steps;
        this.mContext = mContext;
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup viewGroup, int i) {
        View v = LayoutInflater.from(viewGroup.getContext())
                .inflate(R.layout.row_item, viewGroup, false);
        ViewHolder viewHolder = new ViewHolder(v);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(ViewHolder viewHolder, int i) {
        viewHolder.steps.setText(steps.get(i).getData()+" Steps");
        viewHolder.date.setText(steps.get(i).getReceivedDateTime());

        int value = Integer.valueOf(steps.get(i).getData());
        DecimalFormat df = new DecimalFormat("#.00") ;
        String kms = String.valueOf(df.format(value * 0.000762)) + " 
        kms" ;
        viewHolder.calory.setText(String.valueOf((int)(value * 0.045)) 
        + " calories " + "burnt");
        viewHolder.distance.setText("Distance: "+kms);

    }

    @Override
    public int getItemCount() {
        return steps.size();
    }

    public static class ViewHolder extends RecyclerView.ViewHolder  {

        public TextView steps,calory,distance,date;

        public ViewHolder(View itemView) {
            super(itemView);
            steps = (TextView) itemView.findViewById(R.id.steps);
            calory = (TextView) itemView.findViewById(R.id.calories);
            distance = (TextView) itemView.findViewById(R.id.distance);
            date = (TextView) itemView.findViewById(R.id.date);
        }
    }

}

```

在适配器中，我们使用转换因子值显示消耗的卡路里。为了找到通用距离，我们还有另一个值，需要将步数乘以它，如适配器所示。

在`HistoryActivity`中，在类全局范围内，声明以下实例：

```java
Realm realm;
RecyclerView mRecyclerView;
RecyclerView.LayoutManager mLayoutManager;
RecyclerView.Adapter mAdapter;

```

现在，在`HistoryActivity`类的`onCreate`方法中，添加以下代码：

```java
mRecyclerView = (RecyclerView) findViewById(R.id.recycler_view);
mRecyclerView.setHasFixedSize(true);

Realm.init(this);
realm = Realm.getDefaultInstance();
RealmResults<StepCounts> results = realm.where(StepCounts.class).findAll();
// The number of Columns
mLayoutManager = new GridLayoutManager(this, 1);
mRecyclerView.setLayoutManager(mLayoutManager);

mAdapter = new HistoryAdapter(results,HistoryActivity.this);
mRecyclerView.setAdapter(mAdapter);

```

# 完成的`HistoryActivity`类

完整的类代码如下所示：

```java
public class HistoryActivity extends AppCompatActivity {

    Realm realm;
    RecyclerView mRecyclerView;
    RecyclerView.LayoutManager mLayoutManager;
    RecyclerView.Adapter mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_history);
        // Calling the RecyclerView
        mRecyclerView = (RecyclerView) 
        findViewById(R.id.recycler_view);
        mRecyclerView.setHasFixedSize(true);

        Realm.init(this);
        realm = Realm.getDefaultInstance();
        RealmResults<StepCounts> results = 
        realm.where(StepCounts.class).findAll();
        // The number of Columns
        mLayoutManager = new GridLayoutManager(this, 1);
        mRecyclerView.setLayoutManager(mLayoutManager);

        mAdapter = new HistoryAdapter(results,HistoryActivity.this);
        mRecyclerView.setAdapter(mAdapter);
    }
}

```

在`MainActivity`中，当点击`mHistory`按钮时启动`historyActivity`：

```java
mHistory.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        startActivity(new Intent(MainActivity.this, 
        HistoryActivity.class));
    }
});

```

现在，是时候使用我们在 Wear 中使用的方法，将数据从移动设备发送到 Wear 设备了。

我们将创建一个扩展了`Thread`的类，并使用 Node 和 Message API，以下方式发送数据：

```java
class SendToDataLayerThread extends Thread {
    String path;
    String message;

    // Constructor to send a message to the data layer
    SendToDataLayerThread(String p, String msg) {
        path = p;
        message = msg;
    }

    public void run() {
        NodeApi.GetConnectedNodesResult nodes = 
        Wearable.NodeApi.getConnectedNodes(googleClient).await();
        for (Node node : nodes.getNodes()) {
            MessageApi.SendMessageResult result =  
            Wearable.MessageApi.sendMessage(googleClient, 
            node.getId(), path, message.getBytes()).await();
            if (result.getStatus().isSuccess()) {
                Log.v("myTag", "Message: {" + message + "} sent to: " + 
                node.getDisplayName());
            } else {
                // Log an error
                Log.v("myTag", "ERROR: failed to send Message");
            }
        }
    }
}

```

在`mHeartPulse`按钮点击监听器内部，按如下方式启动`SendToDataLayerThread`类：

```java
mHeartPulse.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        new SendToDataLayerThread("/heart", "Start upbeat for heart 
        rate").start();
    }
});

```

现在，切换回 Wear 项目范围，并添加一个扩展了`WearableListenerService`的新类。当它从移动应用接收到消息时，触发一个通知来启动应用程序。完整的类代码如下：

```java
public class MobileListener extends WearableListenerService {

    @Override
    public void onMessageReceived(MessageEvent messageEvent) {

        if (messageEvent.getPath().equals("/heart")) {
            final String message = new String(messageEvent.getData());
            Log.v("myTag", "Message path received on watch is: " + 
            messageEvent.getPath());
            Log.v("myTag", "Message received on watch is: " + message);

            // Broadcast message to wearable activity for display
            Intent messageIntent = new Intent();
            messageIntent.setAction(Intent.ACTION_SEND);
            messageIntent.putExtra("message", message);
            LocalBroadcastManager.getInstance(this)
            .sendBroadcast(messageIntent);

            Intent intent2 = new Intent
            (getApplicationContext(), MainActivity.class);

            PendingIntent pendingIntent = PendingIntent.getActivity
            (getApplicationContext(), 0, intent2,
                    PendingIntent.FLAG_ONE_SHOT);

            Uri defaultSoundUri = RingtoneManager.getDefaultUri
            (RingtoneManager.TYPE_ALARM);

            NotificationCompat.Builder notificationBuilder = 
            (NotificationCompat.Builder) new 
            NotificationCompat.Builder(getApplicationContext())
                    .setAutoCancel(true)   //Automatically delete the 
                    notification
                    .setSmallIcon(R.drawable.ic_heart_icon) 
                    //Notification icon
                    .setContentIntent(pendingIntent)
                    .setContentTitle("Open upbeat")
                    .setContentText("UpBeat to check the pulse")
                    .setCategory(Notification.CATEGORY_REMINDER)
                    .setPriority(Notification.PRIORITY_HIGH)
                    .setSound(defaultSoundUri);

            NotificationManagerCompat notificationManager =    
            NotificationManagerCompat.from
            (getApplicationContext());
            notificationManager.notify(0, notificationBuilder.build());

        }
        else {
            super.onMessageReceived(messageEvent);
        }
    }
}

```

现在，在清单文件中使用正确的路径注册之前提到的服务，使用以下代码为移动应用注册：

```java
<service android:name=".services.MobileListener">
    <intent-filter>
        <action android:name=
        "com.google.android.gms.wearable.DATA_CHANGED" />
        <action android:name=
        "com.google.android.gms.wearable.MESSAGE_RECEIVED" />

        <data
            android:host="*"
            android:pathPrefix="/heart"
            android:scheme="wear" />
    </intent-filter>
</service>

```

让我们切换回移动项目范围，完成重置按钮点击事件。我们将编写一个方法，该方法刷新`RealmDB`数据并重新创建活动：

```java
public void Reset(){
    RealmResults<StepCounts> results = 
    realm.where(StepCounts.class).findAll();

    realm.beginTransaction();

    results.deleteAllFromRealm();

    realm.commitTransaction();
}

```

在点击监听器内部，以下方式添加以下方法：

```java
mReset.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Reset();
        recreate();
    }
});

```

切换到 Wear 项目范围，并为健康建议创建一个新的 Activity，我们将这个活动称为`HealthTipsActivity`。在这个屏幕上，我们将列出一些好的健康建议和提示。

在`activity_health_tips.xml`中，添加以下代码：

```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.wearable.view.BoxInsetLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="5dp"
    app:layout_box="all"
    tools:deviceIds="wear">

    <android.support.wearable.view.WearableRecyclerView
        android:id="@+id/wearable_recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</android.support.wearable.view.BoxInsetLayout>

```

我们需要为建议活动添加一个更多布局的行项目。我们将这个布局称为`health_tips_row.xml`：

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:tag="cards main container">

    <android.support.v7.widget.CardView 
    xmlns:card_view="http://schemas.android.com/apk/res-auto"
        android:id="@+id/card_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        card_view:cardBackgroundColor="@color/colorPrimary"
        card_view:cardCornerRadius="10dp"
        card_view:cardElevation="5dp"
        card_view:cardUseCompatPadding="true">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="12dp"
                android:layout_weight="2"
                android:orientation="vertical">

                <TextView
                    android:id="@+id/health_tip"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_horizontal"
                    android:layout_marginTop="10dp"
                    android:text="HealthTip"
                    android:textColor="@color/white"
                    android:textAppearance="?
                    android:attr/textAppearanceLarge" />

                <TextView
                    android:id="@+id/tip_details"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_horizontal"
                    android:layout_marginTop="10dp"
                    android:text="Details"
                    android:textColor="@color/white"
                    android:textAppearance="?
                    android:attr/textAppearanceMedium" />

        </LinearLayout>

    </android.support.v7.widget.CardView>

</LinearLayout>

```

创建一个包含所需字段的模型。我们将为所有字段创建带有完整参数化构造函数的设置器和获取器：

```java
public class HealthTipsItem {

    public String Title;
    public String MoreInfo;

    public HealthTipsItem(String title, String moreInfo) {
        Title = title;
        MoreInfo = moreInfo;
    }

    public String getTitle() {
        return Title;
    }

    public void setTitle(String title) {
        Title = title;
    }

    public String getMoreInfo() {
        return MoreInfo;
    }

    public void setMoreInfo(String moreInfo) {
        MoreInfo = moreInfo;
    }
}

```

我们将有一个保存所有健康建议的另一个数据类：

```java
public class HealthTips {

   public static String[] nameArray =
           {"Food style",
                   "Food style",
                   "Food style",
                   "Drinking water",
                   "Unhealthy drinks",
                   "Alcohol and drugs",
                   "Body Mass index",
                   "Physical excercise",
                   "Physical activities",
                   "Meditation",
                   "Healthy signs"};

    public static String[] versionArray = {
            "Along with fresh vegetables and fruits, eat lean meats (if 
            you're not vegetarian), nuts, and seeds.",
            "Opt for seasonal and local products instead of those 
            exotic imported foodstuff",
            "Make sure you get a proper balanced diet, as often as 
            possible",
            "Drink water - you need to stay hydrated. It is great for 
            your internal organs, and it also keeps your skin healthy 
            and diminishes acne",
            "Stop drinking too much caffeine and caffeinated 
            beverages",
            "Limit alcohol intake. Tobacco and drugs should be a firm 
            No",
            "Maintain a healthy weight.",
            "Exercise at least four days a week for 20 to 30 minutes 
            each day. Another option is to break your workouts into 
            several sessions",
            "Try to have as much physical activity as you can. Take the 
             stairs instead of elevator; walk to the market instead of 
             taking your car etc",
            "Practice simple meditation. It balances your body, mind, 
            and soul",
            "When speaking about health tips, skin, teeth, hair, and 
            nails are all health signs. Loss of hair or fragile nails 
            might mean poor nutrition"};

}

```

现在，我们将创建一个适配器来处理健康建议列表。以下代码获取数据并在`wearablerecyclerview`中加载：

```java
public class RecyclerViewAdapter
        extends WearableRecyclerView.Adapter
        <RecyclerViewAdapter.ViewHolder> {

    private List<HealthTipsItem> mListTips = new ArrayList<>();
    private Context mContext;

    public RecyclerViewAdapter(List<HealthTipsItem> mListTips, Context 
    mContext) {
        this.mListTips = mListTips;
        this.mContext = mContext;
    }

    static class ViewHolder extends RecyclerView.ViewHolder {
        private TextView Title, info;

        ViewHolder(View view) {
            super(view);
            Title = (TextView) view.findViewById(R.id.health_tip);
            info = (TextView) view.findViewById(R.id.tip_details);
        }
    }

    @Override
    public RecyclerViewAdapter.ViewHolder onCreateViewHolder(ViewGroup 
    parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.health_tips_row, parent, false);

        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {

        holder.Title.setText(mListTips.get(position).getTitle());
        holder.info.setText(mListTips.get(position).getMoreInfo());
    }

    @Override
    public int getItemCount() {
        return mListTips.size();
    }
}

```

在活动的全局范围内，声明以下实例：

```java
private RecyclerViewAdapter mAdapter;
private List<HealthTipsItem> myDataSet = new ArrayList<>();

```

在`oncreate`方法内部，我们可以通过添加以下代码来完成应用：

```java
WearableRecyclerView recyclerView = (WearableRecyclerView) findViewById(R.id.wearable_recycler_view);
recyclerView.setHasFixedSize(true);
LinearLayoutManager mLayoutManager = new LinearLayoutManager(this);
recyclerView.setLayoutManager(mLayoutManager);

myDataSet = new ArrayList<HealthTipsItem>();
for (int i = 0; i < HealthTips.nameArray.length; i++) {
    myDataSet.add(new HealthTipsItem(
            HealthTips.nameArray[i],
            HealthTips.versionArray[i]
    ));
}

mAdapter = new RecyclerViewAdapter(myDataSet,HealthTipsActivity.this);
recyclerView.setAdapter(mAdapter);

```

让我们创建另一个 Activity，用于通用卡路里图表，从国际食物列表中调用活动`CalorychartActivity`。

在`CaloryChartActivity`布局文件中，我们将添加`WearableRecyclerView`组件：

```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.wearable.view.BoxInsetLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="5dp"
    app:layout_box="all"
    tools:deviceIds="wear">

    <android.support.wearable.view.WearableRecyclerView
        android:id="@+id/wearable_recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</android.support.wearable.view.BoxInsetLayout>

```

为每个卡路里图表项目创建另一个布局，并添加以下代码：

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:tag="cards main container">

    <android.support.v7.widget.CardView 
    xmlns:card_view="http://schemas.android.com/apk/res-auto"
        android:id="@+id/card_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        card_view:cardBackgroundColor="@color/colorPrimary"
        card_view:cardCornerRadius="10dp"
        card_view:cardElevation="5dp"
        card_view:cardUseCompatPadding="true">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="12dp"
                android:layout_weight="2"
                android:orientation="vertical">

                <TextView
                    android:id="@+id/health_tip"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_horizontal"
                    android:layout_marginTop="10dp"
                    android:text="calory"
                    android:textColor="@color/white"
                    android:textAppearance="?
                    android:attr/textAppearanceLarge" />

        </LinearLayout>

    </android.support.v7.widget.CardView>

</LinearLayout>

```

我们将以下面的方式为卡路里创建`model`类：

```java
public class CaloryItem {

    public String Calories;

    public CaloryItem(String calories) {
        Calories = calories;
    }

    public String getCalories() {
        return Calories;
    }

    public void setCalories(String calories) {
        Calories = calories;
    }
}

```

我们将创建另一个卡路里图表的适配器。该适配器与`HealthTips`适配器类似。创建一个文件`RecyclerViewCaloryAdapter`并将以下代码添加到其中：

```java
public class RecyclerViewCaloryAdapter
        extends WearableRecyclerView.Adapter<RecyclerViewCaloryAdapter.ViewHolder> {

    private List<CaloryItem> mCalory = new ArrayList<>();
    private Context mContext;

    public RecyclerViewCaloryAdapter(List<CaloryItem> mCalory, Context 
    mContext) {
        this.mCalory = mCalory;
        this.mContext = mContext;
    }

    static class ViewHolder extends RecyclerView.ViewHolder {
        private TextView Title;

        ViewHolder(View view) {
            super(view);
            Title = (TextView) view.findViewById(R.id.health_tip);
        }
    }

    @Override
    public RecyclerViewCaloryAdapter.ViewHolder 
    onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.calory_row, parent, false);

        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        holder.Title.setText(mCalory.get(position).getCalories());
    }

    @Override
    public int getItemCount() {
        return mCalory.size();
    }
}

```

在`CaloryChartActivity`项目的全局范围内，添加以下实例：

```java
private RecyclerViewCaloryAdapter mAdapter;
private List<CaloryItem> myDataSet = new ArrayList<>();

```

在`oncreate`方法内部添加以下代码：

```java
WearableRecyclerView recyclerView = (WearableRecyclerView) findViewById(R.id.wearable_recycler_view);
recyclerView.setHasFixedSize(true);
LinearLayoutManager mLayoutManager = new LinearLayoutManager(this);
recyclerView.setLayoutManager(mLayoutManager);

myDataSet = new ArrayList<CaloryItem>();
for (int i = 0; i < Calory.nameArray.length; i++) {
    myDataSet.add(new CaloryItem(
            Calory.nameArray[i]
    ));
}

mAdapter = new RecyclerViewCaloryAdapter(myDataSet,CaloryChartActivity.this);
recyclerView.setAdapter(mAdapter);

```

以下屏幕显示了完整的移动和 Wear 应用。

下图展示了监听步骤和脉搏的活动屏幕：

![](img/00067.jpeg)

下图展示了 Wear 应用中健康食物建议屏幕。它使用`WearableRecyclerView`进行设计：

![](img/00068.jpeg)

以下图像展示了用于学习不同食品项中可提供热量的热量图表：

![](img/00069.jpeg)

# 总结

在本章中，我们了解了与穿戴设备和移动应用程序协作的基础知识。我们已经探索了从穿戴设备发送和接收数据到移动设备以及反向支持的 API。现在，为任何穿戴项目集成`RealmDB`将会更加容易。

在下一章中，我们将为穿戴设备构建一个谷歌地图应用程序，并且我们将持久化位置数据，了解穿戴设备的不同地图类型和控制方法。
