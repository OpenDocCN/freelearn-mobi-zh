# 第六章：探索 SELinuxFS

在最后几章中，我们多次看到了 SELinuxFS 的出现。从 `/proc/filesystems` 中的条目到 init 守护进程中的策略加载，它在启用 SELinux 的系统中被频繁使用。SELinuxFS 是内核到用户空间的接口，也是更高用户空间习惯和 `libselinux` 的基础。在本章中，我们将探索这个文件系统的功能，以更深入地了解系统是如何工作的。具体来说，我们将：

+   确定如何找到 SELinux 文件系统的挂载点

+   提取我们当前 SELinux 系统的状态信息

+   通过 shell 和代码动态修改我们的 SELinux 系统状态

+   调查 ProcFS 接口

# 定位文件系统

我们需要做的第一件事是定位文件系统的挂载点。`libselinux` 将文件系统挂载在两个位置之一：默认为 `/selinux` 或 `/sys/fs/selinux`。然而，这并不是一个严格的要求，可以通过调用 `void set_selinuxmnt(char *mnt)` 来更改，该函数设置 SELinux 挂载点位置。然而，在大多数情况下，这应该发生，并且不需要任何调整。

在系统中找到挂载点的最佳方式是运行 `mount` 命令并找到文件系统的位置。从串行控制台发出以下命令：

```kt
root@udoo:/ # mount | grep selinux
selinuxfs /sys/fs/selinux selinuxfs rw,relatime 0 0

```

如您所见，挂载点是 `/sys/fs/selinux`。让我们通过在串行终端提示符下发出以下命令进入该目录：

```kt
root@udoo:/ # cd /sys/fs/selinux
root@udoo:/sys/fs/selinux #

```

您现在位于 SELinux 文件系统的根目录。

# 查询文件系统

您可以通过 SELinuxFS 来查询内核支持的最高策略版本。这在您开始使用非源码构建的系统时非常有用。当您没有直接访问 KConfig 文件时，这也很有用。需要注意的是，DAC 和 MAC 权限都适用于此文件系统。关于 MAC 和 SELinux，此文件系统的访问向量在位于 `external/sepolicy/access_vectors` 的策略文件中的类安全部分中列出：

```kt
root@udoo:/sys/fs/selinux # echo 'cat policyvers'
23

```

### 提示

在之前的命令以及接下来的几个命令中，我们不仅仅使用 `cat` 命令打印文件。这是因为这些文件在文件末尾没有换行符。没有换行符，命令执行后的命令提示符会位于输出最后一行的末尾。使用 `echo` 将 `cat` 命令包装起来可以保证有换行。另一种达到相同效果的方法是使用 `cat policyvers ; echo`。

如我们所料，支持的版本是 23。如您所回忆的，我们在 第四章 *在 UDOO 上安装* 中设置了此值，当时在 `kernel_imx` 目录中使用 `make menuconfig` 配置内核以启用 SELinux。这也可以通过 `libselinux` API 访问：

```kt
int security_policyvers(void);

```

这不应该需要任何提升的权限，并且可以被系统上的任何人读取。

## 执法节点

在前面的章节中，我们讨论了 SELinux 在两种模式下运行，**执行** 和 **宽容**。两种模式都会记录策略违规，然而，执行模式会导致内核拒绝访问资源，并将错误返回给调用用户空间进程（例如，`EACCESS`）。SELinuxFS 有一个接口可以查询此状态——文件节点 `enforce`。从该文件读取返回的状态是 `0` 或 `1`，分别取决于我们是否在宽容或执行模式下运行：

```kt
root@udoo:/sys/fs/selinux # echo 'cat enforce' 
0

```

如您所见，我们的系统处于宽容模式。Android 也有一个工具箱命令来打印这个状态。该命令返回的状态是 `Permissive` 或 `Enforcing`，分别取决于我们是否在宽容或执行模式下运行：

```kt
root@udoo:/sys/fs/selinux # getenforce
Permissive

```

您还可以写入 `enforce` 文件。此文件系统的 DAC 权限是：

```kt
Owner: root read, write
Group: root read
Others: read

```

任何人都可以获取执行状态，但要设置它，您必须是 root 用户。为此所需的 MAC 权限是：

```kt
class: security 
vector: setenforce

```

一个名为 `setenforce` 的命令可以更改状态：

```kt
root@udoo:/sys/fs/selinux # setenforce 0

```

要查看命令的作用，请在 `strace` 中运行它：

