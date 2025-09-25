# 第一章 Linux 访问控制

Android 是由两个不同组件组成的操作系统。第一个组件是一个分支的主线 Linux 内核，与 Linux 几乎共享所有共同点。第二个组件，稍后将会讨论，是用户空间部分，这部分非常定制且具有 Android 特性。由于 Linux 内核支撑这个系统并负责大多数访问控制决策，因此这是开始详细研究 Android 的合乎逻辑的地方。

在本章中，我们将：

+   检查离散访问控制的基本知识

+   介绍 Linux 权限标志和能力

+   在验证访问策略时跟踪系统调用

+   为更健壮的访问控制技术辩护

+   讨论利用离散访问控制问题的 Android 漏洞

Linux 的默认和熟悉的访问控制机制被称为 **离散访问控制** (**DAC**)。这只是一个表示关于对象访问权限由其创建者/所有者决定的术语。

在 Linux 中，当进程调用大多数系统调用时，会执行权限检查。例如，一个希望打开文件的进程会调用 `open()` 系统调用。当这个系统调用被调用时，会执行上下文切换，操作系统代码被执行。操作系统有能力确定是否应该将文件描述符返回给请求的进程。在决策过程中，操作系统会检查请求进程和目标文件（它希望获取文件描述符）的访问权限。根据权限检查是否通过，返回文件描述符或 EPERM。

Linux 在内核中维护数据结构来管理这些权限字段，这些字段可以从用户空间访问，并且对于 Linux 和 *NIX 用户来说应该是熟悉的。第一组访问控制元数据属于进程，并构成了其凭证集的一部分。常见的凭证是用户和组。一般来说，我们使用“组”一词来指代主要组以及可能的次要组。您可以通过运行 `ps` 命令来查看这些权限：

```kt
$ ps -eo pid,comm,user,group,supgrp
PID COMMAND         USER     GROUP    SUPGRP
1 init            root     root     -
...
 2993 system-service- root     root     root 
 3276 chromium-browse bookuser sudo fuse bookuser 
...

```

如您所见，我们正在以用户 `root` 和 `bookuser` 运行进程。您还可以看到，它们的主要组只是方程的一部分。进程还有一个称为补充组的次要组集。这个集合可能是空的，这在 `SUPGRP` 字段中的破折号表示。

我们希望打开的文件，称为目标对象、目标或对象，也维护了一组权限。对象维护 `USER` 和 `GROUP`，以及一组权限位。在目标对象的上下文中，`USER` 可以被称为 *所有者* 或 *创建者*。

```kt
$ ls -la
total 296
drwxr-xr-x 38 bookuser bookuser  4096 Aug 23 11:08 .
drwxr-xr-x  3 root     root      4096 Jun  8 18:50 ..
-rw-rw-r--  1 bookuser bookuser   116 Jul 22 13:13 a.c
drwxrwxr-x  4 bookuser bookuser  4096 Aug  4 16:20 .android
-rw-rw-r--  1 bookuser bookuser   130 Jun 19 17:51 .apport-ignore.xml
-rw-rw-r--  1 bookuser bookuser   365 Jun 23 19:44 hello.txt
-rw-------  1 bookuser bookuser 19276 Aug  4 16:36 .bash_history
...

```

如果我们查看前一个命令的输出，我们可以看到`hello.txt`的`USER`是`bookuser`，`GROUP`也是`bookuser`。我们还可以看到输出左边的权限位或标志。同时，我们还需要考虑七个字段。每个空字段都用破折号表示。使用`ls`打印时，第一个字段可能会因为语义而变得复杂。因此，让我们使用`stat`来调查文件权限：

```kt
$ stat hello.txt
 File: `hello.txt'
 Size: 365         Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d  Inode: 1587858     Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/bookuser)   Gid: ( 1000/bookuser)
Access: 2014-08-04 15:53:01.951024557 -0700
Modify: 2014-06-23 19:44:14.308741592 -0700
Change: 2014-06-23 19:44:14.308741592 -0700
 Birth: -

