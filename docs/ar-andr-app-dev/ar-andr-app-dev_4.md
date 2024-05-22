# 第四章. 在世界中的定位

在上一章中，您学习了如何将数字内容叠加在物理世界的视图上。然而，如果您带着设备四处移动，将其指向其他地方，虚拟内容将始终停留在屏幕上的同一位置。这并不是 AR 中真正发生的情况。虚拟内容应该相对于物理世界保持在同一位置（你可以围绕它移动），而不是固定在屏幕上。

在本章中，我们将探讨如何实现数字内容与物理空间之间的**动态注册**。如果我们每次都更新应用程序中移动对象的位置，我们将创造出数字内容与物理世界紧密相连的感觉。跟随场景中移动元素的位置可以定义为**追踪**，这正是我们将在本章中使用和实现的内容。我们将使用基于传感器的 AR 来更新数字内容与物理空间之间的注册。由于这些传感器通常质量不佳，我们将向您展示如何使用一种名为**传感器融合**的技术来改善从它们获得的测量结果。为了更具实用性，我们将向您展示如何开发一个简单的原型，这是最常见基于全局追踪的 AR 应用程序的基本构建块：一个 AR 浏览器（例如 Junaio、Layar 或 Wikitude）。

# 知道你的位置——处理 GPS

在本节中，我们将探讨移动 AR 和基于传感器的 AR（见第一章，*增强现实概念和工具*）的一种主要方法，该方法使用**全局追踪**。全局追踪指的是在全球参考框架（世界坐标系）中的追踪，可以涵盖整个地球。我们首先会看看位置方面，然后是手机上用于 AR 的位置传感器。我们将学习如何使用 Android API 从中获取信息，并将其位置信息整合到 JME 中。

## GPS 和 GNSS

因此，我们需要追踪用户的位置，以了解他在现实世界中的位置。当我们说追踪用户时，手持 AR 应用程序实际上追踪的是设备的位置。

### 注意

**用户追踪与设备追踪**

要创建一个完全沉浸式的 AR 应用程序，理想情况下，您需要知道设备的位置，用户相对于设备的位置，以及用户眼睛相对于身体的位置。这种方法在过去已经被探索过，尤其是在头戴式显示器中。为此，您需要追踪用户头部、身体，并拥有它们之间的所有静态变换（校准）。在移动 AR 中，我们离这还很远；也许将来，用户会佩戴或穿着配备传感器的眼镜或衣物，这将允许创建更精确的注册和追踪。

那么我们如何在一个全球坐标系统中追踪设备的位置呢？你或者你的朋友们肯定使用过 GPS 进行汽车导航、跑步或远足。GPS 是一种用于全球追踪的常见技术，参照地球坐标系统，如下面的图所示：

![GPS 和 GNSS](img/8553_04_01.jpg)

现在大多数手机都配备了 GPS，因此它似乎成为 AR 中全球追踪的理想技术。GPS 是美国版的**全球导航卫星系统**（**GNSS**）。这项技术依赖于一系列地理参考卫星，它们可以使用地理坐标在全球任何地方给出你的位置。GPS 并不是唯一的 GNSS，一个俄罗斯版本（**GLONASS**）目前也在运行中，而一个欧洲版本（**Galileo**）将在 2020 年左右生效。然而，GPS 是目前在移动设备上支持最广泛的 GNSS，因此在本书的其余部分，当我们讨论使用 GNSS 进行追踪时，我们将使用这个术语。

对于依赖 GPS 的常见 AR 应用，你需要考虑两件事：数字内容的位置和设备的位置。如果它们都在参照地球的同一坐标系统中定义，你将能够了解它们相对于彼此的位置（见下面图中的椭圆形图案）。有了这些知识，你可以在用户坐标系统中建模 3D 内容的位置，并通过 GPS 传感器的每次位置更新来更新它。因此，如果你向一个物体移动（从下到上），物体将显得更近（在图像中也更大），复现你在现实世界中的行为。

![GPS 和 GNSS](img/8553_04_02.jpg)

我们在使用这项技术时遇到的一个小问题是与 GPS 使用的坐标系统有关。使用纬度和经度坐标（基本 GPS 提供的数据）并不是使用 AR 的最适应表示。当我们进行 3D 图形处理时，我们习惯于使用欧几里得坐标系统来定位数字内容；使用笛卡尔坐标系统来定义位置，即 X、Y 和 Z 坐标。因此，我们需要通过将这些 GPS 坐标转换成更适应的形式来解决这一问题。

