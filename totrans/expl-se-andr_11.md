# 第十一章 标签属性

在本章中，我们将介绍如何通过 `property_contexts` 文件来标签化属性。

属性是我们在第三章 Android Is Weird 中了解到的一个独特的 Android 功能。我们希望对这些属性进行标签化，以限制我们的属性只由应该设置它们的域设置，防止意外更改值的经典 DAC 根攻击。在本章中，我们将学习如何：

+   创建新属性

+   标签新属性和现有属性

+   解释和处理属性拒绝

+   列举特殊的 Android 属性及其行为

# 通过 property_contexts 标签化

所有属性都使用 `property_contexts` 文件进行标签化，其语法类似于 `file_contexts`。然而，它不是在文件路径上工作，而是在属性名称或属性键（Android 中的属性是键值存储）上工作。属性键通常用点（`.`）分隔。这与 `file_contexts` 类似，但斜杠（`/`）变成了点。一些示例属性及其在 `property_contexts` 中的条目如下所示：

```kt
ctl.ril-daemon  u:object_r:ctl_rildaemon_prop:s0
ctl.   u:object_r:ctl_default_prop:s0
```

注意所有 `ctl.` 属性都使用 `ctl_default_prop` 类型进行标签化，但 `ctl.ril-daemon` 使用不同的类型标签 `ctl_rildaemon_prop`。这代表了你可以从通用开始，根据需要逐步过渡到更具体的值/类型。

此外，任何未明确标签化的内容默认通过 `property_contexts` 中的“匹配所有”表达式使用 `default_prop`：

```kt
# default property context
* u:object_r:default_prop:s0
```

# 属性的权限

可以使用命令行工具 `getprop` 和 `setprop` 查看系统上的当前属性，并创建新的属性，如下面的代码片段所示：

```kt
root@udoo:/ # getprop
...
[sys.usb.state]: [mtp,adb]
[wifi.interface]: [wlan0]
[wlan.driver.status]: [unloaded]

```

从第三章 Android Is Weird 回忆起，属性被映射到每个人的地址空间中，因此任何人都可以读取它们。然而，并非每个人都可以设置（写入）它们。属性的 DAC 权限模型是硬编码在 `system/core/init/property_service.c` 中的：

```kt
/* White list of permissions for setting property services. */
struct {
  const char *prefix;
  unsigned int uid;
  unsigned int gid;
} property_perms[] = {
  { "net.rmnet0.", AID_RADIO, 0 },
  { "net.gprs.", AID_RADIO, 0 },
  { "net.ppp", AID_RADIO, 0 },
...
  { "persist.service.bdroid.", AID_BLUETOOTH, 0 },
  { "selinux." , AID_SYSTEM, 0 },
  { "persist.audio.device", AID_SYSTEM, 0 },
  { NULL, 0, 0 }
```

必须在 `property_perms` 数组中具有 UID 或 GID，才能设置与前缀匹配的任何属性。例如，为了设置 `selinux.` 属性，你必须具有 UID `AID_SYSTEM`（uid 1000）或 root 权限。是的，root 总是能够设置属性，这是将 SELinux 应用于 Android 属性的关键好处。不幸的是，没有方法使用 `getprop -Z` 列出属性及其标签，就像使用 `ls -Z` 和文件一样。

# 重新标签现有属性

为了更熟悉标签属性，让我们重新标签 `wifi.interface` 属性。首先，让我们通过引发拒绝并查看拒绝日志来验证其上下文，如下面的代码所示：

```kt
root@udoo:/ # setprop wifi.interface wlan0
avc: denied { set } for property=wifi.interface scontext=u:r:shell:s0 tcontext=u:object_r:default_prop:s0 tclass=property_service

```

当我们在 UDOO 串行控制台执行`setprop`命令时，发生了一个有趣的行为。AVC 拒绝记录被打印出来。这是因为串行控制台包括了使用`printk()`从内核打印出的任何内容。这里发生的事情是`init`进程，它控制`setprops`，如第三章“Android 很奇怪”中详细说明的，向内核日志写入一条消息。当我们执行`setprop`命令时，这条日志消息就会出现。如果你通过`adb shell`运行它，你会在串行控制台上看到这条消息，但不会在`adb`控制台上看到。但是，要这样做，你必须重新启动你的系统，因为 SELinux 在宽容模式下只会打印一次拒绝记录。

使用`adb shell`的命令如下：

```kt
$ adb shell setprop wifi.interface wlan0

```

使用串行控制台的命令如下：

```kt
root@udoo:/ # avc: denied {set} for property=wifi.interface scontext=u:r:shell:s0 tcontext=u:object_r:default_prop
usb 2-1.3: device descriptor read/64, error -110

```

从拒绝输出中，我们可以看到属性类型标签是`default_prop`。让我们将其更改为`wifi_prop`。

我们首先编辑`sepolicy`目录中的`property.te`，以声明新类型并按以下方式将这些属性标签化，添加以下行：

```kt
type wifi_prop, property_type;
```

类型声明后，下一步是通过修改`property_contexts`来应用标签，添加以下内容：

```kt
# wifi properties
wifi. u:object_r:wifi_prop:s0
```

按照以下步骤构建策略：

```kt
$ mmm external/sepolicy

```

推送新的`property_contexts`文件：

```kt
$ adb push out/target/product/udoo/root/property_contexts /data/security/current
51 KB/s (2261 bytes in 0.042s)

```

触发动态重新加载：

```kt
$ adb shell setprop selinux.reload_policy 1
# setprop wifi.interface wlan0
avc: denied { set } for property=wifi.interface scontext=u:r:shell:s0 tcontext=u:object_r:default_prop:s0 tclass=property_service

```