```

第一条访问行是最有说服力的。它包含了访问控制的所有重要信息。第二条行只是一个时间戳，告诉我们文件最后一次被访问的时间。正如我们所见，对象的`USER`或`UID`是`bookuser`，`GROUP`也是`bookuser`。权限标志（`0664/-rw-rw-r--`）标识了权限标志的两种表示方式。第一种是八进制形式`0664`，将每个三位标志字段压缩成三个基数 8（八进制）数字中的一个。第二种是友好形式，`-rw-rw-r--`，与八进制形式等效，但更容易直观理解。在任何情况下，我们都可以看到最左边的字段是 0，我们接下来的讨论将忽略它。该字段用于`setuid`和`setgid`功能，对于这次讨论并不重要。如果我们将剩余的八进制数字 664 转换为二进制，我们得到 110 110 100。这种二进制表示直接与友好形式相关。每个三元组对应于读取、写入和执行权限。通常你会看到这个权限三元组表示为`RWX`。第一个三元组是赋予`USER`的权限，第二个是赋予`GROUP`的权限，第三个是赋予`OTHERS`的权限。用常规英语翻译就是，“用户`bookuser`有权从`hello.txt`读取和写入。组`bookuser`有权从`hello.txt`读取和写入，而其他人只有权从`hello.txt`读取。”让我们用一些现实世界的例子来测试这一点。

# 修改权限位

让我们在以`bookuser`用户身份运行的示例进程中测试访问控制。大多数进程都是在调用它们的用户上下文中运行的（不包括`setuid`和`getuid`程序），因此我们调用的任何命令都应该继承我们用户的权限。我们可以通过以下命令查看：

```kt
$ groups bookuser
bookuser : bookuser sudo fuse

```

我的用户`bookuser`是`USER bookuser`，`GROUP bookuser`，`SUPGRP sudo`和`fuse`。

要测试读取访问，我们可以使用`cat`命令，该命令打开文件并将内容打印到`stdout`：

```kt
$ cat hello.txt 
Hello, "Exploring SE for Android"
Here is a simple text file for
your enjoyment.
...

```

我们可以通过运行`strace`命令并查看输出来检查系统调用：

```kt
$ strace cat hello.txt 
...
open("hello.txt", O_RDONLY)                   = 3
...
read(3, "Hello, \"Exploring SE for Android\"\n"..., 32768) = 365
...

```

输出可能非常冗长，所以我只显示相关部分。我们可以看到`cat`调用了`open`系统调用并获得了文件描述符`3`。我们可以使用该描述符通过其他系统调用找到其他访问。稍后我们将看到在文件描述符`3`上发生读取操作，它返回`365`，这是读取的字节数。如果我们没有权限从`hello.txt`读取，打开操作将失败，我们永远不会为该文件获得有效的文件描述符。我们还会在`strace`输出中看到失败。

现在读取权限已验证，让我们尝试写入。一种简单的方法是编写一个简单的程序，将某些内容写入现有文件。在这种情况下，我们将写入行`my new text\n`（参考`write.c`）。

使用以下命令编译程序：

```kt
$ gcc -o mywrite write.c

```

现在使用新编译的程序运行：

```kt
$ strace ./mywrite hello.txt

```

验证后，您将看到：

```kt
...
open("hello.txt", O_WRONLY)                   = 3
write(3, "my new text\n", 12)           = 12
...

```

如您所见，写入操作成功并返回`12`，这是写入`hello.txt`的字节数。没有报告错误，所以到目前为止权限似乎没问题。

现在我们尝试执行`hello.txt`并看看会发生什么。我们预计会看到一个错误。让我们像正常命令一样执行它：

```kt
$ ./hello.txt
bash: ./hello.txt: Permission denied

```

这正是我们预期的，但让我们用`strace`来调用它，以深入了解失败的原因：

```kt
$ strace ./hello.txt
...
execve("./hello.txt", ["./hello.txt"], [/* 39 vars */]) = -1 EACCES (Permission denied)
...

```

启动进程的`execve`系统调用因`EACCESS`失败。这就是在没有执行权限时人们希望看到的情况。Linux 访问控制按预期工作！

让我们在另一个用户的上下文中测试访问控制。首先，我们将使用`adduser`命令创建一个名为`testuser`的新用户：

```kt
$ sudo adduser testuser
[sudo] password for bookuser: 
Adding user `testuser' ...
Adding new group `testuser' (1001) ...
Adding new user `testuser' (1001) with group `testuser' ...
Creating home directory `/home/testuser' ...
...

```

验证`testuser`的`USER`、`GROUP`和`SUPGRP`：

```kt
$ groups testuser
testuser : testuser

```

由于`USER`和`GROUP`与`a.S`上的任何权限都不匹配，所有访问都将受到`OTHERS`权限检查的影响，如果您还记得，这是只读的（`0664`）。

首先临时以`testuser`的身份工作：

```kt
$ su testuser
Password: 
testuser@ubuntu:/home/bookuser$ 

```

如您所见，我们仍然在`bookuser`的主目录中，但当前用户已更改为`testuser`。