## JME 和 GPS——追踪你的设备位置

谷歌的 Android API 通过位置管理器服务提供了对 GPS 的访问。位置管理器可以提供你 GPS 数据，但它也可以使用网络（例如 Wi-Fi 和手机网络）来精确你的位置，并给出一个大致的估计。在 Android 术语中，这被称为位置提供者。要使用位置管理器，你需要应用基于监听器类的 Android 通知的标准 Android 机制；在这种情况下是`LocationListener`。

所以打开与本章关联的`LocationAccessJME`项目，这是`SuperimposeJME`项目（第三章，*覆盖世界*）的修改版本。

首先，我们需要修改 Android 清单，以允许访问 GPS 传感器。关于 GPS 有不同的质量模式（估计位置的质量），我们将授权所有这些模式。因此，请将这两个权限添加到您的`AndroidManifest.xml`文件中：

```java
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

该项目与之前一样，有一个 JME 类（`LocationAccessJME`），一个活动类（`LocationAccessJMEActivity`），以及`CameraPreview`。我们需要做的是创建一个`LocationListener`类和一个`LocationManager`类，并将它们添加到我们的`LocationAccessJMEActivity`类中：

```java
private LocationManager locationManager;
```

在`LocationListener`类中，我们需要重写不同的回调函数：

```java
private LocationListener locListener= new LocationListener() {
  …

  @Override
  public void onLocationChanged(Location location) {
    Log.d(TAG, "onLocation: " + location.toString());
    if ((com.ar4android.LocationAccessJME) app != null) {
      ((com.ar4android.LocationAccessJME) app)
        .setUserLocation(xyzposition);
    }
  }
  …
}
```

`onLocationChanged`回调是当用户位置发生变化时的调用；位置参数包含测量的纬度和经度（以度为单位）。为了将转换后的数据传递给我们的 JME，我们将使用与前一个相同的原则：使用位置作为参数调用 JME 类中的方法。因此，每次用户位置更新时都会调用`setUserLocation`，新的值将对 JME 类可用。

接下来，我们需要访问定位管理服务，并使用`requestLocationUpdates`函数向其注册我们的位置监听器：

```java
  public void onResume() {
    super.onResume();
    …
    locationManager = (LocationManager)getSystemService(LOCATION_SERVICE);
    locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 500, 0, locListener);
  }
```

`requestLocationUpdates`方法的参数包括我们想要使用的定位提供者类型（GPS 或网络），更新频率（以毫秒为单位），以及作为我们监听器的位置变化阈值（以米为单位）。

在 JME 方面，我们需要为我们的`LocationAccessJME`类定义两个新变量：

```java
  //the User position which serves as intermediate storage place for the Android
  //Location listener position update
  private Vector3f mUserPosition;

  //A flag indicating if a new Location is available
  private boolean mNewUserPositionAvailable =false;
```

我们还需要定义我们的`setUserLocation`函数，该函数从`LocationListener`中的回调中被调用：

```java
    public void setUserLocation(Vector3f location) {
    if (!mSceneInitialized) {
      return;
    }
    WSG84toECEF(location,mUserPosition);
    //update your POI location in reference to the user position
    ….
    mNewUserPositionAvailable =true;
  }
```

在这个函数中，我们需要将摄像头的位置从纬度/经度格式转换成笛卡尔坐标系。实现这一转换有不同的技术；我们将使用 SatSleuth 网站上的转换算法（[`www.satsleuth.com/GPS_ECEF_Datum_transformation.htm`](http://www.satsleuth.com/GPS_ECEF_Datum_transformation.htm)），将我们的数据转换为**ECEF**（**地球中心，地球固定**）格式。现在我们的 JME 类中有了以 ECEF 格式可用的`mUserPosition`。每次用户位置发生变化时，都会调用`onLocationChange`方法和`setUserLocation`，我们将得到`mUserPosition`的更新值。现在的问题是我们如何将这个变量用于我们的场景图并与地理参考数字内容（例如，POI）相关联？

使用的办法是从您的当前位置本地引用内容。为此，我们需要使用一个附加的坐标系：**东-北-上（ENU）**坐标系。对于您拥有的每个数据（例如，从您位置出发 5 公里半径内的若干个兴趣点），您需要从当前的位置计算其位置。下面让我们看看如何对我们的忍者模型进行这样的操作，如下面的代码所示：

```java
    Vector3f ECEFNinja=new Vector3f();
    Vector3f ENUNinja=new Vector3f();
    WSG84toECEF(locationNinja,ECEFNinja);
    ECEFtoENU(location,mUserPosition,ECEFNinja,ENUNinja);
    mNinjaPosition.set(ENUNinja.x,0,ENUNinja.y);
