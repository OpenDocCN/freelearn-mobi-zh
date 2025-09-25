# 第十三章。进入执行模式

作为一名工程师，你被 handed 一些 Android 设备，要求将 SE for Android 控制应用于设备以增强其安全态势。到目前为止，我们已经看到了所有需要配置的组件以及它们如何工作以启用这样的系统。在本章中，我们将使用所有涵盖的技能将 UDOO 置于执行模式。我们将：

+   运行、评估并响应 CTS 的审计日志

+   为 UDOO 开发安全策略

+   切换到执行模式

# 更新到 SEPolicy master

自 4.3 版本发布以来，AOSP `master`分支中的`sepolicy`目录已经发生了许多变化。在撰写本文时，`external/sepolicy`项目的`master`分支处于 Git 提交 SHA `b5ffb`。作者建议尝试使用最新的提交。然而，为了说明目的，我们将向您展示如何可选地检出提交`b5ffb`，以便您能准确地遵循本章的示例。

首先，您需要克隆`external/sepolicy`项目。在这些说明中，我们假设您的当前工作目录包含在`./udoo`目录中的 UDOO 源代码：

```kt
$ git clone https://android.googlesource.com/platform/external/sepolicy
$ cd sepolicy

```

如果您想精确地遵循本章的示例，您需要使用以下命令检出提交`b5ffb`。如果您跳过它，您将使用`master`分支的最新提交：

```kt
$ git checkout b5ffb

```

现在，我们将用我们从谷歌获得的内容替换 UDOO 4.3 sepolicy：

```kt
$ cd ..
$ rm -rf udoo/external/sepolicy
$ cp -r sepolicy udoo/external/sepolicy

```

可选地，您可以使用以下命令从新复制的 sepolicy 中删除`.git`文件夹，但这不是必需的：

```kt
$ rm –rf udoo/external/sepolicy/.git

```

此外，复制`audit.te`文件并恢复它。

此外，从 NSA Bitbucket `seandroid`存储库恢复`auditd`提交。供您参考，其提交 SHA 为`d270aa3`。

之后，从`udoo/build/core/Makefile`中删除所有对`setool`的引用。此命令将帮助您找到它们：

```kt
$ grep -nw setool udoo/build/core/Makefile

```

# 清除设备

到目前为止，我们的 UDOO 很混乱，所以让我们重新刷新它，包括数据目录，从头开始。我们只想保留代码和`init`脚本更改，不包含额外的 sepolicy。然后我们可以正确地编写策略并应用我们遇到的所有技术和工具。我们将从将状态重置到类似于第四章完成的状态开始，即*在 UDOO 上安装*。然而，主要区别是我们需要为 CTS 构建一个`userdebug`版本而不是一个工程(`eng`)版本。这个版本在设置脚本中选定，最终调用`lunch`。要构建这个版本，请从 UDOO 工作区执行以下命令：

```kt
$ . setup udoo-userdebug
$ make -j8 2>&1 | tee logz

```

使用以下命令刷新系统，启动到 SD 卡，并擦除`userdata`，假设 SD 卡已插入主机且`userdata`未挂载：

```kt
$ mkdir ~/userdata
$ sudo mount /dev/sdd4 ~/userdata
$ cd ~/userdata/
$ sudo rm -rf *
$ cd ..
$ sudo umount ~/userdata

```

# 设置 CTS