我们将首先使用`cat`命令测试`read`：

```kt
$ strace cat hello.txt
...
open("hello.txt", O_RDONLY)                   = 3
...
read(3, "my new text\n", 32768)         = 12
...

```

与早期示例类似，`testuser`可以很好地读取数据，正如预期的那样。

现在让我们继续编写。预期的是，如果没有适当的访问权限，这将失败：

```kt
$ strace ./mywrite hello.txt
...
open("hello.txt", O_WRONLY)                   = -1 EACCES (Permission denied)
...

```

如预期的那样，系统调用操作失败了。当我们尝试以`testuser`身份执行`hello.txt`时，这也应该会失败：

```kt
$ strace ./hello.txt
...
execve("./hello.txt", ["./hello.txt"], [/* 40 vars */]) = -1 EACCES (Permission denied)
...

```

现在我们需要测试组访问权限。我们可以通过向`testuser`添加一个补充组来完成此操作。为此，我们需要退出到`bookuser`，他有权执行`sudo`命令：

```kt
$ exit
exit
$ sudo usermod -G bookuser testuser

```

现在我们检查`testuser`的组：

```kt
$ groups testuser
testuser : testuser bookuser

```

由于之前的`usermod`命令，`testuser`现在属于两个组：`testuser`和`bookuser`。这意味着当`testuser`使用`bookuser`组访问文件或其他对象（如套接字）时，将应用`GROUP`权限，而不是`OTHERS`权限。在`hello.txt`的上下文中，`testuser`现在可以读取和写入文件，但不能执行它。

通过执行以下命令切换到`testuser`：

```kt
$ su testuser

```

通过执行以下命令来测试`read`：

```kt
$ strace cat ./hello.txt
...
open("./hello.txt", O_RDONLY)                 = 3
...
read(3, "my new text\n", 32768)         = 12
...

```

如前所述，`testuser`能够读取文件。唯一的区别是它现在可以通过`OTHERS`和`GROUP`的访问权限来`read`文件。

通过执行以下命令测试`write`：

```kt
$ strace ./mywrite hello.txt
...
open("hello.txt", O_WRONLY)                   = 3
write(3, "my new text\n", 12)           = 12
...

```

这次，`testuser`能够写入文件，而不是像之前那样因为`EACCESS`权限错误而失败。

尝试执行文件应该仍然失败：

```kt
$ strace ./hello.txt
execve("./hello.txt", ["./hello.txt"], [/* 40 vars */]) = -1 EACCES (Permission denied)
...

```

这些概念是 Linux 访问控制权限位、用户和组的基础。

# 更改所有者和组

在上一节中使用`hello.txt`进行探索性工作，我们已经展示了对象的所有者如何通过管理对象的权限位来允许各种形式的访问。更改权限是通过使用`chmod`系统调用完成的。更改用户和/或组是通过使用`chown`系统调用完成的。在本节中，我们将调查这些操作的具体细节。

让我们首先仅授予`hello.txt`文件的所有者`bookuser`读取和写入权限。

```kt
$ chmod 0600 hello.txt
$ stat hello.txt
 File: `hello.txt'
 Size: 12          Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d  Inode: 1587858     Links: 1
Access: (0600/-rw-------)  Uid: ( 1000/bookuser)   Gid: ( 1000/bookuser)
Access: 2014-08-23 12:34:30.147146826 -0700
Modify: 2014-08-23 12:47:19.123113845 -0700
Change: 2014-08-23 12:59:04.275083602 -0700
 Birth: -

```

如我们所见，文件权限现在设置为仅允许`bookuser`读取和写入访问。一个彻底的读者可以执行本章早期部分的命令来验证权限是否按预期工作。

使用`chown`也可以以类似的方式更改组。让我们将组更改为`testuser`：

```kt
$ chown bookuser:testuser hello.txt
chown: changing ownership of `hello.txt': Operation not permitted

```

这并没有按我们的意图工作，但问题是什么？在 Linux 中，只有特权进程可以更改对象的`USER`和`GROUP`字段。初始的`USER`和`GROUP`字段是在对象创建时从有效的`USER`和`GROUP`设置的，这些在尝试执行该进程时进行检查。只有进程可以创建对象。特权进程有两种形式：作为全能的`root`运行的进程和那些设置了其能力设置的进程。我们将在稍后深入了解能力。现在，让我们专注于`root`。

让我们将用户更改为`root`以确保执行`chown`命令将更改该对象的组：

