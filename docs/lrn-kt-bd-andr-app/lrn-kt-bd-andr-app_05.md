# 第五章：类型检查和空安全

如第一章*所述，为 Android 开发设置*，Kotlin 带来的一个伟大特性是**空安全**。在本章中，我们将学习 Kotlin 是如何成为一个空安全语言的，以及我们如何充分利用它。

在本章结束时，我们将学到：

+   非空和可空类型

+   安全调用操作符

+   Elvis 操作符

+   `!!`操作符

+   安全和不安全的类型转换操作符

# 空安全

Java 和其他许多语言中开发者最常见的痛点之一是访问空引用的成员。在大多数语言中，这会导致运行时的空引用异常。大多数 Java 开发者将其称为`NullPointerException`。

Kotlin 旨在尽可能消除空引用和异常的可能性。如第一章中所述，*为 Android 开发设置*，在 Kotlin 中，你可能遇到`NullPointerException`的四个可能原因：

+   外部 Java 代码

+   显式调用抛出`NullPointerException`

+   使用`!!`操作符（我们稍后会学到更多关于这个操作符的知识）

+   关于初始化的数据不一致性

那么 Kotlin 是如何确保这一点的呢？

# 可空和非空类型

**可空类型**是允许保存`null`值的引用，而**非空类型**是不能保存`null`值的引用。

Kotlin 的类型系统设计用于区分这两种引用类型。可空类型通过在类型末尾添加`?`来声明。例如：

```kt
var name: String = "Anna" // non-nullable String
var gender: String? = "Female" //nullable String

name = null // will not compile
gender = null // will compile

print("Length of name is ${name.length}") // will compile

print("Length of gender is ${gender.length}") // will not compile

```

在先前的代码中有一些需要注意的事项：

+   `name`不能被赋予`null`值，因为它是非空类型

+   另一方面，`gender`可以被赋予`null`值，因为它声明为可空类型

+   无法像访问`name`的成员方法或属性一样访问`gender`的成员方法或属性

有多种方式可以访问可空类型的方法或属性。你可以在条件中检查`null`并访问方法或属性。例如：

```kt
if (gender != null) {
    print("Length of gender is ${gender.length}") 
}
```

编译器跟踪`null`检查的结果，因此允许在`if`条件的主体中调用`length`。这是一个**智能转换**的例子：

+   使用安全调用操作符(`?.`)

+   使用 Elvis 操作符(`?:`)

+   使用`!!`操作符

+   执行智能转换

智能转换是 Kotlin 中的一个智能功能，编译器会跟踪`if`语句的结果，并在需要时自动执行转换。

# 安全调用操作符

访问可空类型的方法或属性的另一种方式是使用安全调用操作符：

```kt
val len = gender?.length
print("Length of gender is $len")
```

在先前的代码中，如果 gender 不为`null`，则`len`的值将是`gender.length`的结果。否则，`len`的值将为`null`。

如果在`gender`为`null`时不需要执行任何操作，使用安全调用操作符是很好的。如果我们想在`gender`为`null`时为`len`赋予不同的值，我们可以将安全调用操作符与**Elvis 操作符**结合使用。

# Elvis 操作符

Elvis 操作符类似于 Java 中的三元`if`操作符。它是简化`if-else`语句的一种方式。例如：

```kt
val len = if (gender != null) gender.length else 0
```

代码可以简化为：

```kt
val len = gender?.length ?: 0
```

在先前的代码中，如果`gender?.length`的值为`null`，则`len`的值将为`0`。

# !!操作符

如果我们不在乎遇到空指针异常，那么我们可以使用`!!`操作符。例如：

```kt
val len = gender!!.length
```

如果`gender`为`null`，则会导致空指针异常。

只有在确定变量的值或者不在乎遇到空指针异常时才使用`!!`操作符。

# 类型检查

就像在 Java 中一样，你可以确认变量的类型。在 Kotlin 中，使用`is`操作符来实现。例如：

```kt
if (gender is String) {
    println("Length of gender is ${gender.length}") // gender is automatically cast to a String
}
```

就像之前的 `null` 检查一样，编译器会跟踪类型检查的结果，并自动将 `gender` 转换为 `String`，从而允许调用 `gender.length`。这被称为智能转换。

# 转换运算符

要将变量转换为另一种类型，您必须使用转换运算符（`as`）：

```kt
var fullname: String = name as String
```

如果您尝试将变量转换为的类型不是您要转换的类型，则转换运算符将抛出错误。为了防止这种情况，您可以使用**安全转换运算符**（`as?`）：

```kt
var gen: String? = gender as? String
```

安全转换运算符不会抛出错误，而是在转换不可能时返回 `null`。

# 总结

在本章中，我们学习了 Kotlin 如何帮助使您的代码具有空安全性的不同方式。 Kotlin 中有不同的运算符用于实现这一点，我们讨论了如何使用它们。

在下一章中，我们将在学习 Kotlin 中的函数和 lambda 的同时完成我们的井字棋游戏。