```kt
root@udoo:/sys/fs/selinux # strace setenforce 0

...
open("/proc/self/task/3275/attr/current", O_RDONLY) = 4
brk(0x41d80000) = 0x41d80000
read(4, "u:r:init_shell:s0\0", 4095) = 18
close(4) = 0
open("/sys/fs/selinux/enforce", O_RDWR) = 4
write(4, "0", 1) 
...

```

如我们所见，`enforce` 的接口就像写入 `0` 或 `1` 那么简单。在 `libselinux` 中执行此操作的函数是 `int security_setenforce(int value)`。前述命令的另一个有趣之处是我们可以看到 `procfs` 被访问。SELinux 在 `procfs` 中还有一些额外的条目。这些将在本章中进一步讨论。

## 禁用文件接口

SELinux 还可以通过 `disable` 文件接口在运行时禁用。但是，内核必须使用 `CONFIG_SECURITY_SELINUX_DISABLE=y` 选项构建。我们的内核没有使用此选项。此文件只能由所有者写入，并且没有与之关联的特定 MAC 权限。我们建议保持此选项禁用。此外，SELinux 可以在加载策略之前禁用。即使选项已启用，一旦加载了策略，它就会被禁用。

## 策略文件

`policy` 文件允许您读取已加载到内核中的当前 SELinux 策略文件。这可以读取并保存到磁盘：

```kt
root@udoo:/sys/fs/selinux # cat policy > /sdcard/policy

```

通过启用 `adb` 接口，您现在可以从设备中提取它，并在主机上使用标准的 SELinux 工具进行分析。此文件的 DAC 权限是所有者：`root`，`read`。没有针对此文件的特定 SELinux 权限。

`policy` 文件的逆操作是 `load` 文件。我们已经看到当策略文件通过 `libselinux` API 被 init 加载时，该文件会出现：

```kt
int security_load_policy(void *data, size_t len);

```

## 空文件

`null` 文件在域转换发生时被 SELinux 用于重定向未经授权的文件访问。请记住，域转换是指从一个上下文转换到另一个上下文。在大多数情况下，这发生在程序执行 fork 和 exec 函数时，但这也可能以编程方式发生。在任何情况下，进程都有它无法访问的文件引用，为了帮助防止进程崩溃，它们只是从 SELinux 的空设备中读写。

## mls 文件

我们系统的一个能力是，我们当前的策略正在使用**多级安全**（**MLS**）支持。这取决于加载的策略文件是否使用它，是`0`或`1`。由于我们已启用它，我们预计从这个文件中看到`1`：

```kt
root@udoo:/sys/fs/selinux # echo 'cat mls'
1

```

`mls`文件对所有用户可读，并有一个相应的 SELinux API：

```kt
int is_selinux_mls_enabled(void)

```

## 状态文件

`version`文件允许一种机制，通过它可以通知您在 SELinux 中发生的更新。一个例子就是当策略重新加载发生时。一个**用户空间对象管理器**可以缓存决策结果，并使用`reload`事件作为触发器来刷新它们的缓存。`status`文件只能由每个人读取，并且没有特定的 MAC 权限。`libselinux`API 接口是：

```kt
int selinux_status_open(int fallback);
void selinux_status_close();
int selinux_status_updated(void);
int selinux_status_getenforce(void);
int selinux_status_policyload(void);
int selinux_status_deny_unknown(void);

```

通过检查状态结构，您可以检测到变化并刷新缓存。然而，目前您在`libselinux`中缺少这个 API，但我们在第七章利用审计日志中会纠正这一点，*利用审计日志*。

文件树中有很多 SELinuxFS 文件；我们在这里只介绍几个文件，因为它们的重要性或与我们所做的工作和目标的相关性。我们没有涵盖：

+   `access`

+   `checkreqprot`

+   `commit_pending_bools`

+   `context`

+   `create`

+   `deny_unknown`

+   `member`

+   `reject_unknown`

+   `relabel`

这些文件的使用并不简单，通常由使用`libselinux`API 来抽象复杂性的用户空间对象管理器执行。

## 访问向量缓存

SELinuxFS 也有一些您可以探索的目录。第一个是`avc`。这代表“访问向量缓存”，可以用来获取内核中安全服务器的统计信息：

```kt
root@udoo:/sys/fs/selinux # cd avc/
root@udoo:/sys/fs/selinux/avc # ls
cache_stats
cache_threshold
hash_stats

```

所有这些文件都可以使用`cat`命令读取：

```kt
root@udoo:/sys/fs/selinux/avc # cat cache_stats
lookups hits misses allocations reclaims frees
285710 285438 272 272 128 128
245827 245409 418 418 288 288
267511 267227 284 284 192 193
214328 213883 445 445 288 298

```

