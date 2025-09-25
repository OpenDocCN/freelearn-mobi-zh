# 第八章：将上下文应用于文件

在上一章中，我们升级了系统，收集了审计日志，并开始分析审计记录。我们发现文件系统上的一些对象未标记。在本章中，我们将：

+   学习文件系统和文件系统对象如何获得标签

+   展示更改标签的技术

+   介绍用于标记的扩展属性

+   调查文件上下文和动态类型转换

# 标记文件系统

Linux 上的文件系统起源于挂载，除了 Android 上的 `ramdisk rootfs`。Linux 上的文件系统差异很大。一般来说，为了支持 SELinux 的所有功能，你需要一个支持 `xattr` 和 `security` 命名空间的文件系统。我们在设置内核配置时看到了这个需求。

文件系统对象在创建时，都从一个初始上下文开始，就像所有其他内核对象一样。文件上下文简单地从其父对象继承，所以如果父对象未标记，则子对象未标记，除非有类型转换规则。通常，如果上下文未标记，则推断数据是在启用 SELinux 支持之前在文件系统上创建的，或者 `xattr` 中的类型标签在当前加载的策略中不存在。

初始标签或初始**安全 ID**（**sid**），位于 `sepolicy` 文件中的 `initial_sid_contexts`。每个对象类都有一个关联的初始 `sid`。例如，让我们看一下以下代码片段：

```kt
...
sid fs u:object_r:labeledfs:s0
sid file u:object_r:unlabeled:s0
...
```

## fs_use

文件系统可以通过多种方式进行标记。最佳情况是当文件系统支持 `xattrs`。在这种情况下，策略中应该出现一个 `fs_use_xattr` 语句。这些语句出现在 `sepolicy` 目录下的 `fs_use` 文件中。`fs_use_xattr` 的语法如下：

```kt
fs_use_xattr <fstype> <context>

```

从 `sepolicy` 视角来看 `fs_use`，我们可以参考 `ext4` 文件系统的示例：

```kt
...
fs_use_xattr ext3 u:object_r:labeledfs:s0;
fs_use_xattr ext4 u:object_r:labeledfs:s0;
fs_use_xattr xfs u:object_r:labeledfs:s0;
...
```

这告诉 SELinux，当它遇到 `ext4` `fs` 对象时，在扩展属性中查找标签或文件上下文。

## fs_task_use

文件系统还可以通过在创建对象时使用进程的上下文来进行标记。这对于伪文件系统来说是有意义的，其中对象实际上是进程上下文，例如 `pipefs` 和 `sockfs`。这些伪文件系统管理管道和套接字系统调用，实际上并没有挂载到用户空间。它们存在于内核内部，为内核使用。然而，它们确实有对象，就像任何其他对象一样，它们需要被标记。这就是 `fs_task_use` 策略语句有意义的上下文。这些内部文件系统只能由直接访问它们的进程访问，并为这些进程提供服务。因此，用创建者来标记它们是有意义的。语法如下：

```kt
fs_task_use <fstype> <context>

```

`sepolicy` 文件中的 `fs_use` 示例如下：

```kt
...
# Label inodes from task label.
fs_use_task pipefs u:object_r:pipefs:s0;
fs_use_task sockfs u:object_r:sockfs:s0;
...
```

## fs_use_trans

您可能希望设置伪文件系统（实际上已挂载）的标签的另一种方式是使用`fs_use_trans`。这将在伪文件系统上设置整个文件系统的标签。此语法的格式如下：

```kt
fs_use_trans <fstype> <context>

```

`sepolicy`文件中的`fs_use`示例如下：

```kt
...
fs_use_trans devpts u:object_r:devpts:s0;
fs_use_trans tmpfs u:object_r:tmpfs:s0;
...
```

## genfscon

如果没有`fs_use_*`语句满足您的使用案例，例如对于`vfat`文件系统和`procfs`，那么您将使用`genfscon`语句。为`genfscon`指定的标签适用于该文件系统挂载的所有实例。例如，您可能希望使用`genfscon`与`vfat`文件系统一起。如果您有两个`vfat`挂载，它们将使用每个挂载相同的`genfscon`语句。然而，`genfscon`与`procfs`的行为不同，允许您对文件系统中的每个文件或目录进行标签化。

