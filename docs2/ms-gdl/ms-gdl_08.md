# 第八章。迁移

如果你来自 Ant 或 Maven 背景，首先想到的问题可能是：为什么是 Gradle？我们已经在初始章节中讨论了这个问题。然后，另一个重要的问题出现了。我们已经在 Ant 或 Maven 中编写了大量的构建代码；现在，如果需要在新脚本中写入 Gradle，管理两个构建工具不会很困难吗？在本章中，我们将解释将现有的 Ant 或 Maven 脚本迁移到 Gradle 构建脚本的不同技术。在本章的第一节中，我们将讨论可以应用于从 Ant 迁移到 Gradle 的不同策略，而后续章节将涵盖从 Maven 迁移到 Gradle 的策略。

# 从 Ant 迁移

Ant 是最初成为开发者中非常受欢迎的构建工具之一，因为它们可以控制构建过程的每个步骤。但是，编写每个步骤意味着在构建文件中有很多样板代码。Ant 初始版本中缺少的另一个特性是依赖关系管理的复杂性，后来通过引入 Ivy 依赖关系管理器得到了简化。对于 Ant 用户来说，切换到使用 Gradle 作为他们的构建工具非常简单且容易。Gradle 通过 Groovy 的 `AntBuilder` 提供了对 Ant 的直接集成。在 Gradle 世界中，Ant 任务被视为一等公民。在接下来的几节中，我们将讨论三种不同的策略：将 Ant 文件导入 Gradle、`AntBuilder` 类的脚本使用以及重写为 Gradle。这些策略可以遵循从 Ant 迁移到 Gradle。

## 导入 Ant 文件

这是将 Ant 脚本与 Gradle 脚本集成的一种最简单的方法。作为迁移的第一步，当您有很多用 Ant 编写的构建脚本，并且希望在不改变当前构建结构的情况下开始使用 Gradle 时，它非常有用。我们将从一个示例 Ant 文件开始。假设我们有一个 Java 项目的 `build.xml` Ant 文件，并执行以下任务：

1.  构建项目（编译代码并生成 JAR 文件）。

1.  生成 JAR 文件的校验和。

1.  创建一个包含 JAR 文件和校验和文件的 ZIP 文件。

以下是要执行所有前面提到的三个操作的 `build.xml` 文件：

```java
<project name="sampleProject" default="makeJar" basedir=".">

<property name="src" location="src/main/java"/>
<property name="build" location="build"/>
<property name="classes" location="build/classes"/>
<property name="libs" location="build/libs"/>
<property name="distributions" location="build/distributions"/>
<property name="version" value="1.0"/>

<target name="setup" depends="clean">
   <mkdir dir="${classes}"/>
   <mkdir dir="${distributions}"/>
</target>

<target name="compile" depends="setup" description="compile the source">
   <javac srcdir="${src}" destdir="${build}/classes"  includeantruntime="false"/>
</target>
<target name="makeJar" depends="compile" description="generate the distributions">
   <jar jarfile="${libs}/sampleproject-${version}.jar" basedir="${classes}"/>
</target>
<target name="clean" description="clean up">
   <delete dir="${build}"/>
</target>

<target name="zip" description="zip the jar and checksum" depends="makeJar,checksum">
   <zip destfile="${distributions}/sampleproject.zip" filesonly="true" basedir="${libs}" includes="*.checksum,*.jar"  />
</target>

<target name="checksum" description="generate checksum and store in file" depends="makeJar">
   <checksum file="${libs}/sampleproject-${version}.jar" property="sampleMD5"/>
   <echo file="${libs}/sampleproject.checksum" message="checksum=${sampleMD5}"/>
</target>

<target name="GradleProperties">
<echo message="Gradle comments are:: ${comments}"/>
</target>

</project>
```

要构建项目，您需要运行以下目标（在 Ant 中，我们执行一个可以与 Gradle 任务相比的目标）：

