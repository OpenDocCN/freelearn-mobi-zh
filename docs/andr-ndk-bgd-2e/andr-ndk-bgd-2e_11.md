# 第十一章：后记

在整本书中，你已经学习了入门的基本知识，并概览了进一步学习的路径。你现在知道了驾驭这些小而强大的“怪物”的关键元素，并开始充分利用它们的力量。然而，还有很多东西要学，但时间和空间是有限的。无论如何，掌握一项技术的唯一途径就是实践，再实践。希望你能享受这个过程，并为移动挑战做好准备。所以，我现在最好的建议是收集你新鲜的知识和你所有惊人的想法，在脑海中反复打磨它们，并用键盘将它们实现。

# 我们曾经去过的地方

我们已经具体地了解了如何使用 Eclipse 和 NDK 创建本地项目。我们学习了如何通过 JNI 在 Java 应用程序中嵌入 C/C++库，以及如何在不编写 Java 代码的情况下运行本地代码。

我们使用 OpenGL ES 和 OpenSL ES 测试了 Android NDK 的多媒体能力，这些能力在移动领域正逐渐成为标准（当然，是在忽略 Windows Mobile 之后）。我们甚至与手机的输入外设交互，并通过其传感器感知世界。

此外，Android NDK 不仅与性能有关，还与可移植性有关。因此，我们重用了 STL 框架，它最好的伴侣`Boost`，并且几乎无缝地移植了第三方库。

最后，我们了解了如何使用 RenderScript 技术优化密集计算任务。

# 你可以去的地方

C/C++生态系统已经存在了几十年，并且非常丰富。我们移植了一些库，但还有更多库等待被移植。实际上，下面列出许多库，它们无需完全重写代码就可以工作：

+   Bullet ([`bulletphysics.org/`](http://bulletphysics.org/))是一个物理引擎的例子，它可以在几分钟内直接移植。

+   Irrlicht ([`irrlicht.sourceforge.net/`](http://irrlicht.sourceforge.net/))是可以运行在 Android 上的众多 3D 引擎之一。

+   OpenCV ([`opencv.org/`](http://opencv.org/))是一个计算机视觉和机器学习库，它让你的应用程序能够通过摄像头“看到”并理解外部世界。

+   GLM ([`glm.g-truc.net/`](http://glm.g-truc.net/))是一个有用的库，用于 OpenGL ES 2 以完全 C++的方式处理矩阵计算。

+   Intel Threading Building Block 库 ([`www.threadingbuildingblocks.org/`](https://www.threadingbuildingblocks.org/))，通常称为 TBB，对于那些需要在本地代码中进行大规模并行化的开发者来说是一个有趣的库。

一些库是专门为移动设备设计的，例如：

+   Unity ([`unity3d.com/`](http://unity3d.com/))是一个优秀的编辑器和框架，如果你想要编写移动游戏，绝对应该看看。

+   Unreal Engine ([`www.unrealengine.com/`](https://www.unrealengine.com/))是最强大的引擎之一，现在可以免费使用。

+   Cocos2D-X（[`www.cocos2d-x.org/`](http://www.cocos2d-x.org/)），这是一个在许多 2D 游戏中广泛使用的高度流行的游戏引擎。

+   Vuforia（[`www.qualcomm.com/products/vuforia`](https://www.qualcomm.com/products/vuforia)），这是高通公司推出的增强现实 SDK。

对于那些想要深入 Android 内部工作原理的人，我建议你查看一下 Android 平台代码本身，它可以在[`source.android.com/`](http://source.android.com/)找到。下载、编译甚至部署都不是一件容易的事，但这确实是深入了解 Android 内部工作原理的唯一途径，有时也是找出那些讨厌的 bug 来源的唯一方式！

# 寻求帮助的地方：

Android 社区非常活跃，以下是一些寻找有用信息的地方：

+   Android Google 群组（[`groups.google.com/group/android-developers`](http://groups.google.com/group/android-developers）和 Android NDK 群组（[http://groups.google.com/group/android-ndk](http://groups.google.com/group/android-ndk)），你可以在这些地方获得帮助，有时甚至来自 Android 团队成员。

+   Android 开发者博客（[`android-developers.blogspot.com/`](http://android-developers.blogspot.com/)），在这里你可以找到关于 Android 开发的最新官方信息。

+   Google IO（[`www.google.com/events/io`](https://www.google.com/events/io)）上有一些关于 Android 的最佳视频演讲。

+   英特尔开发者网站（[`software.intel.com/en-us/android`](https://software.intel.com/en-us/android)）上有许多关于 x86 上 NDK 的有趣资源。

+   NVidia 开发者中心（[`developer.nvidia.com/category/zone/mobile-development`](http://developer.nvidia.com/category/zone/mobile-development)）提供了关于 Tegra 的开发资源，同时也包含关于 Android 和 NDK 的一般性资源。

+   高通开发者网络（[`developer.qualcomm.com/`](https://developer.qualcomm.com/)）可以找到关于 NVidia 主要竞争对手的信息。特别是高通的增强现实 SDK 非常具有前景。

+   Stack Overflow（[`stackoverflow.com/`](http://stackoverflow.com/)），虽然不是专门针对 Android 的，但你可以提问并获得准确的答案。

+   GitHub（[`github.com/`](http://github.com/)），上面有大量的 NDK 库和示例。

# 这仅仅是一个开始。

创建应用程序只是过程的一部分。发布和销售是另一部分。这当然超出了本书的范围，但处理碎片化和测试与各种目标设备的兼容性确实是一个需要认真对待的难题。

当心！当你开始处理硬件特性（有很多这样的特性）时，问题就会开始出现，正如我们在输入设备中看到的那样。然而，这些问题并不特定于 NDK。如果 Java 应用程序中存在不兼容性，那么原生代码也不会更好。处理各种屏幕尺寸，加载适当大小的资源，以及适应设备功能，这些最终都是你需要处理的事情。然而，这应该是可以管理的。

简而言之，有很多奇妙但也痛苦的惊喜等着被发现。然而，Android 和移动性仍然是一片待开垦的土地，需要被塑造。看看从最早版本到最新版本的 Android 的演变，你就会相信。革命不会每天都发生，所以不要错过它！

祝你好运！

**西尔万·拉塔布伊尔**
