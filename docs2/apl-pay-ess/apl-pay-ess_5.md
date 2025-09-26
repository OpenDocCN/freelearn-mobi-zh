# 第五章 设计订单管理服务器

如果你有一个向客户提供实体商品的业务，拥有一个可靠的订单管理服务器非常重要。根据客户的需求，你可能需要从运行在 Web 浏览器、Web 服务器或移动设备上的客户端应用提供对服务器的访问。本书重点关注运行在 iPhone 和 iPad 设备上的客户端应用，以及在模拟器应用中。

通常，基于客户端/服务器的订单捕获和订单处理系统具有以下组件：

+   **服务器**：运行订单处理 Web 应用、数据库管理系统和其他服务器端进程。Web 应用实现了基于 HTTP 的 API 客户端，用于请求和提交信息。通过这个 API，Web 应用向客户端提供产品信息，并处理客户端提交的支付和订单信息。

+   **客户端**：运行客户端应用。客户端应用调用订单处理 Web 应用提供的 API，以请求和提交信息到 Web 应用。

为了本书的目的，一个*订单管理 Web 应用*是一个响应 HTTP 请求的过程。在本书的示例项目中，这个过程向客户端提供库存和运输方式信息，并处理客户端提交给它的支付信息（包括关于订购的项目、运输详情等信息）。

客户端应用通过 HTTP 请求与服务器交互。在你的开发环境中，订单管理 Web 应用运行在你的开发计算机上，而客户端应用则运行在支持 Apple Pay 的模拟器或 iOS 设备上。

本章描述以下内容：

+   订单管理服务器的特点

+   其数据库结构和客户端 API

# 配置订单管理服务器 Web 应用

一个*Web 服务器*是一个存储信息、修改信息并向客户端提供访问权限的计算机。这些客户端可以请求信息或要求服务器以特定方式处理数据，这可能导致服务器上存储的数据发生变化，客户端接收到的数据表示也会发生变化。实际上，任何计算机都可以作为 Web 服务器。Web 应用是在 Web 服务器上运行并通过对客户端的 HTTP 请求提供服务的过程。在生产环境中，Web 应用运行在专门配置的 Web 服务器上，这些服务器提供冗余、复制和其他功能，以确保稳健性、高性能和安全，以及其他特性。然而，你不需要一个完整的 Web 服务器来运行 Web 应用。你可以在你的开发计算机上运行 Web 应用。

一个*订单管理服务器*是一个运行在可供客户端访问的计算机上的 Web 应用。在你的开发环境中（由你的计算机、开发工具集和 iOS 设备组成），服务器 Web 应用运行在你的计算机上，而客户端应用则运行在你的计算机上的模拟器或 iOS 设备上。

### 注意

服务器和客户端的硬件和软件配置众多。本章描述了在单个 Mac 上使用模拟器应用和可以发送 HTTP 请求到该计算机的 iOS 设备进行开发的配置。此配置不适用于测试或部署。当您准备在更真实的配置中测试或部署应用程序时，请咨询部署工程师。

要测试支持 Apple Pay 的客户端应用程序的所有组件（例如本书的示例应用程序、商家应用程序），您必须在 iOS 设备上运行客户端应用程序。通常，客户端支付网关软件在模拟器中运行效果不佳。

客户端应用程序可以直接访问服务器网络应用程序（即，如果两个进程都在同一台计算机上运行，例如当客户端在 iOS 模拟器中运行时），通过运行在 iOS 设备上的客户端的内部网络（在子网内），或通过使用可通过公共互联网访问的 Web 服务器（无论是用于大规模测试还是生产）通过互联网访问。示例项目使用 Node.js 实现服务器网络应用程序。Node.js 是用于开发服务器端应用程序的环境。

# 定义订单管理数据结构

订单管理系统至少有表示可供销售的产品（库存）和订单的数据结构。为了支持 Apple Pay 工作流程，系统还应有一个表示支持的运输方式的结构。示例项目使用 MongoDB 数据库存储这些数据，并使用 Node.js 的 Mongoose 数据建模模块。MongoDB 是一种基于文档的数据库系统。Mongoose 使得从模式创建文档和访问这些文档中的数据变得更加容易。

示例订单管理系统数据库有三个集合：`Product`、`ShippingMethod` 和 `Order`。集合在关系型数据库系统中类似于表。

