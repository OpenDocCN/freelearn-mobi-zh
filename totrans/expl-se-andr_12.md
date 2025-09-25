# 第十二章. 掌握工具链

到目前为止，我们已经深入研究了驱动 Android SE 技术的代码和政策，但构建系统和工具通常被忽视。掌握工具链将有助于你改进你的开发实践。在本章中，我们将查看 Android SE 构建的各个组件以及它们是如何工作的。我们将涵盖以下主题：

+   构建特定目标

+   sepolicy 的 `Android.mk` 文件

+   自定义构建策略配置

+   构建工具：

    +   `check_seapp`

    +   `insertkeys.py`

    +   `checkpolicy`

    +   `checkfc`

    +   `sepolicy-check`

    +   `sepolicy-analyze`

# 构建子组件 – 目标和项目

到目前为止，我们已经运行了一些魔法命令，例如 `mm`、`mmm` 和 `make bootimage`，实际上构建了 Android SE 代码的各个部分。谷歌在[`source.android.com/source/building-running.html`](https://source.android.com/source/building-running.html)的文档中官方描述了一些这些工具，但大多数命令并未列出。尽管如此，[`elinux.org/Android_Build_System`](http://elinux.org/Android_Build_System)有一篇更全面的概述。

在谷歌的“构建和运行”文档中，他们描述目标为设备，这最终是你启动的目标。在构建 Android 时，`lunch` 命令为随后执行的 `make` 命令设置环境变量。它设置构建系统以输出目标设备的正确配置。这种目标的概念*并不是*本章将要讨论的内容。相反，当在此处提到“目标”时，它指的是一个特定的 `make` 目标。然而，在需要提及目标设备的情况下，将使用完整的短语“目标设备”。虽然有些令人困惑，但这种术语是标准的，并且将被该领域的工程师所理解。

我们已经多次发出 `make` 命令，可选地提供一个目标作为参数和一个选项，例如 `-j16` 选项。类似于 `make` 或 `make -j16` 的命令实际上构建了整个 Android。你可以指定一个目标或目标列表作为命令参数。一个例子是构建 `boot.img`。可以通过指定 `bootimage` 目标来构建和重新构建 `boot.img` 文件。我们用于此目的的命令是 `make bootimage`。它通过仅重新构建所需的系统部分来加速构建。但如果你只需要重新构建特定的文件怎么办？也许，你只想重新构建 `sepolicy`。你可以指定该目标来构建，就像 `make sepolicy` 一样。这引出了一个问题，“关于其他文件，如 `mac_permissions.xml`、`seapp_contexts` 等怎么办？”它们也可以以相同的方式构建。更引人入胜的问题是，“一个人如何知道目标名称是什么？它是否总是文件输出名称？”

Android 的构建系统建立在 GNU `make`([`www.gnu.org/software/make/`](http://www.gnu.org/software/make/))之上。Android 构建系统 makefiles 系统的核心可以在`build/core`中找到，文档可以在 NDK([`developer.android.com/tools/sdk/ndk/index.html`](https://developer.android.com/tools/sdk/ndk/index.html))中找到。从阅读中可以得出的主要结论是，典型的`Android.mk`文件定义了一个名为`LOCAL_MODULE := mymodulename`的东西，并且会构建名为`mymodulename`的东西。目标名称由这些`LOCAL_MODULE`语句定义。让我们看看外部 sepolicy 的`Android.mk`，并关注其中的 sepolicy 部分，因为在该`Makefile`中定义了其他本地模块或目标。以下是从 Android 4.3 的一个示例：

```kt
include $(CLEAR_VARS)
LOCAL_MODULE := sepolicy
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_PATH := $(TARGET_ROOT_OUT)
...
```

可以通过查找以`LOCAL_MODULE`声明开始的行，并且是整个单词匹配的行，在`Android.mk`文件中找到所有模块：

```kt
$ grep -w '^LOCAL_MODULE' Android.mk
LOCAL_MODULE := sepolicy
LOCAL_MODULE := file_contexts
LOCAL_MODULE := seapp_contexts
LOCAL_MODULE := property_contexts
LOCAL_MODULE := selinux-network.sh
LOCAL_MODULE := mac_permissions.xml
LOCAL_MODULE := eops.xml

```

正则表达式规定`^`是行的开始，`grep`手册页说明`-w`提供整个单词搜索。

上述列表适用于我们在 UDOO 上使用的 Android 版本。然而，你应该在你的确切版本的`Makefile`上运行该命令，以了解可以构建什么内容。

Android 有一些额外的工具，它们与构建目标分开，当你使用`source build/envsetup.sh`时，它们会被添加到你的环境中。这些是`mm`和`mmm`。它们都执行相同的任务，即构建`Android.mk`文件中指定的所有目标，但它们的不同之处在于它们不构建任何依赖项。这两个命令仅在它们源`Android.mk`的位置以搜索构建目标方面有所不同。`mm`命令使用当前工作目录，而`mmm`使用提供的路径。对于这两个命令来说，一个很好的选项是`-B`，它强制重新构建。工程师可以通过使用`mm(m)`命令而不是`make <target>`来节省大量时间。完整的`make`命令会浪费很多时间来确定依赖关系树，所以如果你知道所有的更改都在一个项目中，执行`mmm path/to/project`在先前构建的源树中可以节省几分钟。然而，由于它不构建依赖项，你需要确保它们已经构建并且没有依赖的更改。

# 探索 sepolicy 的 Android.mk

位于`external/sepolicy`的项目使用一个`Android.mk`文件，就像任何其他 Android 项目一样，来构建它们的输出。让我们分析这个文件，看看它做了什么。

## 构建 sepolicy

我们将从中间开始，查看`sepolicy`的目标。它开始于相当标准的`Android.mk`内容：

```kt
...
include $(CLEAR_VARS)
LOCAL_MODULE := sepolicy
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_PATH := $(TARGET_ROOT_OUT)
include $(BUILD_SYSTEM)/base_rules.mk
...
```

下一个部分更像是标准的`make`。它首先声明一个目标文件，该文件将被构建到`intermediates`位置。`intermediates`位置由 Android 构建系统定义。然后，它将`MLS_SENS`和`MLS_CATS`的值分配给一些局部变量以供以后使用。最后一行是最有趣的。它使用一个名为`build_policy`的`make`函数，并接受文件名作为参数：

```kt
...
sepolicy_policy.conf := $(intermediates)/policy.conf
$(sepolicy_policy.conf): PRIVATE_MLS_SENS := $(MLS_SENS)
$(sepolicy_policy.conf): PRIVATE_MLS_CATS := $(MLS_CATS)
$(sepolicy_policy.conf) : $(call build_policy, security_classes initial_sids access_vectors global_macros mls_macros mls policy_capabilities te_macros attributes bools *.te roles users initial_sid_contexts fs_use genfs_contexts port_contexts)
...
```

接下来，我们定义构建这个中间目标`policy.conf`的配方。配方中有趣的部分是`m4`命令和`sed`命令。

### 注意

更多关于`m4`的信息，请参阅[`www.gnu.org/software/m4/manual/m4.html`](http://www.gnu.org/software/m4/manual/m4.html)，更多关于`sed`的信息，请参考[`www.gnu.org/software/sed/manual/sed.html`](https://www.gnu.org/software/sed/manual/sed.html)。

SELinux 策略文件使用`m4`进行处理。`m4`是一种宏处理器语言，通常用作编译器的前端。`m4`命令取一些值，如`PRIVATE_MLS_SENS`和`PRIVATE_MLS_CATS`，并将它们作为宏定义传递。这类似于`gcc -D`选项。然后，它通过`make`展开`$^`将目标依赖项作为输入，并使用`make`展开`$@`将它们输出到目标名称。它还使用该输出生成一个`.dontaudit`版本。该版本从策略文件中删除了所有的`dontaudit`行，使用`sed`完成。MLS 值告诉 SELinux 要生成多少类别和敏感性。这些必须在加载到内核的策略 blob 中静态定义，如下所示：

```kt
...
@mkdir -p $(dir $@)
$(hide) m4 -D mls_num_sens=$(PRIVATE_MLS_SENS) -D mls_num_cats=$(PRIVATE_MLS_CATS) -s $^ > $@
$(hide) sed '/dontaudit/d' $@ > $@.dontaudit
...
```

下一个部分定义了构建实际目标的配方，名为`LOCAL_MODULE_POLICY`，即使这并不明显。`LOCAL_BUILT_MODULE`展开为要构建的中间文件，在这种情况下是`sepolicy`。最终，它由 Android 构建系统作为`LOCAL_INSTALLED_MODULE`在幕后复制。此目标依赖于中间的`policy.conf`文件和`checkpolicy`。它使用`checkpolicy`将`m4`展开的`policy.conf`和`policy.conf.dontaudit`转换为两个 sepolicy 文件，即`sepolicy`和`sepolicy.dontaudit`。实际用于将 SELinux 语句编译成二进制形式以加载到内核的工具是`checkpolicy`，如下所示：

```kt
...
$(LOCAL_BUILT_MODULE) : $(sepolicy_policy.conf) $(HOST_OUT_EXECUTABLES)/checkpolicy
@mkdir -p $(dir $@)
$(hide) $(HOST_OUT_EXECUTABLES)/checkpolicy -M -c $(POLICYVERS) -o $@ $<
$(hide) $(HOST_OUT_EXECUTABLES)/checkpolicy -M -c $(POLICYVERS) -o $(dir $<)/$(notdir $@).dontaudit $<.dontaudit
...
```

最后，它通过设置一个局部变量`built_policy`用于在`Android.mk`文件中的其他地方使用，并清除`policy.conf`以避免污染`make`的全局命名空间，如下所示：

```kt
...
built_sepolicy := $(LOCAL_BUILT_MODULE)
sepolicy_policy.conf :=
...
```

此外，构建`sepolicy`还依赖于`POLICYVERS`变量，如果未设置，则条件性地分配值为`26`。这是`checkpolicy`使用的策略版本号，正如我们在本书前面所看到的，我们不得不为我们的 UDOO 覆盖这个值。

## 控制策略构建

我们看到 `sepolicy` 语句调用了 `build_policy` 函数。我们还看到它在 `Android.mk` 文件中用于构建 `sepolicy`、`file_contexts`、`seapp_contexts`、`property_contexts` 和 `mac_permissions.xml`，因此可以推断它相当重要。此函数输出用于策略文件的完全解析的路径列表。该函数接受一个可变参数列表的文件名，并支持正则表达式（注意 `build_policy` 中的目标 sepolicy 中的 `*.te`）。内部，该函数使用一些魔法来允许你覆盖或追加当前策略构建，而无需直接修改 `external/sepolicy` 目录。这是为了使原始设备制造商和设备构建商能够增强策略以覆盖其特定设备。

当构建策略时，你可以在设备的 `Makefile` 中设置以下 `make` 变量，以控制生成的构建。变量如下：

+   `BOARD_SEPOLICY_DIRS`：这是潜在策略文件的搜索路径

+   `BOARD_SEPOLICY_UNION`：这是一个名为的策略文件，用于追加到所有同名文件

+   `BOARD_SEPOLICY_REPLACE`：这是一个策略文件，用于覆盖基本 `external/sepolicy` 策略文件

+   `BOARD_SEPOLICY_IGNORE`：这是用于从构建中删除特定策略文件的，给定存储库的相对路径

以 UDOO 为例，编写策略的正确方法永远不是修改 `external/sepolicy`，而是在 `device/fsl/udoo/sepolicy` 中创建一个目录：

```kt
$ mkdir <PATH>

```

然后我们修改 `BoardConfig.mk`：

```kt
$ vim BoardConfig.mk

```

接下来，我们添加以下行：

```kt
BOARD_SEPOLICY_DIRS += device/fsl/udoo/sepolicy
```

### 小贴士

在使用 `+=` 而不是 `:=` 时要非常小心。在大型项目树中，这些变量可能由常见的 `BoardConfigs` 在构建树中设置得更高，你可能会清除它们的设置。通常，最安全的做法是使用 `+=`。有关更多详细信息，请参阅 GNU make 手册中的 *变量赋值* 部分，网址为 [`www.gnu.org/software/make/manual/make.html`](http://www.gnu.org/software/make/manual/make.html)。

这将告诉 `Android.mk` 中的 `build_policy()` 函数不仅要在 `external/sepolicy` 中搜索策略文件，还要在 `device/fsl/udoo/sepolicy` 中搜索。

接下来，我们可以在该目录中创建一个 `file_contexts` 文件，并通过在该目录中创建一个新的 `file_contexts` 文件来将我们的标签更改移动到 `device/fsl/udoo/sepolicy`。

之后，我们需要指导构建系统将我们的 `file_contexts` 文件与 `external/sepolicy` 中的文件合并，或联合。我们通过向 `BoardConfig.mk` 文件添加以下语句来完成此操作：

```kt
BOARD_SEPOLICY_UNION += file_contexts
```

你可以为任何策略文件这样做，即使是自定义文件。它仅通过基本名匹配文件名（没有目录）。例如，如果你有一个想要添加到基本 `watchdog.te` 规则文件的 `watchdog.te` 规则文件，你只需添加 `watchdog.te` 即可，如下所示：

```kt
BOARD_SEPOLICY_UNION += file_contexts watchdog.te
```

这将在构建过程中生成一个新的 `watchdog.te` 文件，它将你的新规则与在 `external/sepolicy/watchdog.te` 中找到的规则合并。

还要注意，你可以使用`BOARD_SEPOLICY_UNION`将新文件添加到构建中，因此要为自定义域（如`custom.te`）添加`.te`文件，可以这样做：

```kt
BOARD_SEPOLICY_UNION += file_contexts watchdog.te custom.te
```

假设你想用你自己的文件覆盖`external/sepolicy watchdog.te`文件。你可以将其添加到`BOARD_SEPOLICY_REPLACE`中，如下所示：

```kt
BOARD_SEPOLICY_REPLACE := watchdog.te
```

注意，你不能替换基础策略中不存在的文件。同样，你不能让同一个文件同时出现在`UNION`和`REPLACE`中，因为这会导致歧义。你不能在同一个策略文件上有超过一个`BOARD_SEPOLICY_REPLACE`的指定。

假设我们正在为两个虚构设备（设备 X 和设备 Y）进行分层构建。这两个设备，设备 X 和设备 Y，都从设备 A 继承了`BoardConfigCommon.mk`。设备 A 不是一个真实设备，但由于 X 和 Y 有共同点，所以这些共同的部分被保留在设备 A 中。

假设设备 A 的`BoardConfigCommon.mk`包含以下语句：

```kt
BOARD_SEPOLICY_DIRS += device/OEM/A
BOARD_SEPOLICY_UNION += file_contexts custom.te
```

假设设备 X 的`BoardConfig.mk`包含以下内容：

```kt
BOARD_SEPOLICY_DIRS += device/OEM/X
BOARD_SEPOLICY_UNION += file_contexts custom.te
```

最后，假设设备 Y 的`BoardConfig.mk`包含以下内容：

```kt
BOARD_SEPOLICY_DIRS += device/OEM/Y
BOARD_SEPOLICY_UNION += file_contexts custom.te
```

用于构建设备 X 和设备 Y 的结果策略集如下：

设备 X 策略集：

```kt
device/OEM/A/file_contexts
device/OEM/A/custom.te
device/OEM/X/file_contexts
device/OEM/X/custome.te
external/sepolicy/* (base policy files)
```

设备 Y 还包含：

```kt
device/OEM/A/file_contexts
device/OEM/A/custom.te
device/OEM/Y/file_contexts
device/OEM/Y/custom.te
external/sepolicy/* (base policy files)
```

在一个常见场景中，你可能不希望设备 Y 的结果策略集包含`device/OEM/A/custom.te`。这是一个`BOARD_SEPOLICY_IGNORE`的使用场景。你可以用它来过滤特定的策略文件。然而，你必须具体，并使用存储库的相对路径。例如，在设备 Y 的`BoardConfig.mk`中：

```kt
BOARD_SEPOLICY_IGNORE += device/OEM/A/custom.te
```

现在，当你为设备 Y 构建策略时，策略集将不会包含该文件。`BOARD_SEPOLICY_IGNORE`也可以与`BOARD_SEPOLICY_REPLACE`一起使用，允许在设备层次结构中多次使用，但只有一个`BOARD_SEPOLICY_REPLACE`语句生效。

## 深入分析 build_policy

现在我们已经看到了如何使用一些新的机制来控制策略构建，让我们实际分析策略构建过程发生在哪里。如前所述，策略构建是由`Android.mk`文件控制的。我们之前遇到了对`build_policy()`函数的调用，这正是所有我们设置的`BOARD_SEPOLICY_*`变量发生魔法的地方。检查`build_policy`函数，我们看到对`sepolicy_replace_paths`变量的引用，所以让我们先看看这个变量。

`sepolicy_replace_paths`变量在`Makefile`评估时开始其生命周期。换句话说，它是无条件执行的。代码首先遍历所有的`BOARD_SEPOLICY_REPLACE`文件，并检查是否有任何文件在`BOARD_SEPOLICY_UNION`中。如果找到，会打印错误并导致构建失败，显示`Ambiguous request for sepolicy $(pf). Appears in both BOARD_SEPOLICY_REPLACE and BOARD_SEPOLICY_UNION`，其中`$(pf)`会扩展到有问题的策略文件。之后，它会使用由`BOARD_SEPOLICY_DIRS`设置的搜索路径来扩展`BOARD_SEPOLICY_REPLACE`条目，从而得到从 Android 树根的完整相对路径。然后，它会将这些条目与`BOARD_SEPOLICY_IGNORE`进行过滤，丢弃任何应该被忽略的内容。然后，它确保只找到一个替换文件候选者。否则，它会发出适当的错误信息。最后，它确保文件存在于`LOCAL_PATH`或基本策略中，如果两者都没有找到，它会发出错误信息：

```kt
...
# Quick edge case error detection for BOARD_SEPOLICY_REPLACE.
# Builds the singular path for each replace file.
sepolicy_replace_paths :=
$(foreach pf, $(BOARD_SEPOLICY_REPLACE), \
  $(if $(filter $(pf), $(BOARD_SEPOLICY_UNION)), \
    $(error Ambiguous request for sepolicy $(pf). Appears in both \
      BOARD_SEPOLICY_REPLACE and BOARD_SEPOLICY_UNION), \
  ) \
  $(eval _paths := $(filter-out $(BOARD_SEPOLICY_IGNORE), \
  $(wildcard $(addsuffix /$(pf), $(BOARD_SEPOLICY_DIRS))))) \
  $(eval _occurrences := $(words $(_paths))) \
  $(if $(filter 0,$(_occurrences)), \
    $(error No sepolicy file found for $(pf) in $(BOARD_SEPOLICY_DIRS)), \
  ) \
  $(if $(filter 1, $(_occurrences)), \
    $(eval sepolicy_replace_paths += $(_paths)), \
    $(error Multiple occurrences of replace file $(pf) in $(_paths)) \
  ) \
  $(if $(filter 0, $(words $(wildcard $(addsuffix /$(pf), $(LOCAL_PATH))))), \
    $(error Specified the sepolicy file $(pf) in BOARD_SEPOLICY_REPLACE, \
      but none found in $(LOCAL_PATH)), \
  ) \
)
```

在此之后，对构建策略的调用可以使用`replace_paths`作为在构建过程中将被替换的文件的扩展列表。

`build_policy`函数的参数是你希望将其扩展为其 Android 根相对路径名的文件名，使用`BOARD_SEPOLICY_*`变量家族提供的功能。例如，在我们的设备 A、X 和 Y 的上下文中调用`$(build_policy, file_contexts)`会产生以下结果：

```kt
device/OEM/A/file_contexts
device/OEM/Y/file_contexts
```

`build_policy` 函数的阅读稍微有些棘手。许多嵌套的函数调用导致最深的缩进首先执行。然而，像所有代码一样，我们是自上而下、从左到右阅读的，所以解释将从那里开始。该函数首先遍历所有作为参数传递的文件。然后，它将它们与`BOARD_SEPOLICY_DIRS`进行一次替换和一次联合操作。`sepolicy_replace_paths`变量会进行错误检查，以确保文件不会同时出现在替换和联合位置。对于替换路径的扩展，它会检查扩展的路径是否在`sepolicy_replace_dirs`中，如果是，则进行替换。对于联合部分，它只是进行扩展。这些扩展的结果随后会通过`BOARD_SEPOLICY_IGNORE`的过滤器，从而丢弃任何明确忽略的路径：

```kt
# Builds paths for all requested policy files w.r.t
# both BOARD_SEPOLICY_REPLACE and BOARD_SEPOLICY_UNION
# product variables.
# $(1): the set of policy name paths to build
build_policy = $(foreach type, $(1), \
  $(filter-out $(BOARD_SEPOLICY_IGNORE), \
    $(foreach expanded_type, $(notdir $(wildcard $(addsuffix /$(type), $(LOCAL_PATH)))), \
      $(if $(filter $(expanded_type), $(BOARD_SEPOLICY_REPLACE)), \
        $(wildcard $(addsuffix $(expanded_type), $(sort $(dir $(sepolicy_replace_paths))))), \
        $(LOCAL_PATH)/$(expanded_type) \
      ) \
    ) \
    $(foreach union_policy, $(wildcard $(addsuffix /$(type), $(BOARD_SEPOLICY_DIRS))), \
      $(if $(filter $(notdir $(union_policy)), $(BOARD_SEPOLICY_UNION)), \
        $(union_policy), \
      ) \
    ) \
  ) \
)
...
```

## 构建 mac_permissions.xml

`mac_permissions.xml` 构建过程有些复杂，正如我们在第十章中看到的，*将应用程序放置在域中*。首先，`mac_permissions.xml` 可以与迄今为止引入的所有 `BOARD_SEPOLICY_*` 变量一起使用。最终结果是遵循这些变量规则的单一 XML 文件。此外，原始 XML 文件由位于 `sepolicy/tools` 的工具 `insertkeys.py` 处理。`insertkeys.py` 工具使用 `keys.conf` 将 XML 文件签名段中的标签映射到包含证书的 `.pem` 文件。`keys.conf` 文件也可以在 `BOARD_SEPOLICY_*` 变量中使用。构建配方首先在 `keys.conf` 上调用 `build_policy` 并使用 `m4` 来连接结果。因此，`keys.conf` 中的 `m4` 声明将被尊重。然而，这尚未被使用。最初的意图是使用 `m4 -s` 同步行，以便您可以在 `m4` 处理时跟随 `keys.conf` 文件中的包含链。另一方面，当连接多个文件时，`m4` 会提供同步行，并提供遵循 `#line NUM "FILE"` 行的注释行。这些很有用，因为 `m4` 会接受多个输入文件并将它们组合成一个单一的、展开的输出文件。将会有同步行指示每个文件的开始，这可以帮助您追踪问题。继续回到 `mac_permissions.xml` 构建过程，在 `m4` 扩展 `keys.conf` 之后，该文件以及从 `build_policy()` 调用中获取的所有 `mac_permissions.xml` 文件最终被输入到 `insertkeys.py` 工具中。然后，`insertkeys.py` 工具使用 `keys.conf` 文件将所有匹配的 `signature=<TAG>` 行替换为来自 PEM 文件的实际十六进制编码的 X509，即 `signature=308E3600`。此外，`insertkeys.py` 工具将 XML 文件合并成一个文件，并删除空白和注释以减少其在磁盘上的大小。这没有对其他主要文件（如 `sepolicy`、`seapp_contexts`、`property_contexts` 和 `mac_permissions.xml`）的构建依赖。

## 构建 seapp_contexts

`seapp_contexts`文件也受所有`BOARD_SEPOLICY_*`变量的影响。从`build_policy()`的结果调用生成的所有`seapp_contexts`文件也通过`m4 -s`进行处理，以获得包含同步行的单个`seapp_contexts`文件。同样，就像`mac_permissions.xml`文件的`keys.conf`构建一样，`m4`除了用于同步行外，没有其他用途。这个结果，连接的`seapp_contexts`文件随后被输入到`check_seapp`工具中。这个工具是用 C 编程语言编写的，并在构建过程中构建为可执行文件。源代码可以在`tools/check_seapp`中找到。这个工具读取`seapp_contexts`文件并检查其语法。它验证是否存在无效的关键值对，`levelFrom`是否是一个有效的标识符，以及类型和域字段对于给定的`sepolicy`是否有效。这个构建依赖于`sepolicy`对域和类型字段与策略文件进行严格类型检查。

## 构建 file_contexts

`file_contexts`文件也受所有`BOARD_SEPOLICY_*`变量的影响。生成的集合通过`m4 -s`进行处理，然后单个输出通过`checkfc`工具运行。`checkfc`工具检查文件的语法和语法，并验证类型存在于构建的`sepolicy`中。因此，它依赖于`sepolicy`的构建。

## 构建 property_contexts

`property_contexts`的行为与`file_contexts`构建完全相同，除了它检查一个`property_contexts`文件。它还使用`checkfc`。

## 当前 NSA 研究文件

此外，在国家安全局（NSA）已经开始进行企业操作（`eops`）的工作。由于这个功能尚未合并到主流 Android 中，并且可能会发生巨大变化，因此这里不会涉及。然而，最前沿的地方始终是源代码和 NSA Bitbucket 仓库。`selinux-network.sh`也属于这一类别；它尚未被主流采用，并且可能会从 AOSP 中删除（[`android-review.googlesource.com/#/c/114380/`](https://android-review.googlesource.com/#/c/114380/))）。

# 独立工具

此外，还有一些为 Android 策略评估构建的独立工具，你可能觉得它们很有用。我们将探讨其中的一些及其用法。大多数你在其他参考资料中找到的标准桌面工具在 SE for Android SELinux 策略上仍然有效。请注意，如果你运行以下任何工具并遇到段错误，你可能会需要应用来自[`marc.info/?l=seandroid-list&m=141684060409894&w=2`](http://marc.info/?l=seandroid-list&m=141684060409894&w=2)线程的补丁。

## sepolicy-check

这个工具允许你查看给定的允许规则是否存在于策略文件中。其命令的基本语法如下：

```kt
sepolicy-check -s <domain> -t <type> -c <class> -p <permission> -P <policy_file>

```

例如，如果你想查看`system_app`是否可以写入`system_data_file`类文件，你可以执行以下命令：

```kt
$ sepolicy-check -s system_app -t system_data_file -c file -p write -P $OUT/root/sepolicy

```

## sepolicy-analyze

这是一个检查 SELinux 开发中常见问题的好工具，它可以捕捉到一些新 SELinux 策略编写者的常见陷阱。它可以检查等效域、重复的允许规则。它还可以执行策略类型差异检查。

域等效性检查功能非常有帮助。它显示了你可能（理论上）想要不同的域，尽管它们在实现中已经收敛。这些类型将是合并的理想候选者。然而，它也可能显示政策设计中需要纠正的问题。换句话说，你并没有预料到这些域是等效的。调用命令如下：

```kt
$ sepolicy-analyze -e -P $OUT/root/sepolicy

```

重复允许规则检查是否存在在类型上存在的允许规则，这些类型也存在于该类型继承的属性上。特定类型的允许规则是移除的候选者，因为属性上已经存在一个`allow`。要执行此检查，请运行以下命令：

```kt
$sepolicy-analyze -D -P $OUT/root/sepolicy

```

差异也便于查看文件内的类型差异。如果你想查看两个域之间的差异，可以使用此功能。这对于识别可能合并的域很有用。要执行此检查，请执行以下命令：

```kt
$sepolicy-analyze -d -P $OUT/root/sepolicy

```

# 摘要

在本章中，我们介绍了控制设备策略的各种组件是如何实际构建和创建的，例如`sepolicy`和`mac_permissions.xml`。本章还介绍了用于跨设备和配置管理和构建策略的`BOARD_SEPOLICY_*`变量。然后我们回顾了`Android.mk`组件，详细说明了构建和配置管理的核心工作原理。