```java
SampleProject$ ant makeJar
Buildfile: <path>/Chapter8/SampleProject/build.xml

clean:
 [delete] Deleting directory <path>/Chapter8/SampleProject/build

setup:
 [mkdir] Created dir: <path>/Chapter8/SampleProject/build/classes
 [mkdir] Created dir: <path>/Chapter8/SampleProject/build/distributions

compile:
 [javac] Compiling 2 source files to <path>/Chapter8/SampleProject/build/classes

makeJar:
 [jar] Building jar: <path>/Chapter8/SampleProject/build/libs/sampleproject-1.0.jar

BUILD SUCCESSFUL
Total time: 0 seconds

```

目标将执行其他所需的目标，如编译、清理、设置等。这将生成 `sampleproject-1.0.jar` 文件在 `build/libs` 目录中。现在，为了生成 JAR 的校验和并将其与 JAR 文件捆绑在一起，我们可以运行以下目标：

```java
$ ant zip

```

此目标将运行 `makeJar` 目标以及所有其他依赖目标来创建 JAR 文件，然后它将执行校验和目标以生成 JAR 文件的 `md5` 校验和。最后，ZIP 任务将打包校验和文件和 JAR 文件，并在 `build/distributions` 目录内创建一个 ZIP 文件。

这是一个 Java 项目的示例构建文件；你可以根据定制需求添加额外的目标。我们可以简单地在这个 Gradle 构建文件中导入这个 `Ant` 构建文件，并能够执行 Ant 目标。构建文件的内容如下所示：

```java
ant.importBuild 'build.xml'

```

这一行足以导入 Ant 构建文件并继续使用 Gradle。现在尝试执行以下操作：

```java
$ gradle -b build_import.gradle zip
::clean
:setup
:compile
:makeJar
:checksum
:zip

BUILD SUCCESSFUL

```

在这里，我们将构建文件命名为 `build_import.gradle`。前面的命令依次执行了所有 Ant 任务。你可以在 `build/distributions` 目录中找到创建的 ZIP 文件。

这是从 Ant 迁移到 Gradle 的第一步之一。如果你不想玩现有的构建脚本而想使用 Gradle，这将非常有帮助。只需将 Ant 文件导入 Gradle 构建文件，就可以开始使用。

### 访问属性

Gradle 还允许你访问现有的 Ant 属性并添加新属性。要访问现有的 Ant 属性，你可以使用 `ant.properties`，如下所示：

```java
ant.importBuild 'build.xml'

def antVersion = ant.properties['version']
def src = ant.properties['src']

task showAntProperties << {
      println "Ant Version is "+ antVersion
      println "Source location is "+ src

}
```

```java
$ gradle -b build_import.gradle sAP

:showAntProperties
Ant Version is 1.0
Source location is D:\Chapter8\SampleProject\src\main\java

BUILD SUCCESSFUL

```

在这里，Gradle 脚本已从 Ant 文件中获取属性值，并且我们在 Gradle 任务中打印这些值。以类似的方式，我们可以在 Gradle 中设置 Ant 属性并在 Ant 构建文件中访问这些属性。

使用以下语句更新构建文件：

```java
ant.properties['comments'] = "This comment added in Gradle"

```

此属性将由 Ant 文件中的 `GradleProperties` 目标读取，如下所示：

```java
<target name="GradleProperties">
   <echo message="Gradle comments are ${comments}"/>
</target>
```

现在，在执行 `GradleProperties` 目标时，我们可以在控制台输出中找到显示的注释属性，如下代码片段所示：

```java
$ gradle  -b build_import.gradle GradleProperties

Starting Build
...
Executing task ':GradleProperties' (up-to-date check took 0.015 secs) due to:
 Task has not declared any outputs.
[ant:echo] Gradle comments are:: This comments added in Gradle
:GradleProperties (Thread[main,5,main]) completed. Took 0.047 secs.

BUILD SUCCESSFUL

```

### 更新 Ant 任务

Gradle 还允许你增强现有的 Ant 任务。同样，我们可以使用 `doFirst` 或 `doLast` 闭包来增强任何现有的 Gradle 任务；Ant 任务也可以以类似的方式扩展。在构建文件（文件：`build_import.gradle`）中添加以下语句以将 `doFirst` 和 `doLast` 闭包添加到 `GradleProperties` 任务中：

