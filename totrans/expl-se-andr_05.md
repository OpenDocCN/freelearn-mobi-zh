# 第五章 系统引导

现在我们有了 Android 系统的 SE，我们需要看看我们如何利用它，并将其转换成可用的状态。在本章中，我们将：

+   修改日志级别以在调试时获取更多详细信息

+   跟踪与策略加载器相关的引导过程

+   调查 SELinux API 和 SELinuxFS

+   解决最大策略版本号的问题

+   应用补丁以加载和验证 NSA 策略

你可能已经注意到了一些令人不安的错误消息`dmesg`在第四章“在 UDOO 上的安装”。为了刷新你的记忆，以下是一些例子：

```kt
# dmesg | grep –i selinux
<6>SELinux: Initializing.
<7>SELinux: Starting in permissive mode
<7>SELinux: Registering netfilter hooks
<3>SELinux: policydb version 26 does not match my version range 15-23
...

```

看起来即使 SELinux 已启用，我们也没有一个完全无错误的系统。在这个时候，我们需要了解是什么导致了这个错误，我们可以做些什么来纠正它。在本章结束时，我们应该能够识别 SE for Android 设备的引导过程，以及该策略是如何加载到内核中的。然后我们将解决策略版本错误。

# 策略加载

Android 设备遵循与*NIX 引导序列相似的引导序列。引导加载程序引导内核，内核最终执行 init 进程。init 进程负责通过 init 脚本和守护进程中的某些硬编码逻辑来管理设备的引导过程。像所有进程一样，init 在`main`函数处有一个入口点。这是第一个用户空间进程开始的地方。代码可以通过导航到`system/core/init/init.c`找到。

当 init 进程进入`main`（参考以下代码片段）时，它处理`cmdline`，挂载一些`tmpfs`文件系统，如`/dev`，以及一些伪文件系统，如`procfs`。对于 SE for Android 设备，init 被修改为在引导过程中尽可能早地加载策略到内核中。SELinux 系统中的策略不是内置于内核中的；它位于一个单独的文件中。在 Android 中，早期引导过程中挂载的唯一文件系统是根文件系统，它是内置于`boot.img`中的 ramdisk。策略可以在 UDOO 或目标设备的根文件系统中找到，位于`/sepolicy`。此时，init 进程调用一个函数从磁盘加载策略并将其发送到内核，如下所示：

```kt
int main(int argc, char *argv[]) {
...
  process_kernel_cmdline();
  unionselinux_callback cb;
  cb.func_log = klog_write;
  selinux_set_callback(SELINUX_CB_LOG, cb);

  cb.func_audit = audit_callback;
  selinux_set_callback(SELINUX_CB_AUDIT, cb);

  INFO("loading selinux policy\n");
  if (selinux_enabled) {
    if (selinux_android_load_policy() < 0) {
      selinux_enabled = 0;
      INFO("SELinux: Disabled due to failed policy load\n");
    } else {
      selinux_init_all_handles();
    }
  } else {
    INFO("SELinux:  Disabled by command line option\n");
  }
…
```

在前面的代码中，你会注意到非常漂亮的日志消息，“SELinux: 由于策略加载失败而禁用”，并想知道为什么我们在运行`dmesg`之前没有看到这个消息。这段代码在`init.rc`中的`setlevel`执行之前执行。

默认的 init 日志级别是由`system/core/include/cutils/klog.h`中`KLOG_DEFAULT_LEVEL`的定义设置的。如果我们真的想改变它，重新构建，实际上可以看到那条消息。

既然我们已经确定了策略加载的初始路径，让我们跟随它在系统中的路径。`selinux_android_load_policy()` 函数可以在 `libselinux` 的 Android 分支中找到，该分支位于 UDOO Android 源树中。库可以在 `external/libselinux` 中找到，所有的 Android 修改都可以在 `src/android.c` 中找到。

函数首先挂载一个名为 **SELinuxFS** 的伪文件系统。如果你还记得，这是我们之前在 第四章 中提到的在 `/proc/filesystems` 中看到的新的文件系统之一，*在 UDOO 上安装*。在没有挂载 `sysfs` 的系统上，挂载点为 `/selinux`；在有 `sysfs` 挂载的系统上，挂载点为 `/sys/fs/selinux`。

你可以使用以下命令在运行中的系统上检查 `mountpoints`：

```kt
# mount | grep selinuxfs 
selinuxfs /sys/fs/selinux selinuxfs rw,relatime 0 0

```

