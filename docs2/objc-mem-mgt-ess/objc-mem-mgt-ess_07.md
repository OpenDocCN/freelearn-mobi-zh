# 第七章. 键值编程方法

键值编码是一个真正酷的功能，它与键值观察配合得很好。它允许你编写更少的代码，并创建非常优雅的解决方案和代码模块。在实际应用中，有许多情况是某个东西发生变化，而应用的另一部分应该受到影响。问题是，当实例或类的属性发生变化时，你可以做任何事情，包括但不限于检查其值是否有效，当某个值发生变化时向某人发送消息，等等。选项是无限的。

在本章中，我们将涵盖以下主题：

+   什么是键值编码？

+   NSKeyValueCoding 协议

+   NSKeyValueCoding 行为的手动子集

+   关联对象

+   选择器作为键

+   最大灵活性和处理不寻常的键/值

此外，请注意，NSKeyValueCoding 协议自 Cocoa 中的 Mac OS X 10.0 以来就已经可用，它也出现在了 2008 年 7 月 11 日发布的 iOS 2.0 中。一般来说，iOS 和 Mac 的 API 通常是先在 Mac 平台上出现，然后再在 iOS 平台上出现。

# 什么是键值编码或 KVC？

键值编码基本上是一种间接访问对象属性的方法，而不是通过实例变量显式获取和设置这些属性。使用 KVC，我们使用字符串作为属性键，它充当一个标识符。它通过传递一个“键”，即一个字符串来获取或设置与该键相关的属性。例如，看看以下代码示例：

```swift
@interface DogClass
@property NSString *dog_name;
@property NSInteger number_legs;
@end

DogClass *mydog = [[DogClass alloc] init];
NSString *string = [myDog valueForKey:@"dog_name"];
[mydog setValue:@4 forKey:@"number_legs"];
```

在前面的代码中，我们创建了具有两个 `NSString` 和 `NSInteger` 属性的 `DogClass`。然后，我们使用 `valueForKey` 和 `setValue` 通过键值编码分别获取 `dog_name` 和 `number_legs` 的值。

如果这听起来很熟悉，你可能在使用 NSDictionary 时会注意到语法上的相似性。

另有一个示例代码，你可以参考以获得更多澄清。让我们看看以下代码：

```swift
// The following line sets a property, directly.
//Example A
myObject.myProperty = myValue;

/*
  While this other line sets the same property, this time using KVC.
*/
//Example B
[myObject setValue:myValue forKey:@"myProperty"];
```

一些早期接触 Objective-C 的开发者不喜欢使用点操作符显式设置属性的方法，就像在 `myObject.myProperty = myValue` 中看到的那样，但这种方法本质上是有帮助的，因为它将设置过程中涉及的属性与设置动作本身分离开来。在这个上下文中，一个正常的设置器是适用的，但编写你自己的设置器意味着你将编写大量的样板代码，这会使你的代码更加冗长。

基本上，你的应用程序的访问器方法将实现由 KVC 确定的方法和模式签名。这些访问器方法的任务是提供一种方式来访问应用程序数据模型中的属性值。它们有两个，`set` 和 `get` 访问器。`set` 访问器——也称为 `setters`——设置属性的值，而 `get` 访问器——也称为 `getters`——获取/返回属性的值。

想象一下，有一个`NSTableViewDataSource`方法来处理除了默认之外的一行编辑，而不使用 KVC。它应该看起来像以下代码：

```swift
- (void)tableView:(NSTableView *)aTableView
    setMyObjectValue:(NSString *)anObject
    forMyTableColumn:(NSTableColumn *)aTableColumn
    row:(int)rowIndex
{
    if ([[aTableColumn identifier] isEqual:@"myName"])
    {
        [[myRecords objectAtIndex:rowIndex] setName:anObject];
    }
    else if ([[aTableColumn identifier] isEqual:@"myAddress"])
    {
        [[myRecords objectAtIndex:rowIndex] setAddress:anObject];
    }
}
```

然而，一旦我们可以使用 KVC，方法可以像这样：

```swift
- (void)tableView:(NSTableView *)aTableView
    setMyObjectValue:(NSString *)anObject
    forMyTableColumn:(NSTableColumn *)aTableColumn
    row:(int)rowIndex
{
    [[myRecords objectAtIndex:rowIndex] setValue:anObject forKey:[aTableColumn identifier]];
}
```

