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

# 读者反馈

我们的读者的反馈总是受欢迎的。让我们知道您对本书的看法——您喜欢或不喜欢什么。读者的反馈对我们很重要，因为它可以帮助我们开发您真正能够充分利用的标题。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在主题中提及书名。

如果您在某个专题上有专业知识，并且有兴趣撰写或为一本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经是 Packt 书籍的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册到我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载和勘误**。

1.  在**搜索**框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地方。

1.  点击**代码下载**。

您还可以通过单击 Packt Publishing 网站上该书网页上的**代码文件**按钮来下载代码文件。您可以通过在**搜索**框中输入书名来访问该页面。请注意，您需要登录到您的 Packt 帐户。

文件下载后，请确保使用以下最新版本解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

您还可以从 GitHub 上下载代码文件[`github.com/cardbookvr`](https://github.com/cardbookvr)。

## 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/CardboardVRProjectsforAndroid_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/CardboardVRProjectsforAndroid_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经非常小心地确保了内容的准确性，但错误是难免的。如果您在我们的书中发现错误——可能是文本或代码中的错误——我们将不胜感激地希望您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书，点击**勘误提交表**链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的勘误列表下的勘误部分。

要查看先前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在**勘误**部分下。

## 盗版

互联网上侵犯版权材料的盗版是所有媒体持续存在的问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并附上涉嫌盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