```kt
$ sudo su
# chown bookuser:testuser hello.txt 
Now, we can verify the change occurred successfully:
# stat hello.txt
 File: `hello.txt'
 Size: 12          Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d  Inode: 1587858     Links: 1
Access: (0600/-rw-------)  Uid: ( 1000/bookuser)   Gid: ( 1001/testuser)
Access: 2014-08-23 12:34:30.147146826 -0700
Modify: 2014-08-23 12:47:19.123113845 -0700
Change: 2014-08-23 13:08:46.059058649 -0700
 Birth: -

```

# 对于更多

你可以看到`GROUP`（`GID`）现在是`testuser`，看起来相当安全，因为要更改对象的用户和组，你需要有特权。你只能在拥有对象的情况下更改对象的权限位，除非是`root`用户。这意味着如果你以`root`身份运行，你可以对系统做任何你想做的事情，即使没有权限。这种绝对权力是为什么在`root`运行的进程上的成功攻击或错误可能导致系统严重损坏。此外，对非`root`进程的成功攻击也可能通过意外更改权限位造成损害。例如，假设你的 SSH 私钥上有未预期的`chmod 0666`命令。这将使你的秘密密钥暴露给系统上的所有用户，这几乎肯定不是你希望发生的事情。能力模型部分解决了根限制的问题。

# 能力模型

对于 Linux 上的许多操作，对象权限模型并不完全适用。例如，更改`UID`和`GID`需要一些被称为`root`的神奇`USER`。假设你有一个需要利用这些功能的长运行服务。也许这个服务会监听内核事件并为你创建设备节点？这样的服务是存在的，它被称为`ueventd`或用户事件守护进程。这个守护进程传统上以`root`身份运行，这意味着如果它被入侵，它可能会从你的家目录中读取你的私钥并将其发送给攻击者。这或许是一个非凡的例子，但它的目的是展示以`root`身份运行进程可能会很危险。假设你可以以`root`用户启动一个服务，并且进程可以将其`UID`和`GID`更改为非特权值，但保留执行其工作所需的一小部分特权能力？这正是 Linux 中能力模型的目的。

Linux 中的能力模型是尝试将`root`拥有的权限集分解成更小的子集。这样，进程可以被限制在执行其预期功能所需的最小权限集。这被称为最小权限，是确保系统安全的关键理念，它最大限度地减少了成功攻击可能造成的损害。在某些情况下，它甚至可以通过阻止其他情况下开放的攻击向量来防止成功攻击的发生。

有许多能力。关于能力的 man 页是事实上的文档。让我们看看`CAP_SYS_BOOT`能力：

```kt
$ man capabilities
...
CAP_SYS_BOOT
 Use reboot(2) and kexec_load(2).

```

这意味着具有这种能力的进程可以重新启动系统。然而，该进程不能像作为`root`或带有`CAP_DAC_READ_SEARCH`运行时那样任意更改`USERS`和`GROUP`。这限制了攻击者能做的事情：

```kt
<FROM MAN PAGE>
CAP_DAC_READ_SEARCH
 Bypass file read permission checks and directory read and execute permission checks.