这里展示了 KVC 的本质；它是一个更好的方法，因为每个属性的编辑不需要作为单独的条件来处理。另一个巨大的优点是它的效率，因为一个有数千列的表将由相同的代码处理，甚至不需要添加一行。注意，在第一个例子中，我们需要有两个`if`循环来处理两个不同的标识符，但使用 KVC，我们可以减少冗长的代码，使用`setValue`代替，并且只需一条语句就能达到相同的结果。

除了键值编码简化了你的代码之外，实现其兼容访问器是一个有效的设计原则，它有助于数据封装，并使得与键值观察（我们将在后面介绍）和其他技术（如 Cocoa 绑定、Core Data 等）一起工作变得更加容易。

`NSKeyValueCoding`是一个非正式协议，它提供了 KVC 的基本方法，而`NSObject`提供了其默认实现。键值编码可以访问三种类型的对象值；它们是属性、一对一关系和一对多关系，我们可以通过字符串间接访问属性。

我们所说的属性只是一个简单的值属性，所以它可能是一个`NSString`或`Boolean`值，以及`NSNumber`和其他不可变对象类型。

当一个对象具有自己的属性时，这些属性被称为属性，它们在对象和属性之间建立一对一的关系。这些属性有趣的地方在于，它们可以改变，而对象本身却没有任何改变。为了更好地理解这一点，可以想象一个`NSView`实例的 superview 作为一对一关系。一组相关对象构成一对多关系。我们可以在`NSArray`或`NSSet`实例中看到这一点，其中`NSArray`或`NSSet`实例与一组对象之间有一对多关系。

# `NSKeyValueCoding`协议

我至今展示的每个示例代码中都使用了`NSKeyValueCoding`协议。我也一直称它为协议，但正如我之前所说的，它是一个非正式协议，一个`NSObject`类别。

KVC 是一种机制，允许你通过使用字符串“键”间接访问对象的属性。为了启用 KVC，你的类必须遵守`NSKeyValueCoding`。大多数情况下，你不需要做任何事情就能完成它，因为它是通过`NSObject`来遵守的。

为了使某个属性的键值编码兼容类，必须实现`setValue:forKey:`和`valueForKey:`方法以正常工作。

## 属性和一对一关系的兼容性

如果属性仅仅是属性或一对一关系，你必须确保你的类具有以下规范；一个例子是 `[myObject setValue:myValue forKey:@"myProperty"];`，这是我们之前看到的：

+   有一个名为 `<key>` 或 `_<key>` 的实例变量，或者有一个名为 `-<key>` 的实现方法，这是对键值对中键的引用。一般来说，KVC 键以小写字母开头，但对于像 URL 这样的键，如果首字母大写也是可以接受的。

+   如果属性是可变的，则还需要实现 `-set<Ket>:`。

+   `-set<Key>:` 方法的实现不应包含任何验证，因为验证将由下一点提到的方实现。

+   如果验证适用于该键，则必须在此处实现 `-validate<Key>:error:` 以及你的验证代码。

## 索引一对一关系的兼容性

使用 `NSArrays` 或 `NSMutableArrays` 将会带你了解一对多关系的概念，对于索引多对多关系，你需要确保的关键值编码兼容性要求是：

+   实现 `-<key>` 方法，返回一个数组

+   此外，你可能还有一个名为 `<key>` 或 `_<key>` 的 `NSArray` 实例变量，或者甚至可以继续实现 `-countOf<Key>` 和以下一个或多个：`-<key>AtIndexes:` 或 `-objectIn<Key>AtIndex:`

+   为了提高性能，你也可以实现 `-get<Key>:range:`，但这不是必需的

否则，如果你处理的是可变的索引有序多对多关系，这些是你的要求：

+   至少实现一个方法：`-insertObject:in<Key>AtIndex:` 和 `-insert<Key>:atIndexes:`

+   至少实现一个方法：`-removeObjectFrom<Key>AtIndex:` 和 `-remove<Key>AtIndexes:`

+   作为选项，你甚至可以实现以下方法之一：`-replace<Key>AtIndexes:with` 或 `-replaceObjectIn<Key>AtIndex:withObject:`

## 无序多对多关系的兼容性

NSSets 是无序集合的一个例子，并且也有多对多关系，所以你需要确保的无序多对多关系的键值编码兼容性要求是：

+   实现 `-<key>` 方法，返回一个 `NSSet`

+   否则，设置一个名为 `<key>` 或 `_<key>` 的实例变量

+   或者实现以下方法：`-enumeratorOf<Key>`，`-countOf<Key>`，和 `-memberOf<Key>:`

如果是可变的无序多对多关系，KVC 兼容性会要求你：