```java
GradleProperties.doFirst {
   println "Adding additional behavior before Ant task operations"
}
GradleProperties.doLast {
   println "Adding additional behavior after Ant Task operations"
}
```

现在，`GradleProperties` 目标执行了 `doFirst` 和 `doLast` 闭包，控制台输出如下所示：

```java
$ gradle -b build_import.gradle GP

Starting Build
……
:GradleProperties (Thread[main,5,main]) started.
:GradleProperties
Executing task ':GradleProperties' (up-to-date check took 0.003 secs) due to:
 Task has not declared any outputs.
Adding additional behavior before Ant task operations
[ant:echo] Gradle comments are:: This comments added in Gradle
Adding additional behavior after Ant Task operations
:GradleProperties (Thread[main,5,main]) completed. Took 0.158 secs.

BUILD SUCCESSFUL

```

## 使用 AntBuilder API

我们已经看到，将 Ant 的 `build.xml` 导入 Gradle 并将 Ant 目标作为 Gradle 任务使用是多么简单。另一种方法是使用 `AntBuilder` 类。使用 `AntBuilder`，你可以在 Gradle 脚本中调用 Ant 任务。在 Gradle 构建文件中有一个名为 'ant' 的 `AntBuilder` 类实例可用。使用此实例，当我们调用方法时，它实际上执行了一个 Ant 任务。

在以下示例中，我们将使用相同的 `build.xml` 文件，并解释如何使用 `AntBuilder` 重写任务以使用 Gradle：

1.  设置属性：

    | Ant 方式 | 使用 AntBuilder |
    | --- | --- |

    |

    ```java
    <project 
    name="qualitycheck" default="makeJar" 
    basedir=".">

    <property name="src" location="src/main/java"/>
    <property name="build" location="build"/>
    <property name="lib" location="lib"/>
    <property name="dist" location="dist"/>
    <property name="version" value="1.0"/>
    ```

    |

    ```java
    defaultTasks "makeJar"

    def src = "src/main/java"
    def build = "build"
    def libs = "build/libs"
    def classes = "build/classes"
    def distributions = "build/distributions"
    def version = 1.0
    ```

    |

1.  清理构建目录：

    | Ant 方式 | 使用 AntBuilder |
    | --- | --- |

    |

    ```java
    <delete dir="${build}"/>
    ```

    |

    ```java
    ant.delete(dir:"${build}")
    ```

    |

1.  创建新目录：

    | Ant 方式 | 使用 AntBuilder |
    | --- | --- |

    |

    ```java
    <mkdir dir="${classes}"/>
    <mkdir dir="${distributions}"/>
    ```

    |

    ```java
    ant.mkdir(dir:"${libs}")
    ant.mkdir(dir:"${classes}")
    ```

    |

1.  编译 Java 代码：

    | Ant 方式 | 使用 AntBuilder |
    | --- | --- |

    |

    ```java
    <javac srcdir="${src}"
    destdir="${build}/classes" 
    includeantruntime="false"/>
    ```

    |

    ```java
    ant.javac(srcdir:"${src}",
    destdir:"${classes}",
    includeantruntime:"false")
    ```

    |

1.  从编译后的源代码创建 JAR 文件：

    | Ant 方式 | 使用 AntBuilder |
    | --- | --- |

    |

    ```java
    <jar jarfile 
    ="${libs}/sampleproject-${version}.jar" 
    basedir="${classes}"
    />
    ```

    |

    ```java
    ant.jar(
    destfile: "${libs}/sampleproject-${version}.jar",
    basedir:"${classes}") 
    ```

    |

1.  为 JAR 生成校验和：

    | Ant 方式 | 使用 AntBuilder |
    | --- | --- |

    |

    ```java
    <checksum file="${libs}/sampleproject-${version}.jar" 
    property="sampleMD5"/>

    <echo file ="${libs}/sampleproject.checksum" message="checksum=${sampleMD5}"
    />
    ```

    |

    ```java
    ant.checksum(
    file:"${libs}/sampleproject-${version}.jar",
    property:"sampleMD5"
    )

    ant.echo(file:"${libs}/sampleproject.checksum",
    message:"checksum=${ant.sampleMD5}"
    )
    ```

    |

