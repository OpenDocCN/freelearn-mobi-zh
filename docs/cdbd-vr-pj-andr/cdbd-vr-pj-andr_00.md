# 前言

谷歌纸板是一种低成本、入门级的媒介，用于体验虚拟 3D 环境。它的应用与移动智能手机应用程序本身一样广泛和多样。本书为您提供了使用原生 Java SDK 为谷歌纸板实现各种有趣项目的机会。目的是教育您最佳实践和方法，以制作适用于设备及其预期用户的纸板兼容移动 VR 应用，并指导您制作高质量的内容。

# 本书涵盖的内容

第一章，“每个人的虚拟现实”，定义了谷歌纸板，探讨了它，并讨论了它的用途以及它如何适应虚拟现实设备的范围。

第二章，“骨架纸板项目”，审查了安卓纸板应用程序的结构，介绍了 Android Studio，并通过引入纸板 Java SDK 帮助您构建一个起始纸板项目。

第三章，“纸板盒”，讨论了如何从头开始构建一个基于谷歌的宝藏猎人示例的纸板安卓应用程序，其中包括 3D 立方体模型、变换、立体摄像机视图和头部旋转。本章还包括对 3D 几何、Open GL ES、着色器、矩阵数学和渲染管线的讨论。

第四章，“启动器大堂”，帮助您构建一个应用程序，用于在手机上启动其他纸板应用。这个项目不使用 3D 图形，而是在屏幕空间中模拟立体视图，并实现了凝视选择。

第五章，“RenderBox 引擎”，向您展示了如何创建一个小型图形引擎，用于通过将低级别的 OpenGL ES API 调用抽象为一套`Material`、`RenderObject`、`Component`和`Transform`类来构建新的纸板 VR 应用程序。该库将在后续项目中被使用和进一步开发。

第六章，“太阳系”，通过添加太阳光源、具有纹理映射材料和着色器的球形行星，以及它们在太阳系轨道上的动画和银河星空，构建了一个太阳系模拟科学项目。

第七章，“360 度画廊”，帮助您构建一个用于常规和 360 度照片的媒体查看器，并帮助您将手机相机文件夹中的照片加载到缩略图图像网格中，并使用凝视选择来选择要查看的照片。它还讨论了如何添加进程线程以改善用户体验，并支持 Android 意图以查看来自其他应用程序的图像。

第八章，“3D 模型查看器”，帮助您构建一个用于 OBJ 文件格式的 3D 模型的查看器，使用我们的 RenderBox 库进行渲染。它还向您展示了如何通过移动头部来交互控制模型的视图。

第九章，“音乐可视化器”，构建了一个基于手机当前音频播放器的波形和 FFT 数据进行动画的 VR 音乐可视化器。我们实现了一个通用架构，用于添加新的可视化，包括几何动画和动态纹理着色器。然后，我们添加了一个迷幻轨迹模式和多个并发可视化，随机过渡进出。

# 您需要什么来阅读本书

在整本书中，我们使用 Android Studio IDE 开发环境来编写和构建 Android 应用程序。您可以免费下载 Android Studio，如第二章，“骨架纸板项目”中所述。您需要一部安卓手机来运行和测试您的项目。强烈建议您拥有一个谷歌纸板查看器，以体验立体虚拟现实中的应用程序。

# 本书适合谁

本书适用于对学习和开发使用 Google Cardboard 原生 SDK 的 Google Cardboard 应用程序感兴趣的 Android 开发人员。我们假设读者对 Android 开发和 Java 语言有一定了解，但可能对 3D 图形、虚拟现实和 Google Cardboard 还不熟悉。初学者开发人员或不熟悉 Android SDK 的人可能会发现本书难以入门。那些没有 Android 背景的人可能更适合使用 Unity 等游戏引擎创建 Cardboard 应用程序。

# 惯例

在本书中，您将找到一些区分不同信息类型的文本样式。以下是一些示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“编辑`MainActivity` Java 类，使其扩展`CardboardActivity`并实现`CardboardView.StereoRenderer`。”

代码块设置如下：

```kt
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        CardboardView cardboardView = (CardboardView) findViewById(R.id.cardboard_view);
        cardboardView.setRenderer(this);
        setCardboardView(cardboardView);
    }
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```kt
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        CardboardView cardboardView = (CardboardView) findViewById(R.id.cardboard_view);
 cardboardView.setRenderer(this);
 setCardboardView(cardboardView);
    }
```

任何命令行输入或输出均按以下方式编写：

```kt
git clone https://github.com/googlesamples/cardboard-java.git

```

**新术语**和**重要单词**以粗体显示。屏幕上看到的单词，例如菜单或对话框中的单词，以这种方式出现在文本中：“在 Android Studio 中，选择**文件**|**新建**|**新建模块…**。选择**导入.JAR/.AAR 包**。”

### 注意

警告或重要提示以这样的框出现。

### 提示

技巧和窍门看起来像这样。