```

现在假设我们的重启过程以`CAP_CHOWN`运行。假设它使用这种能力在收到重启请求之前，将每个用户的家目录中的文件备份到服务器上，然后再重启。假设这个文件是`~/backup`，权限为 0600，`USER`和`GROUP`分别是该家目录的相应用户。在这种情况下，我们已经尽可能地最小化了权限，但进程仍然可以访问用户的 SSH 密钥，并可能由于错误或攻击而上传这些密钥。另一种方法是设置组为`backup`，并以`GROUP backup`运行进程。然而，这也有局限性。假设你想与另一个用户共享这个文件。那个用户需要`backup`的附加组，但现在该用户可以读取*所有*的备份文件，而不仅仅是那些预期的文件。一个敏锐的读者可能会想到`bind`挂载，然而执行`bind`挂载和文件权限的进程也运行在某些能力下，因此也受到这种粒度问题的困扰。

主要问题，以及另一个访问控制系统的案例可以总结为一个词，*粒度*。DAC 模型没有所需的粒度来安全地处理复杂的访问控制模型或最小化进程可能造成的损害。这在 Android 上尤为重要，因为整个隔离系统都依赖于这种控制，而一个恶意 root 进程可以破坏整个系统。

# Android 对 DAC 的使用

在 Android 沙箱模型中，每个应用程序都以自己的`UID`运行。这意味着每个应用程序都可以将其存储的数据与其他应用程序分开。用户和组被设置为该应用程序的`UID`和`GID`，因此没有应用程序可以访问另一个应用程序的私有文件，除非该应用程序明确对其对象执行`chmod`。此外，Android 中的应用程序不能具有能力，所以我们不必担心像`CAP_SYS_PTRACE`这样的能力，这是调试另一个应用程序的能力。在 Android 中，在理想的世界里，只有系统组件运行有权限，应用程序不会意外地将私有文件`chmod`为所有用户可读。由于应用程序兼容性问题，当前 AOSP SELinux 策略没有解决这个问题，但可以通过 SELinux 来解决。在 Android 上，在应用程序之间共享数据的正确方式是通过 binder 和共享文件描述符。对于较小的数据量，提供者模型就足够了。

# 快速浏览 Android 漏洞

在我们对 DAC 权限模型及其一些限制的新理解基础上，让我们看看一些针对它的 Android 漏洞。我们将只涵盖几个漏洞，以了解 DAC 模型是如何失败的。

## Skype 漏洞

CVE-2011-1717 于 2011 年发布。在这个漏洞利用中，Skype 应用程序留下了一个 SQLite3 数据库是可读的（类似于 0666 权限）。这个数据库包含用户名和聊天记录，以及姓名和电子邮件等个人信息。一个名为 Skypwned 的应用程序能够展示这种能力。这是一个例子，说明了能够更改对象权限可能会很糟糕，尤其是当情况允许 `READ` 权限给 `OTHERS` 时。

## GingerBreak

CVE-2011-1823 展示了对 Android 的根攻击。Android 上的卷管理守护进程（vold）负责外部 SD 卡的挂载和卸载。守护进程通过 NETLINK 套接字监听消息。守护进程从未检查消息的来源，任何应用程序都可以打开并创建一个 NETLINK 套接字向 vold 发送消息。一旦攻击者打开了 NETLINK 套接字，他们就会发送一个精心设计的消息来绕过健全性检查。该检查测试了一个有符号整数是否超出最大界限，但从未检查它是否为负数。然后它被用来索引一个数组。这种负访问会导致内存损坏，并且通过适当的消息，可能会导致任意代码的执行。GingerBreak 实现导致任意用户获得 root 权限，这是一个教科书式的权限执行攻击。一旦获得 root 权限，设备的沙盒就不再有效。

## 愤怒的笼子

CVE-2010-EASY 是一种通过 fork bomb 攻击的 `setuid` 耗尽攻击。它成功攻击了 Android 上的 `adb` 守护进程，该守护进程以 root 权限启动，如果不需要 root 权限，则降低其权限。这种攻击使 `adb` 保持 `root` 权限，并将 root shell 返回给用户。在 Linux 内核 2.6 中，当达到运行进程数 `RLIMIT_NPROC` 时，`setuid` 系统调用返回错误。`adb` 守护进程代码没有检查 `setuid` 的返回值，这为攻击者留下了一个小的竞争窗口。攻击者需要足够多的进程来达到 `RLIMIT_NPROC` 并杀死守护进程。`adb` 守护进程降级到 shell `UID`，攻击者以 shell `USER` 运行程序，因此杀死操作将成功。此时，`adb` 服务将被重新启动，如果 `RLIMIT_NPROC` 达到最大值，`setuid` 将失败，`adb` 将以 root 权限保持运行。然后，从主机运行 `adb` shell 将返回一个漂亮的 root shell 给用户。

## MotoChopper

CVE-2013-2596 是高通视频驱动程序 `mmap` 功能中的一个漏洞。应用程序通过提供对 GPU 的访问来执行高级图形渲染，例如 OpenGL 调用的情况。`mmap` 中的漏洞允许攻击者 `mmap` 内核地址空间，此时攻击者能够直接更改他们的内核凭证结构。这个漏洞利用是一个 DAC 模型没有错误的例子。实际上，除了修补代码或移除直接图形访问之外，没有其他方法可以防止这种攻击。

# 摘要

DAC 模型非常强大，但其缺乏精细粒度和使用极其强大的`root`用户权限仍有待改进。随着移动电话使用的日益敏感，提高系统安全性的理由充分。幸运的是，Android 是基于 Linux 构建的，因此受益于庞大的工程师和研究人员生态系统。自 Linux 内核 2.6 以来，增加了一个名为**强制访问控制（MAC）**的新访问控制模型。这是一个框架，通过它可以加载模块到内核中，以提供一种新的访问控制模型。第一个模块被称为 SELinux。它被 Red Hat 和其他公司用于保护敏感的政府系统。因此，找到了一种方法，可以为 Android 启用此类访问控制。