`Product` 集合存储有关出售的每个产品的信息。下表显示了集合中产品记录可用的字段：

| 字段名称 | 描述 |
| --- | --- |
| `name` | 这是产品的名称 |
| `description` | 这是一个描述产品的短语 |
| `image_uri` | 这是一个标识产品图像的 URI |
| `quantity_on_hand` | 这是库存中产品的单位数量 |
| `price` | 这是每个产品单位的销售价格 |

`ShippingMethod` 集合存储有关可用运输方式的信息。下表显示了运输记录的字段：

| 字段名称 | 描述 |
| --- | --- |
| `name` | 这是运输方式的名称 |
| `description` | 这是一个表示运输持续时间的天数 |
| `transit_days` | 这是一个表示运输持续时间的天数 |
| `price` | 这是运输方式的费用 |

`Order` 集合存储了订单管理系统处理的每个订单的信息。以下是订单记录的字段：

| 字段名称 | 描述 |
| --- | --- |
| `date` | 这是订单处理的日期时间组 |
| `description` | 这是销售产品的名称（每个订单一个产品） |
| `shipping_contact` | 这是被标识为运输联系人的姓名 |
| `shipping_email` | 这是运输联系人的电子邮件地址 |
| `shipping_street` | 这是运输地址的街道 |
| `shipping_city` | 这是运输地址的城市 |
| `shipping_state` | 这是运输地址的状态 |
| `shipping_zip` | 这是运输地址的邮政编码 |
| `shipping_method_name` | 这是用于订单的运输方式名称 |

这是定义三个集合结构的 Node.js 代码：

```swift
// server_app/lib/inventory_models.js
var mongoose= require('mongoose');
var Schema=   mongoose.Schema;

var Product_schema= new Schema({
   name:             String,
   description:      String,
   image_uri:        String,
   quantity_on_hand: Number,
   price:            String
});

var ShippingMethod_schema= new Schema({
   name:         String,
   description:  String,
   transit_days: Number,
   price:        String
});

var Order_schema= new Schema({
   date                 : String,
   description          : String,
   shipping_contact     : String,
   shipping_email       : String,
   shipping_street      : String,
   shipping_city        : String,
   shipping_state       : String,
   shipping_zip         : String,
   shipping_method_name : String,
   total_price          : String,
   stripe_charge_id     : String
});

exports.Product=
   mongoose.model('Product',        Product_schema);
exports.ShippingMethod=
   mongoose.model('ShippingMethod', ShippingMethod_schema);
exports.Order=
   mongoose.model('Order',          Order_schema);
```

# 向客户端提供库存信息

订单管理系统实现了三个 REST API（在 Node.js 术语中称为中间件）以向客户端提供库存和运输方式信息。具体如下：

+   `/shipping_methods`: 这将返回支持的运输方式列表

+   `/inventory`: 这将返回产品目录

+   `/product_image/<image_name>`: 这将返回用于向用户展示产品的图片

当客户端应用程序启动时，它会请求系统支持的运输方式列表（在 `AppDelegate.m` 中的 `application:didFinishLaunchingWithOptions:` 方法）。在产品列表屏幕上显示可用的产品列表之前，应用程序会请求产品目录（在 `ProductList.m` 中的 `viewDidLoad` 方法）。当应用程序即将显示特定产品的卡片（在用户在产品列表屏幕中选择产品后），它会从服务器请求产品的图片（在 `ProductCard.m` 中的 `viewDidLoad` 方法）。

### 注意

对于开发，示例应用程序（商家应用程序）使用的 URI 指定了进程名称（但不是实际的 Web 地址）和端口号，以访问订单管理系统 Web 应用程序（例如，`http://red:12345/inventory`）。当您将应用程序部署到生产环境（即现实世界）时，您必须使用配置适当的真实 Web 服务器来运行您的 Web 应用程序，并使用指向公共 Web 地址的 URI，例如 `http://red.com/inventory`。

这是实现本节前面介绍的中间件的 Node.js 代码：