`genfscon`的语法如下：

```kt
genfscon <fstype> <path> <context>

```

`sepolicy genfs_contexts`中的示例如下：

```kt
...
# Label inodes with the fs label.
genfscon rootfs / u:object_r:rootfs:s0
# proc labeling can be further refined (longest matching prefix).
genfscon proc / u:object_r:proc:s0
genfscon proc /net/xt_qtaguid/ctrl u:object_r:qtaguid_proc:s0
...
```

注意，`rootfs`的部分路径是`/`。它不是`procfs`，因此不支持对其标签的任何细粒度支持；所以`/`是唯一可以使用的。然而，您可以对`procfs`进行广泛的设置，并设置为所需的任何粒度。

## 挂载选项

如果上述选项都不符合您的需求，您可以通过`mount`命令行传递`context`选项。这将设置整个文件系统的挂载上下文，例如`genfscon`，但在需要具有单独标签的多个文件系统中很有用。例如，如果您有两个挂载的`vfat`文件系统，您可能希望将它们的使用分开。使用`genfscon`语句，这两个文件系统将使用`genfscon`提供的相同标签。通过在挂载时指定标签，您可以将两个`vfat`文件系统挂载为具有不同标签。

以以下命令为例：

```kt
mount -ocontext=u:object_r:vfat1:s0 /dev/block1 /mnt/vfat1
mount -ocontext=u:object_r:vfat2:s0 /dev/block1 /mnt/vfat2

```

除了作为挂载选项的上下文之外，还有：`fscontext`和`defcontext`。这些选项与上下文互斥。`fscontext`选项设置用于某些操作（如挂载）的元文件系统类型，但不会更改每个文件的标签。`defcontext`设置未标记文件的默认上下文，覆盖`initial_sid`语句。最后，另一个选项`rootcontext`允许您在文件系统中设置根 inode 上下文，但仅针对该对象。根据`mount`手册页（`man 8 mount`），在无状态 Linux 中这被发现是有用的。

## 使用扩展属性进行标签化

最后，并且可能是最常用的标签化方式，是通过使用扩展属性支持，也称为`xattr`或 EA 支持。即使有`xattr`支持，新对象也会继承其父目录的上下文；然而，这些标签的粒度是基于文件系统对象或 inode 的。如果您还记得，我们不得不在 Android 文件系统中启用或验证`XATTR(CONFIG_EXT4_FS_XATTR)`支持，并配置 SELinux 通过配置选项`CONFIG_EXT4_FS_SECURITY`来使用它。

扩展属性是文件的关键字-值元数据存储。SELinux 安全上下文使用 `security.selinux` 键，其值是一个字符串，表示安全上下文或标签。

## `file_contexts` 文件

在 `sepolicy` 目录中，你可以找到 `file_contexts` 文件。此文件用于设置支持按文件安全标签的文件系统的属性。请注意，一些伪文件系统也支持此功能，例如 `tmpfs`、`sysfs` 和最近添加的 `rootfs`。`file_context` 文件具有基于正则表达式的语法，如下所示，其中 `regexp` 是路径的正则表达式：

```kt
regexp <type> ( <file label> | <<none>> )

```

如果为文件定义了多个正则表达式，则使用最后一个匹配项，因此顺序很重要。

下面的列表显示了文件系统对象类型的每个类型字段值、它们的含义和 syscall 接口：

+   `--`: 这表示一个普通文件。

+   `-d`: 这表示一个目录。

+   `-b`: 这表示一个块文件。

+   `-s`: 这表示一个套接字文件。

+   `-c`: 这表示一个字符文件。

+   `-l`: 这表示一个链接文件。

+   `-p`: 这表示一个命名管道文件。

如您所见，类型基本上是 `ls -la` 命令输出的模式。如果没有指定，则匹配所有内容。