```

忍者位置的纬度-经度格式（`locationNinja`）也被转换成地心地固坐标系（`ECEFNinja`）格式。从那里，使用当前的 GPS 位置（以纬度-经度格式和地心地固坐标系格式，即位置`mUserPosition`），我们计算忍者在本地方坐标系（`ENUNinja`）中的位置。每次用户移动时，他的 GPS 位置将被更新，转换为地心地固坐标系格式，并且将更新内容的本地位置，这将触发不同的渲染。就是这样！我们已经实现了基于 GPS 的追踪。不同坐标系之间的关系说明如图所示：

![JME 和 GPS——追踪您设备的位置](img/8553_04_03.jpg)

剩下的唯一部分就是使用新的本地位置来更新模型的位置。我们可以通过在`simpleUpdate`函数中添加以下代码来实现这一点：

```java
    if (mNewUserPositionAvailable) {
      Log.d(TAG,"update user location");
      ninja.setLocalTranslation(mNinjaPosition.x+0.0f,mNinjaPosition.y-2.5f,mNinjaPosition.z+0.0f);
      mNewUserPositionAvailable=false;
    }
```

在一个真正的 AR 应用中，您可能会在 GPS 坐标系中以您当前位置为中心放置一些 3D 内容，例如在纽约第五街放置一个虚拟的忍者，或者在巴黎的埃菲尔铁塔前。

由于我们希望确保无论您在何处进行测试和阅读本书（从纽约到廷巴克图），都可以独立运行这个示例。出于教育目的，我们将稍微修改这个演示。我们将要做的是在`setUserLocation`中添加以下调用，在您的初始 GPS 位置 10 米处添加忍者模型（即第一次 GPS 更新时）：

```java
    if (firstTimeLocation) {
      //put it at 10 meters
      locationNinja.setLatitude(location.getLatitude()+0.0001);
      locationNinja.setLongitude(location.getLongitude());
      firstTimeLocation=false;
    }