+   实现以下方法中的至少一个：`-add<Key>:` 或 `-add<Key>Object:`

+   实现以下方法中的至少一个：`-remove<Key>:` 或 `-remove<Key>Object:`

+   为了提高性能，你可以实现 `-set<Key>:` 和 `-insert<Key>:`

使用`NSString`键，你可以使用`setValue:forKey:`和`valueForKey:`方法设置和获取值。这个键是一个简单的字符串，用作对象属性的标识符。一个键必须符合以下规则：以小写字母开头，不应包含空白字符，并使用 ASCII 编码。以下示例键中应用了所有这些规则：`mySampleKey`、`pageNumber`和`oddSum`。

还有键路径，它们基本上是一个由点分隔的两个或多个键的字符串，如`pictures.byOwner.forYear`。如果你很难理解，可以将其视为如这里所示的 UNIX 目录相对路径，`pictures/Vasilkoff/2014`。

很明显，文件夹 2014 相对于`Vasilkoff`，而`Vasilkoff`相对于`pictures`，反过来又相对于用户的当前目录。在键路径中，第一个键——在我们前面的代码示例中是`pictures`——相对于接收对象。

例如，使用地址和街道的概念，你可以从地址中推导出街道。所以，如果你使用相同的概念，`address.street`键路径将从接收对象中获取地址属性的值，然后你可以根据地址对象确定街道属性。

## 键值编码的优势

+   大多数属性默认支持`NSKeyValueCoding`非正式协议。任何从`NSObject`继承的对象都有对`NSKeyValueCoding`的自动支持。所以，你的自定义类将不会支持`NSKeyValueCoding`，除非你明确使其继承自`NSObject`。

+   KVC 会自动查找 setter 和 getter 方法，如果找不到，它甚至会获取或设置实例变量。

+   在处理多个属性对象时，使用键路径的可能性非常有帮助。

+   为了通知状态变化，KVC 可以很容易地与`NSKeyValueObserving`集成，以实现观察者软件模式。

+   处理未定义键的可能性。

+   这提供了回退机制。

## 键值编码的缺点

+   属性键必须是`NSStrings`，这意味着编译器没有关于属性类型或其存在的任何信息。因此，无法从 ID 的返回值中检索任何类型的信息，正如你所知，ID 是一个指向 Objective-C 对象的指针。

+   它的扩展搜索路径使其成为一个非常慢的 KVC 方法。

+   该类必须提供一个方法或实例变量与属性名称匹配，只有这样，它才能被`NSKeyValueCoding`找到。如果你的键有误，你的应用程序将在运行时崩溃，而不是在编译时，所以你必须确保你的键拼写正确，以避免崩溃。

# NSKeyValueCoding 行为的手动子集

`NSKeyValueCoding` 协议在查找方法和实例变量时表现不同。在第一种情况下，它会查找方法选择器的名称，而在最后一种情况下，它会查找实例变量的名称。

这可以通过手动完成，如下面的示例所示：

```swift
// Manual implementation of KVC setter for method.
NSString *mySetterString = [@"set" stringByAppendingString:[myKeyString capitalizedString]];
[myObject performSelector:NSSelectorFromString(mySetterString) withObject:myValue];

// Manual implementation of KVC setter for instance variable.
object_setInstanceVariable(myObject, myKeyString, myValue);
```

由于 KVC 可以自动查找设置器和获取器，如果您想避免 `NSKeyValueCoding` 查找指定的或普通的方法和实例变量，您可能只需要通过创建自己的查找路径来使用前面的方法。

## 创建自己的查找路径的优点

为了避免使用 `NSKeyValueCoding`，寻找通常由 `NSKeyValueCoding` 查找的方法或实例变量，并创建自己的查找路径将是您需要采取的方法。让我们先从优点开始，然后讨论其缺点：

+   这可能比正常的 `NSKeyValueCoding` 路径更快。

+   它让您对路径有更多的控制。与 `NSKeyValueCoding` 路径不同，它也会适用于非 `NSObject` 继承的类。

+   通过手动操作，可以使用非对象值进行获取和设置。

## 创建自己的查找路径的缺点

+   通常，您将花费比仅使用正常的 `NSKeyValueCoding` 路径更多的时间来处理它。

+   它也提供了更少的灵活性，因为您需要编写更多的代码来覆盖任何不寻常的键/值情况，这些情况通常由自动方法覆盖。

# 关联对象

在 iOS 和 64 位 Mac OS X 应用程序使用的 Objective-C 2.0 运行时中，您可以从任何对象设置关联到另一个对象。在这种情况下，对象没有实例变量或方法的支撑，可以在运行时通过键设置一组额外的属性，如下所示：