1.  将校验和文件和 JAR 文件打包成 ZIP 文件：

    | Ant 方式 | 使用 AntBuilder |
    | --- | --- |

    |

    ```java
    <zip 
    destfile ="${distributions}/sampleproject.zip" 
    filesonly="true" basedir="${libs}" includes="*.checksum,*.jar" 
     />
    ```

    |

    ```java
    ant.zip(destfile: "${dist}/sampleproject.zip",
    basedir:"dist") 
    ```

    |

因此，完整的构建文件将如下所示：

```java
defaultTasks "makeJar"

def src="img/java"
def build="build"
def libs="build/libs"
def classes = "build/classes"
def distributions="build/distributions"
def version=1.0

task setup(dependsOn:'clean') << {
   ant.mkdir(dir:"${libs}")
   ant.mkdir(dir:"${classes}")
}

task clean << {
   ant.delete(dir:"${build}")
}

task compileProject(dependsOn:'setup') << {
   ant.javac(srcdir:"${src}",destdir:"${classes}",
includeantruntime:"false")
}

task makeJar << {
   ant.jar(destfile: "${libs}/sampleproject-${version}.jar",
basedir:"${classes}") 
}

task zip(dependsOn:'checksum') << {
   ant.zip(destfile: "${distributions}/sampleproject.zip",
basedir:"${libs}")
}

task checksum(dependsOn:'makeJar') << {
   ant.checksum(file:"${libs}/sampleproject-${version}.jar",
property:"sampleMD5")
   ant.echo(file:"${libs}/sampleproject.checksum",
message:"checksum=${ant.sampleMD5}")
}

makeJar.dependsOn compileProject
```

现在，执行 ZIP 任务并检查分发目录。您将找到以下创建的 `sampleproject.zip` 文件：

```java
$ gradle -b build_ant.gradle zip
:clean
:setup
:compileProject
:makeJar
:checksum
:zip

BUILD SUCCESSFUL

```

注意这里，`AntBuilder` 对于尚未迁移到 Gradle 的自定义 Ant `taskdef` 任务非常有用。

## 重写为 Gradle

到目前为止，我们已经看到将 Ant 文件导入 Gradle 脚本是多么容易。我们还探讨了另一种方法，即使用 `AntBuilder` 实例在从 Ant 迁移到 Gradle 的过程中复制相同的行为。现在，在第三种方法中，我们将用 Groovy 重新编写 Ant 脚本。

我们将继续使用相同的 Ant `build.xml` 文件，并将其转换为 Gradle 构建脚本。在这个例子中，我们正在构建一个 Java 项目。正如我们所知，要构建 Java 项目，Gradle 已经为我们提供了一个 Java 插件；您只需在构建文件中应用 Java 插件即可。Java 插件将负责所有标准约定和配置。

以下是我们已经在 第四章 中讨论过的 Java 插件的某些约定，*插件管理*：

| 使用的约定 | 描述 |
| --- | --- |
| `build` | 默认构建目录名称 |
| `build/libs` | 默认 JAR 文件位置 |
| `src/main/java; src/test/java` | Java 源文件位置 |
| 项目名称 | 归档文件名 |

如果项目也遵循这些约定，我们就不需要为项目编写任何额外的配置。唯一需要的配置是定义版本属性；否则，将创建不带版本信息的 JAR 文件。

因此，我们新的构建脚本将如下所示：

```java
apply plugin :'java'
version = 1.0
```

现在，我们已经完成了。不需要编写任何脚本来创建和删除目录、编译文件、创建 JAR 任务等。在执行构建命令后，您可以在 `build/libs` 目录中找到 `<projectname>-<version>.jar`：

```java
$ gradle build

:clean
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

```

将大约 30 行 Ant 代码缩减为两行 Gradle 代码真是太容易了。这使我们摆脱了所有样板代码，并专注于主要逻辑。然而，并非所有项目都可以通过应用插件或遵循某些约定简单地转换。如果项目不遵循 Gradle 或 Maven 约定，您可能需要配置 `sourceSets` 和其他配置。