```swift
// server_app/red.js

// load required modules
var models=    require('./lib/inventory_models.js');
var assert=    require('assert');
var mongoose=  require('mongoose');
var restify=   require('restify');
var stripe=    require('stripe')('<my_key>')
var server=    restify.createServer();
server.use(restify.bodyParser());

// specify the data models
Product=         models.Product;
ShippingMethod=  models.ShippingMethod;
Order=           models.Order;

// connect to the red MongoDB database
mongoose.connect('mongodb://localhost/red');

protocol=  'http://';
hostname=  'red';
port=      12345;
base_uri=  protocol + hostname + ':' + port;
console.log(base_uri);

// initialize Product collection, if needed
Product.find(function(error, _products)
{
   if (_products.length == 0)
   {
      console.log('initializing product collection');
      models.Product(
      {
         name:             'Clock', 
         description:      'Wooden clock', 
         quantity_on_hand: 10,
         price:            '50.00',
         image_uri:        base_uri + '/product_image/clock.jpeg'
      }).save();
      ...
   }
});

// load Product collection into products
var products= new Array();
Product.find(function(error, _products)
{
   products= _products;
});

// initialize ShippingMethods collection, if needed
ShippingMethod.find(function(error, _shipping_methods)
{
   if (_shipping_methods.length == 0)
   {
      console.log('initializing shipping-method collection');
      models.ShippingMethod({
         name:         'Free',
         description:  'Delivers in seven days',
         transit_days: 7,
         price:        '0.00'
      }).save();
      ...
   }
})

// load ShippingMethod collection into shipping_methods
var shipping_methods= new Array();
ShippingMethod.find(function(error, _shipping_methods)
{
   shipping_methods= _shipping_methods;
});

// ** start middleware (client API) **
//   /product_image/<name> API:
//     provides product image from
//     <project_dir>/public/product_image/<name>
server.get(/\/product_image\/?.*/,
   restify.serveStatic( { directory: '../public' }));

//   /inventory API:
//     provides current inventory from Product document
server.get('/inventory', function(request, response, next)
{
   response.send(products);
   next();
});

//   /shippng_methods API:
//     provides shipping methods from ShippingMethod document
server.get('/shipping_methods', function(request, response, next)
{
   response.send(shipping_methods);
   next();
});
...
// ** end middleware (client API) **

// listen for HTTP requests
server.listen(12345);
```

# 处理来自客户的订单

示例订单管理系统服务器 Web 应用程序通过单个函数 `payment`（在第四章中描述，*支付处理工作流程*）处理订单，客户端通过向 `http://red:12345/payment` URI 发送 HTTP `POST` 请求来执行此函数。

首先，该函数尝试对客户卡片进行收费。如果收费成功，该函数将使用客户端提供的订单信息将一条记录添加到`Order`集合中。完成后，该函数将有关支付交易和新的订单（如果已创建）的信息返回给客户端。

有关`payment`函数实现的详细信息，请参阅第四章中的*流程阶段*部分，*支付处理工作流程*。

# 实现安全通信

前几节中描述的配置仅适用于开发。当部署一个用于真实客户使用的订单管理系统 Web 应用时，你需要做几件事情。你的客户使用的客户端应用也必须是安全的。

首先，客户端应用在 URI 中不应使用进程名称和端口号，而应使用基于 Web 地址的 URI，例如[`red.com`](http://red.com)。你还应配置一台计算机作为你的公开可访问的 Web 服务器，该服务器运行你的服务器进程。根据预期的流量量，你可能需要配置你的 Web 服务器同时运行多个服务器 Web 应用的实例来处理来自多个客户端的并发请求。

其次，你必须确保服务器和客户端之间传输的数据是安全的。通过确保数据传输的安全性，你确保只有授权实体才能接收数据，数据在传输过程中不会被修改，并且数据不能被第三方读取。实现这一目标的一种方法是用**HTTPS**（**HTTP 安全**）协议而不是 HTTP 来在服务器和客户端之间传输消息和数据。

你可以在苹果的开发者库中了解如何向客户端应用添加 HTTPS 支持。有关在 Web 服务器中实现 HTTPS 的信息，请参阅相应的文档。

# 摘要

本章已描述了一个简单 Web 应用的设计和实现，该应用作为订单管理系统，为客户端应用提供库存数据，并处理在 iOS 模拟器应用、iPhone 或 iPad 设备上运行的客户端应用提交的订单，这些设备支持 Apple Pay。本章还提到了在 Web 服务器和客户端应用中需要实施的临界安全措施，以确保它们之间的安全通信。下一章将提供有关客户端应用使用的 Apple Pay 主 API 的基本信息。