```swift
objc_setAssociatedObject(myObject, myKey, myValue, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
```

如果您想从对象外部设置属性，可以使用它。如果您是一个对象，而您的 T 恤颜色是您的属性之一，那么这就像有人从您房子的外面改变它的颜色，而您甚至都没有注意到。

您应该在类似的情况下使用它，即您想在设置属性时让对象远离知道、支持或参与，而您从程序的其它部分设置属性。关联对象不应该是您头脑中想要使用的顶级方法，因为缺乏类型信息使得由于错误的类型而导致崩溃变得容易。

## 使用关联对象的优点

+   键可以是任何指针。在这种情况下，可以使用 `OBJC_ASSOCIATION_ASSIGN`。

+   它可能是最快的键值编码方法。

+   不需要方法或实例变量的支持。

## 使用关联对象的缺点

+   它对对象本身（实例变量或方法）没有影响。对象不会知道自己的变化。

+   在关联对象中，键不再是 `NSString`，而是一个指针。

# 选择器作为键

通常，KVC 会在找到属性键后查找属性键，并在找到后执行操作。另一种方法是在查找过程中对对象的属性进行操作。Objective-C 核心中有一个查找方法，其键用作选择器。

以下代码行是如何实现这个查找方法的：

```swift
objc_msgSend(myObject, mySetterSelector, myValue);
```

### 注意

这种方法与手动实现实例变量的设置器非常相似，但不是使用键来形成一个选择器进行查找，而是使用选择器本身作为键。

## 使用选择器作为键的优势

+   可以获取和设置非对象数据。

+   在所有处理方法的方法中，这是最快的一个。

## 使用选择器作为键的缺点

+   你需要不同的选择器来获取和设置

+   由于选择器不是对象，无法直接存储在`NSArray`和`NSDictionary`中。相反，你可以使用`NSValue`或 Core Foundation

# 最大灵活性和处理不寻常的键/值

在学习了这么多使用键值编码的方法之后，如果你在处理不寻常的键/值时寻求更多灵活性，仍然有一种非常重要的实现方式。就是自己动手做。键值编码的最终方法就是自己处理实现。

创建一个获取器和设置器方法，并在每个方法中适当地在一个对象拥有的字典上返回和设置值，可能是最简单的方法。

我们可以在以下示例代码中查看这种方法：

```swift
/*
//------------------------------
  We create the method called "setCollectionValue:forKey:"
//------------------------------
*/

- (void)setCollectionValue:(id)value forKey:(NSString *)key
{
      /*
     //------------------------------
	 Here we set the value for key in a dictionary owned by the object.
     //------------------------------
     */

      [collectionDictionary setObject:value forKey:key];
}
 /*
//------------------------------
     Then, we create the method called "getCollectionValueForKey:"Note that it's a getter method, so it must return something – (id)
//------------------------------
*/
 - (id)getCollectionValueForKey:(NSString *)key
{
    /* 
   //------------------------------
    Here, we get the object from the dictionary, for the specified key and return it.
   //------------------------------
    */

    return [collectionDictionary objectForKey:key];
}
```

在我们的示例代码中，我们使用了`NSDictionary`作为值的内部存储；然而，你可以使用自己的存储解决方案，甚至 Cocoa 键值存储结构：

+   `NSMutableDictionary`

+   `NSMapTable`

+   `CFMutableDictionaryRef`

## 自行实现的优势

+   单个对象可以暴露多个集合

+   在获取和设置时，可以使用相应集合支持的任何数据类型

+   在所有实现方法中，这是最灵活的一个

## 自行实现的优势

+   对于随机对象来说，这根本不起作用，只有针对目标类才有效

+   除了这个之外，你不能使用其他`NSKeyValueCoding`概念

# 键值观察

键值观察——也称为 KVO——是一种在变量更改时得到通知的方法，但仅当它使用 KVC 更改时。我们可以从这个中突出两点：

+   首先，你需要 KVC 才能进行 KVO

+   其次，如果一个变量直接通过其默认的设置器和获取器方法更改，而没有使用键值编码，你将不会得到任何通知

任何键路径中的任何变量都可以由一个对象进行观察。如果你考虑使用 KVO，这很有用。由于 KVO 建立在 KVC 之上，你需要 KVC 来实现 KVO，使用 KVO 应该是你需要使用 KVC 的原因之一。

## 实现键值观察

