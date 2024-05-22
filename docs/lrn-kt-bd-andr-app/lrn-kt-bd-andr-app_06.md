# 第六章：函数和 Lambda

在本章中，我们将完成 TicTacToe 游戏的工作，并了解 Kotlin 中的函数。

在这个过程中，我们将：

+   了解函数

+   了解高阶函数及其使用方法

+   了解 lambda 及其使用方法

# 函数

在 Kotlin 中，函数的声明格式如下：

![](img/7a43c5b1-8044-4d90-a432-6aadd65fc9b7.png)

**return**类型和**parameters**是可选的。没有`return`类型的函数默认返回`Unit`。`Unit`相当于 Java 中的`void`。

作为其主体的单个表达式的函数也可以省略大括号：

```kt
fun addStudent(name: String, age:Int, classRoomNo: Int = 1, studentId: Int) : Student = Student(name, classRoomNo, studentId, age)
```

如果类型可以被编译器推断出来，则`return`类型也可以省略：

```kt
fun addStudent(name: String, age:Int, classRoomNo: Int = 1, studentId: Int) = Student(name, classRoomNo, studentId, age)
```

# 参数

在 Kotlin 中，使用帕斯卡符号（**parameter_name:Type**）定义函数参数。每个参数的类型都必须明确声明。函数声明中的参数可以被赋予默认值。格式为：**parameter_name:Type = defaultValue**。例如：

```kt
data class Student(var name: String, var classRoomNo: Int, var studentId: Int, var age: Int)

fun addStudent(name: String, age:Int, classRoomNo: Int = 1, studentId: Int) : Student {

 return Student(name, classRoomNo, studentId, age)
}

var anna = addStudent("Anna", 18, 2, 1)
var joseph = addStudent(name = "Joseph", age = 19, studentId = 2)
```

在这个例子中：

+   在调用`addStudent()`函数时，可以省略`classRoomNo`参数。例如，`joseph`将具有默认的`classRoomNo`值为`1`。

+   在某些情况下，如果没有将所有参数传递给函数，则传递的参数必须在其参数名称之前。

# 高阶函数和 lambda

术语**高阶函数**指的是一个函数，它要么接受另一个函数作为参数，要么返回一个函数，或者两者兼而有之。例如：

```kt
// 1
fun logStudent(name: String, age:Int, createStudent:(String, Int) -> Student) {
    Log.d("student creation", "About to create student with name $name")
    val student = createStudent(name, age)
    Log.d("student creation", "Student created with name ${student.name} and age ${student.age}")
}

// 2
logStudent(name = "Anna", age = 20, createStudent = { name: String, age: Int -> Student(name, 1, 3, age)})
```

在这里，`logStudent()`函数接受三个参数：`name`，`age`和`createStudent`。`createStudent`是一个函数，它接受一个`String`和一个`Int`作为参数，并返回一个`Student`对象。

`createStudent`函数未声明，而是作为表达式传递给`logStudent()`函数。这称为**lambda 表达式**。

# Lambda 表达式

Lambda 表达式是一个匿名函数，它不是声明的，而是立即作为表达式传递的。

让我们继续在 TicTacToe 应用程序中使用 lambda 表达式。打开`MainActivity.kt`。在`startNewGame()`函数中，替换以下代码行：

```kt
cell.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
        cellClickListener(i, j)
    }
})
```

用以下代码替换它们：

```kt
cell.setOnClickListener { cellClickListener(i, j) } 
```

在前面的代码行中，我们有一个匿名对象，它实现了一个具有单个抽象方法（`onClick()`）的 Java 接口。所有这些都可以用一个简单的 lambda 表达式来替代。

**单个抽象方法**（**SAM**），通常指的是接口中的功能方法。该接口通常只包含一个称为 SAM 或功能方法的抽象方法。

现在，构建并运行以查看应用程序的状态：

![](img/529b1022-a3fd-4096-896b-e620e549a24c.png)

