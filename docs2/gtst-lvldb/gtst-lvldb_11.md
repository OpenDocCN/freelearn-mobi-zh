# 附录 A. 脚本语言

脚本语言最初被设计为用于脚本常见任务的轻量级语言，因此得名。它们被视为用于命令行工具或作为允许轻松定制更大程序控制流的嵌入式语言。本附录介绍了它们如何与 LevelDB 一起工作，保持简单以帮助您将所学代码与之类比。它并不讨论 LevelDB 作为脚本数据库解决方案的适用性——假设您已经对 LevelDB 感兴趣，或者已经在使用 LevelDB，只是想用它与其他工具一起使用。

在科学界，Python 的可用性使其在更复杂的程序中越来越受欢迎，并且它在使用 Django 等框架的 Web 应用程序中得到了增长。Ruby on Rails 对 Ruby 的普及做出了重大贡献，以至于有些人甚至不认为 Ruby 是一种独立语言。JavaScript，更正式地称为 ECMAScript，最初是一种浏览器语言，它也发展到了服务器端编程。流行的 Node.js 环境使用 Google 的 V8 引擎打包它，以便从原生代码中轻松调用独立编程环境。在这些所有情况下，这些语言提供了一种相对低开销的方式来调用 C 函数，从而使用提供 C 或 C++ 接口的外部库。

无论是在编写 Web 应用程序还是进行本地数据处理时，数据库访问通常很有用。典型的 Web 应用程序使用数据库服务器，虽然它可能由 LevelDB 支持，但不能说它直接使用 LevelDB。然而，如果您需要一个单访问数据库，从脚本语言中调用它很方便，因此已经为 LevelDB 从这些语言编写了多个访问层。

重要的是要记住，LevelDB 的主要接口是 C++，并且这并不会因为您使用脚本语言编程而改变。正如您通过本书中的 Objective-C 示例所了解的，LevelDB 的 C++ 接口非常轻量级。您可以认为我们在这里查看的脚本语言接口是我们一直在使用的 Objective-C 框架的同等物。

我们在多个章节中看到，使用自定义比较器可以为你的数据库增加价值，尤其是如果你使用复合键，或者进行不区分大小写的搜索时。虽然许多包装项目承诺比较器作为未来的功能，但它们似乎缺少这一点。记住，比较器是一个由核心 LevelDB 代码调用的 C++对象。要在脚本语言中实现它们，需要编写一个能够从 C++回调到脚本的代码桥接器。这样的`回调`函数比从脚本到数据库库的正常调用方向要复杂得多。然而，如果你在更大的本地程序中托管你的脚本解释器，你仍然可以用 C++编写自定义比较器，并使其与数据库一起工作。

# 在 Node.js 中使用 LevelDB