如果您的组织寻求 Android 品牌，则必须通过 CTS。然而，即使您不这样做，运行这些测试以确保设备符合应用程序也是一个好主意。根据您的安全目标和愿望，如果您不寻求 Android 品牌，您可能会在 CTS 的某些部分失败。对于我们的案例，我们将 CTS 视为一个测试系统并揭示阻止 UDOO 正常运行的策略问题的方法。其源代码位于 `cts/` 目录中，但我们建议直接从 Google 下载二进制文件。您可以从 [`source.android.com/compatibility/cts-intro.html`](https://source.android.com/compatibility/cts-intro.html) 和 [`source.android.com/compatibility/android-cts-manual.pdf`](https://source.android.com/compatibility/android-cts-manual.pdf) 获取更多信息以及 CTS 二进制文件本身。

从 **下载** 选项卡下载 CTS 4.3 二进制文件。然后选择 CTS 二进制文件。**兼容性定义文档**（**CDD**）也值得阅读。它涵盖了 CTS 的高级细节和兼容性要求。

从 [`source.android.com/compatibility/downloads.html`](https://source.android.com/compatibility/downloads.html) 下载 CTS 并提取它。选择与您的 Android 版本匹配的 CTS 版本。如果您不知道您的设备正在运行哪个版本，您始终可以使用 `getprop ro.build.version.release` 从 UDOO 检查 `ro.build.version.release` 属性：

```kt
$ mkdir ~/udoo-cts
$ cd ~/udoo-cts
$ wget https://dl.google.com/dl/android/cts/android-cts-4.3_r2-linux_x86-arm.zip
$ unzip android-cts-4.3_r2-linux_x86-arm.zip

```

# 运行 CTS

CTS 在设备上测试了许多组件，并有助于测试系统的各个部分。一个好的、通用的策略应该允许 Android 正常运行并通过 CTS。

按照 Android CTS 用户手册中的说明设置您的设备（见 *第 3.3 节*，*设置您的设备*）。通常，如果您没有精确地遵循所有步骤，您可能会看到一些失败，因为您可能没有访问权限或能力获取所有需要的资源。然而，CTS 仍然会测试一些代码路径。至少，我们建议复制媒体文件并激活 Wi-Fi。一旦您的设备设置完成，请确保 `adb` 激活并开始测试：

```kt
$ ./cts-tradefed
11-30 10:30:08 I/: Detected new device 0123456789ABCDEF
cts-tf > run cts --plan CTS
cts-tf > 
time passes here
11-30 10:30:28 I/TestInvocation: Starting invocation for 'cts' on build '4.3_r2' on device 0123456789ABCDEF
11-30 10:30:28 I/0123456789ABCDEF: Created result dir 2014.11.30_10.30.28
11-30 10:31:44 I/0123456789ABCDEF: Collecting device info
11-30 10:31:45 I/0123456789ABCDEF: -----------------------------------------
11-30 10:31:45 I/0123456789ABCDEF: Test package android.aadb started
11-30 10:31:45 I/0123456789ABCDEF: -----------------------------------------
11-30 10:32:15 I/0123456789ABCDEF: com.android.cts.aadb.TestDeviceFuncTest#testBugreport PASS 
...

```

测试需要执行很多小时，所以请耐心等待；但您可以检查测试的状态：

```kt
cts-tf > l i
Command Id  Exec Time Device State 
1 8m:22 0123456789ABCDEF running cts on build 4.3_r2 

```

插入扬声器以享受媒体测试和铃声的声音！此外，CTS 会重新启动设备。如果重新启动后您的 ADB 会话没有恢复，ADB 可能不会执行任何测试。在运行 `cts-tf > run cts --plan CTS --disable-reboot` 计划时，请使用 `--disable-reboot` 选项。

# 收集结果

首先，我们将考虑 CTS 结果。尽管我们预计会有一些失败，但我们也预计在执行强制模式时问题不会变得更糟。其次，我们将查看审计日志。让我们从设备中提取这两个文件。

## CTS 测试结果

每次运行 CTS 时，它都会创建一个测试结果目录。CTS 指示目录名称，但不是位置：

```kt
11-30 10:30:28 I/0123456789ABCDEF: Created result dir 2014.11.30_10.30.28

```

位置在 CTS 手册中提到，可以在 `repository/results` 下的提取 CTS 目录中找到，通常在 `android-cts/repository/results`。测试目录包含一个 XML 测试报告，`testResult.xml`。这可以在大多数网络浏览器中打开。它提供了测试的概述和所有执行测试的详细信息。`pass:fail` 比率是我们的基准。作者有 18,736 个通过，只有 53 个失败，考虑到其中一半是功能问题，如没有蓝牙或返回相机支持为 true，这个结果相当不错。

## 审计日志

我们将使用审计日志来解决我们政策中的不足。使用本书中一直使用的标准 `adb pull` 命令将这些内容从设备上拉取。由于这是一个 `userdebug` 构建且默认的 `adb` 终端是 shell `uid`（不是 root），请使用 `adb root` 以 root 身份启动 `adb`。`su` 也在 `userdebug` 构建中可用。

### 小贴士

你可能会得到一个错误，说 `/data/misc/audit/audit.log` 文件不存在。解决方案是使用 `adb root` 命令以 root 身份运行 `adb`。此外，当运行此命令时，它可能会挂起。只需转到设置，禁用，然后在 **开发者选项** 下的 **USB 调试** 中启用。然后终止 `adb-root` 命令，并通过运行 `adb shell` 验证你是否再次成为 root 用户。

# 编写设备策略

将 `audit.log` 和 `audit.old` 都通过 `audit2allow` 运行，以查看发生了什么。`audit2allow` 的输出按源域分组。我们不会全部通过，而是会突出显示不寻常的情况，从 `audit2allow` 的解释结果开始。假设你处于审计日志目录中，执行 `cat audit.* | audit2allow | less`。任何策略工作都将在该设备特定的 UDOO sepolicy 目录中完成。

## adbd

以下是我们通过 `audit2allow` 过滤后的 `adbd` 拒绝项：

```kt
#============= adbd ==============
allow adbd ashmem_device:chr_file execute;
allow adbd dumpstate:unix_stream_socket connectto;
allow adbd dumpstate_socket:sock_file write;
allow adbd input_device:chr_file { write getattr open };
allow adbd log_device:chr_file { write read ioctl open };
allow adbd logcat_exec:file { read getattr open execute execute_no_trans };
allow adbd mediaserver:binder { transfer call };
allow adbd mediaserver:fd use;
allow adbd self:capability { net_raw dac_override };
allow adbd self:process execmem;
allow adbd shell_data_file:file { execute execute_no_trans };
allow adbd system_server:binder { transfer call };
allow adbd tmpfs:file execute;
allow adbd unlabeled:dir getattr;
```

`adbd` 域中的拒绝项相当奇怪。首先引起我们注意的是对 `/dev/ashmem` 的 `execute` 操作，这是一个字符驱动器。通常，这只需要用于 Dalvik JIT。查看原始审计日志（`cat audit.* | grep adbd | grep execute`），我们看到以下内容：

```kt
type=1400 msg=audit(1417416666.182:788): avc: denied { execute } for pid=3680 comm="Compiler" path=2F6465762F6173686D656D2F64616C76696B2D6A69742D636F64652D6361636865202864656C6574656429 dev=tmpfs ino=412027 scontext=u:r:adbd:s0 tcontext=u:object_r:tmpfs:s0 tclass=file
type=1400 msg=audit(1417416670.352:831): avc: denied { execute } for pid=3753 comm="Compiler" path="/dev/ashmem" dev=tmpfs ino=1127 scontext=u:r:adbd:s0 tcontext=u:object_r:ashmem_device:s0 tclass=chr_file

```

编译器的 `comm` 进程字段在 `ashmem` 上正在执行。我们的猜测是这与 Dalvik 有关，但为什么它在 `adbd` 域中？还有，为什么 `adbd` 正在写入输入设备？所有这些行为都很奇怪。通常，当你看到这类事情时，是因为子进程没有进入正确的域。运行以下命令来检查域并确认我们的怀疑：

```kt
$ adb shell ps -Z | grep adbd
u:r:adbd:s0 root 20046 1 /sbin/adbd
u:r:adbd:s0 root 20101 20046 ps

```

然后，我们运行 `adb shell ps -Z | grep adbd` 来查看哪些内容在 `adb` 域中运行，进一步确认我们的怀疑：

```kt
u:r:adbd:s0 root 20046 1 /sbin/adbd
u:r:adbd:s0 root 20101 20046 ps

```

`ps` 命令不应该在 `adbd` 上下文中运行；它应该在 `shell` 中运行。这证实了 `shell` 不在正确的域中：

```kt
$ adb shell
root@udoo:/ # id
uid=0(root) gid=0(root) context=u:r:adbd:s0

```

首先要检查的是文件上下文：

```kt
root@udoo:/ # ls -Z /system/bin/sh
lrwxr-xr-x root shell u:object_r:system_file:s0 sh -> mksh
root@udoo:/ # ls -Z /system/bin/mksh
-rwxr-xr-x root shell u:object_r:system_file:s0 mksh

```

基础策略定义了当 `adbd` 使用 `exec` 加载 shell 并进入 shell 域时的域转换。这在外部 sepolicy 的 `adbd.te` 中定义为 `domain_auto_trans(adbd, shell_exec, shell)`。

显然，shell 上应用了一个错误的标记，所以让我们查看外部 sepolicy 中的 `file_contexts` 来找出原因。

```kt
$ cat file_contexts | grep shell_exec
/system/bin/sh -- u:object_r:shell_exec:s0

```

两个连字符表示只有常规文件会被标记，而符号链接将被跳过。我们可能不想标记符号链接，而是标记 `mksh` 目标。通过向设备 UDOO sepolicy 添加自定义 `file_contexts` 条目并将文件添加到 `BOARD_SEPOLICY_UNION` 配置中来实现这一点。在 `file_contexts` 中添加 `/system/bin/mksh -- u:object_r:shell_exec:s0`，并在 `sepolicy.mk` 中添加 `BOARD_SEPOLICY_UNION += file_contexts`。

### 提示

在本章剩余部分，无论何时创建或修改策略文件（例如，上下文文件或 `*.te` 文件），别忘了将它们添加到 `sepolicy.mk` 中的 `BOARD_SEPOLICY_UNION`。

由于这是一个与策略和 `adbd` 相关的相当致命的问题，我们现在不会担心拒绝，除了未标记的情况。每当遇到未标记的文件时，都应该解决。导致此拒绝的 `avc` 拒绝如下：

```kt
type=1400 msg=audit(1417405835.872:435): avc: denied { getattr } for pid=4078 comm="ls" path="/device" dev=mmcblk0p7 ino=2 scontext=u:r:adbd:s0 tcontext=u:object_r:unlabeled:s0 tclass=dir

```

因为这个目录挂载在 `/device`，而 Android 的挂载通常在 `/`，我们应该查看挂载表：

```kt
root@udoo:/ # mount | grep device
/dev/block/mmcblk0p7 /device ext4 ro,seclabel,nosuid,nodev,relatime,user_xattr,barrier=1,data=ordered 0 0

```

通常，挂载命令位于初始化脚本中的 `mkdir` 之后，或者在包含初始化内置的 `mount_all` 的 `fstab` 文件中。在 `init.rc` 中搜索 `device` 和 `mkdir` 的快速搜索没有找到任何内容，但在 `fstab.freescale` 中找到了。设备是只读的，因此我们应该能够给它一个类型，用文件上下文标记它，并将其 `getattr` 域应用于其目录类。由于它是只读且为空的，所以没有人需要更多的权限。查看 `make_sd.sh` 脚本，我们注意到块设备的第 7 个分区是 `vender` 目录。这是对 OEM 在其中放置专有二进制文件的常见供应商目录的误拼。我们在 `file.te` 中放置文件类型，并在 `domain.te` 中放置域允许规则。

在 `file.te` 中添加以下内容：

```kt
type udoo_device_file, file_type;
```

在 `domain.te` 中添加以下内容：

```kt
allow domain udoo_device_file:dir getattr;
```

在 `file_contexts` 中添加以下内容：

```kt
/device u:object_r:udoo_device_file:s0
```

如果这个目录不为空，你必须手动运行 `restorecon -R` 来标记现有文件。

如果你多次从 UDOO 中拉取审计日志，你也可能看到表示你这样做拒绝的条目，因为 `adbd` 将无法访问它们。你可能看到如下内容：

```kt
#============= adbd ==============
allow adbd audit_log:file { read getattr open };
```

这条规则来自测试结束时的操作，当时你使用了 `adb pull` 来获取审计日志。我们可以安全地添加 `dontaudit` 并加入 `neverallow` 来确保它不会意外地被允许。审计日志包含恶意软件编写者可能用来在策略中导航的信息，这些信息应该受到保护。在设备的 `sepolicy` 文件夹中，添加一个 `adbd.te` 文件并将其联合到 `sepolicy.mk` 文件中：

在 `adbd.te` 中添加以下内容：

```kt
# dont audit adb pull and adb shell cat of audit logs
dontaudit adbd audit_log:file r_file_perms;
dontaudit shell audit_log:file r_file_perms;
```

在 `auditd.te` 中添加以下内容：

```kt
# Make sure no one adds an allow to the audit logs
# from anything but system server (read only) and
# auditd, rw access.
neverallow { domain -system_server -auditd -init -kernel } audit_log:file ~getattr;
neverallow system_server audit_log:file ~r_file_perms;
```

如果 `auditd.te` 仍然位于 `external/sepolicy` 中，将其移至 `device/fsl/udoo/sepolicy`，并附带所有依赖类型。

`neverallow` 条目显示了如何使用补全运算符 `~` 和集合差运算符 `-` 来进行强断言或简短表达。第一个 `neverallow` 以域开始，并且所有进程类型（域）都是域属性的成员。我们通过集合差来防止访问，留下必须永远没有访问权限的集合。然后我们补全访问向量集合，只允许对日志执行 `getattr` 或 `stat` 操作。第二个 `neverallow` 使用补全来确保 `system_server` 限于读取操作。

## bootanim

`bootanim` 域分配给了启动动画服务，该服务在启动时显示启动画面，通常是运营商的品牌：

```kt
#============= bootanim ==============
allow bootanim init:unix_stream_socket connectto;
allow bootanim log_device:chr_file { write open };
allow bootanim property_socket:sock_file write;
```

任何接触 `init` 域的东西都是一个红旗。在这里，`bootanim` 连接到一个 init Unix 域套接字。这是属性系统的一部分，我们可以看到连接后，它向属性套接字写入。套接字对象及其 URI 是分开的。在这种情况下，它是文件系统，但也可能是匿名套接字：

```kt
type=1400 msg=audit(1417405616.640:255): avc: denied { connectto } for pid=2534 comm="BootAnimation" path="/dev/socket/property_service" scontext=u:r:bootanim:s0 tcontext=u:r:init:s0 tclass=unix_stream_socket

```

`log_device` 在 Android 的新版本中已弃用，并被 `logd` 替换。然而，我们正在将新的主 sepolicy 迁移到 4.3，因此我们必须支持它。移除支持的补丁位于 [`android-review.googlesource.com/#/c/108147/`](https://android-review.googlesource.com/#/c/108147/)。

我们可以不在外部 sepolicy 上应用反向补丁，而是直接在我们的设备策略文件 `domain.te` 中添加规则。我们可以安全地使用设备 UDOO `sepolicy` 文件夹中的适当宏和样式来允许这些操作。在 `bootanim.te` 中添加 `unix_socket_connect(bootanim, property, init)`，并在 `domain.te` 中添加以下内容：

```kt
allow domain udoo_device_file:dir getattr;
allow domain log_device:dir search;
allow domain log_device:chr_file rw_file_perms;
```

## debuggerd

```kt
#============= debuggerd ==============
allow debuggerd log_device:chr_file { write read open };
allow debuggerd system_data_file:sock_file write;
```

在 `bootanim` 下通过添加所有域使用 `log_device` 的允许规则来解决日志设备拒绝问题。`system_data_file:sock_file write` 是奇怪的。在大多数情况下，你几乎永远不会想要允许跨域写入，但这是特殊情况。看看原始拒绝：

```kt
type=1400 msg=audit(1417415122.602:502): avc: denied { write } for pid=2284 comm="debuggerd" name="ndebugsocket" dev=mmcblk0p4 ino=129525 scontext=u:r:debuggerd:s0 tcontext=u:object_r:system_data_file:s0 tclass=sock_file

```

拒绝操作发生在 `ndebugsocket` 上。搜索这个名称类型转换，发现策略版本 23 不支持：

```kt
system_server.te:297:type_transition system_server system_data_file:sock_file system_ndebug_socket "ndebugsocket";

```

我们必须更改代码以设置正确的上下文或直接允许它，我们将这样做。我们不会授予额外的权限，因为它从未请求打开，并且我们正在跨域。防止跨域打开文件是理想的，因为获取此文件描述符的唯一方式是通过进入拥有域的 IPC 调用。在 `debuggerd.te` 中添加 `allow debuggerd system_data_file:sock_file write;`。

## drmserver

```kt
#============= drmserver ==============
allow drmserver log_device:chr_file { write open };
```

这由 `domain.te` 规则处理，所以我们这里没有事情要做。

## dumpstate

```kt
#============= dumpstate ==============
allow dumpstate init:binder call;
allow dumpstate init:process signal;
allow dumpstate log_device:chr_file { write read open };
allow dumpstate node:rawip_socket node_bind;
allow dumpstate self:capability sys_resource;
allow dumpstate system_data_file:file { write rename create setattr };
```

`dumpstate` 上对 `init:binder call` 的拒绝很奇怪，因为 `init` 不使用 binder。某些进程必须保持在 `init` 域中。让我们检查我们的进程列表以查找 `init`：

```kt
$ adb shell ps -Z | grep init
u:r:init:s0 root 1 0 /init
u:r:init:s0 root 2286 1 zygote
u:r:init:s0 radio 2759 2286 com.android.phone

```

在这里，`zygote`和`com.android.phone`不应以`init`运行。这必须是`app_process`文件上的标签错误，该文件是 zygote。`ls -laZ /system/bin/app_process`命令显示`u:object_r:system_file:s0 app_process`，因此向`file_contexts`添加条目以纠正此错误。我们可以在基础 sepolicy 中找到要使用的标签，该基础 sepolicy 定义为`zygote_exec`类型：

```kt
# zygote
type zygote, domain;
type zygote_exec, exec_type, file_type;
```

在`file_contexts`中添加`/system/bin/app_process u:object_r:zygote_exec:s0`。

## installd

添加的`domain.te`规则处理`installd`。

## keystore

```kt
#============= keystore ==============
allow keystore app_data_file:file write;
allow keystore log_device:chr_file { write open };
```

日志设备由`domain.te`规则管理。让我们看看原始的`app_data_file`拒绝项：

```kt
type=1400 msg=audit(1417417454.442:845): avc: denied { write } for pid=15339 comm="onCtsTestRunner" path="/data/data/com.android.cts.stub/cache/CTS_DUMP" dev=mmcblk0p4 ino=131242 scontext=u:r:keystore:s0 tcontext=u:object_r:app_data_file:s0:c512,c768 tclass=file

```

类别在上下文中定义。这意味着 MLS 支持已为`app domains`激活。在`seapp_contexts`基础 sepolicy 中，我们看到：

```kt
user=_app domain=untrusted_app type=app_data_file levelFrom=user
user=_app seinfo=platform domain=platform_app type=app_data_file levelFrom=user
```

应用数据的多级安全（MLS）分离仍在开发中，在 4.3 版本中并未工作，因此我们可以禁用此功能。我们只需在特定设备的`seapp_contexts`文件中声明它们。在`seapp_contexts`中，添加`user=_app domain=untrusted_app type=app_data_file`和`user=_app seinfo=platform domain=platform_app type=app_data_file`。在 4.3 版本中，对数据上下文所做的任何更改都需要进行工厂重置。4.4 版本增加了智能重标记功能。

## mediaserver

```kt
#============= mediaserver ==============
allow mediaserver adbd:binder { transfer call };
allow mediaserver init:binder { transfer call };
allow mediaserver log_device:chr_file { write open };
```

日志设备在`domain.te`规则中得到了处理。我们也将跳过`init`和`adbd`，因为它们的问题是由不正确的进程域触发的。不要盲目添加允许规则很重要，因为现有域的大部分工作可以通过小的标签更改或几个规则来处理。

## netd

```kt
#============= netd ==============
allow netd kernel:system module_request;
allow netd log_device:chr_file { write open };
```

`netd`的日志设备拒绝由`domain.te`处理。然而，我们应该仔细审查任何请求权限的操作。在授予权限时，策略作者需要非常小心。如果某个域被授予加载系统模块的能力，并且该域或模块的二进制文件本身被破坏，这可能导致通过可加载模块将恶意软件注入内核。但是，`netd`需要可加载内核模块支持以支持某些卡片。将允许规则添加到设备 UDOO sepolicy 中的名为`netd.te`的文件中。在`netd.te`中，添加`allow netd self:capability sys_module;`。

## rild

```kt
#============= rild ==============
allow rild log_device:chr_file { write open };
```

这由`domain.te`规则处理，因此我们在这里没有要做的事情。

## servicemanager

```kt
#============= servicemanager ==============
allow servicemanager init:binder transfer;
allow servicemanager log_device:chr_file { write open };
```

再次，日志设备在`domain.te`中处理。我们将跳过`init`，因为其问题是由不正确的进程域触发的。

## surfaceflinger

```kt
#============= surfaceflinger ==============
allow surfaceflinger init:binder transfer;
allow surfaceflinger log_device:chr_file { write open };
```

再次，日志设备在`domain.te`中得到了处理。我们也将跳过`init`，因为其问题是由不正确的进程域触发的。

## system_server

```kt
#============= system_server ==============
allow system_server adbd:binder { transfer call };
allow system_server dalvikcache_data_file:file { write setattr };
allow system_server init:binder { transfer call };
allow system_server init:file write;
allow system_server init:process { setsched sigkill getsched };
allow system_server init_tmpfs:file read;
allow system_server log_device:chr_file write;
```

由于`log_device`由`domain.te`管理，并且`init`和`adbd`已被污染，我们只需处理 Dalvik 缓存拒绝项：

```kt
type=1400 msg=audit(1417405611.550:159): avc: denied { write } for pid=2571 comm="er.ServerThread" name="system@app@SettingsProvider.apk@classes.dex" dev=mmcblk0p4 ino=129458 scontext=u:r:system_server:s0 tcontext=u:object_r:dalvikcache_data_file:s0 tclass=file
type=1400 msg=audit(1417405611.550:160): avc: denied { setattr } for pid=2571 comm="er.ServerThread" name="system@app@SettingsProvider.apk@classes.dex" dev=mmcblk0p4 ino=129458 scontext=u:r:system_server:s0 tcontext=u:object_r:dalvikcache_data_file:s0 tclass=file

```

外部 sepolicy seandroid-4.3 分支允许 `domain.te:allow domain dalvikcache_data_file:file r_file_perms;`。通过 `system_app` 的 `system_app.te:allow system_app dalvikcache_data_file:file { write setattr };` 允许写入。我们应该能够授予这种写入访问权限，因为可能需要更新其 Dalvik 缓存文件。在 `domain.te` 中添加 `allow domain dalvikcache_data_file:file r_file_perms;`，在 `system_server.te` 中添加 `allow system_server dalvikcache_data_file:file { write setattr };`。

## toolbox

```kt
#============= toolbox ==============
allow toolbox sysfs:file write;
```

通常，不应该向 `sysfs` 写入。现在看看对有问题的 `sysfs` 文件的原始拒绝：

```kt
type=1400 msg=audit(1417405599.660:43): avc: denied { write } for pid=2309 comm="cat" path="/sys/module/usbtouchscreen/parameters/calibration" dev=sysfs ino=2318 scontext=u:r:toolbox:s0 tcontext=u:object_r:sysfs:s0 tclass=file

```

从这里，我们正确地标记了 `/sys/module/usbtouchscreen/parameters/calibration`。我们在 `file_contexts` 中添加条目以标记 `sysfs`，在 `file.te` 中声明类型，并允许 `toolbox` 访问它。在 `file.te` 中添加 `type sysfs_touchscreen_calibration, fs_type, sysfs_type, mlstrustedobject;`，在 `file_contexts` 中添加 `/sys/module/usbtouchscreen/parameters/calibration -- u:object_r:sysfs_touchscreen_calibration:s0`，在 `toolbox.te` 中添加 `allow toolbox sysfs_touchscreen_calibration:file w_file_perms;`。

## untrusted_app

```kt
#============= untrusted_app ==============
allow untrusted_app adb_device:chr_file getattr;
allow untrusted_app adbd:binder { transfer call };
allow untrusted_app adbd:dir { read getattr open search };
allow untrusted_app adbd:file { read getattr open };
allow untrusted_app adbd:lnk_file read;
...
```

`untrusted_app` 有许多拒绝。考虑到域标记问题，我们现在不会解决这些问题中的大多数。然而，你应该注意误标记和无标记的目标文件。在通过 `audit2allow` 解释的拒绝日志中搜索时，发现了以下内容：

```kt
allow untrusted_app device:chr_file { read getattr };
allow untrusted_app unlabeled:dir { read getattr open };
```

对于 `chr_file` 设备，我们得到以下结果：

```kt
type=1400 msg=audit(1417416653.742:620): avc: denied { read } for pid=3696 comm="onCtsTestRunner" name="rfkill" dev=tmpfs ino=1126 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417416666.152:784): avc: denied { getattr } for pid=3696 comm="onCtsTestRunner" path="/dev/mxs_viim" dev=tmpfs ino=1131 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417416653.592:561): avc: denied { getattr } for pid=3696 comm="onCtsTestRunner" path="/dev/.coldboot_done" dev=tmpfs ino=578 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:device:s0 tclass=file

```

因此，我们需要正确地标记 `/dev/.coldboot_done`、`/dev/rfkill` 和 `/dev/mxs_viim`。`/dev/rfkill` 应该按照 4.3 策略进行标记：

```kt
file_contexts:/sys/class/rfkill/rfkill[0-9]*/state -- u:object_r:sysfs_bluetooth_writable:s0
file_contexts:/sys/class/rfkill/rfkill[0-9]*/type -- u:object_r:sysfs_bluetooth_writable:s0
```

`/dev/mxs_viim` 设备似乎是一个全局可访问的 GPU。我们建议彻底审查源代码，但到目前为止，我们将它标记为 `gpu_device`。`/dev/.coldboot_done` 是在 `coldboot` 进程完成后由 `ueventd` 创建的。如果 `ueventd` 重新启动，它将跳过冷启动。我们不需要标记这个。这个拒绝是由目标文件上的源域 MLS 引起的，该文件不是源和目标类别子集，并且没有 `mlstrustedsubject` 属性；当我们从应用程序中删除 MLS 支持时，它应该消失。

在 `file_contexts` 中：

```kt
# touch screen calibration
/sys/module/usbtouchscreen/parameters/calibration -- u:object_r:sysfs_touchscreen_calibration:s0
#BT RFKill node
/sys/class/rfkill/rfkill[0-9]*/state -- u:object_r:sysfs_bluetooth_writable:s0
/sys/class/rfkill/rfkill[0-9]*/type -- u:object_r:sysfs_bluetooth_writable:s0
```

## vold

```kt
#============= vold ==============
allow vold log_device:chr_file { write open };
```

再次，日志设备在 `domain.te` 中被处理。

## watchdogd

```kt
#============= watchdogd ==============
allow watchdogd device:chr_file { read write create unlink open };
```

来自 `watchdog` 的原始拒绝描绘了一幅有趣的画面：

```kt
type=1400 msg=audit(1417405598.000:8): avc: denied { create } for pid=2267 comm="watchdogd" name="__null__" scontext=u:r:watchdogd:s0 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417405598.000:9): avc: denied { read write } for pid=2267 comm="watchdogd" name="__null__" dev=tmpfs ino=2580 scontext=u:r:watchdogd:s0 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417405598.000:10): avc: denied { open } for pid=2267 comm="watchdogd" name="__null__" dev=tmpfs ino=2580 scontext=u:r:watchdogd:s0 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417405598.000:11): avc: denied { unlink } for pid=2267 comm="watchdogd" name="__null__" dev=tmpfs ino=2580 scontext=u:r:watchdogd:s0 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417416653.602:575): avc: denied { getattr } for pid=3696 comm="onCtsTestRunner" path="/dev/watchdog" dev=tmpfs ino=1095 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:watchdog_device:s0 tclass=chr_file

```

文件由 `watchdog` 创建并解除链接，它保留了对一个匿名文件的处理权。解除链接后，文件系统引用不存在，但文件描述符是有效的，只有 `watchdog` 可以使用它。在这种情况下，我们可以只允许 `watchdog` 这个规则。在 `watchdogd.te` 中添加 `allow watchdogd device:chr_file create_file_perms;`。然而，这个规则在基本策略中引起了 `neverallow` 违规：

```kt
out/host/linux-x86/bin/checkpolicy: loading policy configuration from out/target/product/udoo/obj/ETC/sepolicy_intermediates/policy.conf
libsepol.check_assertion_helper: neverallow on line 5375 violated by allow watchdogd device:chr_file { read write open };
Error while expanding policy

```

`neverallow` 规则在 `domain.te` 基本策略中作为 `neverallow { domain -init -ueventd -recovery } device:chr_file { open read write };`。对于这样的简单更改，我们只需修改基本 sepolicy 为 `neverallow { domain -init -ueventd -recovery -watchdogd } device:chr_file { open read write };`。

## wpa

```kt
#============= wpa ==============
allow wpa device:chr_file { read open };
allow wpa log_device:chr_file { write open };
allow wpa system_data_file:dir { write remove_name add_name setattr };
allow wpa system_data_file:sock_file { write create unlink setattr };
```

再次，日志设备在 `domain.te` 中已处理。系统数据访问需要进一步调查，从原始拒绝开始：

```kt
type=1400 msg=audit(1417405614.060:193): avc: denied { setattr } for pid=2639 comm="wpa_supplicant" name="wpa_supplicant" dev=mmcblk0p4 ino=129295 scontext=u:r:wpa:s0 tcontext=u:object_r:system_data_file:s0 tclass=dir
type=1400 msg=audit(1417405614.060:194): avc: denied { write } for pid=2639 comm="wpa_supplicant" name="wlan0" dev=mmcblk0p4 ino=129318 scontext=u:r:wpa:s0 tcontext=u:object_r:system_data_file:s0 tclass=sock_file
type=1400 msg=audit(1417405614.060:195): avc: denied { write } for pid=2639 comm="wpa_supplicant" name="wpa_supplicant" dev=mmcblk0p4 ino=129295 scontext=u:r:wpa:s0 tcontext=u:object_r:system_data_file:s0 tclass=dir
type=1400 msg=audit(1417405614.060:196): avc: denied { remove_name } for pid=2639 co

```

使用 `ls -laR` 定位到有问题的文件：

```kt
/data/system/wpa_supplicant:
srwxrwx--- wifi wifi 2014-12-01 06:43 wlan0

```

此套接字是由 `wpa_supplicant` 本身创建的。由于没有类型转换，无法重新标记它，因此我们必须允许它。在 `wpa.te` 中添加 `allow wpa system_data_file:dir rw_dir_perms;` 和 `allow wpa system_data_file:sock_file create_file_perms;`。未标记的设备已经处理过；它位于 `rfkill`：

```kt
type=1400 msg=audit(1417405613.640:175): avc: denied { read } for pid=2639 comm="wpa_supplicant" name="rfkill" dev=tmpfs ino=1126 scontext=u:r:wpa:s0 tcontext=u:object_r:device:s0 tclass=chr_file

```

# 第二次策略遍历

在加载草拟的策略后，设备在启动时仍然有拒绝：

```kt
#============= init ==============
allow init rootfs:file { write create };
allow init system_file:file execute_no_trans;
#============= shell ==============
allow shell device:chr_file { read write getattr };
allow shell system_file:file entrypoint;
```

所有这些拒绝都应该进行调查，因为它们针对敏感类型，特别是 `tcontext`。

## init

`init` 的原始拒绝如下：

```kt
<5>type=1400 audit(4.380:3): avc: denied { create } for pid=2268 comm="init" name="tasks" scontext=u:r:init:s0 tcontext=u:object_r:rootfs:s0 tclass=file
<5>type=1400 audit(4.380:4): avc: denied { write } for pid=2268 comm="init" name="tasks" dev=rootfs ino=3080 scontext=u:r:init:s0 tcontext=u:object_r:rootfs:s0 tclass=file

```

这些发生在 `init` 将 `/` 重新挂载为只读之前。我们可以安全地允许这些操作，并且由于 `init` 正在运行未限制，我们只需将其添加到 `init.te`。我们本可以将 `allow` 规则添加到未限制集中，但由于它即将消失，让我们将权限最小化到仅 `init`：

```kt
allow int rootfs:file create_file_perms;
```

### 注意

未限制的并不是完全未限制的。随着 AOSP 越来越接近零未限制域，规则将从该域中移除。

然而，这样做会导致另一个 `neverallow` 失败。我们可以修改 `external/sepolicy domain.te` 来绕过这个问题。将 `neverallow` 从以下内容更改为：

```kt
# Nothing should be writing to files in the rootfs.
neverallow { domain -recovery} rootfs:file { create write setattr relabelto append unlink link rename };
```

更改为以下内容：

```kt
# Nothing should be writing to files in the rootfs.
neverallow { domain -recovery -init } rootfs:file { create write setattr relabelto append unlink link rename };
```

### 注意

如果您需要修改 `neverallow` 条目以构建，您将无法通过 CTS。正确的方法是从 `init` 中移除这种行为。

此外，我们还需要查看没有域转换的 `exec` 加载了什么，导致 `execute_no_trans` 拒绝：

```kt
<5>type=1400 audit(4.460:6): avc: denied { execute_no_trans } for pid=2292 comm="init" path="/system/bin/magd" dev=mmcblk0p5 ino=146 scontext=u:r:init:s0 tcontext=u:object_r:system_file:s0 tclass=file
<5>type=1400 audit(4.460:6): avc: denied { execute_no_trans } for pid=2292 comm="init" path="/system/bin/rfkill" dev=mmcblk0p5 ino=148 scontext=u:r:init:s0 tcontext=u:object_r:system_file:s0 tclass=file

```

为了解决这个问题，我们可以用 `magd` 的自身类型重新标记它，并将其放置在其自己的未限制域中。基本策略中的 `neverallow` 强迫我们将每个可执行文件移动到其自己的域中。

创建一个名为 `magd.te` 的文件，将其添加到 `BOARD_SEPOLICY_UNION`，并将以下内容添加到其中：

```kt
type magd, domain;
type magd_exec, exec_type, file_type;
permissive_or_unconfined(magd);
```

还需要更新 `file_contexts` 以包含以下内容：

```kt
/system/bin/magd  u:object_r:magd_exec:s0
```

重复为`magd`所做的步骤，对于`rfkill`进行操作。只需在先前的示例中将`magd`替换为`rfkill`。后续测试显示了一个入口点拒绝，源上下文是`init_shell`，目标是`rfkill_exec`。在添加 shell 规则后，发现`rfkill`是通过`exec`从`init_shell`域加载的，因此让我们也向`rfkill.te`文件添加`domain_auto_trans(init_shell, rfkill_exec, rfkill)`。此外，与这一发现一起的是`rfkill`尝试打开、读取和写入`/dev/rfkill`。因此，我们必须将`/dev/rfkill`标记为`rfkill_device`，允许`rfkill`访问它，并将`allow rfkill rfkill_device:chr_file rw_file_perms;`追加到`rfkill.te`文件。创建一个新文件来声明此设备类型，命名为`device.te`，并添加`type rfkill_device, dev_type;`。之后，通过添加`file_contexts`来标记它，添加`/dev/rfkill u:object_r:rfkill_device:s0`。

## shell

我们将评估的第一个 shell 拒绝是`entrypoint`上的拒绝：

```kt
<5>type=1400 audit(4.460:5): avc: denied { entrypoint } for pid=2279 comm="init" path="/system/bin/mksh" dev=mmcblk0p5 ino=154 scontext=u:r:shell:s0 tcontext=u:object_r:system_file:s0 tclass=file
```

由于我们没有标记`mksh`，现在需要对其进行标记。我们可以为`init`启动的 shell 创建一个不受限制的域，以便最终进入`init_shell`域。控制台仍然通过显式的`seclabel`进入`shell`域，其他调用最终成为`init_shell`。创建一个新文件，`init_shell.te`，并将其添加到`BOARD_SEPOLICY_UNION`。

## init_shell.te

```kt
type init_shell, domain;
domain_auto_trans(init, shell_exec, init_shell);
permissive_or_unconfined(init_shell);
```

更新`file_contexts`以包含以下内容：

```kt
/system/bin/mksh  u:object_r:shell_exec:s0;
```

现在我们将处理对原始设备的 shell 访问：

```kt
<5>type=1400 audit(6.510:7): avc: denied { read write } for pid=2279 comm="sh" name="ttymxc1" dev=tmpfs ino=122 scontext=u:r:shell:s0 tcontext=u:object_r:device:s0 tclass=chr_file
<5>type=1400 audit(7.339:8): avc: denied { getattr } for pid=2279 comm="sh" path="/dev/ttymxc1" dev=tmpfs ino=122 scontext=u:r:shell:s0 tcontext=u:object_r:device:s0 tclass=chr_file

```

这只是一个错误标记的`tty`，因此我们可以将其标记为`tty_device`。向文件上下文中添加以下条目：

```kt
/dev/ttymxc[0-9]*  u:object_r:tty_device:s0
```

# 现场试验

到目前为止，重新构建源树，擦除数据文件系统，刷新，并重新运行 CTS。重复此操作，直到所有拒绝都被解决。

一旦完成 CTS 和内部 QA 测试，我们建议在允许模式下进行现场试验。在此期间，你应该收集日志并完善策略。如果域不稳定，你可以在策略文件中将它们声明为允许，同时仍然将设备置于执行模式；执行一些域比不执行任何域要好。

# 进入执行模式

你可以通过`bootloader`（这里将不会涉及）或通过在引导时间早期运行的`init.rc`脚本来传递执行模式。你可以在`setcon`之后这样做：

```kt
setcon u:r:init:s0
setenforce 1
```

一旦此语句编译到`init.rc`脚本中，就只能通过后续构建和重新刷新`boot.img`来撤销。你可以通过运行`getenforce`命令来检查这一点。此外，作为一个有趣的测试，你可以尝试从 root 串行控制台运行`reboot`命令并观察它失败：

```kt
root@udoo:/ # getenforce
Enforcing
root@udoo:/ # reboot
reboot: Operation not permitted

```

# 摘要

在本章中，你之前对系统的所有理解都被用来为全新的设备开发实际的 Android SE 政策。你现在拥有了如何编写 Android SELinux 政策的知识，系统组件在哪里以及如何工作，以及如何在各种 Android 平台上移植和启用这些功能。由于这是一个相对较新的功能，它会影响许多系统交互，因此将出现需要代码更改以及政策更改的问题。理解这两者至关重要。

作为政策制定者和一般的安全人员，确保系统的安全责任落在了我们的肩上。在大多数组织中，你需要在黑暗中工作。然而，如果你能的话，尽可能多地在工作列表中工作和提问，永远不要接受现状。Android 和 AOSP 项目对所有人的贡献都持欢迎态度，通过贡献，你将帮助使项目变得更好，并增强所有平台的功能集。