```

测试时间：将应用程序部署到您的移动设备上，并前往一个可以获得良好 GPS 信号的位置（您应该能够看到天空，并避免在非常多云的日子进行测试）。不要忘记在设备上激活 GPS。启动应用程序，四处移动，您应该能看到忍者位置的变化。恭喜您，您已经开发出了您的第一个增强现实（AR）应用的追踪实例！

# 了解您的注视方向——处理惯性传感器

通过前面的示例和访问 GPS 位置，我们现在可以更新用户的位置，并能够在增强现实中进行基本的追踪。然而，这种追踪只考虑了用户的位置，而没有考虑他的方向。例如，如果用户旋转手机，将不会发生任何变化，只有在他移动时变化才有效。为此，我们需要能够检测用户方向的变化；这时就需要用到惯性传感器。惯性传感器可以用来检测方向的变化。

## 理解传感器

在当前一代的手机中，有三种类型的传感器可用于定位：

+   **加速度计**：这些传感器检测手机的正加速度，也称为**g 力**加速度。手机通常配备有多轴模型，可以提供三个轴上的加速度：手机的俯仰、翻滚和倾斜。它们是手机上最早使用的传感器，用于基于传感器的游戏，生产成本低廉。通过加速度计和一点基础物理知识，你可以计算出手机的方向。然而，它们相当不准确，测量数据非常嘈杂（可能导致你的 AR 应用出现抖动）。

+   **磁力计**：它们可以检测地球的磁场，就像指南针一样。理想情况下，你可以通过测量三维磁场来获取北方方向，从而知道手机指向哪里。磁力计的挑战在于，它们很容易受到周围金属物体的影响，比如用户手腕上的手表，进而指示出错误的北方方向。

+   **陀螺仪**：它们通过**科里奥利效应**测量角速度。手机中使用的是**多轴微型机械系统**（**MEMS**），采用振动机制。它们比之前的传感器更准确，但主要问题是漂移：测量精度随时间降低；短时间后，测量开始变得非常不准确。

你可以将它们的测量值结合起来，以解决它们的局限性，我们将在本章后面看到。在手机使用之前，惯性传感器已经被广泛使用，最著名的应用是在飞机上测量其方向或速度，用作**惯性测量单元**（**IMU**）。由于制造商总是试图降低成本，不同移动设备之间的传感器质量差异很大。噪声、漂移和不准确的影响将导致你的 AR 内容在你没有移动手机的情况下跳跃或移动，或者可能导致内容定位在错误的方向。如果你想要商业部署你的应用，确保你测试了它们的一系列性能。

## JME 中的传感器

在 Google Android API 中，通过`SensorManager`访问传感器，并使用`SensorListener`获取测量值。`SensorManager`不仅提供对惯性传感器的访问，还提供对所有手机上传感器的访问。在 Android API 中，传感器分为三类：运动传感器、环境传感器和位置传感器。加速度计和陀螺仪被定义为运动传感器；磁力计被定义为位置传感器。Android API 还实现了一些软件传感器，这些传感器结合了这些不同传感器的值（可能包括位置传感器）来提供运动和方向信息。可用的五个运动传感器包括：

+   `TYPE_ACCELEROMETER`

+   `TYPE_GRAVITY`

+   `TYPE_GYROSCOPE`

+   `TYPE_LINEAR_ACCELERATION`

+   `TYPE_ROTATION_VECTOR`

请参考谷歌开发者安卓网站[`developer.android.com/guide/topics/sensors/sensors_overview.html`](http://developer.android.com/guide/topics/sensors/sensors_overview.html)，了解更多关于它们各自特性的信息。现在让我们打开`SensorAccessJME`项目。像之前一样，我们定义了一个`SensorManager`类，并为每个这些运动传感器添加一个`Sensor`类：

```java
  private SensorManager sensorManager;
  Sensor rotationVectorSensor;
  Sensor gyroscopeSensor;
  Sensor magneticFieldSensor;
  Sensor accelSensor;
  Sensor linearAccelSensor; 
```

我们还需要定义`SensorListener`，它将处理来自运动传感器的任何传感器变化：

```java
private SensorEventListener sensorListener = new SensorEventListener() {    
    …
@Override
public void onSensorChanged(SensorEvent event) {
  switch(event.sensor.getType()) {
      …
      case Sensor.TYPE_ROTATION_VECTOR:
  float[] rotationVector = {event.values[0],event.values[1], event.values[2]};
  float[] quaternion = {0.f,0.f,0.f,0.f};
  sensorManager.getQuaternionFromVector(quaternion,rotationVector);
  float qw = quaternion[0]; float qx = quaternion[1];
  float qy = quaternion[2];float qz = quaternion[3];
    double headingQ = Math.atan2(2*qy*qw-2*qx*qz , 1 - 2*qy*qy - 2*qz*qz);
  double pitchQ = Math.asin(2*qx*qy + 2*qz*qw); 
  double rollQ = Math.atan2(2*qx*qw-2*qy*qz ,1 - 2*qx*qx - 2*qz*qz);
  if ((com.ar4android.SensorAccessJME) app != null) {
  ((com.ar4android.SensorAccessJME) app).setRotation((float)pitchQ, (float)rollQ, (float)headingQ);
  }
  }
}
};
```

### 注意事项

旋转变化也可以仅使用四元数来处理，但我们明确使用欧拉角以便更直观地理解。优先使用四元数，因为它们组合旋转更容易，并且不会出现“万向节死锁”。

我们的监听器重写了两个回调：`onAccuracyChanged`和`onSensorChanged`。当我们注册到`SensorManager`的传感器有任何变化时，将调用`onSensorChanged`通道。这里我们通过查询`event.sensor.getType()`来确定是哪种类型的传感器发生了变化。对于每种类型的传感器，你可以使用生成的测量值来计算设备的新方向。在这个例子中，我们只向你展示如何使用`TYPE_ROTATION_VECTOR`传感器的值（软件传感器）。这个传感器提供的方向需要映射到与虚拟相机坐标帧相匹配。我们将欧拉角（偏航，俯仰和翻滚）传递给 JME 应用程序，在 JME 应用程序的`setRotation`函数中实现这一点（欧拉角只是方向的另一种表示，可以从传感器事件中提供的四元数和轴角表示计算得出）。

现在，有了我们的`SensorListener`，我们需要查询`SensorManager`以获取传感器服务并初始化我们的传感器。在你的`onCreate`方法中添加：

```java
  // sensor setup
  sensorManager = (SensorManager)getSystemService(SENSOR_SERVICE);
  List<Sensor> deviceSensors = sensorManager.getSensorList(Sensor.TYPE_ALL);
  Log.d(TAG, "Integrated sensors:");
  for(int i = 0; i < deviceSensors.size(); ++i ) {
    Sensor curSensor = deviceSensors.get(i);
    Log.d(TAG, curSensor.getName() + "\t" + curSensor.getType() + "\t" + curSensor.getMinDelay() / 1000.0f);
  }