下一个字段是文件标签或特殊标识符 `<<none>>`。两者都可以提供上下文或标识符 `<<none>>`。如果你指定了上下文，则咨询 `file_contexts` 的 SELinux 工具使用指定上下文的最后一个匹配项。如果指定的上下文是 `<<none>>`，则表示没有分配上下文。因此，保留我们找到的那个。关键字 `<<none>>` 在 AOSP 参考 `sepolicy` 中没有使用。

重要的是要注意，前面的段落明确指出 SELinux 工具使用 `file_contexts` 策略。内核不知道这个文件的存在。SELinux 通过从用户空间使用查找 `file_context` 的工具或通过 `fs_use_*` 和 `genfs` 策略语句显式设置来为所有对象设置标签。换句话说，`file_contexts` 不是内置于核心策略文件中，也不是由内核直接加载或使用的。在构建时，`file_contexts` 文件在 ramdisk rootfs 中构建，可以在 `/file_contexts` 中找到。此外，在构建时，系统镜像被标记，从而减轻了设备本身的负担。

在 Android 中，`init`、`ueventd` 和 `installd` 都已被修改以查找它们创建的对象的上下文；这样它们就可以正确地标记它们。因此，所有创建文件系统对象的 init 内置命令，如 `mkdir`，都已修改为使用 `file_contexts` 文件（如果存在），同样适用于 `installd` 和 `ueventd`。

让我们看看位于 `sepolicy` 中的 `file_context` 文件的一些片段：

```kt
...
/dev(/.*)? u:object_r:device:s0
/dev/accelerometer u:object_r:sensors_device:s0
/dev/alarm u:object_r:alarm_device:s0
...
```

在这里，我们正在为 `/dev` 目录下的文件设置上下文。注意条目是如何从最通用的 `dev` 文件开始，逐渐变得更为具体的。因此，任何未被更具体条目覆盖的文件将最终带有上下文 `u:object_r:device:s0`，而与更下面条目匹配的文件将带有更具体的标签。例如，位于 `/dev/accelerometer` 的加速度计将获得上下文 `u:object_r:sensors_device:s0`。请注意，省略了类型字段，这意味着它匹配所有文件系统对象，例如目录（`type -d`）。

你可能想知道 `/dev` 目录本身是如何获得文件上下文的。查看一些片段，我们说 `/` 或根是通过 `genfs_context` 文件中的 `genfscon rootfs / u:object_r:rootfs:s0` 语句进行标签化的。本章之前提到，“新对象继承其父目录的上下文。”因此，我们可以推断 `/dev` 的上下文是 `u:object_r:rootfs:s0`，因为这是 `/` 的标签。我们可以通过传递 `-Z` 标志给 `ls` 来测试这一点，以显示 `/dev` 的标签。在 UDOO 串行连接上，执行以下命令：

```kt
130|root@udoo:/ # ls -laZ / 
...
drwxr-xr-x root root u:object_r:device:s0 dev
...

```

看起来假设是不正确的，但请注意，确实一切都是有一个标签的，如果没有指定，则从父级继承。回顾 `sepolicy`，我们可以看到 `dev` 文件系统最初是通过 `fs_use_trans devtmpfs u:object_r:device:s0;` 策略语句设置的。因此，当文件系统挂载时，它被设置为整个文件系统。后来，当 `init` 或 `ueventd` 添加条目时，它们使用 `file_contexts` 条目来设置新创建的文件系统对象的上下文，使其与 `file_contexts` 文件中指定的上下文相匹配。位于 `/dev` 的文件系统，这是一个 `devtmps` 伪文件系统，是一个既有通过 `fs_use_trans` 语句设置的文件系统范围标签，也可以通过 `file_contexts;` 支持细粒度标签的文件系统的例子。在 Linux 上，文件系统在功能上并不非常一致。

## 动态类型转换

