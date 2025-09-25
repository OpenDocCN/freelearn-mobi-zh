# 第九章. 将服务添加到域中

在上一章中，我们介绍了在适当域中获取文件对象的过程。在大多数情况下，文件对象是目标。然而，在本章中，我们将：

+   强调对进程进行标记——特别是由 init 运行和管理的服务

+   管理由 init 创建的辅助关联对象

# Init – 守护进程之王

在 Linux 系统中，init 进程至关重要，Android 在这个方面也不例外。然而，Android 有自己的 init 实现。Init 是系统上的第一个进程，因此其 **进程 ID**（**PID**）为 1。所有其他进程都是直接从 init 的 `fork()` 调用的结果，因此所有进程最终都直接或间接地成为 init 的子进程。Init 负责清理和维护这些进程。例如，任何父进程死亡的孩子进程都会由内核重新分配给 init。这样，init 可以在进程退出时调用 `wait()`（有关更多详细信息，请参阅 `man 2 wait`）来清理进程。

### 注意

一个已经终止但尚未调用 `wait()` 的进程是一个 **僵尸进程**。内核必须保留进程数据结构，直到这个调用。未能这样做将无限期地消耗内存。

由于 init 是所有进程的根，它还提供了一种通过自己的脚本语言声明和执行命令的机制。使用这种语言来控制 init 的文件被称为 init 脚本，我们已经修改了一些。在源树中，我们使用了 `init.rc` 文件，您可以通过导航到 `device/fsl/imx6/etc/init.rc` 来访问它，但在设备上，它与 ramdisk 一起打包在 `/init.rc` 中，并可供 init 使用，init 也打包在 ramdisk 中的 `/init`。

要将服务添加到 init 脚本中，您可以修改 `init.r` 并添加一个声明，如下所示：

```kt
service <name> <path> [ <argument>... ]
```

这里，`name` 是服务名称，`path` 是可执行文件的路径，而 `argument` 是要传递给可执行文件的空格分隔的参数字符串，这些字符串将添加到其 `argv` 数组中。

例如，以下是 `rild` 的服务声明，**Radio Interface Layer Daemon**（**RILD**）：

```kt
Service ril-daemon /system/bin/rild
```

通常情况下，可以并且需要添加额外的服务选项。init 脚本的 `service` 语句支持丰富的选项集。有关完整列表，请参阅位于 `system/core/init/readme.txt` 的信息文件。此外，我们在 第三章 中介绍了 Android 特定的 SE 变更，*Android 是奇怪的*。

继续分析 `rild`，我们看到 UDOO `init.rc` 中的其余声明如下：

```kt
Service ril-daemon /system/bin/rild
 class main
 socket rild stream 660 root radio
 socket rild-debug stream 660 radio system
 socket rild-ppp stream 660 radio system
 user root
 group radio cache inet misc audio sdcard_rw log

```

这里值得注意的有趣之处在于它创建了很多套接字。`init.rc` 中的 `socket` 关键字由 `readme.txt` 文件描述：

### 注意

从源树文件 `system/core/init/readme.txt`：

```kt
socket <name> <type> <perm> [ <user> [ <group> [ <context> ] ] ]

```

创建一个名为`/dev/socket/<name>`的 Unix 域套接字，并将它的`fd`传递给启动的进程。类型必须是`dgram`、`stream`或`seqpacket`。`user`和`group` IDs 默认为`0`。套接字的 SELinux 安全上下文是`context`。它默认为服务安全上下文，如`seclabel`中指定的，或者基于服务可执行文件的安全上下文计算得出。

让我们看看这个目录，看看我们发现了什么。

```kt
root@udoo:/dev/socket # ls -laZ | grep adb
srw-rw---- system system u:object_r:adbd_socket:s0 adbd

```