initSensors();
```

在获取传感器服务访问权限后，我们查询所有可用传感器的列表，并在我们的 logcat 上显示结果。为了初始化传感器，我们调用我们的`initSensors`方法，并定义如下：

```java
    protected void initSensors(){
      //look specifically for the gyroscope first and then for the rotation_vector_sensor (underlying sensors vary from platform to platform)
    gyroscopeSensor = initSingleSensor(Sensor.TYPE_GYROSCOPE, "TYPE_GYROSCOPE");
    rotationVectorSensor = initSingleSensor(Sensor.TYPE_ROTATION_VECTOR, "TYPE_ROTATION_VECTOR");
    accelSensor = initSingleSensor(Sensor.TYPE_ACCELEROMETER, "TYPE_ACCELEROMETER");
      linearAccelSensor = initSingleSensor(Sensor.TYPE_LINEAR_ACCELERATION, "TYPE_LINEAR_ACCELERATION");
    magneticFieldSensor = initSingleSensor(Sensor.TYPE_MAGNETIC_FIELD, "TYPE_MAGNETIC_FIELD");
    }
```

`initSingleSensor`函数将创建一个`Sensor`实例，并注册我们之前创建的监听器，传递特定类型的传感器参数：

```java
    protected Sensor initSingleSensor( int type, String name ){
    Sensor newSensor = sensorManager.getDefaultSensor(type);
    if(newSensor != null){
      if(sensorManager.registerListener(sensorListener, newSensor, SensorManager.SENSOR_DELAY_GAME)) {
        Log.i(TAG, name + " successfully registered default");
      } else {
        Log.e(TAG, name + " not registered default");
      }
    } …
    return newSensor;
    }
```

当我们退出应用程序时，我们不应该忘记注销监听器，因此按照以下方式修改你的`onStop`方法：

```java
    public void onStop() {
      super.onStop();
      sensorManager.unregisterListener(sensorListener);
    }
```

因此，我们现在在`Activity`中设置好了。在我们的`SensorAccessJME`类中，我们添加以下变量：

```java
  private Quaternion mRotXYZQ;
  private Quaternion mInitialCamRotation;
  private Quaternion mCurrentCamRotation; 