实现 KVO 相对容易，正如我们将在下面的代码示例中看到的那样。在指定的键路径上添加一个观察者。之后，你可以创建一个方法，当观察者在其键路径上看到变量发生变化时，该方法将被调用。

可以使用以下方法从`NSKeyCodingProtocol`注册对象作为观察者：`addObserver:forKeyPath:options:context:`。任何修改执行时，都会调用以下方法`observeValueForKeyPath:ofObject:change:context:`。

首先，进入你的类并添加以下方法：

```swift
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
}
```

如您之前所见，此方法在执行任何修改时被调用。然而，该协议比这更强大；它允许您在变化发生之前和之后通过使用相应的方法：`willChangeValueForKey`和`didChangeValueForKey`来通知变化。如果您需要时间特定的通知，您可以考虑这些方法。

让我们检查以下代码，其中我们注册一个对象作为观察者：

```swift
/*
//------------------------------
   We register the object "developmentManager" as the observer of "developer". It will then notify you when any change will take place for the key path "developmentStage".
//------------------------------
*/
[developer addObserver:developmentManager forKeyPath:@"developmentStage" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
```

如果您仔细观察，您会注意到我们使用了选项`NSKeyValueObservingOptionNew`和`NSKeyValueObservingOptionOld`。这两个选项都在我们需要知道旧值和新值时使用。这些值将存储在我们的更改字典中。

在我们的例子中，让我们假设开发阶段由级别表示，`NSInteger`值从*0*到*10*，并且每次修改时我们都需要通知我们的进度。在这种情况下，我们将创建两个简单的方法来为我们完成这项工作：

```swift
- (void)informNoProgress
{
  NSLog(@"We had no progress today");
}

- (void)informRealProgress
{
  NSLog(@"Our today's progress is of %@ level", developer.developmentStage);
}
```

前面的两个方法现在已完成；一个将在开发阶段没有变化时不会通知进度——在我们的场景中，我们将考虑这是不可能减少的，另一个将在开发阶段变化时通过级别通知真实进度。然而，现在，我们想要在比较值之后调用适当的方法。记住我们使用了选项`NSKeyValueObservingOptionNew`和`NSKeyValueObservingOptionOld`；它们将在变化后保存旧值和新值。

当观察者通知修改时，旧值和新值将在调用该方法的内部进行处理，如下所示：

```swift
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    if([keyPath isEqualToString:@"developmentStage"])
    {
    /*
    //------------------------------
        Here we store the old and new values for further comparison.
    //------------------------------
    */
     NSInteger oldStage = [changeobjectForKey:NSKeyValueChangeOldKey];
    NSInteger newStage = [changeobjectForKey:NSKeyValueChangeNewKey];

    /*
    //------------------------------
        Then, we check whether the oldStage level is lower than the newStage level
    //------------------------------
    */
     if(oldStage < newStage)
    {
      /*
      //------------------------------
          If the value is lower, there is progress and we call the properly method to inform it
      //------------------------------
      */
       [self informRealProgress];

    } else {
      /*
      //------------------------------
	  However, if the old level is not lower, it means there was no progress, we call the method to inform it.
      //------------------------------
      */
       [self informNoProgress];
    }
      }
}
```

在前面的代码中，我们确保如果观察到的键是我们真正要找的，只是为了确保——在我们的例子中，键是`developmentStage`。然后，我们将旧值和新值存储起来以便比较。如果有积极的变化，通知进度，如果没有，调用其他方法通知坏消息。

这是一个非常实用的工具，尤其是如果它被巧妙地使用，因为它真的很强大，因为它允许我们观察或监视对象上的 KVC 键路径，并在对象值发生变化时收到通知，这在某些编程环境中可能很有用。即使在属性变化上也能有所控制，这是一个非常强大的功能，我相信您将在自己的项目中找到很多很好的用例。

## 性能考虑

在重写 KVC 方法实现时，你必须小心，因为默认实现会缓存 Objective-C 运行时信息，以便更有效且更少出错，不必要的重写实现可能会影响你应用程序的性能。

# 摘要

到目前为止，我们已经深入探讨了键值编码和其他细节，例如各种实现方法、它们的优缺点，以及键值观察——这是建立在键值编码之上的一个机制。

我们还看到了一些关于键值编码和键值观察的工作代码，并解释了为什么我们更倾向于使用键值编码而不是其他类似方法，例如使用点操作符来访问属性。

通过这些，我希望这能帮助你理解键值编码和键值观察。因此，让我们继续前进到下一章，我们将深入探讨苹果公司推出的全新语言 Swift。