SELinux 策略声明`type_transition`指示的动态类型转换是一种允许文件动态确定其类型的方法。因为这些是编译到策略中的，所以它们与`file_contexts`文件没有任何关系。这些策略声明允许策略作者根据文件创建的上下文动态指定文件上下文。这在您不控制源代码或不想以任何方式耦合 SELinux 的情况下很有用。例如，`wpa`客户端，这是一个为 Wi-Fi 支持而运行的服务，并在其数据目录中创建一个套接字文件。其数据目录被标记为类型`wifi_data_file`，正如预期的那样，套接字最终也带有这个标签。然而，这个套接字是由系统服务器共享的。现在，我们可以只允许系统服务器访问类型和对象类，然而，`hostapd`和其他一些东西在这个目录中创建套接字和其他对象，因此这些对象也具有这种类型。我们真正想要确保的是，两个相关的套接字，即`hostapd`使用的套接字和系统服务器使用的套接字，彼此之间是独立的。为了做到这一点，我们需要能够以更细的粒度标记其中一个套接字，为此，我们可以修改代码或使用动态类型转换。与其修改代码，不如使用类型转换，如下所示：

```kt
type_transition wpa wifi_data_file:sock_file wpa_socket;

```

这是从`sepolicy`文件`wpa_supplicant.te`中的一个实际声明。它表示，当类型为`wpa`的进程创建类型为`wifi_data_file`的文件，并且对象类是`sock_file`时，在创建时将其标记为`wpa_socket`。该语句的语法如下：

```kt
type_transition <creating type> <created type>:<class> <new type>;
```

到 SELinux 策略版本 25 为止，`type_transition`语句可以支持命名类型转换，其中存在第四个参数，并且是该文件的名字：

```kt
type_transition <creating type> <created type>:<class> <new type> <file name>;
```

我们将在`sepolicy`文件中看到一个使用此文件名的示例，`system_server.te`：

```kt
type_transition system_server system_data_file:sock_file system_ndebug_socket "ndebugsocket";
```

注意文件名或基本名，而不是路径，并且它必须完全匹配。不支持正则表达式。还有一点值得注意的是，动态转换不仅限于文件对象，还包括任何对象类事件进程。我们将在第九章中看到如何使用动态进程转换，*向域添加服务*。

# 示例和工具

在掌握理论之后，让我们来看看在系统中标记文件的工具和技术。让我们先挂载一个`ramfs`文件系统。我们将从重新挂载`/`开始，因为它只读，并为文件系统创建一个挂载点。通过 UDOO 串行控制台执行以下命令：

```kt
root@udoo:/ # mount -oremount,rw /
root@udoo:/ # mkdir /ramdisk
root@udoo:/ # mount -t ramfs -o size=20m ramfs /ramdisk

```

现在，我们想看看文件系统具有哪个标签：

```kt
# ls -laZ / | grep ramdisk 
drwxr-xr-x root root u:object_r:unlabeled:s0 ramdisk

```

如您所回忆的，`initial_sid_context`文件为文件系统设置了以下初始`sid`：

```kt
sid file u:object_r:unlabeled:s0
```

如果我们想要将这个 ramdisk 转换成一个新的标签，我们需要在策略中创建类型，并设置一个新的`genfscon`语句来使用它。我们将在`file.te`策略文件中声明新的类型：

```kt
type ramdisk, file_type, fs_type;
```

类型策略语句的语法如下：

```kt
type <new type>, <attribute0,attribute1…attributeN>;
```

SELinux 中的属性是允许你定义常见组的语句。它们通过`attribute`语句定义。在 Android SELinux 策略中，我们已经为我们定义了`file_type`和`fs_type`。我们将在这里使用它们，因为我们要创建的新类型具有`file_type`和`fs_type`属性。`file_type`属性与文件的类型相关联，而`fs_type`属性表示此类型也与文件系统相关联。目前，属性并不十分重要；所以不要陷入细节。

下一步要修改的是`sepolicy`文件，`genfs_context`，通过添加以下内容：

```kt
genfscon ramfs / u:object_r:ramdisk:s0
```

现在，我们将编译引导映像并将其闪存到设备上，或者更好的是，让我们使用以下动态策略重新加载支持。

从 UDOO 项目树的根目录构建`sepolicy`项目：

```kt
$ mmm external/sepolicy/

```

按照以下方式通过`adb`推送新的策略：