`cache_stats`文件对所有用户可读，不需要特殊的 MAC 权限。

接下来要查看的文件是`hash_stats`：

```kt
root@udoo:/sys/fs/selinux/avc # cat hash_stats
entries: 512
buckets used: 284/512
longest chain: 7

```

访问向量缓存的基础数据结构是一个哈希表；`hash_stats`列出了当前属性。正如我们可以在前一个命令的输出中看到的那样，表中共有 512 个槽位，其中 284 个正在使用。对于冲突，我们有最长的链有 7 个条目。此文件对所有人可读，不需要特殊的 MAC 权限。您可以通过`cache_threshold`文件修改此表中的条目数。

`cache_threshold`文件用于调整`avc`哈希表中的条目数。它是全球可读的，所有者可写。它需要 SELinux 权限`setsecparam`，可以使用以下简单的命令进行写入和读取：

```kt
root@udoo:/sys/fs/selinux/avc # echo "1024" > cache_threshold 
root@udoo:/sys/fs/selinux/avc # echo 'cat cache_threshold'
1024

```

您可以通过写入`0`来禁用缓存。然而，在基准测试之外，这并不鼓励。

## 布尔值目录

接下来要检查的第二个目录是`booleans`。SELinux 的`boolean`允许通过`boolean`条件动态地改变策略声明。通过改变`boolean`状态，您可以影响加载的策略的行为。当前策略没有定义任何`boolean`；因此这个目录是空的。在定义了`boolean`的策略中，这个目录会被填充以每个`boolean`命名的文件。然后您可以读取和写入这些文件来改变`boolean`状态。Android 工具箱已经被修改，包括`getsebool`和`setsebool`命令。`libselinux` API 也公开了这些功能：

```kt
int security_get_boolean_names(char ***names, int *len);
int security_get_boolean_pending(const char *name);
int security_get_boolean_active(const char *name);
int security_set_boolean(const char *name, int value);
int security_commit_booleans(void);
int security_set_boolean_list(size_t boolcnt, SELboolean * boollist, int permanent);

```

`boolean`是事务性的。这意味着它是一个全有或全无的集合。当您使用`security_set_boolean*`时，您必须调用`security_commit_booleans()`以使其生效。与 Linux 桌面系统不同，不支持永久`boolean`。改变运行时值不会在重启之间持久化。此外，在 Android 上，如果您正在尝试 Android**兼容性测试套件**（**CTS**）合规性，`boolean`会导致测试失败。`boolean`可以根据目标具有不同的 DAC 权限，但它们始终需要 SELinux 权限`setbool`。

### 小贴士