我们在第七章中介绍了 Node.js 和 LevelUP 以及 LevelDOWN 包装器的安装，*使用 REPL 和命令行进行调试*，作为安装**lev**实用程序的一部分。LevelDOWN 基本上是将 C++接口重新发布为 Node.js JavaScript 语法。两者都可通过**npm**（Node 包管理器）安装，现在已包含在 level 包中。主页提供了更多安装选项，网址为[`github.com/rvagg/node-levelup`](https://github.com/rvagg/node-levelup)，如果你想为项目做出贡献，也可以从该地址克隆 GIT 仓库。

LevelUP 的有趣演变过程是一个抽象层，隐藏了底层使用 LevelDB 的过程，以至于它不再需要安装 LevelDOWN，也可以与其他实现一起工作，包括内存存储 MemDown（更多详情请参阅之前提到的主页）。

Node.js 程序以一系列异步调用的回调函数的形式编写，考虑到它原本是作为 Web 应用的服务器端语言，这样做是有意义的。因此，一个简单的写入一些数据并读取回的数据程序结构如下：

```swift
var levelup = require('levelup')
// open a data store
var db = levelup('/tmp/testleveldb11_node.db')
// a simple Put operation
db.put('name', 'William Bloggs', function (err) {
  // a Batch operation made up of 3 Puts (after 1st Put finished)
  db.batch([
      { type: 'put', key: 'spouse', value: 'Josephine Bloggs' }
    , { type: 'put', key: 'dog', value: 'Kilroy' }
    , { type: 'put', key: 'occupation', value: Dev' }
  ], function (err) {
    // asynch after batch put finishes, another nest
    // read store as a stream and print each entry to stdout
    db.createReadStream()
      .on('data', console.log)
      .on('close', function () {
        db.close()
      })
  })  // end of batch Put
})  // end of top-level Put
```

使用 LevelDB 作为模块化数据库的节点包生态系统非常庞大。其中一个特别有趣的是 **levelgraph**。您可以从主页 [`github.com/mcollina/levelgraph`](https://github.com/mcollina/levelgraph) 下载它，并与 LevelUP 一起使用以提供数据库层。它使用配对键以类似我们第八章中描述的方案支持的方式提供完整的图数据库抽象，*更丰富的键和数据结构*。然而，levelgraph 进一步支持来自图数据库理论的经典三元组操作。它可以扩展以支持来自 [`github.com/mcollina/levelgraph-n3`](https://github.com/mcollina/levelgraph-n3) 的 levelgraph-n3 插件，该插件允许紧凑的 N3 语法。如果您对图数据库和基于三元组的知识表示感兴趣，请参阅 [`www.w3.org/TeamSubmission/n3/`](http://www.w3.org/TeamSubmission/n3/)。

# 使用 Python 从 LevelDB 读取

有许多针对 LevelDB 的 Python 包装器，它们使用 `Cython` 工具集生成一个接口层来与 C++ 类进行通信。最新且经常推荐的包装器是 plyvel：[`plyvel.readthedocs.org/en/latest/installation.html`](https://plyvel.readthedocs.org/en/latest/installation.html)。

然而，还有一个更底层的纯 C API 用于 LevelDB，它允许您直接调用共享库中的函数。一个简单的 Python 包装器 [`code.google.com/p/leveldb-py/`](https://code.google.com/p/leveldb-py/) 非常简单，以至于它仅用一个文件实现。该文件 `leveldb.py` 和单元测试 `test_leveldb.py` 包含在本章的示例代码中。您不需要使用 `pip` 安装或其他命令，只需将文件包含在您的调用旁边即可。

这个简单的包装器，就像许多其他脚本语言包装器一样，期望在标准系统位置有一个动态库。这反映了它们的 Unix 血统。许多安装程序实际上会重新构建 LevelDB 并将其推送到这个位置，但这个需要您自己完成这项工作。为了提供这个库，请回到第一章中关于构建 LevelDB 的说明 下载 LevelDB 和使用 OS X 构建，但这次，在您构建了 LevelDB 之后，而不是重命名静态库，请将四个文件复制到 `/usr/local/lib`：

```swift
libleveldb.a
libleveldb.dylib
libleveldb.dylib.1
libleveldb.1.12
```

这确保了在标准位置存在一个动态库，因此尝试打开名为 `leveldb` 的动态库将能够成功。

编写类似之前看到的 Node.js 代码的数据库的代码看起来与我们已经看到的 C++ 示例非常相似：

```swift
#!/usr/bin/env python
import leveldb

# open a data store
db = leveldb.DB("/tmp/testleveldb11_py.db",
 create_if_missing=True)

# a simple Put operation
db.put('name', 'William Bloggs')

# a Batch operation made up of 3 Puts
b = db.newBatch()
db.putTo(b, key = 'spouse', val= 'Josephine Bloggs')
db.putTo(b, key = 'dog', val= 'Kilroy')
db.putTo(b, key = 'occupation', val= 'Dev')
db.write(b, sync=True)
db.close()
```

为了读取这些内容，我们可以简单地遍历所有键并获取它们关联的值：

```swift
for k in db.keys(): 
  print k, db.get(k)
```

除了支持我们迄今为止看到的所有基本 LevelDB 功能外，`leveldb.py`还包括与我们在 Objective-C 中添加的前缀逻辑类似的逻辑，这样你可以获取键的子集。`test_leveldb.py`中的单元测试包括如下代码：

```swift
def testPutGet(self):
    db = self.db_class(self.db_path, create_if_missing=True)
    db.put("key1", "val1")
    db.put("key2", "val2", sync=True)
    self.assertEqual(db.get("key1"), "val1")
...
    self.assertEqual(list(db.keys()), ["key1", "key2"])
    self.assertEqual(list(db.keys(prefix="key")), ["1", "2"])
```

从最后一行可以看出，前缀检索键会自动去除前缀，这与我们在 Objective-C 中获取`TableView`键时所做的代码类似。

# 使用 Ruby 从 LevelDB 中获取

最常推荐的 LevelDB Ruby 包装器来自[`github.com/wmorgan/leveldb-ruby`](https://github.com/wmorgan/leveldb-ruby)，如本章代码示例中包含的日志文件所示，可以使用 gem 命令进行安装：

```swift
sudo gem install leveldb-ruby

```

注意，它自 2011 年以来一直处于停滞状态，且非常基础，甚至不包括批处理支持。然而，它使用与之前 Python 代码类似的代码支持了基本功能：

```swift
require 'leveldb'
# open a data store
db = LevelDB::DB.new("/tmp/testleveldb11_ruby.db")
# a simple Put operation
db.put('name', 'William Bloggs')
db.put('spouse', 'Josephine Bloggs')
db.put('dog', 'Kilroy')
db.put('occupation', 'Dev')
db.close()
```

与 Python 代码不同，Ruby 代码在读取时是惯用的，你可以直接将数据库当作字典来处理，并对其应用一个块：

```swift
db.each do |k,v|
  puts "Key=#{k}, Value=#{v}"
end
```

一个更实用且更完整的包装器是[`github.com/DAddYE/leveldb`](https://github.com/DAddYE/leveldb)，它包括更友好的迭代器和批处理，但安装过程更复杂，需要 Ruby 2.0。它增加了批处理支持：

```swift
db.batch do |b|
  b.put 'spouse', 'Josephine Bloggs'
  b.put 'dog', 'Kilroy'
  b.delete 'name'
end
```

这个示例作为 Ruby 风格的代码块使用了一个惯用表达式，即块包含所有要应用到批处理中的逻辑，因此暗示在块末尾进行写入。

# 摘要

我们在三种主流脚本语言中看到了不同的编码风格。抓住机会进一步探索那些链接，并考虑使用脚本语言作为 REPL 来探索 LevelDB 中的想法。你可能想用它快速生成大量数据库或玩转不同的键结构。