```kt
$ adb push $OUT/root/sepolicy /data/security/current/sepolicy
544 KB/s (86409 bytes in 0.154s)

```

使用`setprop`命令触发重新加载：

```kt
$ adb shell setprop selinux.reload_policy 1

```

如果你连接了串行控制台，你应该会看到：

```kt
SELinux: Loaded policy from /data/security/current/sepolicy

```

如果你没有，只有`adb`，检查`dmesg`：

```kt
$ adb shell dmesg | grep "SELinux: Loaded"
<4>SELinux: Loaded policy from /sepolicy
<6>init: SELinux: Loaded property contexts from /property_contexts
<4>SELinux: Loaded policy from /data/security/current/sepolicy

```

一个成功的加载应该使用我们的策略在路径上，`/data/security/current/sepolicy`。让我们卸载 ramdisk 并重新挂载它以检查其类型：

```kt
root@udoo:/ # umount /ramdisk 
root@udoo:/ # mount -t ramfs -o size=20m ramfs /ramdisk
root@udoo:/ # ls -laZ / | grep ramdisk
drwxr-xr-x root root u:object_r:ramdisk:s0 ramdisk

```

我们能够修改策略并使用`genfscon`来更改文件系统类型，现在为了显示继承，让我们继续在文件系统上使用`touch`创建一个文件：

```kt
root@udoo:/ # cd /ramdisk
root@udoo:/ramdisk # touch hello
root@udoo:/ramdisk # ls -Z
-rw------- root root u:object_r:ramdisk:s0 hello

```

如我们所预期，新文件被标记为 ramdisk 类型。现在，假设我们从 shell 执行`touch`时，我们希望文件具有不同的类型，例如`ramdisk_newfile`；我们如何做到这一点？我们可以通过修改`touch`本身来咨询`file_contexts`，或者我们可以定义动态类型转换；让我们尝试动态类型转换方法。`type_transition`语句的第一个参数是创建类型；那么我们的 shell 在什么类型？你可以通过执行以下操作来获取：

```kt
root@udoo:/ramdisk # echo `cat /proc/self/attr/current`
u:r:init_shell:s0

```

一个更简单的方法是运行`id -Z`命令，它使用上述`proc`文件。对于串行控制台，执行：

```kt
root@udoo:/ramdisk # id -Z
uid=0(root) gid=0(root) context=u:r:init_shell:s0

```

并且运行相同的命令用于`adb` shell：

```kt
$ adb shell id -Z
uid=0(root) gid=0(root) context=u:r:shell:s0

```

注意我们的串行控制台 shell 和`adb` shell 之间的差异，在第九章，*向域添加服务*；我们将修复这个问题。因此，我们现在编写的策略将解决这两种情况。

首先打开`sepolicy`文件，`init_shell.te`，并将以下内容附加到文件末尾：

```kt
type_transition init_shell ramdisk:file ramdisk_newfile;
```

对`sepolicy`文件，`shell.te`也这样做：

```kt
type_transition shell ramdisk:file ramdisk_newfile;
```

现在，我们需要声明新的类型；因此打开`sepolicy`文件，`file.te`，并附加以下内容：

```kt
type ramdisk_newfile, file_type;
```

注意，我们只使用了`file_type`属性。这是因为文件系统永远不应该有`ramdisk_newfile`类型，只有位于该文件系统内的文件应该有。

现在，构建`adb`策略，将其推送到设备，并触发重新加载。完成此操作后，创建文件并检查结果：

```kt
$ adb shell 'touch /ramdisk/shell_newfile'
$ adb shell 'ls -laZ /ramdisk'
-rw-rw-rw- root root u:object_r:ramdisk:s0 shell_newfile

```

所以它没有成功。让我们通过一个 `ext4` 文件系统的例子来调查原因。让我们使用以下命令：

```kt
root@udoo:/ # cd /data/
root@udoo:/data # mkdir ramdisk

```

现在，检查它的上下文：

```kt
root@udoo:/data # ls -laZ | grep ramdisk
drwx------ root rootu:object_r:system_data_file:s0 ramdisk

```