SELinuxFS 是一个重要的文件系统，因为它提供了内核和用户空间之间控制和管理 SELinux 的接口。因此，它必须挂载才能使策略加载工作。策略加载使用文件系统将策略文件字节发送到内核。这发生在 `selinux_android_load_policy()` 函数中：

```kt
int selinux_android_load_policy(void)
{
  char *mnt = SELINUXMNT;
  int rc;
  rc = mount(SELINUXFS, mnt, SELINUXFS, 0, NULL);
  if (rc < 0) {
    if (errno == ENODEV) {
      /* SELinux not enabled in kernel */
      return -1;
    }
    if (errno == ENOENT) {
      /* Fall back to legacy mountpoint. */
      mnt = OLDSELINUXMNT;
      rc = mkdir(mnt, 0755);
      if (rc == -1 && errno != EEXIST) {
        selinux_log(SELINUX_ERROR,"SELinux: Could not mkdir:  %s\n",
        strerror(errno));
        return -1;
      }
      rc = mount(SELINUXFS, mnt, SELINUXFS, 0, NULL);
    }
  }
  if (rc < 0) {
    selinux_log(SELINUX_ERROR,"SELinux:  Could not mount selinuxfs:  %s\n",
    strerror(errno));
    return -1;
  }
  set_selinuxmnt(mnt);

  return selinux_android_reload_policy();
}
```

`set_selinuxmnt(car *mnt)` 函数在 `libselinux` 中改变一个全局变量，以便其他例程可以找到这个重要接口的位置。从那里它调用另一个辅助函数，`selinux_android_reload_policy()`，该函数位于相同的 `libselinux` `android.c` 文件中。它按优先级顺序遍历一个可能的策略位置数组。这个数组定义如下：

```kt
Static const char *const sepolicy_file[] = {
  "/data/security/current/sepolicy",
  "/sepolicy",
  0 };
```

由于只有根文件系统被挂载，它此时选择 `/sepolicy`。另一个路径是用于策略的动态运行时重新加载。在获取到策略文件的合法文件描述符后，系统将其内存映射到其地址空间，并调用 `security_load_policy(map, size)` 将其加载到内核。这个函数定义在 `load_policy.c` 中。在这里，map 参数是策略文件开始的指针，size 参数是文件的大小（以字节为单位）：

```kt
int selinux_android_reload_policy(void)
{
  int fd = -1, rc;
  struct stat sb;
  void *map = NULL;
  int i = 0;

  while (fd < 0 && sepolicy_file[i]) {
    fd = open(sepolicy_file[i], O_RDONLY | O_NOFOLLOW);
    i++;
  }
  if (fd < 0) {
    selinux_log(SELINUX_ERROR, "SELinux:  Could not open sepolicy:  %s\n",
    strerror(errno));
    return -1;
  }
  if (fstat(fd, &sb) < 0) {
    selinux_log(SELINUX_ERROR, "SELinux:  Could not stat %s:  %s\n",
    sepolicy_file[i], strerror(errno));
    close(fd);
    return -1;
  }
  map = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
  if (map == MAP_FAILED) {
    selinux_log(SELINUX_ERROR, "SELinux:  Could not map %s:  %s\n",
    sepolicy_file[i], strerror(errno));
    close(fd);
    return -1;
  }

  rc = security_load_policy(map, sb.st_size);
  if (rc < 0) {
    selinux_log(SELINUX_ERROR, "SELinux:  Could not load policy:  %s\n",
    strerror(errno));
    munmap(map, sb.st_size);
    close(fd);
    return -1;
  }

  munmap(map, sb.st_size);
  close(fd);
  selinux_log(SELINUX_INFO, "SELinux: Loaded policy from %s\n", sepolicy_file[i]);

  return 0;
}
```

安全加载策略打开 `<selinuxmnt>/load` 文件，在我们的例子中是 `/sys/fs/selinux/load`。在这个时候，策略通过这个伪文件写入内核：

```kt
int security_load_policy(void *data, size_t len)
{
  char path[PATH_MAX];
  int fd, ret;

  if (!selinux_mnt) {
    errno = ENOENT;
    return -1;
  }

  snprintf(path, sizeof path, "%s/load", selinux_mnt);
  fd = open(path, O_RDWR);
  if (fd < 0)
  return -1;

  ret = write(fd, data, len);
  close(fd);
  if (ret < 0)
  return -1;
  return 0;
}
```

# 修复策略版本

