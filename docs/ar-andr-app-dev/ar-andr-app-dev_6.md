# 第六章：让它互动——创建用户体验

在前面的章节中，我们已经学习了使用两种最常见的 AR 方法创建增强现实的要点：基于传感器和基于计算机视觉的 AR。我们现在能够将数字内容叠加在物理世界的视图上，支持 AR 跟踪，以及处理账户注册（在目标上或户外）。

然而，我们仅仅能在增强的世界中导航。如果能让用户以直观的方式与虚拟内容互动，岂不是很好吗？用户互动是任何应用程序开发的重要组成部分。由于我们这里专注于用户与 3D 内容（3D 交互）的互动，以下是三种主要的交互技术类别，可以加以开发：

+   **导航**：在场景中移动并选择一个特定的视角。在增强现实（AR）中，这种导航是通过物理移动来完成的（例如，在街上行走或转动桌子），并且可以辅以额外的虚拟功能（例如，地图视图，导航路径，冻结模式等）。

+   **操作**：选择、移动和修改对象。在 AR 中，这可以应用于物理和虚拟元素，通过一系列传统方法（例如，射线选择），以及新颖的交互范式（例如，有形用户界面）。

+   **系统控制**：调整应用程序的参数，包括渲染、轮询过程和依赖于应用程序的内容。在 AR 中，它可以对应于调整跟踪参数或可视化技术（例如，在 AR 浏览器中显示到您的兴趣点（POI）的距离）。

在本章中，我们将向您展示一些常用的 AR 交互技术的一个子集。我们将向您展示如何开发三种交互技术，包括射线选择、基于接近度的交互和基于 3D 运动手势的交互。这是设计 AR 应用程序的下一步，也是我们 AR 层的基本构建块（请参阅第一章，*增强现实概念和工具*）。

# 拿起棍子——使用射线选择进行 3D 选择

在台式计算机上，3D 交互使用的是一组有限的设备，包括键盘、鼠标或游戏操纵杆。在智能手机（或平板电脑）上，交互主要由触摸或传感器输入驱动。从交互输入（传感器数据，如在屏幕上的 x 和 y 坐标，或事件类型，如点击或悬停）开始，您可以开发不同的交互技术，如射线选择、转向导航等。对于移动 AR，可以使用大量交互技术进行 2D 或 3D 交互。在本节中，我们将探讨结合触摸输入和名为**射线选择**的技术。

射线拣选的概念是使用一个从你的设备到你的环境（即目标）的虚拟射线，并检测沿途它击中了什么。当你在某个对象上得到一个击中（例如，射线与你的虚拟角色之一相交），你可以认为这个对象已被拣选（选中）并开始操作它。在这里，我们只看如何在 JME 中拣选一个对象。在示例代码中，你可以扩展对象以支持进一步的操作，例如，当一个对象被击中并拣选，你可以检测滑动触摸动作并平移对象，让它爆炸，或者将击中用作某些游戏的射击射线，等等。

那么让我们开始。在 JME 中，你可以使用特定于 Android 的输入功能（通过`AndroidInput`）或与桌面应用程序相同的输入（`MouseInput`）。默认情况下，JME 在 Android 上将任何触摸事件映射为鼠标事件，这允许我们在 Android 和桌面几乎使用相同的代码。我们将为这个项目选择以下解决方案；作为一个练习，你可以尝试使用`AndroidInput`（查看`AndroidTouchInputListener`以使用`AndroidInput`）。