这引发了问题，“它是如何进入该域的？”根据上一章的知识，我们知道`/**` `dev`是一个`tmpfs`，所以我们知道它不是通过`xattrs`进入这个域的。它必须是一个代码修改或类型转换。让我们检查是否是类型转换。如果是，我们预计会在展开的`policy.conf`中看到一个语句。SELinux 策略基于`m4`宏语言。在构建过程中，它被展开为`policy.conf`，然后编译。第十二章，*掌握工具链*，对此有更多细节。

我们可以通过使用 sesearch 来查找`adbd_socket`的类型转换来发现这一点：

```kt
$ sesearch -T -t adbd_socket $OUT/sepolicy

```

如您从空输出中看到的，没有这样的行，所以这不是策略在这样做，而是一个代码更改。

在 Linux 中，进程是通过`fork()`然后是`exec()`创建的。正因为如此，我们能够提供强大的关键词来搜索 init 守护进程。我们怀疑设置套接字的代码就在子进程中的`fork()`调用之后和`exec()`调用之前：

```kt
$ grep -n fork system/core/init/init.c 
235: pid = fork();

```

因此，我们正在寻找的`fork`在`init.c`的第 235 行；让我们在文本编辑器中打开`init.c`并查看。我们将找到以下片段以进行检查：

```kt
...
NOTICE("starting '%s'\n", svc->name);

  pid = fork();

  if (pid == 0) {
    struct socketinfo *si;
    struct svcenvinfo *ei;
    char tmp[32];
    int fd, sz;

    umask(077);
    if (properties_inited()) {
      get_property_workspace(&fd, &sz);
      sprintf(tmp, "%d,%d", dup(fd), sz);
      add_environment("ANDROID_PROPERTY_WORKSPACE", tmp);
    }

    for (ei = svc->envvars; ei; ei = ei->next)
      add_environment(ei->name, ei->value);

    for (si = svc->sockets; si; si = si->next) {
      int socket_type = (
        !strcmp(si->type, "stream") ? SOCK_STREAM :
          (!strcmp(si->type, "dgram") ? SOCK_DGRAM : SOCK_SEQPACKET));
      int s = create_socket(si->name, socket_type,
            si->perm, si->uid, si->gid, si->socketcon ?: scon);
      if (s >= 0) {
        publish_socket(si->name, s);
      }
...
```

根据`man 2 fork`，子进程中的`fork()`返回代码是`0`。子进程在这个`if`语句中执行，而父进程跳过它。函数`create` **_** `socket()`也似乎很有趣。它似乎接受服务名称、套接字类型、权限标志、`uid`、`gid`和`socketcon`。什么是`socketcon`？让我们检查我们是否可以追踪到它被设置的地方。

如果我们在`fork()`之前查看，我们可以看到父进程根据两个因素获取其`scon`：

```kt
...
    if (svc->seclabel) {
      scon = strdup(svc->seclabel);
      if (!scon) {
        ERROR("Out of memory while starting '%s'\n", svc->name);
        return;
      }
      } else {
...
```

通过`if`语句的第一个路径发生在`svc->seclabel`不为空时。这个`svc`结构被填充了可以与一个服务关联的选项。作为对第三章，*Android 很奇怪*的复习，`seclabel`允许你显式地设置一个服务上的上下文，硬编码到`init.rc`中的值。`else`子句要复杂和有趣得多。

在`else`子句中，我们通过调用`getcon()`获取当前进程的上下文。由于我们在 init 中运行，该函数应返回`u:r:init:s0`并将其存储在`mycon`中。下一个函数`getfilecon()`传递可执行文件的路径，并检查文件本身的上下文。第三个函数是这里的功臣：`security_compute_create()`。它接受`mycon`、`fcon`和`target`类，并计算安全上下文`scon`。给定这些输入，它尝试根据策略类型转换确定子进程的结果域。如果没有定义转换，`scon`将与`mycon`相同。

`create_socket()`函数中的条件表达式还决定了传递的 socket 上下文。变量`si`是一个结构，包含 init `service`部分中 socket 语句的所有选项。根据`readme.txt`文件，`si->socketcon`是 socket 上下文参数。换句话说，socket 上下文可以来自以下三个地方（按优先级降序）：