标签是 `system_data_file`。这没有帮助，因为它不适用于我们的类型转换规则；为了修复这个问题，我们可以使用 `chcon` 命令显式地更改文件上下文：

```kt
root@udoo:/data # chcon u:object_r:ramdisk:s0 ramdisk
root@udoo:/data # ls -laZ | grep ramdisk
drwx------ root root u:object_r:ramdisk:s0 ramdisk

```

现在将上下文更改为我们之前尝试的 ramdisk 的上下文，让我们尝试在这个目录中创建一个文件：

```kt
root@udoo:/data/ramdisk # touch newfile
root@udoo:/data/ramdisk # ls -laZ
-rw------- root root u:object_r:ramdisk_newfile:s0 newfile

```

正如你所见，类型转换已经发生。这旨在说明你在使用 SELinux 和 Android 时可能遇到的问题。既然我们已经证明我们的 `type_transition` 语句是有效的，那么这个失败只有两种可能性：文件系统不支持它，或者我们在某处遗漏了“开启”它的东西。结果证明，后者是正确的；我们遗漏了我们的 `fs_use_trans` 语句。所以，请打开 `sepolicy` 文件，在 `fs_use` 中添加以下行：

```kt
fs_use_trans ramfs u:object_r:ramdisk:s0;
```

这条语句启用了此文件系统上的 SELinux 动态转换。现在，重新构建 `sepolicy` 项目，使用 `adb push` 推送策略文件，并通过 `setprop` 命令启用动态重新加载：

```kt
$ mmm external/sepolicy
$ adb push $OUT/root/sepolicy /data/security/current/sepolicy546 KB/s (86748 bytes in 0.154s)
$ adb shell setprop selinux.reload_policy 1
root@udoo:/ # cd ramdisk
root@udoo:/ramdisk # touch foo
root@udoo:/ramdisk # ls -Z
-rw------- root root u:object_r:ramdisk_newfile:s0 foo

```

就这样，对象有了由动态类型转换确定的正确值。我们遗漏了 `fs_use_trans`，它启用了不支持 `xattrs` 的文件系统上的类型转换。

现在，假设我们想要挂载另一个 ramdisk，会发生什么？嗯，因为它被标记为 `genfscon` 语句，所以所有使用该类型的挂载文件系统都应该获得上下文，`u:object_r:ramdisk:s0`。我们将在这个文件系统上挂载 `/ramdisk2`，并验证这种行为：

```kt
root@udoo:/ # mkdir ramdisk2
root@udoo:/ # mount -t ramfs -o size=20m ramfs /ramdisk2

```

此外，也要检查上下文：

```kt
root@udoo:/ # ls -laZ | grep ramdisk
drwxr-xr-x root root u:object_r:ramdisk:s0 ramdisk
drwxr-xr-x root root u:object_r:ramdisk:s0 ramdisk2

```

如果我们想要为这些文件系统的单独访问编写允许规则，我们需要将它们的目标文件放在不同的类型中。为此，我们可以使用上下文选项挂载新的 ramdisk。但首先，我们需要创建新的类型；让我们转到 `sepolicy` 文件，`file.te` 中并添加一个名为 `ramdisk2` 的新类型：

```kt
type ramdisk2, file_type, fs_type;
```

现在，使用 `mmm` 命令构建 `sepolicy`，然后使用 `adb push` 命令推送策略，并通过 `setprop` 命令触发重新加载：

```kt
$ mmm external/sepolicy/
$ adb push out/target/product/udoo/root/sepolicy /data/security/current/sepolicy542 KB/s (86703 bytes in 0.155s)
$ adb shell setprop selinux.reload_policy 1

```

在这个阶段，让我们卸载 `/ramdisk2` 并使用 `context=` 选项重新挂载它：

```kt
root@udoo:/ # umount /ramdisk2/ 
root@udoo:/ # mount -t ramfs -osize=20m,context=u:object_r:ramdisk2:s0 ramfs /ramdisk2

```

现在，验证上下文：

```kt
root@udoo:/ # ls -laZ | grep ramdisk 
drwxr-xr-x root root u:object_r:ramdisk:s0 ramdisk
drwxr-xr-x root root u:object_r:ramdisk2:s0 ramdisk2

```

