# 第十章。将应用程序放置在域中

在第三章中，我们介绍了 zygote 以及所有 Android 中的应用程序（APKs）都像服务从`init`进程产生一样，源自 zygote。因此，它们需要被标记，就像我们在上一章中所做的那样。回想一下，标记与将进程放置在具有该标记的域中是相同的。应用程序也需要被标记。

### 注意

APK 是 Android 上可安装应用程序包的文件扩展名和格式。它与桌面包格式类似，如 RPM（基于 Redhat）或 DEB（基于 Debian）。

在本章中，我们将学习：

+   正确标记应用程序私有数据目录及其运行时上下文

+   进一步检查 zygote 及其安全方法

+   发现如何将完成的`mac_permssions.xml`文件分配`seinfo`值

+   创建一个新的自定义域名

# 保护 zygote 的案例

具有提升权限和能力的 Android 应用程序是从 zygote 产生的。一个例子是系统服务器，这是一个由本地和非本地代码组成的大型进程，托管了各种服务。系统服务器包含活动管理器、包管理器、GPS 数据等。系统服务器还以高度敏感的 UID `system` (`1000`) 运行。此外，许多 OEMs 打包了所谓的**系统应用**，这些是运行在`system` UID 上的独立应用程序。

zygote 还会产生不需要提升权限的应用程序。所有第三方应用程序都代表这一点。第三方应用程序以它们自己的 UID 运行，与敏感的 UID（如`system`）分开。此外，应用程序还会被产生到各种 UID 中，如`media`、`nfc`等。OEMs 倾向于定义额外的 UID。

重要的是要注意，要进入特殊 UID，如`system`，你必须使用适当的密钥进行签名。Android 有四个主要密钥用于签名应用程序：`media`、`platform`、`shared`和`testkey`。它们位于`build/target/product/security`中，还有一个`README`。

根据`README`，关键用法如下：

+   `testkey`: 未指定密钥的包的通用密钥。

+   `platform`: 核心平台部分包的测试密钥。

+   `shared`: 在家庭/联系人过程中共享的测试密钥。

+   `media`: 媒体/下载系统部分包的测试密钥。

为了请求应用程序的`system` UID，你必须使用`platform`密钥进行签名。在这些更受特权的环境中执行需要拥有私钥。

如您所见，我们有在多种权限级别和信任级别上执行的应用程序，我们不能信任第三方应用程序，因为它们是由未知实体创建的，我们可以信任用我们的私钥签名的应用程序。然而，在 SELinux 之前，应用程序权限仍然受到与第一章中确定的相同 DAC 权限限制的约束，*Linux 访问控制*。由于这些属性，这使得受精卵成为攻击的主要目标，以及使用 SELinux 进行加固。

# 加强受精卵

现在我们已经确定了受精卵的问题，下一步是了解如何将应用程序放入适当的域中。我们需要 SELinux 策略或代码更改来将新进程放入一个域。在第九章 *将服务添加到域* 中，我们介绍了基于 init 的服务动态域转换，并在章节末尾提到“应用标签限制”部分中的`exec()`系统调用的作用。这是动态域转换发生的触发器。如果没有`exec`在路径中，我们就必须依赖代码更改。然而，在这个安全模型中，还必须考虑签名密钥，而在纯 SELinux 策略语言中无法表达进程所签名的密钥。

而不是探索整个受精卵，我们可以剖析以下引入应用标签到 Android 中的补丁。此外，我们还可以发现引入的设计如何满足尊重签名密钥、在 SELinux 和受精卵设计中工作的要求。

## 探索受精卵套接字