回到示例，我们只创建了 JAR 文件；还有两个任务待完成。我们必须生成一个文件来存储校验和，我们需要将校验和文件和 JAR 文件捆绑到一个 ZIP 文件中。我们可以定义两个额外的任务来完成此操作，如下所示：

```java
apply plugin:'java'
version = 1.0

task zip(type: Zip) {
    from "${libsDir}"
    destinationDir project.distsDir
}

task checksum << {
	ant.checksum(file:"${libsDir}/${project.name}-${version}.jar",property:"sampleMD5")
	ant.echo(file:"${libsDir}/${project.name}.checksum",message:"checksum=${ant.sampleMD5}")
}

zip.dependsOn checksum
checksum.dependsOn build
```

在前面的构建脚本中，校验和任务将为 jar 文件创建校验和。这里我们再次使用 Ant。校验和任务创建校验和，因为这是 Gradle 中最简单的方式。我们已经配置了 ZIP 任务（类型为 ZIP）以创建 ZIP 文件。Gradle 已经为构建/分发目录提供了约定，即 `project.distsDir`：

```java
$ gradle clean zip

:clean
:compileJava
:processResources UP-TO-DATE
:classes
....
:check UP-TO-DATE
:build
:checksum
:zip

BUILD SUCCESSFUL

```

### 配置

如果你不想遵循约定，Gradle 提供了一种简单的方法来根据需求配置项目。我们将展示如何配置之前创建的 Ant 任务在 Gradle 中：

1.  清理构建目录：

    | Ant 方式 | Gradle 方式 |
    | --- | --- |

    |

    ```java
    <target 
    name="clean" description="clean up">

       <delete dir =   "${build}"/>

        <delete dir = "${dist}"/>
    </target>
    ```

    |

    ```java
    task cleanDir(type: Delete) {
      delete "${build}"
    }
    ```

    |

1.  创建新目录：

    | Ant 方式 | Gradle 方式 |
    | --- | --- |

    |

    ```java
    <target 
    name="setup" depends="clean">

       <mkdir dir =  "${build}"/>
    </target>
    ```

    |

    ```java
    task setup(dependsOn:'cleanDir') << {
            def classesDir = file("${classes}")
            def distDir = file("${distributions}")
            classesDir.mkdirs()
            distDir.mkdirs()
    }
    ```

    |

1.  编译 Java 代码：

    | Ant 方式 | Gradle 方式 |
    | --- | --- |

    |

    ```java
    <target name="compile" depends="setup" description="compile the source">
        <javac srcdir="${src}" destdir="${build}" />
    </target>
    ```

    |

    ```java
    compileJava {
            File classesDir = file("${classes}")
            FileTree srcDir = fileTree(dir: "${src}")
            source srcDir
            destinationDir classesDir
    }
    ```

    |

1.  将编译的类 JAR 化：

    | Ant 方式 | Gradle 方式 |
    | --- | --- |

    |

    ```java
    <target name="dist" depends="compile" description="generate the distribution">
       <mkdir dir="${dist}"/>
       <jar jarfile="${dist}/sampleproject-${version}.jar" basedir="${build}"/>
    </target>
    ```

    |

    ```java
    task myJar(type: Jar) {
            manifest {
            attributes 'Implementation-Title': 'Sample Project',
                    'Implementation-Version': version,
                    'Main-Class': 'com.test.SampleTask'
        }
        baseName = project.name +"-" +version
        from "build/classes"
        into project.libsDir
    }
    ```

    |

因此，带有配置的最终构建文件（`build_conf.gradle`）将如下所示：