到目前为止，我们已经清楚地了解了策略是如何加载到内核中的。这非常重要。SELinux 与 Android 的集成始于 Android 4.0，因此在移植到各种分支和片段时，这会导致问题，代码通常会缺失。了解系统的所有部分，即使是非常粗略的了解，也将帮助我们纠正野外出现的问题并开发。这些信息也有助于理解整个系统，因此当需要做出修改时，你会知道该往哪里看以及事情是如何运作的。到目前为止，我们已经准备好纠正策略版本。

日志和内核配置都很清晰；只支持政策版本 23 及以下，而我们正在尝试加载政策版本 26。考虑到内核通常过时，这可能是 Android 的一个常见问题。

Google 提供的 4.3 sepolicy 也存在问题。Google 的一些改动使得配置设备变得稍微困难一些，因为他们根据发布目标定制了策略。本质上，该策略允许几乎所有操作，因此产生的拒绝日志非常少。策略中的某些域通过每个域的允许性声明完全允许，这些域也有允许所有操作的规则，因此不会生成拒绝日志。为了纠正这个问题，我们可以使用来自 NSA 的更完整的策略。将`external/sepolicy`替换为从[`bitbucket.org/seandroid/external-sepolicy/get/seandroid-4.3.tar.bz2`](https://bitbucket.org/seandroid/external-sepolicy/get/seandroid-4.3.tar.bz2)下载的内容。

在我们提取 NSA 的策略后，我们需要纠正策略版本。策略位于`external/sepolicy`，并使用名为`check_policy`的工具编译。sepolicy 的`Android.mk`文件必须将此版本号传递给编译器，因此我们可以在这里调整它。在文件顶部，我们找到了问题所在：

```kt
...
# Must be <= /selinux/policyvers reported by the Android kernel.
# Must be within the compatibility range reported by checkpolicy -V.
POLICYVERS ?= 26
...

```

由于变量可以通过`?=`赋值来覆盖。我们可以在`BoardConfig.mk`中覆盖它。编辑`device/fsl/imx6/BoardConfigCommon.mk`，在文件底部添加以下`POLICYVERS`行：

```kt
...
BOARD_FLASH_BLOCK_SIZE := 4096
TARGET_RECOVERY_UI_LIB := librecovery_ui_imx
# SELinux Settings
POLICYVERS := 23
-include device/google/gapps/gapps_config.mk

```

由于策略位于`boot.img`镜像中，因此需要构建策略和`bootimage`：

```kt
$ mmm -B external/sepolicy/
$ make –j4 bootimage 2>&1 | tee logz
!!!!!!!!! WARNING !!!!!!!!! VERIFY BLOCK DEVICE !!!!!!!!!
$ sudo chmod 666 /dev/sdd1
$ dd if=$OUT/boot.img of=/dev/sdd1 bs=8192 conv=fsync

```

弹出 SD 卡，将其放入 UDOO 中并启动。

### 小贴士

前面的第一个命令应该产生以下日志输出：

```kt
out/host/linux-x86/bin/checkpolicy: writing binary representation (version 23) to out/target/product/udoo/obj/ETC/sepolicy_intermediates/sepolicy

```

在这一点上，通过使用`dmesg`检查 SELinux 日志，我们可以看到以下内容：

```kt
# dmesg | grep –i selinux
<6>init: loading selinux policy
<7>SELinux: 128 avtab hash slots, 490 rules.
<7>SELinux: 128 avtab hash slots, 490 rules.
<7>SELinux: 1 users, 2 roles, 274 types, 0 bools, 1 sens, 1024 cats
<7>SELinux: 84 classes, 490 rules
<7>SELinux: Completing initialization.

```

我们还需要运行另一个命令是`getenforce`。`getenforce`命令获取 SELinux 强制执行状态。它可以处于三种状态之一：

+   **禁用**：没有加载策略或没有内核支持

+   **允许**：策略已加载且设备记录拒绝（但不在强制执行模式）

+   **强制执行**：此状态与允许状态类似，除了策略违规会导致返回给用户空间的 EACCESS 外

启动 SELinux 系统时的一个目标之一是达到强制执行状态。允许模式用于调试，如下所示：

```kt
# getenforce
Permissive

```

# 摘要

在本章中，我们介绍了通过 init 进程的重要策略加载流程。我们还更改了策略版本以适应我们的开发努力和内核版本。从那里，我们能够加载 NSA 策略并验证系统已加载它。本章还展示了 SELinux API 及其与 SELinuxFS 的交互。在下一章中，我们将检查文件系统，然后继续我们的努力将系统置于强制执行模式。