+   `service`声明中 socket 选项的`socketcon`选项

+   `service`关键字上的`seclabel`选项

+   从源上下文和目标上下文动态计算

socket 上下文传递给`create_socket()`。现在，让我们看看`create_socket()`。这个函数定义在`system/core/init/util.c:87`。围绕`socket()`的代码片段似乎很有趣：

```kt
...
  if (socketcon)
    setsockcreatecon(socketcon);

  fd = socket(PF_UNIX, type, 0);
  if (fd < 0) {
    ERROR("Failed to open socket '%s': %s\n", name, strerror(errno));
    return -1;
  }

  if (socketcon)
    setsockcreatecon(NULL);
...
```

`setsockcreatecon()`函数设置进程的 socket 创建上下文。这意味着通过`socket()`调用创建的 socket 将具有通过`setsockcreatecon()`设置的上下文。创建后，进程通过使用`setsockcreatecon(NULL)`将其重置为原始上下文。

下一段有趣的代码是围绕`bind()`：

```kt
...
  filecon = NULL;
  if (sehandle) {
    ret = selabel_lookup(sehandle, &filecon, addr.sun_path, S_IFSOCK);
    if (ret == 0)
      setfscreatecon(filecon);
  }

  ret = bind(fd, (struct sockaddr *) &addr, sizeof (addr));
  if (ret) {
    ERROR("Failed to bind socket '%s': %s\n", name, strerror(errno));
    goto out_unlink;
  }

  setfscreatecon(NULL);
  freecon(filecon);
...
```

在这里，我们设置了文件创建上下文。这些函数与`setsock_creation()`类似，但适用于文件系统对象。然而，`selabel_lookup()`函数在`file_contexts`中查找文件的上下文。你可能遗漏的部分是，对于基于路径的 socket，`bind()`调用在`sockaddr_un struct`指定的路径上创建一个文件。因此，socket 对象和文件系统节点条目是明显分开的，可以有不同的上下文。通常，socket 属于进程的上下文，而文件系统节点被赋予其他上下文。

# 动态域转换

我们看到了 init 计算 init socket 的上下文，但在设置子进程的域时从未遇到过。在本节中，我们将深入了解两种技术：使用 init 脚本的显式设置和 sepolicy 动态域转换。

获取子进程域的第一种方式是在 init 脚本服务声明中的`seclabel`语句。在`fork()`之后的子进程执行中，我们发现这个语句：

```kt
if (svc->seclabel) {
if (is_selinux_enabled() > 0 && setexeccon(svc->seclabel) < 0) {
ERROR("cannot setexeccon('%s'): %s\n", svc->seclabel, strerror(errno));
_exit(127);
}
}
```

为了澄清，`svc`变量是包含服务选项和参数的结构，所以`svc->seclabel`是`seclabel`。如果它被设置，它将调用`setexeccon()`，这将设置进程通过`exec()`执行的执行上下文。向下看，我们看到`exec()`函数调用被做出。`exec()`系统调用在成功时永远不会返回；它只在失败时返回。

设置子进程域的另一种方式，也是首选的方式，是使用 sepolicy。它之所以被首选，是因为策略没有依赖于其他任何东西。通过将上下文硬编码到 init 中，你将 init 脚本和 sepolicy 之间的依赖耦合起来。例如，如果 sepolicy 删除了在 init 脚本中硬编码的类型，init 的`setcon`将失败，但两个系统都将正确编译。如果你删除了一个类型转换的类型并留下了转换语句，你可以在编译时捕获错误。既然我们已经查看了`rild`服务语句，让我们看看位于`sepolicy`中的`rild.te`策略文件。我们应该使用`grep`在这个文件中搜索`type_transition`关键字：

```kt
$ grep -c type_transition rild.te 
0

```