```java
apply plugin:'java'

def src="img/java"
def build="$buildDir"
def libs="$buildDir/libs"
def classes = "$buildDir/classes"
def distributions="$buildDir/distributions"
def version=1.0

task setup(dependsOn:'cleanDir') << {
   def classesDir = file("${classes}")
   def distDir = file("${distributions}")
   classesDir.mkdirs()
   distDir.mkdirs()
}

task cleanDir(type: Delete) {
  delete "${build}"
}

compileJava {
   File classesDir = file("${classes}")
   FileTree srcDir = fileTree(dir: "${src}")
   source srcDir
   destinationDir classesDir
}
task myJar(type: Jar) {
   manifest {
        attributes 'Implementation-Title': 'Sample Project',  
        	'Implementation-Version': version,
        	'Main-Class': 'com.test.SampleTask'
    }
    baseName = project.name +"-" +version
    from "build/classes"
    into project.libsDir
}
task zip(type: Zip) {
    from "${libsDir}"
    destinationDir project.distsDir
}
task checksum << {
   ant.checksum(file:"${libsDir}/${project.name}-${version}.jar",
   property:"sampleMD5")
   ant.echo(file:"${libsDir}/${project.name}.checksum",
   message:"checksum=${ant.sampleMD5}")
}

myJar.dependsOn setup
compileJava.dependsOn myJar
checksum.dependsOn compileJava
zip.dependsOn checksum
```

现在，尝试执行 ZIP 命令。你可以在 `build/libs` 目录中找到创建的 JAR 文件和校验和文件，以及构建/分发目录中的 ZIP 文件：

```java
$ gradle -b build_conf.gradle zip
:cleanDir
:setup
:myJar
:compileJava
:checksum
:zip

BUILD SUCCESSFUL

```

# 从 Maven 迁移

随着企业软件中 Ant 文件的大小和复杂性的增加，开发者开始寻找更好的解决方案。Maven 很容易成为解决方案，因为它引入了约定优于配置的概念。如果你遵循某些约定，通过跳过样板代码可以节省大量时间。Maven 还提供了一种依赖管理解决方案，这是 Ant 工具的主要缺点之一。Ant 没有提供任何依赖管理解决方案，而 Maven 带有内置的依赖管理器。

当我们讨论从 Ant 迁移到 Gradle 的策略时，你了解到最简单的解决方案是导入 Ant 的 `build.xml` 文件并直接使用它。对于 Maven 迁移，我们没有这样的功能。Maven 用户可能会发现从 Maven 迁移到 Gradle 很容易，因为两者遵循这些共同原则：

+   约定优于配置

+   依赖管理解决方案

+   仓库配置

要从 Maven 迁移到 Gradle，我们需要编写一个新的 Gradle 脚本，以模拟其功能。如果你已经使用过 Maven，你可能已经注意到 Gradle 使用了大多数 Maven 概念；因此，从 Maven 迁移到 Gradle 不会非常困难。主要区别之一是 Gradle 使用 Groovy 作为构建脚本语言，而 Maven 使用 XML。在本节中，我们将讨论一些将 Maven 脚本转换为 Gradle 脚本的最常见任务。

## 构建文件名和项目属性

`GroupId`、`artifactId` 和 `version` 是用户在 Maven `pom` 文件中需要提供的最小必需属性，而在 Gradle 中这些不是必需的。如果用户没有配置，则假定默认值。始终建议指定这些值以避免任何冲突：

| Maven 方法 | Gradle 方法 |
| --- | --- |

|

+   `<`groupId>ch8.example</groupId>`

+   `<artifactId>SampleMaven</artifactId>`

+   `<version>1.0</version>`

+   `<packaging>jar</packaging>`

|

+   `groupid`：不必要。

+   `artifactId`：默认值为项目目录名。

+   `version`：如果未提及，则将创建不带版本的组件。

+   打包取决于你在构建脚本中应用的插件。

|

### 提示

如果 Maven 中没有提及打包，则默认为 JAR。其他核心打包方式包括：`pom`、`jar`、`maven-plugin`、`ejb`、`war`、`ear`、`rar` 和 `par`。

## 属性

以下是在 Maven 和 Gradle 中定义属性的方法：

| Maven 方法 | Gradle 方法 |
| --- | --- |

|

```java
<properties>
     <src>src/main/java
     </src>
    <build>build</build>
     <classes>
       build/classes
    </classes>
    <libs>
        build/libs
    </libs>
<distributions>
     build/distributions
     </distributions>
    <version>
        1.0
     </version>