您必须通过 Android 兼容性测试套件（Android branding）测试。更多关于 CTS 的信息可以在[`source.android.com/compatibility/cts-intro.html`](https://source.android.com/compatibility/cts-intro.html)找到。

## `class`目录

下一个要查看的目录是`class`。`class`目录包含在`access_vectors` SELinux 策略文件中定义的所有类，或者通过 SELinux 策略语言中的`class`关键字。对于策略中定义的每个类，都存在一个具有相同名称的目录。例如，在串行终端上运行以下命令：

```kt
root@udoo:/sys/fs/selinux/class # ls -la
...
dr-xr-xr-x root root 1970-01-02 01:58 peer
dr-xr-xr-x root root 1970-01-02 01:58 process
dr-xr-xr-x root root 1970-01-02 01:58 property_service
dr-xr-xr-x root root 1970-01-02 01:58 rawip_socket
dr-xr-xr-x root root 1970-01-02 01:58 security
...

```

如您从前面的命令中可以看到，有很多目录。让我们来检查一下`property_service`目录。这个目录被选中是因为它是在 Android 上唯一定义的。然而，每个目录中存在的文件是相同的，包括`index`和`perms`：

```kt
root@udoo:/sys/fs/selinux/class/property_service # ls
index
perms

```

在 SELinux 内核模块中定义的字符串和某些任意整数之间的映射是`index`。包含该类所有可能权限的目录是`perms`：

```kt
root@udoo:/sys/fs/selinux/class/property_service # cd perms/
root@udoo:/sys/fs/selinux/class/property_service/perms # ls
set

```

如您所见，`set`访问向量对`property_service`类可用。`class`目录可以非常有益于观察系统中已加载的策略文件。

## `initial_contexts`目录

下一个要查看的目录条目是`initial_contexts`。这是初始安全上下文的静态映射，也称为**安全标识符**（**sid**）。这个映射告诉 SELinux 系统应该使用哪个上下文来启动每个内核对象：

```kt
root@udoo:/sys/fs/selinux/initial_contexts # ls
any_socket
devnull
file
...

```

我们可以通过执行以下操作来查看`file`的初始 sid：

```kt
root@udoo:/sys/fs/selinux/initial_contexts # echo 'cat file'
u:object_r:unlabeled:s0

```

这对应于`external/sepolicy/initial_sid_contexts`中的条目：

```kt
...
sid file u:object_r:unlabeled:s0
...
```

## `policy_capabilities`目录

最后要查找的目录是 `policy_capabilities`。此目录定义了策略可能具有的任何附加功能。对于我们的当前设置，我们应该有：

```kt
root@udoo:/sys/fs/selinux/policy_capabilities # ls
network_peer_controls
open_perms

```

每个文件条目都包含一个布尔值，指示该功能是否启用：

```kt
root@udoo:/sys/fs/selinux/policy_capabilities # echo 'cat open_perms'
1

```

条目对所有用户都是可读的，但对所有用户都是不可写的。

## ProcFS

我们提到了一些正在导出的 procfs 接口。所讨论的大部分内容是安全上下文，这意味着 shell 应该与一些安全上下文相关联...但我们如何实现这一点？由于这是一个所有 LSM 都使用的通用机制，安全上下文是通过 procfs 读取和写入的：

```kt
root@udoo:/sys/fs/selinux/policy_capabilities # echo 'cat /proc/self/attr/current'
u:r:init_shell:s0

```

您还可以获取每个线程的上下文：

```kt
root@udoo:/sys/fs/selinux/policy_capabilities # echo '/proc/self/task/2278/attr/current'
u:r:init_shell:s0

```

只需将 `2278` 替换为您想要的线程 ID。

当前文件上的 DAC 权限对所有人都是可读和可写的，但这些文件通常受到 MAC 权限的严格限制。通常，只有拥有 procfs 条目的进程可以读取这些文件，并且您必须同时拥有标准写权限和 `setcurrent` 的组合。请注意，“from” 和 “to” 域必须通过 **dyntransition** 允许。要读取，您必须拥有 `getattr`。所有这些权限都是从安全类 `process` 获得的。`libselinux` API 函数 `getcon` 和 `setcon` 允许您操作 `current`。

`prev` 文件可用于查找您之前切换的上下文。此文件不可写：

```kt
root@udoo:/proc/self/attr # echo 'cat prev'
u:r:init:s0

```

我们串行终端之前的有效域或安全上下文是 `u:r:init:s0`。

`exec` 文件用于设置子进程的标签。这在进行 `exec` 之前设置。这些文件上的所有权限与实际设置它们时使用的 MAC 权限相同。尝试设置此权限的调用者必须也持有 `process` 类的 `setexec`。libselinux API 中的 `int setexeccon(security_context_t context)` 和 `int getexeccon(security_context_t *context)` 可用于设置和检索标签。

`fscreate`、`keycreate` 和 `sockcreate` 文件执行类似的事情。当进程创建相应的对象之一时，即 `fs` 对象（文件、命名管道或其他对象）、密钥或套接字，这里设置的值将被使用。调用者还必须持有 `process` 类的 `setfscreate`、`setsockcreate` 和 `setkeycreate`。以下 SELinux API 用于更改这些：

```kt
int set*createcon(security_context_t context);
int get*createcon(security_context_t *con);

```

其中 `*` 可以是 `fs`、`key` 或 `socket`。

需要注意的是，这些特殊的 `process` 类权限赋予您更改 `proc/attr` 文件的权限。您仍然需要通过 DAC 权限以及文件对象上设置的任何 SELinux 权限。然后，并且只有在这种情况下，您才需要额外的权限，例如 `setfscreate`。

# Java SELinux API

与之前讨论的 C API 相似的 API 也存在于 Java 中。在这种情况下，假设你将与平台一起构建代码，因为这些 API 不是与 Android SDK 一起发布的公共 API。API 位于`frameworks/base/core/java/android/os/SELinux.java`。然而，这只是 API 的一个非常有限的子集。

# 摘要

在本章中，我们探讨了与 SELinux 相关的内核与用户空间之间的接口，并强化了访问向量类和安全上下文的概念。在下一章中，我们将对我们的系统进行一些升级，并查看审计日志，逐步接近我们的最终目标——在 SELinux 强制模式下可操作的设备。我们称之为可操作，因为我们现在可以将其置于强制模式。然而，如果你现在通过在 UDOO 上执行`setenforce 1`，你的设备将变得不稳定。例如，在我们的系统中，如果我们这样做，浏览器将无法启动。