让我们继续利用我们迄今为止学到的所有知识，以便完成游戏的工作。

Android Studio 提供了一个默认的工具链，支持大多数 JAVA 8 功能，包括 lambda 表达式。强烈建议您使用默认的工具链，并禁用所有其他选项，如 jackoptions 和 retrolambda。

# 实现游戏状态检查

在本节中，我们将处理一些函数，以帮助我们找出游戏的赢家。

首先，将以下函数添加到`MainActivity`类中：

```kt
private fun isBoardFull(gameBoard:Array<CharArray>): Boolean {
    for (i in 0 until gameBoard.size) { 
        for (j in 0 until gameBoard[i].size) { 
            if(gameBoard[i][j] == ' ') {
                return false
            }
        }
    }
    return true
}
```

此函数用于检查游戏板是否已满。在这里，我们遍历棋盘上的所有单元格，如果有任何一个单元格为空，则返回`false`。如果没有一个单元格为空，则返回`true`。

接下来，添加`isWinner()`方法：

```kt
private fun isWinner(gameBoard:Array<CharArray>, w: Char): Boolean {
    for (i in 0 until gameBoard.size) {
        if (gameBoard[i][0] == w && gameBoard[i][1] == w && 
        gameBoard[i][2] == w) {
            return true
        }

        if (gameBoard[0][i] == w && gameBoard[1][i] == w && 
        gameBoard[2][i] == w) {
            return true
        }
    }
    if ((gameBoard[0][0] == w && gameBoard[1][1] == w && gameBoard[2]
    [2] == w) ||
            (gameBoard[0][2] == w && gameBoard[1][1] == w && 
        gameBoard[2][0] == w)) {
        return true
    }
    return false
}
```

在这里，您可以检查传递的字符是否是赢家。如果字符在水平、垂直或对角线行中出现三次，则该字符是赢家。

现在添加`checkGameStatus()`函数：

```kt
private fun checkGameStatus() {
    var state: String? = null
    if(isWinner(gameBoard, 'X')) {
        state = String.format(resources.getString(R.string.winner), 'X')
    } else if (isWinner(gameBoard, 'O')) {
        state = String.format(resources.getString(R.string.winner), 'O')
    } else {
        if (isBoardFull(gameBoard)) {
            state = resources.getString(R.string.draw)
        }
    }

    if (state != null) {
        turnTextView?.text = state
        val builder = AlertDialog.Builder(this)
        builder.setMessage(state)
        builder.setPositiveButton(android.R.string.ok, { dialog, id ->
            startNewGame(false)

        })
        val dialog = builder.create()
        dialog.show()

    }
}
```

上述函数利用`isBoardFull()`和`isWinner()`函数来确定游戏的赢家是谁。如果 X 和 O 都没有赢，而且棋盘已满，则是平局。显示一个警报，显示游戏的赢家或告诉用户游戏是平局的消息。

接下来，在`cellClickListener()`函数的末尾添加一个调用`checkGameStatus()`。

构建并运行：

![](img/60fca23e-aee9-4834-9797-a9bc33c7a48b.png)

最后，实现**FloatingActionButton**的功能。在`onCreate()`函数中，将以下内容替换为：

```kt
fab.setOnClickListener { view -> HelloKotlin("Get ready for a fun game of Tic Tac Toe").displayKotlinMessage(view) }
```

将其替换为：

```kt
fab.setOnClickListener {startNewGame(false)}
```

再次构建并运行。现在，当您点击**FloatingActionButton**时，棋盘将被清空，以便您重新开始游戏：

![](img/4c213cc7-02c2-4256-aac7-5693c4995769.png)

# 总结

在本章中，我们学习了如何在 Kotlin 中使用函数和 lambda，并完成了对我们的 TicTacToe 游戏的工作。

在接下来的几章中，我们将学习如何在 Android 上使用 Google 位置服务以及执行网络调用，同时致力于创建基于位置的闹钟。