</properties>
```

|

```java
def src="img/java"
def build="build"
def lib="lib"
def dist="dist"
def version=1.0
```

|

## 依赖管理

Maven 和 Gradle 都提供了依赖管理功能。它们会自行管理依赖。你不需要担心项目中任何二级或三级依赖。类似于 Maven 的范围（如编译和运行时），Gradle 也提供了不同的配置，例如编译、运行时、`testCompile` 等。以下表格列出了 Maven 和 Gradle 支持的范围：

| Maven 范围 | Gradle 范围 |
| --- | --- |

|

+   编译

+   provided

+   运行时

+   测试

+   system

|

+   `编译`

+   `providedCompile`

+   `runtime`

+   `providedRuntime`

+   `testCompile`

+   `testRutime`

|

与 Maven 中的提供范围相比，Gradle 的 `war` 插件增加了两个额外的范围，`providedCompile` 和 `providedRuntime`。对于测试用例，Gradle 也提供了两个范围，`testCompile` 和 `testRuntime`。以下是如何在定义依赖时使用范围的一个示例：

| Maven 方法 | Gradle 方法 |
| --- | --- |

|

```java
<dependency>
<groupId>org.apache.commons</groupId>
<artifactId>commons-lang3</artifactId>
<version>3.1</version>
<scope>compile</scope>
</dependency>
```

|

```java
dependencies {
    compile group:  'org.apache.commons', name: 'commons-lang3', version:'3.1'
}
```

|

### 排除传递依赖

在 第五章 中，*依赖管理*，你学习了如何使用依赖管理。Maven 在依赖管理功能方面没有太大差异。以下是一个示例，展示了如何在构建工具中排除传递依赖：

| Maven 方法 | Gradle 方法 |
| --- | --- |

|

```java
<dependencies>
    <dependency>
      <groupId>
          commons-httpclient
      </groupId>
      <artifactId>
          commons-httpclient
       </artifactId>
      <version>3.1</version>
      <exclusions>
        <exclusion>
          <groupId>
            commons-codec
           </groupId>
          <artifactId>
             commons-codec
          </artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    ...
  </dependencies>

```

|

```java
dependencies{

compile('commons-httpclient:commons-httpclient:3.1') {
 exclude group:'commons-codec',
module:'commons-codec'
}

}
```

|

## 插件声明

Maven 插件是一组可以应用于项目的目标集合。插件中的目标通过`mvn [plugin-name]:[goal-name]`命令执行。在 Maven 中，通常有两种类型的插件：构建插件和报告插件。构建插件将在构建过程中执行，应在`pom.xml`的`<build/>`元素中进行配置。报告插件将在生成站点时执行，并使用`<reporting/>`元素进行配置。报告插件的例子包括 Checkstyle、PMD 等。在 Gradle 中，你只需在 Gradle 脚本中添加一个插件声明，或者需要在`buildscript`闭包中定义它。

以下是一个示例代码，它描述了如何在 Maven 中包含插件以及我们如何在 Gradle 中定义相同的内容：

| Maven 方式 | Gradle 方式 |
| --- | --- |

|

```java
<build>
<plugins>
<plugin>
<artifactId>maven-compiler-plugin</artifactId>
<version>2.3.2</version>
<configuration>
<source>1.7</source>
<target>1.7</target>
</configuration>
</plugin>
</plugins>
</build>
```

|

+   `apply plugin:'<pluginid>'`

+   对于自定义插件：

```java
buildscript {
ext {
springBootVersion = '1.2.2.RELEASE'
}
repositories {
jcenter()
maven { url "http://repo.spring.io/snapshot" }
maven { url "http://repo.spring.io/milestone" }
}
dependencies {
classpath("org.springframework.boot:spring-boot-gradle-plugin:1.2.2.RELEASE")
}
}
```

|

## 仓库配置

仓库用于下载资源和工件，并且也用于发布工件。在 Maven 中，你可以在`pom.xml`或`settings.xml`中定义仓库。在 Gradle 中，你可以在`init`脚本（`init.gradle`）或`build.gradle`文件中添加仓库：

| Maven 方式 | Gradle 方式 |
| --- | --- |

|

```java
<repositories>
    <repository>
      <id>rep1</id>
      <name>org repo1</name>
      <url>
       http://company.repository1
       </url>
    </repository>
