# 创建OTA包

在上一章中，我们分析了恢复的内部结构，并学习了它是如何工作的。正如我们所见，恢复的主要功能之一是支持OTA更新。在本章中，我们将研究OTA包，并研究OTA包更新的过程。我们将涵盖以下主题：

+   我们将查看OTA包内部的内容。我们将研究`updater`和`updater-script`的内部结构。

+   我们将学习如何构建OTA包的过程。

+   最后，我们需要改进恢复以从Android系统中移除依赖。

# OTA包内部有什么

在我们开始构建OTA包之前，让我们看看OTA包内部的内容。OTA包可以用来将系统更新到新版本。新版本可以是主要版本或次要版本。例如，它可能是对现有Android版本的小幅更新，以修复关键问题或安全漏洞。它也可能是从Android 6到Android 7的主要更新。让我们看看本章将要创建的OTA包的内容，以了解OTA包内部有什么。本章将要创建的OTA包是我们整个ROM的OTA更新包。我们可以使用恢复来将OTA包刷入我们的VirtualBox设备。这是将我们构建的系统镜像安装到虚拟设备上的另一种方法。

让我们看看本章将要构建的OTA包的内容。OTA包本身是一个ZIP文件。在我们解压ZIP文件后，我们可以列出ZIP文件的内容如下：

[PRE0]

我们可以看到它包括两个文件和三个文件夹。在我们使用恢复刷写这个更新包后，它将更新`/boot`分区和`/system`分区：

+   `boot.img`: 这是`/boot`分区的镜像，其中包含内核和ramdisk。

+   `file_contexts`: 此文件用于根据SELinux策略为文件分配标签。SELinux在最新的Android系统中默认启用。在恢复更新系统分区后，它必须使用此文件应用标签。

+   `META-INF`: 这个文件夹包含OTA包、更新程序和更新脚本的签名。我们将在稍后查看这个文件夹的详细信息。

+   `recovery`: 这个文件夹包含一个`install-recovery.sh`shell脚本和一个`recovery-from-boot.p`补丁文件。

+   `system`: 这是恢复将要更新到`/system`分区的`system`文件夹。

OTA包通常用于更新`/boot`和`/system`分区。它不会更新自身。`/recovery`分区的更新在正常的启动过程中进行。在启动过程中，init将通过以下`flash_recovery`服务在`init.rc`脚本中执行`install-recovery.sh`：

[PRE1]

`install-recovery.sh`脚本使用`recovery-from-boot.p`补丁文件安装恢复，如下所示：

[PRE2]

在我们的环境设置中，`/recovery`分区位于`/dev/block/sda7`分区。此脚本将检查`/dev/block/sha7`分区的`sha1`哈希值。如果`sha1`哈希值不同，它将更新`/recovery`分区。

现在，让我们看看下面的截图所示的`META-INF`文件夹：

![](img/image_13_001.png)

如我们所见，更新包、更新器和更新脚本的签名包含在`META-INF`文件夹中。在恢复应用更新之前，它将验证`META-INF`文件夹中的包签名与`/system/etc/security/otacerts.zip`中的受信任证书。

更新器位于`META-INF/com/google/android/update-binary`的可执行文件。它解释`META-INF/com/google/android/updater-script`文件中的脚本。该脚本是用一种可扩展的脚本语言（edify）编写的，支持典型更新相关任务的命令。

由于更新器和更新脚本是在OTA包中支持OTA更新的关键组件，我们将深入了解它们的细节。

# 更新器

更新器是AOSP源树中针对目标设备的单个可执行文件。它可以在`$AOSP/bootable/recovery/updater`文件夹中找到。让我们看看`updater.cpp`文件中的主函数。由于`main`函数比较长，我们将分几个段落来看：

[PRE3]

更新器有四个参数。它首先会检查是否传入了四个参数。从代码中我们可以看到，这四个参数是：

+   第一个参数是可执行文件名，在这里是`update-binary`

+   第二个参数是更新器版本

+   第三个参数是可以用来与恢复进行通信的管道

+   第四个参数是OTA包的路径

在继续之前，它将检查更新器版本：