没有找到`type_transition`的实例，但这个关键字必须存在，类似于文件。然而，它可能隐藏在一个未展开的宏中。SELinux 策略文件是用 m4 宏语言编写的，并且在编译之前会被展开。让我们查看`rild.te`并检查我们是否可以找到一些宏。它们是有区别的，看起来像带有参数的函数。我们遇到的第一个宏是`init_daemon_domain(rild)`宏。现在，我们需要在`sepolicy`中找到这个宏的定义。m4 语言使用`define`关键字来声明宏，所以我们可以搜索它：

```kt
$ grep -n init_daemon_domain * | grep define
te_macros:99:define(`init_daemon_domain', `

```

我们的宏在`te_macros`中声明，这个文件巧合地包含了所有与**类型强制**（**TE**）相关的宏。让我们更详细地看看这个宏做了什么。首先，它的定义是：

```kt
...
#####################################
# init_daemon_domain(domain)
# Set up a transition from init to the daemon domain
# upon executing its binary.
define(`init_daemon_domain', `
domain_auto_trans(init, $1_exec, $1)
tmpfs_domain($1)
')
...
```

在前面的代码中注释的行（m4 中以`#`开头的行），表示它设置了一个从 init 到守护进程域的转换。这听起来像是我们想要的东西。然而，这两个包含的语句都是宏，我们需要递归地展开它们。我们将从`domain_auto_trans()`开始：

```kt
...
#####################################
# domain_auto_trans(olddomain, type, newdomain)
# Automatically transition from olddomain to newdomain
# upon executing a file labeled with type.
#
define(`domain_auto_trans', `
# Allow the necessary permissions.
domain_trans($1,$2,$3)
# Make the transition occur by default.
type_transition $1 $2:process $3;
')
...
```

这里的注释表明我们正在朝着正确的方向前进；然而，我们需要继续展开宏以进行搜索。根据注释，`domain_trans()`宏只允许发生转换。记住，在 SELinux 中，几乎所有的事情都需要从策略中显式获得权限才能发生，包括类型转换。宏中的最后一个语句就是我们一直在寻找的：

```kt
type_transition $1 $2:process $3;
```

如果你展开这个语句，你会得到：

```kt
type_transition init rild_exec:process rild;
```

这句话传达的意思是，如果你在一个类型为`rild_exec`的文件上执行`exec()`系统调用，并且执行域是 init，那么将子进程的域设置为`rild`。

# 通过 seclabel 显式上下文

设置上下文的另一种选项非常直接。它是通过在 `service` 声明中的 init 脚本中硬编码它们来实现的。在 `service` 声明中，正如我们在第三章（第三章。Android 是奇怪的）中看到的，“Android 是奇怪的”中进行了对 init 语言的修改。新增的一项是 `seclabel`。此选项仅允许 init 明确将服务的上下文更改为 `seclabel` 给定的参数。以下是一个 `adbd` 的示例：

```kt
Service adbd /sbin/adbd
  class core
  socket adbd stream 660 system system
  disabled
  seclabel u:r:adbd:s0
```

那么为什么在某个地方使用动态转换，而在其他地方使用 `seclabel`？答案取决于你从哪里执行。像 `adbd` 这样的东西在 ramdisk 上运行得早，由于 ramdisk 真的没有使用每个文件的标签，因此无法正确设置转换——目标具有相同的上下文。

# 重新标记进程

现在我们已经拥有了动态进程转换的能力，并且需要从 init 脚本中设置套接字上下文。让我们尝试重新标记处于不恰当上下文中的服务。我们可以通过检查以下规则来判断它们是否不恰当：

+   除了 init 之外，不应有任何其他进程处于 init 上下文中

+   不应该在 `init_shell` 域中运行长时间运行的过程

+   zygote 域中除了 zygote 之外不应有任何内容

### 注意

更全面的测试套件是 AOSP 上的 CTS 的一部分。有关更多详细信息，请参阅 Android CTS 项目：（git clone）[`android.googlesource.com/platform/cts`](https://android.googlesource.com/platform/cts)。注意 `./hostsidetests/security/src/android/cts/security/SELinuxHostTest.java` 和 `./tests/tests/security/src/android/security/cts/SELinux.*.java` 测试。

让我们运行一些基本命令并评估我们的 UDOO 通过 `adb` 连接的状态：

```kt
$ adb shell ps -Z | grep init
u:r:init:s0 root 1 0 /init
u:r:init:s0 root 2267 1 /sbin/watchdogd
u:r:init_shell:s0 root 2278 1 /system/bin/sh
$ adb shell ps -Z | grep zygote
u:r:zygote:s0 root 2285 1 zygote

```

我们有两个不恰当的域中的进程。第一个是 `watchdogd`，第二个是一个 `sh` 进程。我们需要找到这些并纠正它们。

我们将从神秘的 `sh` 程序开始。如您在上一章中回忆的那样，我们的 UDOO 串行控制台进程的上下文是 `init_shell`，所以这是一个很好的嫌疑人。让我们检查 PID 并找出。从 UDOO 串行控制台执行：

```kt
root@udoo:/ # echo $$ 
2278

```

我们可以将这个 PID 与这里 `adb shell ps` 输出的 PID 字段进行比较（PID 字段是第三个字段，索引 2），如您所见，我们有一个匹配。

从那里，我们需要找到这个服务的声明。我们知道它在 `init.rc` 中，因为它在 `init_shell` 中运行，这是一种只能由 init 直接转换的类型，根据 SELinux 策略。此外，init 只通过服务声明来处理事物，因此为了在 `init_shell` 中，你必须通过服务声明由 init 启动。

### 注意

使用 `sesearch` 在编译好的 sepolicy 二进制文件中查找此类问题：

```kt
$ sesearch -T -s init -t shell_exec -c process $OUT/root/sepolicy

```

如果我们在 `init.rc` 中搜索 UDOO，它在 `udoo/device/fsl/imx6/etc` 中，我们可以使用 `grep` 搜索 `/system/bin/sh`，即有问题的命令。如果我们这样做，我们会找到：

```kt
$ grep -n "/system/bin/sh" init.rc 
499:service console /system/bin/sh
702:service wifi_mac /system/bin/sh /system/etc/check_wifi_mac.sh

```

让我们看看 `499`，因为我们与 Wi-Fi 没有任何关系：

```kt
service console /system/bin/sh
 class core
 console
 user root
 group root

```

如果这是我们要处理的服务，我们应该能够禁用它，并验证我们的串行连接不再工作：

```kt
$ adb shell setprop ctl.stop console

```

我的实时串行连接在以下位置中断：

```kt
root@udoo:/ # avc: denied { set } for property=ctl.console scontext=u:r:shell:s0 tcontext=u:e

```

现在我们已经确认了问题所在，我们可以重新启动它：

```kt
$ adb shell setprop ctl.start console

```

系统恢复到工作状态后，我们现在需要解决如何正确修正这个服务的标签。我们有两种选择：

+   在 `init.rc` 中使用显式的 `seclabel` 条目

+   使用类型转换

我们在这里将使用第一个选项。原因是 init 会时不时地执行 shell，我们不希望所有这些都在控制台进程域中。我们希望最小权限来隔离运行中的进程。通过使用显式的 seclabel，我们不会改变沿途执行的其他 shell。

要做到这一点，我们需要修改控制台（console）的 `init.rc` 条目；添加：

```kt
service console /system/bin/sh
 class core
 console
 user root
 group root
 seclabel u:r:shell:s0

```

这个可执行文件的正确域是 `shell`，因为它应该具有与 `adb shell` 相同的权限集。在做出这个更改后，重新编译引导镜像（bootimage），刷写（flash），然后重启。我们可以看到它现在位于 shell 域中。为了验证，从 UDOO 串行连接执行以下命令：

```kt
root@udoo:/ # id -Z 
uid=0(root) gid=0(root) context=u:r:shell:s0

```

或者，使用 `adb` 执行以下命令：

```kt
$ adb shell ps -Z | grep "system/bin/sh"
u:r:shell:s0 root 2279 1 /system/bin/sh

```

我们接下来需要处理的是 `watchdogd`。`watchdogd` 进程已经有一个域，并且允许在 `watchdog.te` 中设置规则；所以我们只需要添加一个 `seclabel` 语句并将其放入这个正确的域中。修改 `init.rc`：

```kt
# Set watchdog timer to 30 seconds and pet it every 10 seconds to get a 20 second margin
service watchdogd /sbin/watchdogd 10 20
  class core
  seclabel u:r:watchdogd:s0
```

要使用 `adb` 进行验证，执行以下命令：

```kt
$ adb shell ps -Z | grep watchdog
u:r:watchdogd:s0 root 2267 1 /sbin/watchdogd

```

到目前为止，我们已经对 UDOO 所需的实际策略进行了修正。然而，我们需要练习动态域转换的使用。一个好的教学示例将会有来自其自身域的子 shell。让我们首先定义一个新的域并设置转换。

我们将在 `sepolicy` 中创建一个新的 `.te` 文件，命名为 `subshell.te`，并编辑它，使其内容包含以下内容：

```kt
type subshell, domain, shelldomain, mlstrustedsubject;
# domain_auto_trans(olddomain, type, newdomain)
# Automatically transition from olddomain to newdomain
# upon executing a file labeled with type.
#
domain_auto_trans(shell, shell_exec, subshell)

```

现在，书中之前使用过的 `mmm` 技巧可以用来编译策略。此外，使用 `adb push` 命令将新策略推送到 `/data/security/current/sepolicy`，并执行 `setprop` 来重新加载策略，就像我们在第八章中做的那样，*将上下文应用于文件*。

为了测试这一点，我们应该能够输入 `sh` 并验证域转换。我们将首先获取当前上下文：

```kt
root@udoo:/ # id -Z
uid=0(root) gid=0(root) context=u:r:shell:s0

```

然后通过以下方式执行一个 shell：

```kt
root@udoo:/ # sh
root@udoo:/ # id -Z
uid=0(root) gid=0(root) context=u:r:subshell:s0

```

我们能够使用动态类型转换将一个新的进程放入一个域中。如果你结合使用标签文件，如第八章中所述，*将上下文应用于文件*，你将拥有一个强大的工具来控制进程权限。

# 应用标签的限制

这些动态进程转换的一个基本限制是它们需要执行一个`exec()`系统调用。只有在这种情况下，SELinux 才能计算新的域，并触发上下文切换。另一种实现这一点的唯一方法是通过修改代码，这本质上就是当你指定`seclabel()`时 init 所做的事情。init 代码为其进程设置执行上下文，导致下一个`exec`最终进入指定的域。实际上，我们可以在`init.c`代码中看到这一点：

```kt
if (svc->seclabel) {
if (is_selinux_enabled() > 0 && setexeccon(svc->seclabel) < 0) {
ERROR("cannot setexeccon('%s'): %s\n", svc->seclabel, strerror(errno));
_exit(127);
}
}
```

在这里，子进程在`exec()`系统调用将控制权交给新的二进制镜像之前，通过调用`setexeccon()`来设置其执行上下文。在 Android 中，应用程序不是以这种方式产生的，并且在进程创建路径中不存在`exec()`系统调用；因此需要一个新的机制。

# 摘要

在本章中，我们学习了如何通过类型转换以及通过`seclabel`语句来标记进程。我们还研究了 init 如何管理服务套接字，以及如何正确地标记它们。然后我们纠正了串行控制台以及看门狗守护进程的进程上下文。

Android 中的应用程序永远不会有一个显式的`exec()`调用来启动它们的程序执行。由于没有`exec()`，我们必须通过代码更改来标记应用程序。在下一章中，我们将讨论这是如何发生的，以及应用程序是如何被标记的。