</repositories>

```

|

```java
repositories {
maven {
url "http://company.repository1"
}
}
```

|

## 多模块声明

Maven 和 Gradle 都提供了创建多模块项目的方法，如下所示：

| Maven 方式 | Gradle 方式 |
| --- | --- |

|

```java
<project>
  <groupId>
    com.test.multiproject
   </groupId>
   <artifactId>
     rootproject
  </artifactId>
  <version>
     1.0
  </version>
  <packaging>
     Pom
  </packaging>
  <modules>
    <module>subproject1</module>
    <module>subproject2</module>
  </modules>
</project>

```

| 在根项目下添加`settings.gradle`并按如下方式包含子项目：

```java
include 'subproject1', 'subproject2',
```

|

## 默认值

Gradle 和 Maven 都为某些属性提供了默认值。如果你想遵循约定，可以直接使用它们，如果需要则可以更新它们：

|   | Maven 方式 | Gradle 方式 |
| --- | --- | --- |
| 项目目录 | `${project.basedir}` | `${project.rootDir}` |
| 构建目录 | `${project.basedir}/target` | `${project.rootDir}/build` |
| 类目录 | `${project.build.directory}/classes` | `${project.rootDir}/build/classes` |
| JAR 名称 | `${project.artifactId}-${project.version}` | `${project.name}` |
| 测试输出目录 | `${project.build.directory}/test-classes` | `${project.testResultsDir}` |
| 源目录 | `${project.basedir}/src/main/java` | `${project.rootDir}/src/main/java` |
| 测试源目录 | `${project.basedir}/src/test/java` | `${project.rootDir}/src/test/java` |

## Gradle init 插件

`Build init`插件可以从`pom`文件生成`build.gradle`文件。命令`gradle init --type pom`或`gradle init`会在有有效`pom.xml`文件的项目或目录中创建 Gradle 构建文件和其他工件。

我们创建了一个项目，`SampleMaven`，该项目包含位于根项目目录下的`src\main\java\ch8`目录中的 Java 文件和`pom.xml`文件。以下为`pom.xml`文件的内容：

```java
<project 

   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>

   <groupId>ch8.example</groupId>
   <artifactId>SampleMaven</artifactId>
   <version>1.0</version>
   <packaging>jar</packaging>
   <dependencies>
     <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-lang3</artifactId>
       <version>3.1</version>
       <scope>compile</scope>
     </dependency>
   </dependencies>
 </project>
```

现在，执行以下命令：

```java
$ gradle init --type pom
:wrapper
:init
Maven to Gradle conversion is an incubating feature.

BUILD SUCCESSFUL

```

注意，从 Maven 到 Gradle 是一个孵化特性。当前的 DSL 和其他配置可能在将来发生变化。执行前面的命令后，你将在项目目录中找到`build.gradle`、`settings.gradle`和 Gradle 包装文件。自动生成的`build.gradle`文件包含以下内容。它自动添加插件详情，以及组和版本信息。

系统生成的构建文件内容如下所示（以下代码片段）：

```java
apply plugin: 'java'
apply plugin: 'maven'

group = 'ch8.example'
version = '1.0'
description = """"""

sourceCompatibility = 1.5
targetCompatibility = 1.5

repositories {
     maven { url "http://repo.maven.apache.org/maven2" }
}

dependencies {
    compile group: 'org.apache.commons', name: 'commons-lang3', version:'3.1'
}
```

此插件还支持多项目构建、仓库配置、依赖管理以及许多其他功能。

# 摘要

在本章中，我们讨论了从 Ant 和 Maven 迁移到 Gradle 的过程。这在任何组织中都是一个非常常见的场景，其中现有的构建脚本是用 Ant 或 Maven 编写的，并且正在尝试升级到 Gradle。本章提供了一些分析和不同的方法，这些方法可能有助于更好地、更有组织地规划 Gradle 迁移。

在下一章中，我们将讨论使用 Docker 构建自动化的部署方面。