```

变量`mInitialCamRotation`保存初始摄像头方向，`mRotXYZQ`保存映射到摄像头坐标系的传感器方向，`mCurrentCamRotation`存储最终摄像头旋转，它是由`mInitialCamRotation`与`mRotXYZQ`相乘得到的。`setRotation`函数从 Android 活动中获取传感器值并将它们映射到摄像头坐标系。最后，它将当前旋转值与初始摄像头方向相乘。

```java
  public void setRotation(float pitch, float roll, float heading) {
    if (!mSceneInitialized) {
      return;
    }
    mRotXYZQ.fromAngles(pitch , roll - FastMath.HALF_PI, 0);
    mCurrentCamRotation = mInitialCamRotation.mult(mRotXYZQ);
    mNewCamRotationAvailable = true;
```

作为最后一步，我们需要将这个旋转值用于我们的虚拟摄像头，就像我们在 GPS 示例中所做的那样。在`simpleUpdate`中，你现在需要添加：

```java
    if (mNewCamRotationAvailable) {
      fgCam.setAxes(mCurrentCamRotation);
      mNewCamRotationAvailable = false;
    }
```

现在，我们准备运行应用程序。重要的是要考虑设备的自然方向，它定义了运动传感器的坐标系，并不是所有设备都相同。如果你的设备默认是纵向模式，而你将其改为横向模式，坐标系将会旋转。在我们的示例中，我们明确将设备方向设置为横向。使用此默认方向模式将你的应用程序部署到设备上。你可能需要旋转设备，以便在屏幕上看到忍者的移动，如下面的截图所示：

![JME 中的传感器](img/8553_04_04.jpg)![JME 中的传感器](img/8553_04_05.jpg)

# 改进方向追踪——处理传感器融合

基于传感器的追踪的一个限制是传感器本身。正如我们之前所介绍的，一些传感器不准确、有噪声或存在漂移。一种补偿它们各自问题的技术是将它们的值结合起来，以提高你可以获得的整体旋转。这种技术称为传感器融合。融合传感器有不同的方法，我们将使用*Paul Lawitzki*提出的方法，并提供一个在 MIT 许可下的源代码，可访问[`www.thousand-thoughts.com/2012/03/android-sensor-fusion-tutorial/`](http://www.thousand-thoughts.com/2012/03/android-sensor-fusion-tutorial/)。在本节中，我们将简要解释这项技术是如何工作的，以及如何将传感器融合集成到我们的 JME AR 应用程序中。

## 传感器融合简述

*Paul Lawitzki*提出的融合算法将加速度计、磁力计和陀螺仪传感器数据融合在一起。类似于 Android API 中的软件传感器处理方式，首先将加速度计和磁力计融合在一起，以获得绝对方向（磁力计作为指南针，为你提供真正的北方向）。为了补偿两者的噪声和不准确，使用陀螺仪。陀螺仪精确但随时间漂移，在系统中以高频率使用；加速度计和磁力计则考虑更长的周期。以下是算法的概述：

![传感器融合简述](img/8553_04_06.jpg)

你可以在*Paul Lawitzki*的网页上找到更多关于算法（补充滤波器）细节的信息。

## JME 中的传感器融合

打开`SensorFusionJME`项目。传感器融合使用了一定数量的内部变量，你在`SensorFusionJMEActivity`的开始部分声明这些变量。

```java
// angular speeds from gyro
private float[] gyro = new float[3]; …
```

还要添加算法使用的不同子程序的代码：

+   `calculateAccMagOrientation`：从加速度计和磁力计的测量值计算方向角度

+   `getRotationVectorFromGyro`：从陀螺仪角速度测量计算旋转矢量

+   `gyroFunction`：将基于陀螺仪的方向写入`gyroOrientation`

+   **两个矩阵变换函数**：`getRotationMatrixFromOrientation`和`matrixMultiplication`

处理的主要部分在`calculatedFusedOrientationTask`函数中完成。这个函数作为`TimerTask`的一部分生成新的融合方向，`TimerTask`是一个可以在特定时间安排的任务。在这个函数的末尾，我们将生成的数据传递给我们的 JME 类：

```java
  if ((com.ar4android.SensorFusionJME) app != null) {
        ((com.ar4android.SensorFusionJME) app).setRotationFused((float)(fusedOrientation[2]), (float)(-fusedOrientation[0]), (float)(fusedOrientation[1]));
      }
  }
