# `Activity`生命周期

应用程序中的每一个`Activity`都在 App 进程中运行，但每个`Activity`也都有自己的生命周期。这些方法是在`Activity`即将改变状态时由平台触发的，例如，当用户暂时退出`Activity`去使用另一个`Activity`（无论它是在同一应用程序中，还是另一个应用程序中）时。`Fragment`对象也受到生命周期的约束，虽然它主要遵循与`Activity`生命周期相同的模式，但它也有从其父`Activity`中“附加”和“分离”的概念。

下面的流程图详细解释了`Activity`的生命周期：

![图片](img/4dafc4c3-acbe-4094-8a74-42f86db92d15.png)