好吧，这没有起作用！`property_contexts`文件必须在`/data/security`，而不是`/data/security/current`。

为了发现这个问题，搜索`libselinux/src/android.c`文件。这个文件中没有提到`property_contexts`；因此，它必须在其他地方提到。这引导我们去搜索`system/core`，它包含了该文件的使用属性服务。匹配项在`init.c`中的代码，用于从优先位置加载文件。

```kt
$ grep -rn property_contexts *
init/init.c:745: { SELABEL_OPT_PATH, "/data/security/property_contexts" },
init/init.c:746: { SELABEL_OPT_PATH, "/property_contexts" },
init/init.c:760: ERROR("SELinux: Could not load property_contexts: %s\n",

```

让我们将`property_contexts`文件推送到正确的位置并再次尝试：

```kt
$ adb push out/target/product/udoo/root/property_contexts /data/security
51 KB/s (2261 bytes in 0.042s)
$ adb shell setprop selinux.reload_policy 1
root@udoo:/ # setprop wifi.interface wlan0
avc: received policyload notice (seqno=3)
init: sys_prop: permission denied uid:0 name:wifi.interface

```

哇！它再次失败了。这个练习本意是想指出，如果你忘记做某事，这可能会多么棘手。没有显示任何有信息量的拒绝消息，只有一个指示表明*它*被拒绝了。这是因为包含`wifi_prop`类型声明的`sepolicy`文件从未被推送。这导致`system/core/init/property_service.c`中的`check_mac_perms()`在`selinux_check_access()`函数中失败，因为它找不到用于计算访问检查的类型，尽管在`property_contexts`中的查找是成功的。没有详细的错误日志。

我们可以通过确保`sepolicy`也被推送来纠正这个问题：

```kt
$ adb push out/target/product/udoo/root/sepolicy /data/security/current/
550 KB/s (87385 bytes in 0.154s)
$ adb shell setprop selinux.reload_policy 1
root@udoo:/ # setprop wifi.interface wlan0
avc: received policyload notice (seqno=4)
avc: denied { set } for property=wifi.interface scontext=u:r:shell:s0 tcontext=u:object_r:wifi_prop:s0 tclass=property_service

```

现在我们看到了预期的拒绝消息，但目标（或属性）的标签是`u:object_r:wifi_prop:s0`。

现在目标属性已经标记，你可以允许对其的访问。请注意，这是一个虚构的例子，在现实世界中，你可能*不*希望允许从 shell 访问大多数属性。策略应与你的安全目标和对最小权限的要求相一致。

我们可以在`shell.te`中添加一个`allow`规则，如下所示：

```kt
# wifi prop
allow shelldomain wifi_prop:property_service set;
```

编译策略，将其推送到手机，并触发动态重新加载：

```kt
$ mmm external/sepolicy/
$ adb push out/target/product/udoo/root/sepolicy /data/security/current/
547 KB/s (87397 bytes in 0.155s)
$ adb shell setprop selinux.reload_policy 1

```

现在尝试设置`wifi.interface`属性，并注意拒绝的缺失。

```kt
root@udoo:/ # setprop wifi.interface wlan0
avc: received policyload notice (seqno=5)

```

# 创建和标记新属性

所有属性都是通过`setprop`调用或执行等效操作的函数调用在系统中动态创建的（从 C 的`bionic/libc/include/sys/system_properties.h`和 Java 的`android.os.SystemProperties`）。请注意，`System.getProperty()`和`System.setProperty()` Java 调用在应用程序私有属性存储上工作，并且与全局存储无关。

对于 DAC 控制，你需要修改`property_perms[]`，如之前所述，以便非 root 用户有创建或设置属性的权限。请注意，root 用户始终可以`set`和`create`，除非受到 SELinux 策略的限制。

假设我们想要创建`udoo.name`和`udoo.owner`属性；我们只想让 root 用户和 shell 域访问它们。我们可以这样创建它们：

```kt
root@udoo:/ # setprop udoo.name udoo
avc: denied { set } for property=udoo.name scontext=u:r:shell:s0 tcontext=u:object_r:default_prop:s0 tclass=property_service
root@udoo:/ # setprop udoo.owner William

```

注意拒绝显示这些为`default_prop`类型。为了纠正这一点，我们需要重新标记这些属性，就像我们在上一节中做的那样，*重新标记现有属性*。

# 特殊属性

在 Android 中，有一些特殊属性具有不同的行为。我们在后续章节中列举了属性名称和含义。

## 控制属性

以`ctl`开头的属性被保留为控制属性，用于通过`init`控制服务：

+   `start`：启动一个服务（`setprop ctl.start <servicename>`）

+   `stop`：停止一个服务（`setprop ctl.stop <servicename>`）

+   `restart`：重新启动一个服务（`setprop ctl.restart <servicename>`）

## 持久属性

以`persist`为前缀的任何属性在重启后都会持续存在，并会被恢复。数据被保存到`/data/property`目录中，文件名与属性同名。

```kt
root@udoo:/ # ls /data/property/
persist.gps.oacmode
persist.service.bdroid.bdaddr
persist.sys.profiler_ms
persist.sys.usb.config

```

## SELinux 属性

`selinux.reload_policy`属性是特殊的。正如我们所见，它的用途是触发动态重新加载事件。

# 概述

在本章中，我们探讨了如何创建和标记新的和现有的属性，以及在进行此操作时出现的某些奇怪现象。我们还检查了`property_service.c`中属性的硬编码 DAC 权限表，以及硬编码的特殊属性，如`ctl.`系列。在下一章中，我们将探讨工具链如何构建和创建我们一直在使用的所有策略文件。