在第三章 *Android 很奇怪* 中，我们了解到受精卵正在监听从套接字生成新应用程序的请求。要检查的第一个补丁是[`android-review.googlesource.com/#/c/31066/`](https://android-review.googlesource.com/#/c/31066/)。此补丁修改了 Android 基础框架中的三个文件。第一个文件是`Process.java`中的`startViaZygote()`方法。此方法是构建字符串参数并将其传递给受精卵的`zygoteSendArgsAndGetResult()`方法的入口点。补丁引入了一个名为`seinfo`的新参数。稍后我们将看到它是如何被使用的。看起来这个补丁是通过套接字将这个新的`seinfo`参数进行管道传输。请注意，此代码是在受精卵进程外部调用的。

在这个补丁中要查看的下一个文件是`ZygoteConnection.java`。此代码在上下文中执行。补丁首先在`ZygoteConnection`类中声明一个字符串成员变量`peerContext`。在构造函数中，这个`peerContext`成员被设置为从调用`SELinux.getPeerContext(mSocket.getFileDescriptor())`获得的值。

由于`LocalSocket`的`mSocket`底层是一个 Unix 域套接字，你可以获取已连接客户端的凭证。在这种情况下，`getPeerContext()`调用获取客户端的安全上下文，或者更正式地说，进程标签。初始化之后，在`runOnce()`方法中进一步使用，我们看到它在调用`applyUidSecurityPolicy`和其他`apply*SecurityPolicy`例程中使用。受保护的`runOnce()`方法被调用以从套接字读取一个启动命令和参数。最终，在`apply*SecurityPolicy`检查之后，它调用`forkandSpecialize()`。每个安全策略检查都已修改为在现有的 DAC 安全控制之上使用 SELinux。如果我们审查`applyUidSecurityPolicy`，我们会看到他们进行了调用：

```kt
boolean allowed = SELinux.checkSELinuxAccess(peerSecurityContext, peerSecurityContext, "zygote", "specifyids");
```

这是一个用户空间利用已知为对象管理器的强制访问控制的示例。此外，在`applyseInfoSecurityPolicy()`方法中，还增加了一个对神秘`seinfo`字符串的安全检查。这里的所有 SELinux 安全检查都指定了目标类`zygote`。因此，如果我们查看`sepolicy access_vectors`，我们会看到添加的类`zygote`。这是一个为 Android 定义的定制类，并定义了安全检查中检查的所有向量。

我们将考虑的这个补丁的最后一个是`ActivityManagerService.java`。`ActivityManager`负责启动应用程序并管理它们的生命周期。它是`Process.start` API 的消费者，并需要指定`seinfo`。这个补丁很简单，目前只是发送`null`。稍后，我们将看到启用其使用的补丁。

下一个补丁[`android-review.googlesource.com/#/c/31063/`](https://android-review.googlesource.com/#/c/31063/)在 Android Dalvik VM 的上下文中执行，并在 VM 受精卵进程空间中编码。我们在`ZygoteConnection`中看到的`forkAndSpecialize()`最终结束在这个本地例程中。它通过`static pid_t forkAndSpecializeCommon(const u4* args, bool isSystemServer)`进入。这个例程负责创建成为应用程序的新进程。

它从将家务代码从 Java 迁移到 C 开始，并设置了`niceName`和`seinfo`值作为 C 风格的字符串。最终，代码调用`fork()`，子进程开始执行一些操作，例如执行`setgid`和`setuid`。`uid`和`gid`值通过`Process.start`方法指定给受精卵连接。我们还看到了对`setSELinuxContext()`的新调用。顺便说一下，这些事件的顺序在这里很重要。如果你过早地设置新进程的 SELinux 上下文，进程在新上下文中执行`setuid`和`setgid`等操作将需要额外的能力。然而，这些权限最好留给`zygote`域，因此我们进入的应用程序域可以尽可能小。

继续看，`setSELinuxContext` 最终调用 `selinux_android_setcontext()`。注意，在这次提交之后移除了 `HAVE_SELINUX` 条件编译宏，但在 4.3 版本之前。另外，注意 `selinux_android_setcontext()` 定义在 `libselinux` 中，所以我们的旅程将带我们到那里。在这里我们看到神秘的 `seinfo` 仍然被传递。

下一个需要评估的补丁是[`android-review.googlesource.com/#/c/39601/`](https://android-review.googlesource.com/#/c/39601/). 这个补丁实际上从 Java 层传递了一个更有意义的 `seinfo` 值。而不是设置为 `null`，这个补丁引入了一些从 XML 文件中解析的逻辑，并将其传递给 `Process.start` 方法。

这个补丁修改了两个主要组件：`PackageManager` 和 `installd`。`PackageManager` 在 `system_server` 内运行，并执行应用程序安装。它维护系统中所有已安装包的状态。第二个组件，一个名为 `installd` 的服务，是一个非常特权的根服务，在磁盘上创建所有应用程序的私有目录。而不是给系统服务器和因此 `PackageManager`，创建这些目录的能力，只有 `installd` 有这些权限。使用这种方法，即使系统服务器也无法读取你的私有数据目录中的数据，除非你将其设置为全局可读。

这个补丁比其他补丁要大，所以我们只将检查与我们的讨论直接相关的部分。我们将从查看 `PackageManagerService.java` 开始。这个类是 Android 的包管理器。在 `PackageManagerService()` 构造函数中，我们看到添加了 `mFoundPolicyFile = SELinuxMMAC.readInstallPolicy();`。

根据命名，我们可以推测这个方法正在寻找某种类型的策略配置文件，如果找到，则返回 true，设置 `mFoundPolicyFile` 成员变量。我们还看到一些对 `createDataDirs` 和 `mInstaller.*` 的调用。我们可以忽略这些调用，因为这些调用是前往 `installd` 的。

下一个主要部分添加了以下内容：

```kt
if (mFoundPolicyFile) {
  SELinuxMMAC.assignSeinfoValue(pkg);
}
```

重要的是要注意，这段代码被添加到了 `scanPackageLI()` 方法中。每次需要扫描安装包时都会调用这个方法。所以从高层次来看，如果在服务启动期间找到了某个策略文件，那么就会为该包分配一个 `seinfo` 值。

下一个要查看的文件是 `ApplicationInfo.java`，这是一个用于维护包元信息的容器类。正如我们所见，`seinfo` 值在这里指定用于存储目的。此外，还有一些代码用于通过 Android 特定的 `Parcel` 实现序列化和反序列化该类。

在这一点上，我们应该更仔细地查看 `SELinuxMMAC.java` 代码，以确认我们对正在发生的事情的理解。该类首先声明了两个策略文件的位置。

```kt
// Locations of potential install policy files.
private static final File[] INSTALL_POLICY_FILE = {
  new File(Environment.getDataDirectory(), "system/mac_permissions.xml"),
  new File(Environment.getRootDirectory(), "etc/security/mac_permissions.xml"),
  null };
```

根据这个，策略文件可以存在于两个位置 - `/data/system/mac_permissions.xml` 和 `/system/etc/security/mac_permissions.xml`。最终，我们看到从 `PackageManagerService` 初始化调用类 `readInstallPolicy()` 中定义的方法的调用，这最终简化为一个调用：

```kt
private static boolean readInstallPolicy(File[] policyFiles) {
  FileReader policyFile = null;
  int i = 0;
  while (policyFile == null && policyFiles != null && policyFiles[i] != null) {
    try {
      policyFile = new FileReader(policyFiles[i]);
      break;
    } catch (FileNotFoundException e) {
      Slog.d(TAG,"Couldn't find install policy " + policyFiles[i].getPath());
    }
  i++;
  }
...
```

将 `policyFiles` 设置为 `INSTALL_POLICY_FILE` 时，此代码使用数组在指定位置查找文件。它是基于优先级的，其中 `/data` 位置比 `/system` 位置具有更高的优先级。此方法中的其余代码看起来像是解析逻辑，并填充了在类声明中定义的两个哈希表：

```kt
// Signature seinfo values read from policy.
private static final HashMap<Signature, String> sSigSeinfo =
new HashMap<Signature, String>();
// Package name seinfo values read from policy.
private static final HashMap<String, String> sPackageSeinfo =
new HashMap<String, String>();
```

`sSigSeinfo` 映射 `Signatures`（或签名密钥）到 `seinfo` 字符串。另一个映射 `sPackageSeinfo` 将包名映射到字符串。

在这一点上，我们可以从 `mac_permissions.xml` 文件中读取一些格式化的 XML，并从签名密钥到 `seinfo` 以及包名到 `seinfo` 创建内部映射。

来自 `PackageManagerService` 到这个类的另一个调用来自 `void assignSeinfoValue(PackageParser.Package pkg)`。

让我们来调查一下这个方法能做什么。它首先检查应用程序是否是系统 UID 或系统安装的应用程序。换句话说，它检查该应用程序是否是第三方应用程序：

```kt
if (((pkg.applicationInfo.flags & ApplicationInfo.FLAG_SYSTEM) != 0) ||
((pkg.applicationInfo.flags & ApplicationInfo.FLAG_UPDATED_SYSTEM_APP) != 0)) {
```

此代码已被谷歌删除，最初是合并的要求。然而，我们可以继续评估。代码遍历包中的所有签名，并与哈希表进行比对。如果它使用该映射中的某个签名，则使用关联的 `seinfo` 值。另一种情况是它通过包名进行匹配。在任一情况下，包的 `ApplictionInfo` 类的 `seinfo` 值都会更新以反映这一点，并由 `installd` 和 zygote 应用程序启动使用：

```kt
// We just want one of the signatures to match.
for (Signature s : pkg.mSignatures) {
  if (s == null)
    continue;
  if (sSigSeinfo.containsKey(s)) {
    String seinfo = pkg.applicationInfo.seinfo = sSigSeinfo.get(s);
    if (DEBUG_POLICY_INSTALL)
      Slog.i(TAG, "package (" + pkg.packageName +
        ") labeled with seinfo=" + seinfo);
    return;
    }
  }
  // Check for seinfo labeled by package.
  if (sPackageSeinfo.containsKey(pkg.packageName)) {
    String seinfo = pkg.applicationInfo.seinfo = sPackageSeinfo.get(pkg.packageName);
    if (DEBUG_POLICY_INSTALL)
      Slog.i(TAG, "package (" + pkg.packageName +
        ") labeled with seinfo=" + seinfo);
      return;
    }
  }
}
```

作为旁白，合并到主线 AOSP 的内容与在 NSA Bitbucket 仓库中维护的内容略有不同。NSA 在这些策略文件中增加了额外的控制，这可能导致应用程序安装中断。可以说，谷歌和 NSA 在这个问题上“分道扬镳”。在 NSA 版本的 `SELinuxMMAC.java` 中，你可以指定匹配特定签名或包名的应用程序可以拥有某些 Android 级别的权限集。例如，你可以阻止所有请求 `CAMERA` 权限的应用程序安装，或者阻止使用特定密钥签名的应用程序。这也突显了在大型代码库中找到补丁并快速了解项目如何演变的重要性，这通常可能显得令人畏惧。

我们需要考虑的这个补丁中的最后一个文件是 `ActivityManagerService.java`。此补丁将 `null` 替换为 `app.info.seinfo`。在所有这些工作和所有这些管道之后，我们最终完全解析了神秘的 `seinfo` 值，将其与每个应用程序包相关联，并将其发送到 zygote 以用于 `selinux_android_setcontext()`。

现在，坐下来思考一下我们想要在标签应用中实现的一些属性将对我们有益。其中之一是 somehow 将安全上下文与应用签名密钥相关联，这正是`seinfo`的主要好处。这是一个高度敏感且受信任的字符串关联值，与签名密钥相关联。字符串的实际内容是任意的，并在`mac_permissions.xml`中指定，这是我们冒险的下一站。

## `mac_permissions.xml`文件

`mac_permissions.xml`文件有一个非常令人困惑的名称。展开后，名称是 MAC 权限。然而，其主要主要功能是将签名密钥映射到`seinfo`字符串。其次，它还可以用于配置非主流的安装时权限检查功能，称为安装时 MMAC。MMAC 控制是 NSA 在中间件层实施强制访问控制的工作的一部分。MMAC 代表“中间件强制访问控制”。谷歌没有合并任何 MMAC 功能。然而，由于我们使用了 NSA Bitbucket 存储库，我们的代码库包含这些功能。

`mac_permissions.xml`是一个 XML 文件，应遵循以下规则，其中斜体部分仅在 NSA 分支上受支持：

+   签名是一个十六进制编码的 X.509 证书，并且对于每个签名者标签都是必需的。

+   `<signer signature="" >`元素可以有多个子元素：

    +   `allow-permission`：它生成一组最大允许权限（白名单）。

    +   `deny-permission`：它生成一个拒绝权限的黑名单。

    +   `allow-all`：它是一个通配符标签，将允许请求的所有权限。

    +   `package`：它是一个复杂的标签，为受签名保护的特定包名称定义允许、拒绝和通配符子元素。

+   允许存在零个或多个全局`<package name="">`标签。这些标签允许为特定包名称设置策略，而不考虑任何签名。

+   允许存在一个`<default>`标签，它可以包含未使用先前列出的证书签名的所有应用程序的安装策略，并且没有每个包的全局策略。

+   任何级别的未知标签都将被跳过。

+   允许存在零个或多个签名者标签。

+   每个签名者标签允许存在零个或多个包标签。

+   `<package name="">`标签可能不包含另一个`<package name="">`标签。如果找到，则跳过。

+   当一个标签出现多个子元素时，使用以下逻辑最终确定执行类型：

    +   如果至少找到一个拒绝权限的标签，则使用黑名单。

    +   如果不是黑名单，并且至少找到一个允许权限的标签，则使用白名单。

    +   如果既不是黑名单也不是白名单，并且至少有一个允许所有权限的标签存在，则使用通配符（接受所有权限）策略。

    +   如果找到一个`<package name="">`子元素，则根据之前的逻辑使用该子元素的策略，并覆盖任何签名全局策略类型。

    +   为了强制执行策略段落，至少必须满足前面的一个条件。这意味着，空签名者、默认或包标签将不被接受。

+   每个`signer/default/package`（全局或附加到签名者）标签都可以包含一个`<seinfo value=""/>`标签。此标签代表每个应用程序可以在最终进程中设置 SELinux 安全上下文时使用的附加信息。

+   在大多数情况下，不会强制执行任何 XML 段落的严格规则。这主要适用于重复的标签，它们是被允许的。如果已存在标签，则原始标签将被替换。

+   对于权限名称的有效性也没有进行检查。尽管预期有效的 Android 权限，但没有任何东西阻止未知权限。

+   以下是一些强制执行决策：

    +   用于签名应用程序的所有签名都根据签名者标签进行检查策略。然而，只需要通过一个签名策略。

    +   如果没有任何签名策略通过，或者甚至没有匹配，那么会寻找全局包策略。如果找到，则此策略将调解安装。

    +   如果需要，最后会咨询默认标签。

    +   本地包策略始终覆盖任何父策略。

    +   如果没有任何情况适用，则应用程序将被拒绝。

以下示例忽略了安装 MMAC 支持，并专注于`seinfo`映射的主线用法。以下是将所有使用平台密钥签名的项目映射到`seinfo`值`platform`的段落映射示例：

```kt
<!-- Platform dev key in AOSP -->
<signer signature="@PLATFORM" >
  <seinfo value="platform" />
</signer>
```

以下是一个示例，将所有使用发布密钥签名的项目映射到发布域，除了浏览器。浏览器被分配了一个`seinfo`值为`browser`，如下所示：

```kt
<!-- release dev key in AOSP -->
<signer signature="@RELEASE" >
  <seinfo value="release" />
  <package name="com.android.browser" >
  <seinfo value="browser" />
  </package>
</signer>
...
```

任何具有未知密钥的项目都将映射到默认标签：

```kt
...
<!-- All other keys -->
<default>
  <seinfo value="default" />
</default>
```

签名标签很有趣，`@PLATFORM`和`@RELEASE`是在构建期间使用的特殊处理字符串。另一个映射文件将这些映射到实际键值。处理并放置到设备上的文件将所有键引用替换为十六进制编码的公钥，而不是这些占位符。它还删除了所有空白和注释以减小大小。让我们通过从设备中提取构建文件并格式化它来查看。

```kt
$ adb pull /system/etc/security/mac_permissions.xml
$ xmllint --format mac_permissions.xml

```

现在，滚动到格式化输出的顶部；你应该看到以下内容：

```kt
<?xml version="1.0" encoding="iso-8859-1"?>
<!-- AUTOGENERATED FILE DO NOT MODIFY -->
<policy>
  <signer signature="308204ae30820396a003020102020900d2cba57296ebebe2300d06092a864886f70d0101050500308196310b300906035504061302555331133...
dec513c8443956b7b0182bcf1f1d">
    <allow-all/>
    <seinfo value="platform"/>
  </signer>
```

注意，`signature=@PLATFORM`现在是一个十六进制字符串。这个十六进制字符串是一个有效的 X509 证书。

## keys.conf

实际上执行从`mac_permissions.xml`中的`signature=@PLATFORM`映射的魔法是`keys.conf`。此配置文件允许您将 pem 编码的 x509 映射到任意字符串。惯例是以`@`开头，但这不是强制性的。文件的格式基于 Python 配置解析器，并包含部分。部分名称是您希望在`mac_permissions.xml`文件中用键值替换的标签。平台示例如下：

```kt
[@PLATFORM]
ALL : $DEFAULT_SYSTEM_DEV_CERTIFICATE/platform.x509.pem
```

在 Android 中，当你构建时，你可以有三个构建级别：`engineering`、`userdebug` 或 `user`。在 `keys.conf` 文件中，你可以使用 `ALL` 部分属性关联一个用于所有级别的密钥，或者你可以为每个级别分配不同的密钥。这对于使用非常特殊的发布密钥构建发布或用户构建非常有用。我们可以在 `@RELEASE` 部分看到一个例子：

```kt
[@RELEASE]
ENG       : $DEFAULT_SYSTEM_DEV_CERTIFICATE/testkey.x509.pem
USER      : $DEFAULT_SYSTEM_DEV_CERTIFICATE/testkey.x509.pem
USERDEBUG : $DEFAULT_SYSTEM_DEV_CERTIFICATE/testkey.x509.pem
```

该文件还允许通过传统的 `$` 特殊字符使用环境变量。pem 文件默认位置是 `build/target/product/security`。然而，你绝对不应该在用户发布构建中使用这些密钥。这些密钥是 AOSP 测试密钥，是公开的！这样做的话，任何人都可以使用系统密钥来签名他们的应用程序并获得系统权限。`keys.conf` 文件仅在构建期间使用，并且不在系统上。

## seapp_contexts

到目前为止，我们已经探讨了如何一个完成的 `mac_permssions.xml` 文件分配 `seinfo` 值。现在我们应该解决如何实际配置和利用这个值。应用程序的标签化管理在另一个配置文件 `seapp_contexts` 中进行。与 `mac_permissions.xml` 类似，它被加载到设备上。然而，默认位置是 `/seapp_contexts`。`seapp_contexts` 的格式是每行 `key=value` 对映射，遵循以下规则：

+   输入选择器：

    +   `isSystemServer`（布尔值）

    +   `user`（字符串）

    +   `seinfo`（字符串）

    +   `name`（字符串）

    +   `sebool`（字符串）

+   输入选择器规则：

    +   `isSystemServer=true` 只能使用一次。

    +   未指定的 `isSystemServer` 默认为 false。

    +   未指定的字符串选择器将匹配任何值。

    +   以 `*` 结尾的用户字符串选择器将执行前缀匹配。

    +   `user=_app` 将匹配任何常规应用程序 UID。

    +   `user=_isolated` 将匹配任何隔离服务 UID。

    +   一个条目中所有指定的输入选择器必须匹配（逻辑与）。

    +   匹配不区分大小写。

    +   优先规则顺序：

        +   在 `isSystemServer=false` 之前 `isSystemServer=true`

        +   指定的 `user=` 字符串在未指定的 `user=` 字符串之前。

        +   在 `user=` 前缀（以 `*` 结尾）之前修复了 `user=` 字符串。

        +   较长的 `user=` 前缀在较短的 `user=` 前缀之前。

        +   在未指定的 `seinfo=` 字符串之前指定 `seinfo=` 字符串。

        +   在未指定的 `name=` 字符串之前指定 `name=` 字符串。

        +   在未指定的 `sebool=` 字符串之前指定 `sebool=` 字符串。

+   输出：

    +   `domain`（字符串）：它指定应用程序的进程域。

    +   `type`（字符串）：它指定应用程序私有数据目录的磁盘标签。

    +   `levelFrom`（字符串；`none`、`all`、`app` 或 `user` 之一）：它给出 MLS 指定器。

    +   `level`（字符串）：它显示硬编码的 MLS 值。

+   输出规则：

    +   只有指定 `domain=` 的条目才会用于应用程序进程标签。

    +   只有指定 `type=` 的条目才会用于应用程序目录标签。

    +   `levelFrom=user` 只支持 `_app` 或 `_isolated` UIDs。

    +   `levelFrom=app` 或 `levelFrom=all` 只支持 `_app` UIDs。

    +   `level`可以用来指定任何 UID 的固定级别。

在应用程序启动期间，此文件由`selinux_android_setcontext()`和`selinux_android_setfilecon2()`函数使用，分别查找适当的应用程序域或文件系统上下文。这些函数的源代码可以在`external/libselinux/src/android.c`中找到，建议阅读。例如，此条目将所有 UID 为`bluetooth`的应用程序放置在`bluetooth`域中，数据目录标签为`bluetooth_data_file`：

```kt
user=bluetooth domain=bluetooth type=bluetooth_data_file
```

此示例将所有第三方或“默认”应用程序放入`untrusted_app`进程域和`app_data_file`数据目录中。它还使用`levelFrom=app`的 MLS 类别来帮助提供基于 MLS 的额外分离。 

```kt
user=_app domain=untrusted_app type=app_data_file levelFrom=app
```

目前，这个功能是实验性的，因为它破坏了一些已知的应用兼容性问题。在撰写本文时，这是谷歌和 NSA 工程师关注的焦点。由于它是实验性的，让我们验证其功能，然后禁用它。

我们还没有安装任何第三方应用程序，因此我们需要这样做以便进行实验。FDroid 是一个寻找第三方应用程序的有用地方，因此让我们从那里下载一些并安装它。我们可以使用位于[`f-droid.org/repository/browse/?fdid=org.zeroxlab.zeroxbenchmark`](https://f-droid.org/repository/browse/?fdid=org.zeroxlab.zeroxbenchmark)的`0xbenchmark`应用程序，其 APK 位于[`f-droid.org/repo/org.zeroxlab.zeroxbenchmark_9.apk`](https://f-droid.org/repo/org.zeroxlab.zeroxbenchmark_9.apk)，如下所示：

```kt
$ wget https://f-droid.org/repo/org.zeroxlab.zeroxbenchmark_9.apk
$ adb install org.zeroxlab.zeroxbenchmark_9.apk 
567 KB/s (1193455 bytes in 2.052s)
pkg: /data/local/tmp/org.zeroxlab.zeroxbenchmark_9.apk
Success

```

### 小贴士

检查`logcat`以获取安装时的`seinfo`值：

```kt
$ adb logcat | grep SELinux
I/SELinuxMMAC( 2557): package (org.zeroxlab.zeroxbenchmark) installed with seinfo=default

```

从您的 UDOO 设备启动`0xbenchmark` APK。我们应该在`ps`中看到它运行并带有其标签：

```kt
$ adb shell ps -Z | grep untrusted
u:r:untrusted_app:s0:c40,c256 u0_a40 17890 2285 org.zeroxlab.zeroxbenchmark

```

注意上下文字符串的级别部分`s0:c40,c256`。这些类别是从`seapp_contexts`中的`level=app`设置创建的。

要禁用它，我们可以简单地从`seapp_contexts`条目中删除`level`的键值对，或者我们可以利用`sebool`条件赋值。让我们使用布尔方法。修改`sepolicy`的`seapp_contexts`文件，以便修改现有的`untrusted_app`条目，并添加一个新的条目。将`user=_app domain=untrusted_app type=app_data_file`更改为`user=_app sebool=app_level domain=untrusted_app type=app_data_file levelFrom=app`。

使用以下命令构建：`mmm external/sepolicy`：

```kt
Error:
out/host/linux-x86/bin/checkseapp -p out/target/product/udoo/obj/ETC/sepolicy_intermediates/sepolicy -o out/target/product/udoo/obj/ETC/seapp_contexts_intermediates/seapp_contexts out/target/product/udoo/obj/ETC/seapp_contexts_intermediates/seapp_contexts.tmp
Error: Could not find selinux boolean "app_level" on line: 42 in file: out/target/product/udoo/obj/ETC/seapp_contexts_intermediates/seapp_contexts
Error: Could not validate

```

好吧，有一个构建错误，抱怨在`seapp_contexts`的第 42 行找不到`selinux`布尔值。让我们通过声明布尔值来尝试纠正这个问题。在`app.te`中添加：`bool app_level false;`。现在将新构建的`seapp_contexts`和 sepolicy 文件推送到设备并触发动态重新加载：

```kt
$ adb push $OUT/root/sepolicy /data/security/current/
$ adb push $OUT/root/seapp_contexts /data/security/current/
$ adb shell setprop selinux.reload_policy 1

```

我们可以通过以下方式验证布尔值的存在：

```kt
$ adb shell getsebool -a | grep app_level
app_level --> off

```

由于设计限制，我们需要卸载并重新安装应用程序：

```kt
$ adb uninstall org.zeroxlab.zeroxbenchmark

```

重新安装并检查启动应用程序后的进程上下文：

```kt
$ adb shell ps -Z | grep untrusted
u:r:untrusted_app:s0:c40,c256 u0_a40 17890 2285 org.zeroxlab.zeroxbenchmark

```

太好了！失败了。经过一些调试，我们发现问题的根源是路径 `/data/security` 不是对所有人可搜索的，导致 DAC 权限失败。

### 注意

我们通过打印 `android.c` 中的结果和错误代码找到了这个问题，我们在 `selinux_android_seapp_context_reload()` 中检查 `fp = fopen(seapp_contexts_file[i++], "r")` 的结果时，在 `seapp_contexts_file[]` 数组（按优先级排序的文件）上看到了 `fopen`，并使用 `selinux_log()` 将数据转储到 `logcat`。

```kt
$ adb shell ls -la /data | grep security
drwx------ system system 1970-01-04 00:22 security

```

记住 `set selinux` 上下文发生在 UID 切换之后，因此我们需要使其对其他人可搜索。我们可以通过更改 `device/fsl/imx6/etc/init.rc` 中的 UDOO `init.rc` 脚本的权限来修复权限。具体来说，将行 `mkdir /data/security 0700 system system` 更改为 `mkdir /data/security 0711 system system`。构建并刷新 `bootimage`，然后再次尝试上下文测试。

```kt
$ adb uninstall org.zeroxlab.zeroxbenchmark
$ adb install ~/org.zeroxlab.zeroxbenchmark_9.apk
<launch apk>
$ adb shell ps -Z | grep org.zeroxlab.zeroxbenchmark
u:r:untrusted_app:s0 u0_a40 3324 2285 org.zeroxlab.zeroxbenchmark

```

到目前为止，我们已经展示了如何使用 `sebool` 选项在 `seapp_contexts` 上禁用 MLS 类别。重要的是要注意，在更改 APK 上的类别或类型时，需要删除和安装 APK，否则你将使进程与其数据目录分离，因为在大多数情况下它将没有访问权限。

接下来，让我们取这个 APK，卸载它，并通过更改其 `seinfo` 字符串为其分配一个唯一的域。通常，你使用此功能将使用共同密钥签名的应用程序集放入自定义域以执行自定义操作。例如，如果你是 OEM，你可能需要允许使用 OEM 控制的密钥未签名的第三方应用程序自定义权限。首先，卸载 APK：

```kt
$ adb uninstall org.zeroxlab.zeroxbenchmark

```

在 `mac_permissions.xml` 中通过添加以下内容创建一个新的条目：

```kt
<signer signature="@BENCHMARK" >
<allow-all />
<seinfo value="benchmark" />
</signer>

```

现在我们需要为 `keys.conf` 获取一个 pem 文件。所以解包 APK 并提取公钥证书：

```kt
$ mkdir tmp
$ cd tmp
$ unzip ~/org.zeroxlab.zeroxbenchmark_9.apk
$ cd META-INF/
$ $ openssl pkcs7 -inform DER -in *.RSA -out CERT.pem -outform PEM  -print_certs

```

我们需要从生成的 `CERT.pem` 文件中删除任何冗余内容。如果你打开它，你应该在顶部看到这些行：

```kt
subject=/C=UK/ST=ORG/L=ORG/O=fdroid.org/OU=FDroid/CN=FDroid
issuer=/C=UK/ST=ORG/L=ORG/O=fdroid.org/OU=FDroid/CN=FDroid
-----BEGIN CERTIFICATE-----
MIIDPDCCAiSgAwIBAgIEUVJuojANBgkqhkiG9w0BAQUFADBgMQswCQYDVQQGEwJV
SzEMMAoGA1UECBMDT1JHMQwwCgYDVQQHEwNPUkcxEzARBgNVBAoTCmZkcm9pZC5v
...
```

它们需要被移除，所以只移除主题和发行者行。文件应从 `BEGIN CERTIFICATE` 开始，以 `END CERTIFICATE` 剪刀线结束。

让我们把这项工作移到我们工作区的新文件夹中，命名为 `certs`，并将证书以更好的名称移动到这个文件夹中：

```kt
$ mkdir UDOO_SOURCE_ROOT/certs
$ mv CERT.pem UDOO_SOURCE_ROOT/certs/benchmark.x509.pem

```

我们可以通过添加以下内容来设置我们的 `keys.conf`：

```kt
[@BENCHMARK]
ALL : certs/benchmark.x509.pem

```

不要忘记更新 `seapp_contexts` 以使用新的映射：

```kt
user=_app seinfo=benchmark domain=benchmark_app type=benchmark_app_data_file

```

现在声明要使用的新类型。域类型应在 `sepolicy` 中的名为 `benchmark_app.te` 的文件中声明：

```kt
# Declare the new type
type benchmark_app, domain;
# This macro adds it to the untrusted app domain set and gives it some allow rules
# for basic functionality as well as object access to the type in argument 2.
untrustedapp_domain(benchmark_app, benchmark_app_data_file)

```

还要在 `file.te` 中添加 `benchmark_app_data_file`：

```kt
type benchmark_app_data_file, file_type, data_file_type, app_public_data_type;

```

### 小贴士

你可能并不总是想要所有这些属性，特别是如果你正在做一些安全关键的事情。确保你查看每个属性和宏的用法。你不希望通过过度宽容的域打开一个意外的漏洞。

重建策略，推动所需的组件，并触发重新加载。

```kt
$ mmm external/sepolicy/
$ adb push $OUT/system/etc/security/mac_permissions.xml /data/security/current/
$ adb push $OUT/root/sepolicy /data/security/current/
$ adb push $OUT/root/seapp_contexts /data/security/current/
$ adb shell setprop selinux.reload_policy 1

```

启动一个 shell 并使用 grep 查找 logcat 来查看基准 APK 安装的 `seinfo` 值。然后安装 APK：

```kt
$ adb install ~/org.zeroxlab.zeroxbenchmark_9.apk
$ adb logcat | grep -i SELinux

```

在`logcat`输出中，你应该看到：

```kt
I/SELinuxMMAC( 2564): package (org.zeroxlab.zeroxbenchmark) installed with seinfo=default

```

应该是`seinfo=benchmark`！可能发生了什么？

问题出在`frameworks/base/services/java/com/android/server/pm/SELinuxMMAC.java`文件中。它查找`/data/security/mac_permissions.xml`；所以我们只需推送`mac_permissions.xml`。这是动态策略重新加载中的另一个错误，与这个加载过程的历史变化有关。罪魁祸首就在`frameworks/base/services/java/com/android/server/pm/SELinuxMMAC.java`文件中：

```kt
private static final File[] INSTALL_POLICY_FILE = {
new File(Environment.getDataDirectory(), "security/mac_permissions.xml"),
new File(Environment.getRootDirectory(), "etc/security/mac_permissions.xml"),
null};
```

为了解决这个问题，请重新挂载`system`并将其推送到默认位置。

```kt
$ adb remount
$ adb push $OUT/system/etc/security/mac_permissions.xml /system/etc/security/

```

这并不需要执行`setprop selinux.reload_policy 1`。请卸载并重新安装基准 APK，并检查日志：

```kt
I/SELinuxMMAC( 2564): package (org.zeroxlab.zeroxbenchmark) installed with seinfo=default

```

好的。它仍然没有工作。当我们检查代码时，发现`mac_permissions.xml`文件在包管理器服务启动时被加载。这个文件在没有重启的情况下不会重新加载，所以让我们卸载基准 APK，并重启 UDOO。启动后，启用`adb`，触发动态重新加载，安装 APK，并检查`logcat`。它应该有：

```kt
I/SELinuxMMAC( 2559): package (org.zeroxlab.zeroxbenchmark) installed with seinfo=benchmark

```

现在我们通过启动 APK、检查`ps`和验证其应用程序私有目录来验证进程域：

```kt
<launch apk>
$ adb shell ps -Z | grep org.zeroxlab.zeroxbenchmark
u:r:benchmark_app:s0 u0_a45 3493 2285 org.zeroxlab.zeroxbenchmark
$ adb shell ls -Z /data/data | grep org.zeroxlab.zeroxbenchmark
drwxr-x--x u0_a45 u0_a45 u:object_r:benchmark_app_data_file:s0 org.zeroxlab.zeroxbenchmark

```

这次，所有类型都通过了检查。我们成功创建了一个新的自定义域。

# 摘要

在本章中，我们探讨了如何通过配置文件和 SELinux 策略正确标记应用程序私有数据目录及其运行时上下文。我们还研究了使所有这些工作正常运行的子系统以及可能出错的一些基本事项。在下一章中，我们将扩展如何通过查看 SE for Android 构建系统来构建策略和配置文件。
