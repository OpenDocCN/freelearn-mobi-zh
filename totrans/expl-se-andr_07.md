# 第七章。利用审计日志

到目前为止，我们已经看到 AVC 记录或 SELinux 拒绝消息出现在`dmesg`中，但`dmesg`是一个环形内存缓冲区，其频繁滚动取决于内核的详细程度。通过使用审计内核子系统，我们可以将这些消息路由到用户空间并将它们记录到磁盘上。在桌面环境中，执行此操作的守护进程称为`auditd`。然而，`auditd`的最小端口在 NSA 分支中得到了维护，但它尚未正式合并到 AOSP 中。由于我们正在开发 Android 4.3，我们将使用 NSA 分支中的`auditd`版本。截至 2014 年 4 月 7 日官方合并的版本可以在[`android-review.googlesource.com/#/c/89645/`](https://android-review.googlesource.com/#/c/89645/)找到。它是通过`logd`实现的，并在[`android-review.googlesource.com/#/c/83526/`](https://android-review.googlesource.com/#/c/83526/)合并。

在本章中，我们将：

+   使用针对**Android 开源社区**（**AOSP**）的快速开发版更新我们的系统

+   调查审计子系统的工作原理

+   学习阅读 SELinux 审计日志并开始编写策略

+   查看与日志相关的上下文

所有 LSM 都应该将它们的消息记录到审计子系统中。审计子系统可以使用`printk`将消息路由到内核环形缓冲区，或者如果存在，路由到用户空间的审计守护进程。内核和用户空间日志守护进程使用`AUDIT_NETLINK`套接字进行通信。我们将在本章中进一步分析此接口。

最后，当策略违规发生时，审计子系统具有打印详细记录的能力。尽管您不需要此功能来启用和使用 SELinux，但它可以使您的生活更加轻松。要启用此系统，您必须使用`auditd`，因为`logd`目前还没有这项支持。您需要使用`CONFIG_AUDITSYSCALL=y`构建内核，并在`/data/misc/audit/`中放置一个`audit.rules`文件。在按照以下说明修补您的树之后，请阅读`system/core/auditd/README`。

不幸的是，UDOO 内核版本 3.0.35 不支持`CONFIG_AUDITSYSCALL`。位于[`git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/commit/?id=29ef73b7a823b77a7cd0bdd7d7cded3fb6c2587b`](https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/commit/?id=29ef73b7a823b77a7cd0bdd7d7cded3fb6c2587b)的补丁应该可以启用支持。然而，在 UDOO 上，它导致了一个我们无法追踪的死锁。

# 升级——大量补丁

虽然谷歌发布的 Android 4.3 版本支持 Android 安全增强功能，但它仍然有限，尤其是在审计领域。将此功能带到更实用的状态的最简单方法之一是从 NSA 的 Android 4.3 安全增强功能分支获取一些项目的补丁。在这里，社区已经部署了许多在 4.3 时间框架内未合并的更高级功能。

NSA 在 [`bitbucket.org/seandroid/`](https://bitbucket.org/seandroid/) 维护仓库。由于项目众多，确定使用哪个项目和哪个分支可能会令人感到困难。找到它们的一种方法是遍历每个项目，找到具有 `SEAndroid-4.3` 分支的项目。我们不需要深入到设备树中，因为我们不是在构建 AOSP 设备。此类项目的列表如下：

+   [`bitbucket.org/seandroid/system-core`](https://bitbucket.org/seandroid/system-core)

+   [`bitbucket.org/seandroid/frameworks-base`](https://bitbucket.org/seandroid/frameworks-base)

+   [`bitbucket.org/seandroid/external-libselinux`](https://bitbucket.org/seandroid/external-libselinux)

+   [`bitbucket.org/seandroid/build`](https://bitbucket.org/seandroid/build)

+   [`bitbucket.org/seandroid/frameworks-native`](https://bitbucket.org/seandroid/frameworks-native)

我们也可以安全地跳过 `sepolicy`，因为我们已经将其更新到最新版本，但内核稍微复杂一些。我们需要从 `kernel-common ([`bitbucket.org/seandroid/kernel-common`](https://bitbucket.org/seandroid/kernel-common))` 和 `binder` 补丁 `[`android-review.googlesource.com/#/c/45984/`](https://android-review.googlesource.com/#/c/45984/)` 中获取更改，如下所示：

```kt
$ mkdir ~/sepatches
$ cd ~/sepatches
$ git clone https://bitbucket.org/seandroid/system-core.git
$ git clone https://bitbucket.org/seandroid/frameworks-base.git
$ git clone https://bitbucket.org/seandroid/external-libselinux.git
$ git clone https://bitbucket.org/seandroid/build.git
$ git clone https://bitbucket.org/seandroid/frameworks-native.git

```

我们可以通过查看 `build/core/build_id.mk` 文件来确定需要修补的确切版本，并使用网页 [`source.android.com/source/build-numbers.html`](https://source.android.com/source/build-numbers.html) 进行查找。

文件显示 `BUILD_ID` 是 `JSS15J`，查找结果显示我们正在使用 UDOO 的 `android-4.3_r2.1` 版本。

对于迄今为止下载的每个项目，通过运行命令 `git checkout origin/seandroid-4.3_r2` 生成补丁。最后，执行 `git format-patch origin/jb-mr2.0-release`。由于没有 `4.3._r2.1` 分支，我们使用 `r2`。

对于这些补丁中的每一个，你都需要将它们应用到从相应的 `udoo/<project>` 文件夹中的树中。对于每个项目，按照从 `0001*` 补丁开始，接着是 `0002*`，依此类推的数字顺序应用补丁非常重要。以下是如何为一个项目应用特定补丁的示例，让我们看看为 `system-core` 需要的第一个补丁。请注意，这些 Git 仓库在源树中使用连字符代替斜杠；因此 `frameworks-base` 与 `frameworks/base` 相关联。

首先，生成补丁：

```kt
$ cd sepatches/system-core
$ git checkout origin/seandroid-4.3_r2
$ git format-patch origin/jb-mr2.0-release

```

按照以下方式应用第一个补丁：

```kt
$ cd <udoo_root>/system/core
$ patch -p1 < ~/sepatches/system-core/0001-Add-writable-data-space-for-radio.patch 
patching file rootdir/init.rc
Reversed (or previously applied) patch detected! Assume -R? [n] 

```

### 注意

注意，对于 UDOO 来说，在 `frameworks/base` 中应用高于 `0005` 的补丁编号非常重要。对于其他项目，你应该应用所有补丁。

注意错误。只要看到这个错误，就按*Ctrl* + *C*来退出补丁过程。Git 树并不完美，因此，一些补丁已经包含在 UDOO 源代码中。patch 命令会通知我们，当警告时，我们可以通过按*Ctrl* + *C*取消它们，继续通过补丁，取消已应用的补丁，并修复任何失败。在修补用户空间后，强烈建议你构建以确保没有出错。

一旦用户空间完全修补，我们需要修补内核。首先，使用`git clone https://bitbucket.org/seandroid/kernel-common.git`命令从 Bitbucket 克隆 kernel-common 项目。我们将使用与其他项目相同的方法来修补内核，除了 binder 补丁。通过查看提到的 binder 补丁链接[`android-review.googlesource.com/#/c/45984/`](https://android-review.googlesource.com/#/c/45984/)，我们发现 Git SHA 散列是`a3c9991b560cf0a8dec1622fcc0edca5d0ced936`，正如以下截图中的**Patch set 4**参考字段所给出的：

![升级 - 补丁众多](img/0594OS_7_1.jpg)

我们可以生成这个 SHA 散列的补丁：

```kt
$ git format-patch -1 a3c9991b560cf0a8dec1622fcc0edca5d0ced936
0001-Add-security-hooks-to-binder-and-implement-the-hooks.patch

```

然后，使用与之前相同的方法使用 patch 命令应用该补丁。该补丁有一个失败的 hunk 用于头文件包含；就像其他补丁一样，通过使用 reject 文件来修复它。当你构建时，你将在内核中得到这个错误：

```kt
security/selinux/hooks.c:1846:9: error: variable 'sad' has initializer but incomplete type
security/selinux/hooks.c:1846:28: error: storage size of 'sad' isn't known

```

继续删除这一行及其所有引用。这是在 3.0 内核中做出的更改：

```kt
struct selinux_audit_data sad = {0,};
ad.selinux_audit_data = &sad;
```

### 注意

我们通过查看原始的 3.0 补丁来解决这个问题，这些补丁可以在以下链接找到：

[`bitbucket.org/seandroid/kernel-omap/commits/59bc19226c746f479edc2acca9a41f60669cbe82?at=seandroid-omap-tuna-3.0`](https://bitbucket.org/seandroid/kernel-omap/commits/59bc19226c746f479edc2acca9a41f60669cbe82?at=seandroid-omap-tuna-3.0)

如你所记，UDOO 使用自定义的`init.rc`。我们需要将任何对`init.rc`的更改添加到 UDOO 实际使用的那个`init.rc`中。所有可以修改`init.rc`的补丁都在 system-core 项目中，具体如下：

+   `0003-Auditd-initial-commit.patch`

+   `0007-Handle-policy-reloads-within-ueventd-rather-than-res.patch`

+   `0009-Allow-system-UID-to-set-enforcing-and-booleans.patch`

继续查找这些补丁中`init.rc`的变化，并使用相同的补丁技术将其应用到`device/fsl/imx6/etc/init.rc`。

# 审计系统

在上一节中，我们做了很多补丁工作；其目的是为了启用在 Android 及其依赖项上完成的审计集成工作。这些补丁还修复了代码中的某些错误，并且非常重要，它们启用了 SELinux/LSM binder 钩子和策略控制。

Linux 中的审计系统被 LSM 用于打印拒绝记录，以及收集非常详尽和完整的事件记录。无论如何，当 LSM 打印消息时，它都会传播到审计子系统并打印出来。然而，如果审计子系统已被启用，那么你将获得更多与拒绝相关的上下文。审计子系统甚至支持加载规则以监视这一点。例如，你可以监视所有由系统 UID 未执行的对 `/system` 的写入操作。

## auditd 守护进程

`auditd` 守护进程或服务在用户空间运行，并通过 NETLINK 套接字监听审计子系统。守护进程将自己注册以接收内核消息，并且也可以通过这个套接字加载审计规则。一旦注册，`auditd` 守护进程就会接收所有的审计事件。`auditd` 守护进程只是最小化移植的，并且曾尝试将其主线集成到 Android 中，但后来被拒绝了。然而，`auditd` 已被各种 OEM（如三星）和 NSA 的 4.3 分支所使用。后来，将记录放入 logcat 的另一种方法被合并到 Android 中（更多信息，请参阅 [`android-review.googlesource.com/89645`](https://android-review.googlesource.com/89645))。

之前，我们在 `dmesg` 中看到了 SELinux 的 AVC 拒绝消息。这个问题在于，当有大量的拒绝或一个健谈的内核时，循环内存日志容易溢出。使用 `auditd`，所有消息都会发送到守护进程，并写入 `/data/misc/audit/audit.log` 文件。这个日志文件，在此处称为 `audit.log`，可能在设备启动时存在，并旋转到 `/data/misc/audit/audit.old` 文件，称为 `audit.old`。守护进程将重新开始向新的 `audit.log` 文件记录。当大小阈值 `AUDITD_MAX_LOG_FILE_SIZEKB`（在 `system/core/auditd/Android.mk` 文件编译时设置）被超过时，就会发生这个旋转事件。这个阈值通常是 1000 KB，但可以在设备的 `makefile` 中更改。此外，使用 `kill` 发送 `SIGHUP` 也会像以下示例中那样导致旋转。

验证守护进程正在运行并获取其 PID：

```kt
root@udoo:/ # ps -Z | grep audit
u:r:auditd:s0 audit 2281 1 /system/bin/auditd
u:r:kernel:s0 root 2293 2 kauditd

```

验证只存在一个日志：

```kt
root@udoo:/ # ls -la /data/misc/audit/
-rw-r----- audit system 79173 1970-01-02 00:19 audit.log

```

旋转日志：

```kt
root@udoo:/ # kill -SIGHUP 2281

```

验证 `audit.old`：

```kt
root@udoo:/ # ls -la /data/misc/audit/
-rw-r----- audit system 319 1970-01-02 00:20 audit.log
-rw-r----- audit system 79173 1970-01-02 00:19 audit.old

```

## auditd 内部机制

由于 Linux 桌面的 `auditd` 和 `libaudit` 代码具有 GPL 许可证，因此为 Android 进行了重写，并发布了 Apache 许可证。重写是最小化的，因此你只会找到支持守护进程所需的功能。尽管如此，功能性和头文件接口应该保持相同。

`auditd`守护进程在其生命周期中从`system/core/auditd.c`中的`main()`函数开始。它迅速将其权限从 UID root 更改为特殊的`auditd` UID。在这个过程中，它保留了`CAPSYS_AUDIT`权限，这是使用`AUDIT` NETLINK 套接字所必需的 DAC 能力检查。它是通过调用`drop_privileges_or_die()`来实现的。从那里，它使用`getopt()`进行一些选项解析，然后我们最终到达审计特定的调用，第一个调用是使用`audit_open()`打开 NETLINK 套接字。这个函数简单地调用`socket(PF_NETLINK, SOCK_RAW, NETLINK_AUDIT)`，从而打开一个指向 NETLINK 套接字的数据文件描述符。在打开套接字之后，守护进程通过调用`audit_log_open(const char *logfile, const char *rotatefile, size_t threshold)`打开对`audit.log`的句柄。这个函数检查`audit.log`文件是否存在，如果存在，则将其重命名为`audit.old`。然后，它创建一个新的空日志文件，用于记录数据。

下一步是将守护进程注册到审计子系统中，以便它知道向谁发送消息。通过设置守护进程的 PID，你确保只有这个守护进程会接收到消息。由于 NETLINK 可以支持多个读取者，你不希望有一个“流氓`auditd`”读取消息。有了这个前提，守护进程调用`audit_set_pid(audit_fd, getpid(), WAIT_YES)`，其中`audit_fd`是来自`audit_open()`的 NETLINK 套接字，`getpid()`返回守护进程的 PID，而`WAIT_YES`使得守护进程在操作完成前阻塞。接下来，守护进程通过调用`audit_set_enabled(audit_fd, 1)`启用审计子系统的先进功能，并通过`audit_rules_read_and_add(audit_fd, AUDITD_RULES_FILE)`将规则添加到审计子系统中。这个函数从该文件读取规则，格式化一些结构，并将这些结构发送到内核。

`audit_set_enabled()`和`audit_rules_read_and_add()`只有在内核构建时带有`CONFIG_AUDITSYSCALL`配置选项时才有效。在此之后，守护进程检查是否指定了`-k`选项。`-k`选项告诉`auditd`在`dmesg`中查找任何丢失的审计记录。它这样做是因为在捕获审计记录以防止环形缓冲区溢出和用户空间启动许多服务、生成审计事件和政策违规之间存在竞争。本质上，这有助于将早期引导中的审计事件合并到相同的日志文件中。

之后，守护进程进入一个循环，从 NETLINK 套接字读取，格式化消息，并将它们写入日志文件。它通过使用 `poll()` 等待 NETLINK 套接字上的 IO 来启动这个循环。如果 `poll()` 因为错误而退出，循环将继续检查 `quit` 变量。如果引发 `EINTR`，循环保护器 `quit` 在信号处理程序中设置为 `true`，守护进程退出。如果 `poll()` 在 NETLINK 上有数据，守护进程会调用 `audit_get_reply(audit_fd, &rep, GET_REPLY_BLOCKING, 0)`，并通过 `rep` 参数返回一个 `audit_reply` 结构。然后，它使用 `audit_log_write(alog, "type=%d msg=%.*s\n", rep.type, rep.len, rep.msg.data)` 将格式化的 `audit_reply` 结构写入 `audit.log` 文件。它一直这样做，直到引发 `EINTR`，此时守护进程退出。

当守护进程退出时，它会清除与内核注册的 PID (`audit_set_pid(audit_fd, 0)`)，通过 `audit_close()` 关闭审计套接字（实际上就是系统调用 `close(audit_fd)`），并使用 `audit_log_close()` 关闭 `audit.log` 文件。`audit_log_*` 函数族不是 GPL 授权的审计接口的一部分，而是一种自定义的写入方式。

当谷歌将 `auditd` 移植到 Android 的 `logd` 基础设施时，它使用了守护进程 `main()` 中使用的相同函数和库代码，并将其包装到 `logd` 中。然而，谷歌 *没有* 使用 `audit_set_enabled()` 和 `audit_rules_read_and_add()` 函数。

# 解析 SELinux 拒绝日志

SELinux 拒绝信息会被路由到内核审计子系统，到 `auditd`，最终到 `audit.log` 和 `audit.old`。由于日志存储在 `audit.log` 中，让我们通过 `adb` 将此文件拉取过来，并仔细查看它。

从主机运行以下命令，确保 `adb` 已启用：

```kt
$ adb pull /data/misc/audit/audit.log

```

现在，让我们查看该文件并寻找以下行：

```kt
$ tail audit.log
...
type=1400 msg=audit(88526.980:312): avc: denied { getattr } for pid=3083 comm="adbd" path="/data/misc/audit/audit.log" dev=mmcblk0p4 ino=42 scontext=u:r:adbd:s0 tcontext=u:object_r:audit_log:s0 tclass=file
type=1400 msg=audit(88527.030:313): avc: denied { read } for pid=3083 comm="adbd" name="audit.log" dev=mmcblk0p4 ino=42 scontext=u:r:adbd:s0 tcontext=u:object_r:audit_log:s0 tclass=file
type=1400 msg=audit(88527.030:314): avc: denied { open } for pid=3083 comm="adbd" name="audit.log" dev=mmcblk0p4 ino=42 scontext=u:r:adbd:s0 tcontext=u:object_r:audit_log:s0 tclass=file

```

这里的记录由两个主要部分组成：`type` 和 `msg`。`type` 字段指示消息的类型。类型为 1400 的消息是 AVC 消息，即 SELinux 拒绝消息（还有其他类型）。前面的策略中的 `msg`（简称消息）部分是我们需要分析的部分。

我们最后执行的命令是 `adb pull /data/misc/audit/aduit.log`，如您所见，我们在 `audit.log` 文件的末尾有一些 `adb` 策略违规。让我们先看看这个事件：

```kt
type=1400 msg=audit(88526.980:312): avc: denied { getattr } for pid=3083 comm="adbd" path="/data/misc/audit/audit.log" dev=mmcblk0p4 ino=42 scontext=u:r:adbd:s0 tcontext=u:object_r:audit_log:s0 tclass=file

```

我们可以看到 `comm` 字段是 `adbd`。然而，由于它可以通过 `prctl()` 接口从用户空间控制，所以这个值并不可信。它只能作为一个提示。验证这个值最好的方法是使用 `ps -Z` 检查 PID：

```kt
# ps -Z | grep adbd
u:r:adbd:s0 root 3083 1 /sbin/adbd

```

在验证守护进程后，我们现在可以更详细地检查消息。消息由以下字段组成（可选字段用 `*` 标识）：

+   `avc: denied`：这部分总是会发生，表示这是一个拒绝记录。

+   `{权限}`：这是被拒绝的权限，在本例中是 `getattr`。

+   `for`：这总是会被打印出来，使得输出可读。

+   `Path*`: 这是一个可选字段，包含所讨论对象的路径。它仅在文件系统访问拒绝的情况下才有意义。

+   `dev*`: 这是一个可选字段，用于标识挂载文件系统的块设备。它仅在文件系统访问拒绝的情况下才有意义。

+   `ino*`: 这是一个可选的文件 inode。只有 Linux 中的匿名文件会打印 inode。它仅在文件系统访问拒绝的情况下才有意义。

+   `tclass`: 这是对象的目标类，在我们的例子中是 `file`。

在这一点上，我们需要非常精炼地理解拒绝记录中的 `msg` 部分在告诉我们什么。它表示 Android 调试桥接守护进程想要能够调用我们的策略文件上的 `getattr`。在接下来的几个事件中，我们会看到它还想要 `read` 和 `open`。这是运行 `adb pull` 的副作用。`getattr` 权限拒绝来自 `stat()` 系统调用，而 `read/open` 来自 `read()` 和 `open()` 系统调用。如果你想在策略中允许这一点，这将是一个基于你的威胁模型的安全决策，你应该添加：

```kt
allow adbd audit_log:file { getattr read open };

```

或者，使用在 `global_macros` 中定义的宏集：

```kt
allow adbd audit_log:file r_file_perms;

```

大多数时候，你应该使用在 `global_macros` 中定义的宏来访问文件权限。通常，逐个添加它们是非常耗时且繁琐的。宏将权限分组在一个类似于读取、写入和执行 DAC 权限的上下文中。例如，如果你给它 `open` 和 `read`，那么在某个时候源域可能需要获取该文件的统计信息。因此，`r_file_perms` 宏已经包含了这些权限。

你应该将此规则添加到 `external/sepolicy/adbd.te`。`.te` 文件（也称为类型强制文件）按源上下文组织，所以请确保将其添加到正确的文件。我们不推荐添加此允许规则——`adbd` 需要访问审计日志没有合法的理由——我们可以安全地忽略这些内容，使用 `dontaudit` 规则：

```kt
dontaudit adbd audit_log:file r_file_perms;

```

`dontaudit` 规则是一个策略声明，表示不要审计（打印）与该规则匹配的拒绝。

如果你不确定该怎么做，最好的建议是利用 SE for Android、SELinux 和 audit 的邮件列表。只需确保消息与特定邮件列表的主题相关。

存在一个名为 `audit2allow` 的工具，可以帮助你编写策略允许规则。然而，它只是一个工具，可能会被误用。它将策略文件转换为策略的允许规则：

```kt
$ cat audit.log | audit2allow 
#============= adbd ==============
allow adbd audit_log:file { read getattr open };

```

`audit2allow` 工具不具备宏意识，或者如果你真的想将此允许规则添加到策略文件中，它也不具备这种意识。只有策略作者才能做出这个决定。

还有一个名为 `fixup.py` 的工具，可以启用 `r_file_*` 宏映射。你可以在 [`bitbucket.org/billcroberts/fixup/overview`](https://bitbucket.org/billcroberts/fixup/overview) 获取该工具。下载后，使其可执行，并将其放置在可执行路径中的某个位置：

```kt
$ chmod a+x fixup.py
$ cat audit.log | audit2allow | fixup.py 
#============= adbd ==============
allow adbd audit_log:file r_file_perms;

```

# 上下文

在最简单的意义上，编写策略的活动就是识别策略违规并添加适当的允许规则到策略文件中。然而，为了使 SELinux 有效，源上下文和目标上下文必须正确。如果不正确，允许规则就毫无意义。

你可能会遇到的目标类型未标记的拒绝。在这种情况下，需要设置正确的目标标签（参考第十一章，*标记属性*）。此外，进程标签也可能错误。多个进程可以属于一个域，除非通过策略明确执行，否则父进程的子进程会继承父进程的域。然而，在 Android 中，具有多个进程的域相当有限。你永远不会在`init`、`system_server`、`adbd`、`auditd`、`debuggerd`、`dhcp`、`servicemanager`、`vold`、`netd`、`surfaceflinger`、`drmserver`、`mediaserver`、`installd`、`keystore`、`sdcardd`、`wpa`和`zygote`域中看到多个进程。

在以下域中看到多个进程是可以的：

+   `system_app`

+   `untrusted_app`

+   `platform_app`

+   `shared_app`

+   `media_app`

+   `release_app`

+   `isolated_app`

+   `shell`

在已发布的设备上，`su`、`recovery`和`init_shell`域中不应运行任何内容。以下表格提供了域到预期可执行文件和容量的完整映射：

| 域 | 可执行文件 | 容量（N） |
| --- | --- | --- |
| `u:r:init:s0"` | `/init` | `N == 1` |
| `u:r:ueventd:s0` | `/sbin/ueventd` | `N == 1` |
| `u:r:healthd:s0` | `/sbin/healthd` | `N == 1` |
| `u:r:servicemanager:s0` | `/system/bin/servicemanager` | `N == 1` |
| `u:r:vold:s0` | `/system/bin/vold` | `N == 1` |
| `u:r:netd:s0` | `/system/bin/netd` | `N == 1` |
| `u:r:debuggerd:s0` | `/system/bin/debuggerd, /system/bin/debuggerd64` | `N == 1` |
| `u:r:surfaceflinger:s0` | `/system/bin/surfaceflinger` | `N == 1` |
| `u:r:zygote:s0` | `zygote, zygote64` | `N == 1` |
| `u:r:drmserver:s0` | `/system/bin/drmserver` | `N == 1` |
| `u:r:mediaserver:s0` | `/system/bin/mediaserver` | `N >= 1` |
| `u:r:installd:s0` | `/system/bin/installd` | `N == 1` |
| `u:r:keystore:s0` | `/system/bin/keystore` | `N == 1` |
| `u:r:system_server:s0` | `system_server` | `N ==1` |
| `u:r:sdcardd:s0` | `/system/bin/sdcard` | `N >=1` |
| `u:r:watchdogd:s0` | `/sbin/watchdogd` | `N >=0 && N < 2` |
| `u:r:wpa:s0` | `/system/bin/wpa_supplicant` | `N >=0 && N < 2` |
| `u:r:init_shell:s0` | `null` | `N == 0` |
| `u:r:recovery:s0` | `null` | `N == 0` |
| `u:r:su:s0` | `null` | `N == 0` |

围绕这一点已经编写了几个**兼容性测试套件**（**CTS**）测试，并已提交到 AOSP，[`android-review.googlesource.com/#/c/82861/`](https://android-review.googlesource.com/#/c/82861/)。

基于对良好策略应具备的通用断言，让我们评估我们的策略。

首先，我们将检查未标记的对象。从主机，使用您通过`adb pull`获得的`audit.log`文件：

```kt
$ cat audit.log | grep unlabeled
...
type=1400 msg=audit(86527.670:341): avc: denied { rename } for pid=3206 comm="pool-1-thread-1" name="com.android.settings_preferences.xml" dev=mmcblk0p4 ino=129664 scontext=u:r:system_app:s0 tcontext=u:object_r:unlabeled:s0 tclass=file
...

```

看起来我们有一些文件和其他东西没有正确标记；我们将在第十一章*标记属性*中解决这些问题。现在，让我们检查那些不应该有多个进程的域，并找出这些域中的不适当二进制文件（参考前表以获取完整映射）。

Init：

```kt
$ adb shell ps -Z | grep u:r:init:s0
u:r:init:s0 root 1 0 /init
u:r:init:s0 root 2267 1 /sbin/watchdogd

```

Zygote：

```kt
$ adb shell ps -Z | grep u:r:zygote:s0
u:r:zygote:s0 root 2285 1 zygote
$ adb shell ps -Z | grep u:r:init_shell
u:r:init_shell:s0 root 2278 1 /system/bin/sh
… through all domains

```

在完成这项工作后，我们发现了一些问题，因为某些东西正在`init_shell`域中运行，而`watchdogd`却在`init`域中。这些问题必须得到纠正。

# 摘要

编写`sepolicy`相对容易，编写好的策略则是一门艺术。它要求策略作者理解系统以及`allow`规则的影响。策略本身是一种元编程语言，其中语言控制用户空间和内核如何相处，并且与任何程序一样，策略可以针对特定用途进行架构。策略可能过于松散（本质上无用）或非常紧密，难以更改而不破坏已经正常工作的部分。

一项好的策略需要保留系统的预期正确功能，因此对 Android 系统中所有系统的彻底测试是必不可少的。CTS 在测试 Android 系统方面非常有帮助，但它通常并不涵盖所有情况；建议进行用户测试。在下一章中，我们将介绍文件系统和文件系统对象是如何获得其安全标签的，以及我们如何可以更改它们。稍后，我们将讨论如何将 CTS 用作工具来测试系统并生成针对预期行为的策略违规。