打开`RayPickingJME`示例。它使用与`VuforiaJME`相同的基代码，我们的拣选方法基于 JME 的一个示例，对于这种拣选方法，请访问以下链接：[`jmonkeyengine.org/wiki/doku.php/jme3:beginner:hello_picking`](http://jmonkeyengine.org/wiki/doku.php/jme3:beginner:hello_picking)。

首先要做的就是在我们的`RayPickingJME`类中添加不同的射线拣选所需的包：

```java
import com.jme3.math.Ray;
import com.jme3.collision.CollisionResult;
import com.jme3.collision.CollisionResults;
import com.jme3.input.MouseInput;
import com.jme3.input.controls.ActionListener;
import com.jme3.input.controls.KeyTrigger;
import com.jme3.input.controls.MouseButtonTrigger;
```

为了能够拣选一个对象，我们需要在`RayPicking`类的范围内声明一些全局变量：

+   `Node shootables`

+   `Geometry geom`

下一步是向我们的类中添加一个监听器。如果你从未做过 Android 或 JME 编程，你可能不知道监听器是什么。**监听器**是一种事件处理技术，可以监听类中发生的任何活动，并提供特定方法来处理任何事件。例如，如果你有一个鼠标按钮点击事件，你可以为它创建一个监听器，该监听器有一个`onPushEvent()`方法，你可以在其中安装你自己的代码。在 JME 中，事件管理和监听器被组织成两个组件，通过使用`InputManager`类进行控制。

+   **触发器映射**：使用这个你可以将设备输入与一个触发器名称关联起来，例如，点击鼠标可以与`Press`或`Shoot`或`MoveEntity`等关联。

+   **监听器**：使用这个你可以将触发器名称与特定的监听器关联起来；`ActionListener`（用于离散事件，如“按钮按下”）或`AnalogListener`（用于连续事件，如操纵杆移动的幅度）。

因此，在你的`simpleInitApp`过程中，添加以下代码：

```java
  inputManager.addMapping("Shoot",      // Declare...
    newKeyTrigger(KeyInput.KEY_SPACE), // trigger 1: spacebar, or
    newMouseButtonTrigger(0));         // trigger 2: left-button click
  inputManager.addListener(actionListener, "Shoot");
```

所以这里，我们将按下空格键（即使使用虚拟键盘）和鼠标点击（这是在我们移动设备上的触摸动作）映射到触发器名称 `Shoot`。这个触发器名称与名为 `actionListener` 的 `ActionListener` 事件监听器相关联。动作监听器将是我们进行射线拾取的地方；因此，在触摸屏设备上，通过触摸屏幕，你可以激活 `actionListener`（使用触发器 `Shoot`）。

我们下一步是定义可能被我们的射线拾取命中的对象列表。一个不错的技术是将它们重新组合在一个特定的组节点下。在下面的代码中，我们将创建一个盒子对象并将其放置在名为 `shootables` 的组节点下：

```java
Box b = new Box(7, 4, 6); // create cube shape at the origin
geom = new Geometry("Box", b);  // create cube geometry from the shape
Material mat = new Material(assetManager,
"Common/MatDefs/Misc/Unshaded.j3md");  // create a simple material
mat.setColor("Color", ColorRGBA.Red);   // set color of material to blue
geom.setMaterial(mat);        // set the cube's material
geom.setLocalTranslation(new Vector3f(0.0f,0.0f,6.0f));

shootables = new Node("Shootables");
shootables.attachChild(geom);
rootNode.attachChild(shootables);
```

现在我们有了触摸映射和可以被击中的对象。我们只需要实现我们的监听器。在 JME 中进行射线投射的方式与许多其他库相似；我们使用击中坐标（在屏幕坐标中定义），通过我们的摄像机进行变换，创建一个射线，并进行击中测试。在我们的 AR 示例中，我们将使用由基于计算机视觉的追踪器 `fgCam` 更新的 AR 摄像机。因此，在 AR 中的代码与另一个虚拟游戏中的相同，不同之处在于，这里的摄像机位置是由追踪器更新的。

我们创建一个 `Ray` 对象并通过调用 `collideWith` 来对我们的可击中对象列表（`shootables`）进行拾取测试（击中测试）。碰撞结果将被存储在一个 `CollisionResults` 对象中。因此，我们的监听器的代码如下所示：

```java
  privateActionListeneractionListener = new ActionListener() {

  public void onAction(String name, booleankeyPressed, float tpf) {
      Log.d(TAG,"Shooting.");

      if (name.equals("Shoot") && !keyPressed) {

        // 1\. Reset results list.
        CollisionResults results = new CollisionResults();

        // 2\. Mode 1: user touch location.
        Vector2f click2d = inputManager.getCursorPosition();
        Vector3f click3d = fgCam.getWorldCoordinates(
        new Vector2f(click2d.x, click2d.y), 0f).clone();
        Vector3f dir = fgCam.getWorldCoordinates(
        new Vector2f(click2d.x, click2d.y), 1f).subtractLocal(click3d).normalizeLocal();
        Ray ray = new Ray(click3d, dir);

        // 2\. Mode 2: using screen center
        //Aim the ray from fgcamloc to fgcam direction.
        //Ray ray = new Ray(fgCam.getLocation(), fgCam.getDirection());

        // 3\. Collect intersections between Ray and Shootables in results list.
        shootables.collideWith(ray, results);
…
```

那么，我们应该如何处理这个结果呢？正如本书前面所解释的，你可以以不同的方式操作它。这里我们将做一件简单的事情；我们会检测我们的盒子是否被选中，如果被选中，在没有交点的情况下将其颜色变为红色，如果存在交点则变为绿色。我们首先打印结果以便调试，你可以使用 `getCollision()` 函数来检测哪个对象被击中（`getGeometry()`），在什么距离（`getDistance()`）以及接触点（`getContactPoint()`）：

```java
  for (int i = 0; i<results.size(); i++) {
    // For each hit, we know distance, impact point, name of geometry.
    floatdist = results.getCollision(i).getDistance();
    Vector3f pt = results.getCollision(i).getContactPoint();
    String hit = results.getCollision(i).getGeometry().getName();

    Log.d(TAG,"* Collision #" + i + hit);
    //         Log.d(TAG,"  You shot " + hit + " at " + pt + ", " + dist + "wu away.");
  }
```

因此，使用前面的代码我们可以检测是否有任何结果，由于我们的场景中只有一个对象，我们认为如果我们有击中，那就是我们的对象，所以我们把对象的颜色变为绿色。如果我们没有任何击中，因为只有我们的对象，我们将其变为红色：

```java
  if (results.size() > 0) {
    // The closest collision point is what was truly hit:
  CollisionResult closest = results.getClosestCollision();

  closest.getGeometry().getMaterial().setColor("Color", ColorRGBA.Green);
  } else {
    geom.getMaterial().setColor("Color", ColorRGBA.Red);
  }
```

你应该得到一个类似于以下截图所示的结果（击中：左，未击中：右）：

![选择棒子 - 使用射线拾取的 3D 选择](img/8553OS_06_01a.jpg)

现在你可以部署并运行这个例子；在屏幕上触摸对象，看看我们的盒子颜色变化！

# 基于邻近的交互

AR 中的另一种交互方式是利用相机与物理对象之间的关系。如果你在桌子上放置了一个目标，并且你带着设备围绕它移动以从不同角度观察虚拟对象，你也可以使用这种方式来创建交互。这个想法很简单：你可以检测到移动设备上的相机（你的设备）与放在桌子上的目标之间的空间变换的任何变化，并触发一些事件。例如，你可以检测相机是否处于特定角度，是否从上方看向目标，等等。

在本例中，我们将实现一种**接近性**技术，该技术可用于创建一些酷炫的动画和效果。接近性技术利用了 AR 相机与基于计算机视觉的目标之间的距离。

因此，请在你的 Eclipse 中打开`ProximityBasedJME`项目。同样，这个项目也是基于`VuforiaJME`示例的。

首先，我们使用三种不同的颜色——红色、绿色和蓝色，创建三个对象——一个盒子、一个球体和一个圆环，如下所示：

```java
    Box b = new Box(7, 4, 6); // create cube shape at the origin
    geom1 = new Geometry("Box", b);  // create cube geometry from the shape
    Material mat = new Material(assetManager,"Common/MatDefs/Misc/Unshaded.j3md");  // create a simple material
    mat.setColor("Color", ColorRGBA.Red);   // set color of material to red
    geom1.setMaterial(mat);                   // set the cube's material

    geom1.setLocalTranslation(new Vector3f(0.0f,0.0f,6.0f));

    rootNode.attachChild(geom1);              // make the cube appear in the scene

    Sphere s = new Sphere(12,12,6);
    geom2 = new Geometry("Sphere", s);  // create sphere geometry from the shape
    Material mat2 = new Material(assetManager,"Common/MatDefs/Misc/Unshaded.j3md");  // create a simple material
    mat2.setColor("Color", ColorRGBA.Green);   // set color of material to green
    geom2.setMaterial(mat2);                   // set the sphere's material

    geom2.setLocalTranslation(new Vector3f(0.0f,0.0f,6.0f));

    rootNode.attachChild(geom2);              // make the sphere appear in the scene

    Torus= new Torus(12, 12, 2, 6); // create torus shape at the origin
    geom3 = new Geometry("Torus", t);  // create torus geometry from the shape
    Material mat3 = new Material(assetManager,"Common/MatDefs/Misc/Unshaded.j3md");  // create a simple material
    mat3.setColor("Color", ColorRGBA.Blue);   // set color of material to blue
    geom3.setMaterial(mat3);                   // set the torus material
    geom3.setLocalTranslation(new Vector3f(0.0f,0.0f,6.0f));

    rootNode.attachChild(geom3);              // make the torus appear in the scene
```

在许多场景图库中，你经常会找到一个允许基于某些参数（如对象到相机的距离）切换对象表示的开关节点。JME 没有开关节点，因此我们将模拟其行为。我们将根据物体与相机的距离来改变显示的对象（盒子、球体或圆环）。实现这一点的简单方法是，在特定距离处添加或移除不应该显示的对象。

要实现接近性技术，我们查询 AR 相机（`fgCam.getLocation()`）的位置。从该位置，你可以计算到某些对象或仅是目标的距离。根据定义，到目标的距离类似于相机的位置（用三维向量表示）的距离。所以，我们要做的是为我们的对象定义三个范围，如下所示：

+   **相机距离 50 及以上**：显示立方体

+   **相机距离 40-50**：显示球体

+   **相机距离 40 以下**：显示圆环

在`simpleUpdate`方法中的生成代码相当简单：

```java
      Vector3f pos=new Vector3f();

      pos=fgCam.getLocation();

      if (pos.length()>50.)
      {
        rootNode.attachChild(geom1);           
        rootNode.detachChild(geom2);       
        rootNode.detachChild(geom3); 

      }
      else
        if (pos.length()>40.)
        {
          rootNode.detachChild(geom1);           
          rootNode.attachChild(geom2);       
          rootNode.detachChild(geom3); 
        },
        else
        {
          rootNode.detachChild(geom1);           
          rootNode.detachChild(geom2);       
          rootNode.attachChild(geom3); 
        }
```

运行你的示例，并改变设备与追踪目标之间的距离。这将影响所呈现的对象。当你远离时（如下图左侧所示）会出现立方体，靠近时（如下图右侧所示）会出现圆环，在中间距离时（如下图中心所示）会出现球体：

![基于接近性的交互](img/553OS_06_02a.jpg)

# 使用加速度计的简单手势识别

在第四章《在世界中定位》中，你已经了解了典型的 Android 设备内置的各种传感器。你学会了如何使用它们来推导出设备方向。然而，你可以用这些传感器做更多的事情，特别是加速度计。如果你玩过 Wii 游戏，你肯定会对通过挥动 Wiimote 实现自然交互而着迷（例如，在玩网球或高尔夫 Wii 游戏时）。有趣的是，Wiimote 使用的加速度计与许多 Android 智能手机类似，因此你实际上可以实现与 Wiimote 类似的交互方法。对于复杂的 3D 运动手势（如在空中画八字），你将需要一些机器学习背景或使用以下链接中的库：[`www.dfki.de/~rnessel/tools/gesture_recognition/gesture_recognition.html`](http://www.dfki.de/~rnessel/tools/gesture_recognition/gesture_recognition.html)。但是，如果你只想识别简单的手势，你可以很容易地在几行代码中实现。接下来，我们将向你展示如何识别简单的手势，比如摇动手势，即快速地前后挥动手机几次。

如果你查看示例项目`ShakeItJME`，你会发现它很大程度上与第四章《在世界中定位》中的`SensorFusionJME`项目相同。实际上，我们只需执行几个简单的步骤，就可以扩展任何已经使用加速度计的应用程序。在`ShakeItJMEActivity`中，你首先添加一些与摇动检测相关的变量，主要包括存储加速度计事件时间戳的变量（`mTimeOfLastShake`、`mTimeOfLastForce`和`mLastTime`），存储过去的加速度力的变量（`mLastAccelValX`、`mLastAccelValY`和`mLastAccelValZ`），以及用于摇动持续时间、超时（`SHAKE_DURATION_THRESHOLD`、`TIME_BETWEEN_ACCEL_EVENTS_THRESHOLD`和`SHAKE_TIMEOUT`）以及加速度力和传感器样本的最小数量（`ACCEL_FORCE_THRESHOLD`和`ACCEL_EVENT_COUNT_THRESHOLD`）的阈值。

接下来，你只需在你的`SensorEventListener::onSensorChanged`方法中的`Sensor.TYPE_ACCELEROMETER`代码部分，简单地添加对`detectShake()`方法的调用。

`detectShake()`方法是你的摇动检测的核心：

```java
public void detectShake(SensorEvent event) {
…
  floatcurAccForce = Math.abs(event.values[2] - mLastAccelValZ) / timeDiff;
  if (curAccForce> ACCEL_FORCE_THRESHOLD) {
    mShakeCount++;
    if ((mShakeCount>= ACCEL_EVENT_COUNT_THRESHOLD) && (now - mTimeOfLastShake> SHAKE_DURATION_THRESHOLD)) {
      mTimeOfLastShake = now;mShakeCount = 0;			
      if ((com.ar4android.ShakeItJME) app != null) {
        ((com.ar4android.ShakeItJME) app).onShake();
      }
    }
…
  }    
}
```

在这种方法中，你基本上是检查在特定时间框架内的加速度计数值是否大于阈值。如果是，就调用你的 JME 应用程序的`onShake()`方法，并将事件整合到你的应用程序`逻辑中。`注意`，在这个例子中，我们只使用了沿 z 轴的加速度计数值，即与摄像头指向的方向平行。你可以很容易地扩展这一点，也包括侧向摇动运动，通过在`curAccForce`的计算中纳入加速度计的 x 和 y 值。以下是如何使用摇动检测来触发事件的示例，在你的 JME 应用程序的`onShake()`方法中，我们触发了走路忍者的新动画：

```java
public void onShake() {
  mAniChannel.setAnim("Spin");
}
```

为了避免忍者现在一直旋转；我们将在旋转动画结束后切换到行走动画：

```java
public void onAnimCycleDone(AnimControl control, AnimChannel channel, String animName) {
if(animName.contains("Spin")) {
    mAniChannel.setAnim("Walk");
  }
}
```

如果你现在启动应用程序并在观看方向上摇动设备，你应该会看到忍者停止行走并做了一个轻柔的旋转，正如下面这幅图所示：

![使用加速度计的简单手势识别](img/8553OS_06_03.jpg)

# 概述

在本章中，我们向你介绍了三种交互技术，适用于各种增强现实（AR）应用。选择技术允许你通过触摸屏幕来选择 3D 对象，就像你在 2D 选择中所做的那样。基于接近度的摄像头技术允许你通过改变设备的距离和方向来触发应用事件。最后，我们向你展示了一个简单的 3D 手势检测方法示例，为你的应用程序添加更多交互可能性。这些技术应作为你创建针对特定应用场景的交互方法的构建块。在最后一章，我们将介绍一些高级技术以及进一步阅读材料，帮助你充分利用增强现实应用。 