```

传递给我们的 JME 活动桥接函数（`setRotationFused`）的参数是在欧拉角格式中定义的融合方向。

我们还需要修改我们的`onSensorChanged`回调，以调用`calculatedFusedOrientationTask`使用的子程序：

```java
public void onSensorChanged(SensorEvent event) {
  switch(event.sensor.getType()) {
  case Sensor.TYPE_ACCELEROMETER:
    System.arraycopy(event.values, 0, accel, 0, 3);
    calculateAccMagOrientation();
    break;
  case Sensor.TYPE_MAGNETIC_FIELD:  
    System.arraycopy(event.values, 0, magnet, 0, 3);
    break;
  case Sensor.TYPE_GYROSCOPE:
    gyroFunction(event)
    break;
}
```

对于我们的活动类，最后的更改是指定一个定时器的任务，指定计划速率以及首次执行前的延迟。我们在调用`initSensors`之后，在`onCreate`方法中添加这个：

```java
fuseTimer.scheduleAtFixedRate(new calculateFusedOrientationTask(), 1000, TIME_CONSTANT);
```

在 JME 方面，我们定义了一个新的桥接函数用于更新旋转（再次将传感器方向转换为虚拟相机的适当方向）：

```java
public void setRotationFused(float pitch, float roll, float heading) {
  if (!mSceneInitialized) {
    return;
  } // pitch: cams x axis roll: cams y axisheading: cams z axis
  mRotXYZQ.fromAngles(pitch + FastMath.HALF_PI , roll - FastMath.HALF_PI, 0);
  mCurrentCamRotationFused = mInitialCamRotation.mult(mRotXYZQ);
  mNewUserRotationFusedAvailable = true;
}
```

最后，我们在`simpleUpdate`中与`setRotation`一样使用这个函数，通过`fgCam.setAxes(mCurrentCamRotationFused)`更新相机方向。你现在可以部署应用程序并在你的设备上查看结果。

如果你将`LocationAccessJME`和`SensorAccessJME`示例结合起来，你现在将获得完整的 6 自由度（6DOF）跟踪，这是基于经典传感器增强现实应用的基础。

# 为你的增强现实浏览器获取内容——使用 Google Places API

在知道如何获取你的 GPS 位置和手机的方向之后，你现在可以准备将优秀的内容集成到相机的实时视图中。如果能够物理探索你周围的兴趣点，如地标和商店，岂不是很酷？现在我们将向你展示如何集成流行的基于位置的服务，如 Google Places API，以实现这一点。为了成功集成到你的应用程序中，你需要执行以下步骤：

+   查询你当前位置周围的兴趣点（POIs）

+   解析结果并提取属于 POIs 的信息。

+   在你的增强现实视图中可视化信息

在开始之前，你必须确保你的应用程序有一个有效的 API 密钥。为此，你需要一个 Google 账户。你可以通过使用你的 Google 账户登录[`code.google.com/apis/console`](https://code.google.com/apis/console)来获取。

为了测试你的应用程序，你可以使用默认项目`API Project`，或者创建一个新的。要创建新的 API 密钥，你需要：

1.  点击左侧菜单中的**服务**链接。

1.  激活 Places API 状态开关。

1.  通过点击左侧菜单中的**API 访问**菜单项，并查看**简单 API 访问**区域来获取你的密钥。

你可以在`LocationAccessJME`项目中将密钥存储在`String mPlacesKey = "<YOUR API KEY HERE>"`变量中。

接下来，我们将向你展示如何查询设备位置周围的 POI，并获得一些基本信息，例如它们的名称和位置。将这些信息集成到 AR 视图中的原则与*JME 和 GPS——追踪你的设备位置*一节中描述的原则相同。

## 查询你当前位置周围的 POI

在本章前面，你已经学习了如何获取你在世界上的当前位置（纬度和经度）。你现在可以使用这些信息来获取你周围的 POI 位置。Google Places API 允许你通过 HTTP 请求查询用户周边的地标和商家，并以 JSON 或 XML 字符串的形式返回结果。所有查询都将指向以[`maps.googleapis.com/maps/api/place/`](https://maps.googleapis.com/maps/api/place/)开头的 URL。

尽管你可以在网页浏览器中轻松地进行查询，但你会希望在你的 Android 应用程序内部发送请求并处理响应。由于调用 URL 并等待响应可能需要花费数秒钟，因此你需要以不阻塞主程序执行的方式来实现这种请求-响应处理。下面我们将展示如何使用线程来实现这一点。

在你的`LocationAccessJME`项目中，你定义了一些新的成员变量，它们负责与 Google Places API 的交互。具体来说，你创建了一个`HttpClient`来发送请求，以及一个`List<POI> mPOIs`列表，用于存储关于 POIs 的最重要信息。`POI`类是一个简单的帮助类，用于存储 Google Places 引用字符串（在 Google Places 数据库中的唯一标识符，POI 名称，纬度和经度）：

```java
private class POI {
  public String placesReference;
  public String name;
  public Location location;
…
}
```

当然，你可以轻松地扩展这个类以保存其他信息，例如街道地址或图片 URL。为了查询 POI，你调用了`sendPlacesQuery`函数。我们在程序启动时进行调用，但你可以很容易地在固定时间间隔内进行（例如，当用户移动一定距离时）或明确地在按钮点击时进行。

```java
public void sendPlacesQuery(final Location location,  final Handler guiHandler) throws Exception  {
Thread t = new Thread() {
public void run() {
  Looper.prepare();
  BufferedReader in = null;
  try {
    String url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=" + location.getLatitude() + "," + location.getLongitude() + "&radius=" +  mPlacesRadius + "&sensor=true&key=" + mPlacesKey;
    HttpConnectionParams.setConnectionTimeout(mHttpClient.getParams(), 10000);
    HttpResponse response;
    HttpGet get = new HttpGet(url);
    response = mHttpClient.execute(get);
    Message toGUI = guiHandler.obtainMessage();
…
guiHandler.sendMessage(toGUI);
…
```

在此方法中，我们为每次对 Google Places 服务的查询创建一个新线程。这对于不阻塞主程序的执行非常重要。Places API 的响应应该是一个 JSON 字符串，我们将它传递给主线程中的`Handler`实例来解析 JSON 结果，接下来我们将讨论这一点。

## 解析 Google Places API 的结果

Google Places 以轻量级的 JSON 格式（XML 是另一种选择）返回结果。您可以使用作为标准 Android 包提供的`org.json`库方便地解析这些结果。

您查询的典型 JSON 结果将如下所示：

```java
{
   …
   "results" : [
      {
         "geometry" : {
            "location" : {
               "lat" : 47.07010720,
               "lng" : 15.45455070
            },
   …
         },
         "name" : "Sankt Leonhard",
         "reference" : "CpQBiQAAADXt6JM47sunYZ8vZvt0GViZDLICZi2JLRdfhHGbtK-ekFMjkaceN6GmECaynOnR69buuDZ6t-PKow-J98l2tFyg3T50P0Fr39DRV3YQMpqW6YGhu5sAzArNzipS2tUY0ocoMNHoNSGPbuuYIDX5QURVgncFQ5K8eQL8OkPST78A_lKTN7icaKQV7HvvHkEQJBIQrx2r8IxIYuaVhL1mOZOsKBoUQjlsuuhqa1k7OCtxThYqVgfGUGw",
         …
      },
…
}
```

在我们的处理程序`placesPOIQueryHandler`的`handleMessage`中，我们将解析这个 JSON 字符串到一个 POI 列表，然后可以在您的 AR 视图中进行可视化：

```java
public void handleMessage(Message msg) {
  try {
    JSONObject response = new JSONObject(msg.obj.toString());
    JSONArray results = response.getJSONArray("results");
    for(int i = 0; i < results.length(); ++i) {
      JSONObject curResult = results.getJSONObject(i);
      String poiName = curResult.getString("name");
      String poiReference = curResult.getString("reference");
      double lat = curResult.getJSONObject("geometry").getJSONObject("location").getDouble("lat");
      double lng = curResult.getJSONObject("geometry").getJSONObject("location").getDouble("lng");
      Location refLoc = new Location(LocationManager.GPS_PROVIDER);
      refLoc.setLatitude(lat);
      refLoc.setLongitude(lng);
      mPOIs.add(new POI(poiReference, poiName, refLoc));
      …
    }
  …
  }
} 
```

就这样。现在您已经有了基本的 POI 信息，并且有了纬度和经度信息，您可以在 JME 中轻松实例化新的 3D 对象，并将它们相对于相机位置正确地定位，就像您对忍者所做的那样。您还可以查询有关 POI 的更多详细信息，或者根据各种标准对它们进行过滤。有关 Google Places API 的更多信息，请访问[`developers.google.com/places/documentation/`](https://developers.google.com/places/documentation/)。

### 提示

如果您想在 3D 场景中包含文本，我们建议避免使用 3D 文本对象，因为它们会导致需要渲染的额外多边形数量增多。您可以使用位图文本替代，将其渲染为可以在网格上生成的纹理。

# 总结

在本章中，我们向您介绍了移动 AR 的第一种流行方法：基于 GPS 和传感器的增强现实。我们介绍了跟踪设备在全球参考框架中的位置的基本构建块，动态确定设备方向，提高方向跟踪的鲁棒性，并最终使用流行的 Google Places API 获取关于用户周围 POI 的信息，然后可以将这些信息集成到 AR 视图中。

在下一章中，我们将向您介绍实现移动 AR 的第二种流行方式：基于计算机视觉的增强现实。