我们可以使用 `mount` 选项 `context=<context>` 覆盖 `genfscon` 上下文。实际上，如果我们查看 `dmesg`，我们可以看到一些很好的信息。当我们没有上下文选项挂载 `ramfs` 时，我们得到了：

```kt
<7>SELinux: initialized (dev ramfs, type ramfs), uses genfs_contexts

```

当我们使用 `context=<context>` 选项挂载时，我们得到了：

```kt
<7>SELinux: initialized (dev ramfs, type ramfs), uses mountpoint labeling

```

我们可以看到，在尝试确定其标签来源时，SELinux 给出了一些有用的信息。

现在，让我们使用具有 `xattr` 支持的文件系统进行标签化，例如 `ext4`。我们将从工具箱命令 `chcon` 开始。`chcon` 命令允许你显式地设置文件系统对象的上下文，它不会咨询 `file_contexts`。

让我们看看 `/system/bin` 目录，以及其中的前 10 个文件：

```kt
$ adb shell ls -laZ /system/bin | head -n10
-rwxr-xr-x root shell u:object_r:system_file:s0 InputDispatcher_test
-rwxr-xr-x root shell u:object_r:system_file:s0 InputReader_test
-rwxr-xr-x root shell u:object_r:system_file:s0 abcc
-rwxr-xr-x root shell u:object_r:system_file:s0 adb
-rwxr-xr-x root shell u:object_r:system_file:s0 am
-rwxr-xr-x root shell u:object_r:zygote_exec:s0 app_process
-rwxr-xr-x root shell u:object_r:system_file:s0 applypatch
-rwxr-xr-x root shell u:object_r:system_file:s0 applypatch_static
drwxr-xr-x root shell u:object_r:system_file:s0 asan
-rwxr-xr-x root shell u:object_r:system_file:s0 asanwrappe

```

我们可以看到，其中许多都带有`system_file`标签，这是该文件系统的默认标签；让我们将`am`类型更改为`am_exec`。同样，我们需要通过向`sepolicy`文件中的`file.te`添加以下内容来创建一个新的类型：

```kt
type am_exec, file_type;
```

现在，重建策略文件，将其推送到 UDOO，并触发重新加载。之后，让我们开始重新挂载系统，因为它现在是只读的：

```kt
root@udoo:/ # mount -orw,remount /system

```

现在执行`chcon`：

```kt
root@udoo:/ # chcon u:object_r:am_exec:s0 /system/bin/am 

```

验证结果：

```kt
root@udoo:/ # la -laZ /system/bin/am 
-rwxr-xr-x root shell u:object_r:am_exec:s0 am

```

此外，`restorecon`命令将使用`file_contexts`，并将该文件恢复到`file_contexts`文件中设置的状态，应该是`system_file`：

```kt
root@udoo:/ # restorecon /system/bin/am 
root@udoo:/ # la -laZ /system/bin/am 
-rwxr-xr-x root shell u:object_r:system_file:s0 am

```

正如您所看到的，`restorecon`能够咨询`file_contexts`并在该对象上恢复指定的上下文。

Android 系统的文件系统在构建时构建，因此，所有文件对象在构建过程中都会被标记。我们也可以通过更改`file_contexts`在构建时更改这一点。更改后，系统分区将被重建，系统刷新后，我们应该看到带有`am_exec`类型的`am`文件。我们可以通过在`system/bin`部分的末尾添加此行来测试`sepolicy`文件，`file_contexts`：

```kt
/system/bin/am u:object_r:am_exec:s0
```

使用以下命令重建整个系统：

```kt
$ make -j8 2>&1 | tee logz

```

现在刷新并重启，让我们看一下以下`/system/bin/am`上下文：

```kt
root@udoo:/ # ls -laZ /system/bin/am 
-rwxr-xr-x root shell u:object_r:am_exec:s0 am

```

这表明系统分区尊重构建时标签的文件上下文，以及我们如何控制这些标签。

## 修复/data