[PRE4]

下一步是打开管道以建立与恢复的通信通道。然后它从OTA包中提取`updater-script`以准备执行脚本：

[PRE5]

在开始执行更新脚本之前，它需要注册函数以解释更新脚本内部的edify语言。从前面的代码中我们可以看到，这些函数包括以下四个类别：

+   内置函数以支持edify语言语法。这些函数在`bootable/recovery/edify/expr.cpp`中实现。

+   与包安装相关的函数。这些函数在`bootable/recovery/updater/install.cpp`中实现。

+   处理基于块OTA包的函数。在Android 4.4及更早版本中，使用基于文件的OTA更新。在Android 5.0及以后版本中，使用基于块的OTA更新。有关文件与块OTA的比较，请参阅以下URL：

    [https://source.android.com/devices/tech/ota/block.html](https://source.android.com/devices/tech/ota/block.html)

    基于块的函数在`bootable/recovery/updater/blockimg.cpp`中实现。

+   开发者可以扩展恢复和更新器以提供特定设备的 OTA 扩展。

在注册所有函数后，它调用 `parse_string` 函数来解析脚本。最后，它调用 `Evaluate` 函数来执行脚本。

# 更新器脚本

在我们探索更新器的实现之后，我们将在本节中查看更新器脚本。更新器脚本是在目标设备上执行更新操作的那个脚本。更新器脚本是用一种简单的脚本语言 edify 编写的。edify 脚本是一系列表达式，每行一个表达式。它支持以下运算符：

+   比较运算符，例如 `==`（字符串相等）和 `!=`（字符串不等）

+   逻辑运算符，例如 `||`（逻辑或）、`&&`（逻辑与）和 `!`（逻辑非）

+   连接运算符 `+`

唯一的保留关键字是条件关键字 `if`、`then`、`else` 和 `endif`。

edify 中的所有值都是字符串。在布尔上下文中，空字符串为 `false`，所有其他字符串为 `true`。

您可以参考以下 URL 了解 edify 语法的更多信息：

[https://source.android.com/devices/tech/ota/inside_packages](https://source.android.com/devices/tech/ota/inside_packages)

# Edify 函数

edify 语言的主体功能以 edify 函数的形式实现，而 edify 函数则注册在先前的更新器源代码中。为了支持 OTA 更新，edify 函数包括内置函数、安装函数、块镜像函数和设备扩展。我们将在接下来的几节中查看每个类别。

# 内置函数

内置函数用于支持 edify 语言语法。内置函数通过 `RegisterBuiltins` 注册。我们可以查看以下源代码：

[PRE6]

`RegisterBuiltins` 函数注册以下内置函数：

+   `ifelse(cond, e1[, e2])`: 评估 `cond`，如果为 true，则评估并返回 `e1` 的值，否则评估并返回 `e2`（如果存在）。

+   `abort([msg])`: 立即终止脚本的执行，可选的 `msg` 参数。如果用户已开启文本显示，`msg` 将出现在恢复日志和屏幕上。

+   `assert(expr[, expr, ...])`: 依次评估每个 `expr`。如果其中任何一个为 false，则立即终止执行并显示消息 `assert failed`。

+   `concat(expr[, expr, ...])`: 评估每个表达式并将它们连接起来。

+   `is_substring(substring, string)`: 如果可以找到子字符串，则返回 true。

+   `stdout(expr[, expr, ...])`: 评估每个表达式并将它们的值输出到 `stdout`。这在调试中非常有用。

+   `sleep(secs)`: 等待 `secs` 秒。

+   `less_than_int(a, b)`: 如果且仅当 `a`（解释为整数）小于 `b`（解释为整数）时返回 true。

+   `greater_than_int(a, b)`: 如果且仅当 `a`（解释为整数）大于 `b`（解释为整数）时返回 true。

# 安装函数

与安装相关的函数通过 `RegisterInstallFunctions` 注册。以下是其源代码：

[PRE7]

如我们所见，大多数函数都注册在这里；我们现在将查看它们：

+   `mount(fs_type, partition_type, name, mount_point)`: 此函数在 `mount_point` 处挂载 `fs_type` 文件系统。`partition_type` 参数必须是 MTD 或 EMMC 之一。`name` 参数是分区名称（系统、userdata 或 cache 等）。恢复默认不挂载任何文件系统，更新脚本必须挂载它需要修改的任何分区。

+   `is_mounted(mount_point)`: 如果在 `mount_point` 处挂载了文件系统，则返回 true。

+   `unmount(mount_point)`: 卸载在 `mount_point` 处挂载的文件系统。

+   `format(fs_type, partition_type, location, fs_size, mount_point)`: 此函数格式化给定的分区。`fs_type` 参数可以是 yaffs2、ext4 或 f2fs。`partition_type` 参数可以是 MTD 或 EMMC。`location` 参数是分区或设备的名称。`fs_size` 参数是文件系统大小，`mount_point` 是挂载点名称。

+   `show_progress(frac, secs)`: 在 `secs` 秒内将进度条向前推进到其长度的 `frac` 部分。`secs` 参数可以是零，在这种情况下，进度条不会自动前进，而是通过以下定义的 `set_progress` 函数使用：

    +   `set_progress(frac)`: 此函数设置进度条在最近一次 `show_progress` 调用定义的块中的位置。

+   `delete([filename, ...])`: 删除列出的所有文件名。返回成功删除的文件数量。

+   `delete_recursive([dirname, ...])`: 递归删除 `dirname` 及其所有内容。返回成功删除的目录数量。

+   `package_extract_dir(package_dir, dest_dir)`: 从 `package_dir` 下的包中提取所有文件，并将它们写入 `dest_dir` 下的相应树中。任何现有文件都将被覆盖。

+   `package_extract_file(package_file[, dest_file])`: 从 `update` 包中提取单个 `package_file` 并将其写入 `dest_file`，如果需要则覆盖现有文件。

+   `symlink(target[, source, ...])`: 将所有源创建为指向目标的符号链接。

+   `set_metadata(filename, key1, value1[, key2, value2, ...])`: 将给定文件名的键设置为值。

+   `set_metadata_recursive(dirname, key1, value1[, key2, value2, ...])`: 递归地将给定 `dirname` 及其所有子目录的键设置为值。

+   `getprop(key)`: 返回系统属性键的值（如果没有定义，则返回空字符串）。

+   `file_getprop(filename, key)`: 读取给定的文件名，将其解释为属性文件（例如，`/system/build.prop`），并返回给定键的值，如果键不存在，则返回空字符串。

+   `write_raw_image(filename_or_blob, partition)`: 将 `filename_or_blob` 中的镜像写入 MTD 分区。

+   `apply_patch(src_file, tgt_file, tgt_sha1, tgt_size, patch1_sha1, patch1_blob, [...])`: 将二进制补丁应用到 `src_file` 上以生成 `tgt_file`。

+   `apply_patch_check(filename, sha1[, sha1, ...])`: 如果 `filename` 的内容或缓存分区中的临时副本（如果存在）的 SHA1 校验和等于给定的 `sha1` 值之一，则返回 true。

+   `apply_patch_space(bytes)`: 如果至少有 bytes 的临时空间可用于应用二进制补丁，则返回 true。

+   `wipe_block_device(block_dev, len)`: 清除给定块设备 `block_dev` 的 `len` 字节。

+   `read_file(filename)`: 读取 `filename` 并将其内容作为二进制块返回。

+   `sha1_check(blob[, sha1])`: `blob` 参数是 `read_file` 返回的类型或 `package_extract_file` 的一参数形式。如果没有 `sha1` 参数，此函数返回 blob 的 SHA1 哈希。如果有一个或多个 `sha1` 参数，此函数返回等于其中一个参数的 SHA1 哈希，如果不等于任何一个参数，则返回空字符串。

+   `rename(src_filename, tgt_filename)`: 将 `src_filename` 重命名为 `tgt_filename`。

+   `wipe_cache()`: 在成功安装结束时清除缓存分区。

+   `ui_print([text, ...])`: 连接所有文本参数并将结果打印到 UI。

+   `run_program(path[, arg, ...])`: 使用参数 `arg` 执行 `path` 上的二进制文件。返回程序的退出状态。

+   `reboot_now(name[, arg, ...])`: 立即重启设备。`name` 参数是传递给 Android 重启属性的分区名称。

+   `get_stage(name)`: 此函数返回由 `set_stage` 函数保存的值。`name` 参数是 `/misc` 分区的块设备。

+   `set_stage(name, stage)`: 这个函数存储一个字符串值，以便未来的恢复调用可以访问。`name` 参数是 `/misc` 分区的块设备。`stage` 是要存储的字符串。

+   `enable_reboot()`: 通过管道发送 `enable_reboot` 命令到恢复。

+   `tune2fs(arg, ...)`: 在 ext2/ext3 文件系统上更改文件系统参数。

# 块图像函数

在 Android 5.0 或更高版本中，可以使用基于块的 OTA 包。基于块的 OTA 包将整个分区视为单个文件，并在块级别进行更新。基于块的 OTA 包的函数通过 `RegisterBlockImageFunctions` 函数注册：

[PRE8]

基于块的更新实现包括三个函数：

+   `block_image_verify(partition, transfer_list, new, patch)`: `partition` 参数是更新将进行的设备。通常，它是 `/system` 分区。`transfer_list` 参数是一个包含在 `target` 分区上从一个地方传输到另一个地方的命令的文本文件。此命令仅执行干运行，不写入，以测试更新是否可以继续。

+   `block_image_update(partition, transfer_list, new, patch)`: 此函数与 `block_image_verify` 相同，但它执行实际更新。

+   `range_sha1(partition, range)`: 这个函数检查指定范围内的分区的SHA1哈希值。

# 设备扩展

作为Android系统开发者，我们可以扩展edify语言以满足我们设备的特定需求。要使用我们自己的函数扩展edify语言，我们可以使用以下函数调用注册我们的函数：

[PRE9]

我们将在下一章解释如何扩展edify语言。

# 准备x86vbox的OTA包

到目前为止，我们已经了解了OTA包内的更新器和更新器脚本。现在我们可以为我们的x86vbox设备构建OTA包了。要构建OTA包，我们可以使用以下命令：

[PRE10]

Android 5及以上版本默认构建的OTA包是构建基于块的OTA包，但我们在为x86vbox构建基于块的OTA包时会遇到错误。在我们的环境中，还需要进行很多配置才能支持基于块的OTA包。所有第三方恢复包也无法使用基于块的更新包。

为了避免这个错误，我们需要将以下`build/core/Makefile`文件更改为移除`--block`选项：

[PRE11]

构建完成后，我们可以按照以下方式检查OTA包：

[PRE12]

让我们看看我们刚刚构建的OTA包内的更新器脚本：

[PRE13]

在更新器脚本中，它首先检查当前系统的构建信息。如果当前系统比OTA包新，则不会更新系统。之后，它还会检查运行系统的设备名称和OTA包，两者应该匹配。否则，我们可能会使用错误的OTA包来更新系统。

在完成所有验证工作后，脚本将格式化`/system`分区，并从OTA包中创建一个新的`system`文件夹。一旦系统文件安装完成，脚本将创建所有必要的软链接，并应用SELinux属性。

最后，它将使用新的内核和ramdisk更新`/boot`分区。

一旦我们为x86vbox设备构建了OTA包，并在[第12章](5eff5635-ac58-4b48-80d4-b7e69b464d8e.xhtml)“介绍恢复”中构建了恢复，我们就可以更新我们的系统到OTA包。我们应该能够使用这个OTA包来更新系统，但此时系统可能无法启动。在我们能够进行更多操作之前，我们有两个问题需要解决。

回顾我们为x86vbox构建恢复的过程，我们尽可能重用了从[第8章](acf2363a-2a0f-40b9-a35f-c8bb0e523737.xhtml)“在VirtualBox上创建您的设备”到[第11章](3c6453e9-98bb-4979-9c61-f0df071b1255.xhtml)“启用VirtualBox特定硬件接口”中开发的源代码。这意味着我们在[第12章](5eff5635-ac58-4b48-80d4-b7e69b464d8e.xhtml)“介绍恢复”中构建的恢复中继承了以下功能：

+   从两个阶段的启动继承的第一个问题是，我们使用 Android `system` 文件夹中的组件来启动恢复。理想情况下，恢复不应该依赖于其他任何东西。它应该是一个自包含的系统。例如，即使系统镜像损坏，恢复也应该能够正常工作。我们可以使用恢复来修复系统。

+   我们使用 Android-x86 项目的两个阶段启动过程。正如我们从前几章中看到的那样，两个阶段启动的系统磁盘布局与标准 Android 系统不同。我们使用 OTA 包创建的系统是标准的 Android 系统磁盘布局。我们只能在 OTA 更新后使用标准的启动过程来启动系统。这意味着我们必须使用 `ramdisk.img` 而不是 `initrd.img` 来启动系统。

# 移除对 /system 的依赖

对 Android `/system` 文件夹的依赖包括两部分：

+   所有设备驱动程序的内核模块都位于：`$OUT/system/lib/modules/4.x.x-android-x86`。

+   在恢复启动过程中，我们需要运行一些基本的 Linux 命令。例如，我们使用以下命令进行硬件初始化：

    `on init exec -- /system/bin/logwrapper /system/bin/sh /system/etc/init.sh`

让我们在接下来的几节中逐一讨论前两点。

# 恢复中的硬件初始化

为了加载恢复所需的最低限度的设备驱动程序，我们必须更改 Android 系统启动的 shell 脚本执行。这是一个从一般到具体的定制过程，这与 Android-x86 项目的目标不同。在 Android-x86 项目中，所有可能的设备驱动程序都是可用的，而在这里我们只应该包含恢复所需的 VirtualBox 驱动程序。正如我们在介绍两个阶段启动时看到的那样，所有可能的设备驱动程序都在 `$OUT/system/lib/modules/4.x.x-android-x86` 文件夹中编译并可用。

内核模块将根据内核动态找到的硬件加载到系统中。在我们的例子中，我们将移除动态加载过程，只保留恢复启动所必需的最小内核模块。让我们看看 x86vbox 的原始启动脚本：

[PRE14]

在启动过程中，`init` 进程将运行前面的命令行来执行 `/system/etc/init.sh` 脚本。`/system/bin/logwrapper` 和 `/system/bin/sh` 命令都是 Android 系统中 `/system/bin` 文件夹的一部分。它们在恢复模式下不可用，因为恢复启动后没有挂载 `/system` 分区。

为了解决这个问题，我们将使用 `initrd.img` 中的 `busybox` 二进制文件在恢复环境中提供一个最小的环境来执行 Linux shell 命令。我们也不能执行 `/system/etc/init.sh` 脚本，因为它存储在 `/system/etc` 文件夹中，这个文件夹在恢复模式下也不可用。我们将通过在恢复环境中的 `/sbin` 下创建另一个脚本 `init.x86vbox.sh` 来替换它。

我们将`init.recovery.x86vbox.rc`更改为以下内容以移除对`/system`的依赖：

[PRE15]

在`early-init`阶段，我们创建软链接以使`/bin/sh`可用。我们将`/system/bin/sh`替换为位于恢复ramdisk中的`/bin/sh`。

在`init.x86vbox.sh`脚本中，我们加载恢复所需的设备驱动程序如下：

[PRE16]

如我们所见，在shell脚本`init.x86vbox.sh`中，我们首先创建了`busybox`的所有软链接。然后，我们加载了所有必要的设备驱动程序。我们还挂载了VirtualBox在`/vendor`文件夹下的共享文件夹，以便我们可以在主机和客户机之间交换数据。我们将在下一章中使用此文件夹。

# 恢复时的最小执行环境

如我们从两个脚本中可以看出，`init.recovery.x86vbox.rc`和`init.x86vbox.sh`，我们需要执行一些Linux命令，以便我们可以在启动过程中执行我们的任务。

我们需要在`ramdisk-recovery.img`中包含所有这些Linux命令，以便它们在恢复时可用。然而，问题并不像我们迄今为止所认为的那么简单。大多数命令在AOSP构建输出中是动态链接的，而不是静态链接的。

在我们的情况下，我们需要在`ramdisk-recovery.img`中包含两套共享库。Android-x86中的`initrd.img`中的`busybox`二进制文件是从AOSP树中预构建的，因此它们有自己的依赖关系。如果我们转到`newinstaller`文件夹`bootable/newinstaller/initrd`，我们可以看到可执行文件和共享库的列表：

[PRE17]

除了`busybox`二进制文件外，还有八个共享库，如前所述的片段所示。

除了`busybox`外，我们还有一些作为AOSP源树一部分构建的可执行文件。它们有一组不同的共享库，这些库也需要包含在`ramdisk-recovery.img`中。例如，显示`uvesafb`驱动程序需要一个用户空间守护进程`/sbin/v86d`，它是作为AOSP树的一部分构建的。如果没有一组共享库，它将无法正常工作。为了使我们能够运行这些可执行文件，我们需要在`ramdisk-recovery.img`中包含以下共享库：

[PRE18]

你可能想知道如何找到共享库的依赖关系。我们可以通过以下命令获取链接信息的一种方法：

[PRE19]

如前所述的输出所示，我们可以使用`readelf`命令找到`/sbin/v86d`所需的共享库。我们还需要通过恢复环境中的测试来验证依赖关系，我们将在下一章中进一步讨论。

为了将所有讨论过的内核模块和共享库包含在`ramdisk-recovery.img`中，我们更改了`x86vbox.mk`的一部分，如下所示：

![图片](img/image_13_002.png)

# 构建和测试

在本章完成所有分析后，我们现在可以构建和测试我们的代码了。

如常，我们为每一章都准备了一个清单文件。我们根据[第12章](5eff5635-ac58-4b48-80d4-b7e69b464d8e.xhtml)，“介绍恢复”的源代码对这一章进行了修改。以下是我们在这一章中更改的项目：

[PRE20]

我们可以看到我们需要更改四个项目：`recovery`、`newinstaller`、`common`和`x86vbox`。我们有一个`android-7.1.1_r4_x86vbox_ch13_r1`标签作为本章源代码的基线。

要从GitHub和AOSP直接获取源代码，可以使用以下命令：

[PRE21]

源代码准备就绪后，我们可以设置环境并按照以下步骤构建系统：

[PRE22]

要构建`initrd.img`，我们可以运行以下命令：

[PRE23]

要为x86vbox设备构建OTA包，我们可以运行以下命令：

[PRE24]

要在VirtualBox中测试AOSP镜像，我们需要使用我们在[第9章](c8d10155-cc8e-4b8c-a5e0-f359520c894a.xhtml)，“使用PXE/NFS启动x86vbox”中介绍过的PXE引导和NFS。

构建完成后，我们可以在PXE引导配置文件`$HOME/.VirtualBox/TFTP/pxelinux.cfg/default`中添加一个条目，如下以测试恢复：

[PRE25]

恢复启动后，我们可以在x86vbox设备上看到以下恢复屏幕：

![图片](img/image_13_003.png)

x86vbox的恢复用户界面在任何Android设备上看起来都一样。

在你下载源代码并自行构建所有内容之前，你也可以下载并测试本章中提供的预构建镜像，链接为[https://sourceforge.net/projects/android-system-programming/files/android-7/ch13/ch13.zip/download](https://sourceforge.net/projects/android-system-programming/files/android-7/ch13/ch13.zip/download)。

# 摘要

在这一章中，我们学习了更新器的流程，它是实际执行OTA更新工作的工具。更新器解释OTA包内的更新脚本以执行更新。我们不必自己创建更新脚本。它是在构建过程中自动创建的。你可能在这里有一些疑问，因为你可能使用了一些开源开发者或ROM开发者创建的恢复包。你甚至可能使用LineageOS/CyanogenMod或TWRP分发的恢复。它们与我们本章讨论的主题有何关联？这些是我们将在下一章中讨论的主题。