此外，在审计日志中，我们看到了许多未标记的文件，例如以下拒绝：

```kt
type=1400 msg=audit(86559.780:344): avc: denied { append } for pid=2668 comm="UsbDebuggingHan" name="adb_keys" dev=mmcblk0p4 ino=42 scontext=u:r:system_server:s0 tcontext=u:object_r:unlabeled:s0 tclass=file
```

我们可以看到设备是`mmcblk0p4`，挂载命令将在其输出中告诉我们它挂载到了哪个文件系统：

```kt
root@udoo:/ # mount | grep mmcblk0p4
/dev/block/mmcblk0p4 /data ext4 rw,seclabel,nosuid,nodev,noatime,nodiratime,errors=panic,user_x0

```

那么，为什么`/data`文件系统有这么多未标记的文件呢？原因是 SELinux 旨在从空设备开启，也就是说，从第一次启动开始。Android 按需构建数据目录结构。因此，`/data`的所有标签都由`file_contexts`文件处理，因为它使用的是`ext4`。此外，它由创建`/data`文件和目录的系统处理。这些系统已被修改，根据`file_contexts`规范标记数据分区。因此，这提供了两种选择：擦除`/data`并重启，或`restorecon -R /data`。

第一个选项有点严厉，但如果您拔出 SD 卡并删除数据分区（分区 4）上的所有文件，Android 将重新构建，您将不再看到任何未标记的问题。然而，当您升级部署设备时，这并不推荐；您将破坏所有用户的数据。

第二种选择在部署场景中更为可接受，但有其局限性。值得注意的是，执行`restorecon -R /data`将花费很长时间，并且必须在引导早期、挂载之后立即完成。然而，这真的是目前唯一的选项。然而，谷歌在这一领域做了很多工作，并创建了一个系统，该系统能够在策略更新时智能地重新标记`/data`。对于我们的使用，我们将选择第二种选择的变体，特别是在考虑到`/data`文件系统稀疏度很高的情况下；我们实际上还没有安装或生成很多用户数据。据此，执行：

```kt
root@udoo:/ # restorecon -R /data
root@udoo:/ # reboot

```

由于我们的系统处于宽容模式，并且我们不在部署场景中，所以我们不需要在引导早期执行`restorecon`。现在，让我们拉取`audit.log`文件，并将其与已经拉取的`audit.log`进行比较：

```kt
$ adb pull /data/misc/audit/audit.log audit_data_relabel.log
170 KB/s (14645 bytes in 0.084s)

```

让我们使用`grep`来计算每个文件中出现的次数：

```kt
$ grep -c unlabeled audit.log 
185
$ grep -c unlabeled audit_data_relabel.log 
0

```

太好了，我们解决了`/data`上的所有未标记问题！

# 关于安全性的一个侧记

注意，尽管我们正在运行所有这些命令并更改所有这些设置，但这并不是 SELinux 中的安全漏洞。能够更改类型标签、挂载文件系统以及将文件系统与类型关联，所有这些都要求允许规则。如果你查看审计日志，你会看到一系列拒绝；以下是一个示例： 

```kt
type=1400 msg=audit(90074.080:192): avc: denied { associate } for pid=3211 comm="touch" name="foo" scontext=u:object_r:ramdisk_newfile:s0 tcontext=u:object_r:ramdisk:s0 tclass=filesystem
type=1400 msg=audit(90069.120:187): avc: denied { mount } for pid=3205 comm="mount" name="/" dev=ramfs ino=1992 scontext=u:r:init_shell:s0 tcontext=u:object_r:ramdisk:s0 tclass=filesystem
```

如果我们处于强制模式，我们就无法执行这里展示的任何实验。

# 摘要

在本章中，我们学习了如何通过重新标记文件将文件放入上下文中。我们使用了各种技术来完成这项任务，从工具箱命令如`chcon`和`restorecon`，到挂载选项和动态转换。有了这些工具，我们可以确保所有文件系统对象都被正确标记。这样，我们最终得到正确的目标上下文，从而确保我们编写的策略是有效的。在下一章中，我们将关注进程，确保它们位于正确的域或上下文中。
