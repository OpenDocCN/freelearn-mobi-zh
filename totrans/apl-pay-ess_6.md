# 第 6 章。Apple Pay API 概述

您在三个阶段处理 Apple Pay 交易：

1.  **准备**：创建一个支付请求，并用支付信息和用户购买的产品详情填充它。

1.  **支付表单交互**：通过 `PKPaymentAuthorizationViewControllerDelegate` 协议的方法响应用户与支付表单的交互（例如更改送货地址或所需的运输方式）。

1.  **支付处理**：通过您的支付网关提供的 API 和对您的订单处理 Web 应用的 HTTP 请求，在用户的设备和您的服务器上处理 Apple Pay 交易。

早期章节展示了您用于与 Apple Pay 交易中的主要参与者交互的工作流程。本章提供了有关 Apple Pay API 的类、方法和属性的有用信息，这些信息是您为了有效使用它而必须熟悉的。然而，当您需要更详细的信息时，您应该查阅 Apple 开发者文档。

在本章中，您将了解涉及支付交易的 Apple Pay 主要类，例如 `PKPaymentReqest` 和 `PKPaymentSummaryItem` 类。您还将了解如何通过 `PKPaymentAuthorizationViewControllerDelegate` 协议响应用支付表单事件。最后，本章将描述 Apple Pay API 用于处理地址的附加类，例如 `PKContact` 类。

本章涵盖了以下主题：

+   涉及支付交易的 Apple Pay 主要类，例如 `PKPaymentReqest` 和 `PKPaymentSummaryItem` 类

+   通过 `PKPaymentAuthorizationViewControllerDelegate` 协议对支付表单事件进行响应

+   Apple Pay API 使用的其他类来工作

# 主要类

本节描述了您用于处理 Apple Pay 交易的 Apple Pay 主要类。这些类包括：

+   `PKPaymentButton`: 您使用这个类来显示一个**支付**按钮，用户通过这个按钮开始 Apple Pay 交易。

+   `PKPaymentRequest`: 这个类代表一个支付请求，用户在支付表单中授权或取消该请求。

+   `PKPaymentSummaryItem`: 这个类的实例代表支付表单中的摘要项（例如小计、运费和总计）。

+   `PKPaymentMethod`: 这个类提供了访问 Apple Pay 交易中使用的支付卡信息的方式。

+   `PKShippingMethod`: 这个类的实例代表您支持的运输方式。

+   `PKPaymentAuthorizationViewController`: 这个类用于确定用户是否可以在设备上使用 Apple Pay。它还用于在屏幕上显示支付表单。

+   `PKPayment`: 这个类的实例存储 Apple Pay 交易的支付信息。

+   `PKPaymentToken`: 这个类的实例包含加密的支付信息，准备提交给支付处理器以对用户的支付卡进行收费。

## PKPaymentButton 类

`PKPaymentButton` 类为用户提供了一个点击以启动 Apple Pay 交易的按钮。您应该仅使用 Apple Pay 按钮来启动 Apple Pay 交易。

在显示 **Apple** **Pay** 按钮之前，您必须通过调用 `PKPaymentAuthorizationViewController` 类的 `canMakePayments` 方法来确保设备支持 Apple Pay。您还应该调用同一类的 `canMakePaymentsUsingNetworks:` 或 `canMakePaymentsUsingNetworks:capabilities:` 来确保用户的支付卡支持您所需的支付网络和支付处理器功能。有关详细信息，请参阅本节后面的 *`PKPaymentAuthorizationViewController` 类*。

如果设备不支持 Apple Pay，或者用户添加到设备上的支付卡与您所需的支付网络和支付处理器功能不兼容，请使用带有标题如 **Buy** 的标准按钮来启动标准支付交易。

### 创建按钮

使用此方法创建一个 **Apple** **Pay** 按钮。仅使用 Apple Pay 按钮进行 Apple Pay 交易。

#### +buttonWithType:style

此方法提供了一个 **Apple** **Pay** 按钮。

这里是参数：

+   `type` (`PKPaymentButtonType`): 这指定了按钮的标题。选项有普通（**Pay**）、购买（**Buy with Pay**）和设置（**Set up Pay**）。要了解更多关于使用最后一个选项的信息，请参阅 Apple 开发者库中的 *`Apple Pay Programming Guide`*。

+   `style` (`PKPaymentButtonStyle`): 这指定了按钮的外观。使用它来确保按钮从背景中突出。选项有白色、白色轮廓和黑色。

## `PKPaymentRequest` 类

`PKPaymentRequest` 类代表对特定商品的支付请求。这是您在 Apple Pay 交易中使用的主体类。您可以在每个 Apple Pay 交易中使用此类的单个实例。支付请求包含有关请求支付的商业信息、支付处理的国家、使用的货币、请求支付的项目的价格（包括运费、税费等），以及支持的支付网络（如 Amex、Discover 和 Visa）。

当用户在支付表中授权支付请求时，您将获得一个 `PKPayment` 对象，您需要将其提交给您的支付网关以获取一个 `PKPaymentToken` 对象（或替代品），然后您可以使用它来在您的订单处理系统中对用户的支付卡进行收费。

### 支付处理信息

这些必需的属性指定了您的支付网关用于处理支付的信息。这些应该是您在支付请求上设置的第一件事。

#### countryCode (NSString*)

这是一个表示支付处理国家（ISO 3166）的两个字母代码，例如 `"US"`。

此代码将支付请求的 `countryCode` 属性设置为 `US`，如下所示：

[PRE0]

#### currencyCode (NSString*)

这是用于处理支付的货币的三位代码（ISO 4217），例如 `"USD"`。

#### merchantCapabilities (PKMerchantCapability)

这些是您的支付网关支持的支付处理协议。3-D Secure 协议是必需的。其他可用的协议包括 EMV（欧联、万事达卡、维萨卡）、信用卡和借记卡。

这些常量标识了苹果支付支持的以下支付处理能力：

+   `PKMerchantCapability3DS`

+   `PKMerchantCapabilityEMV`

+   `PKMerchantCapabilityCredit`

+   `PKMerchantCapabilityDebit`

以下代码将支付请求的 `merchantCapabilities` 属性设置为 3D Secure 和 EMV：

[PRE1]

#### merchantIdentifier (NSString*)

这是一个存储在您项目 `Entitlements` 文件中的商户标识符。

#### supportedNetworks (NSArray<NSString*>*)

这包括您支持的支付网络（或关联）。可用的选项有发现卡、万事达卡、维萨卡和私人标签（商店卡）。

这些常量标识了苹果支付支持的以下支付网络：

+   `PKPaymentNetworkAmex`

+   `PKPaymentNetworkDiscover`

+   `PKPaymentNetworkMasterCard`

+   `PKPaymentNetworkPrivateLabel`

+   `PKPaymentNetworkVisa`

例如，此代码将支付请求的 `supportedNetworks` 属性设置为美国运通和万事达卡：

[PRE2]

### 支付摘要项目

摘要项目标识并定价客户购买的产品（小计）和相关项目，如税费、运费和总计。

#### paymentSummaryItems (NSArray<PKPaymentSummaryItem*>*)

这是一个摘要项目数组，指定每个项目的标签和价格。此外，您可以将项目标识为 *待定*（价格尚未设置，但在服务完成后将确定，例如出租车行程）或 *最终*。有关详细信息，请参阅 *PKSummaryItem 类* 部分。

### 必要的地址字段

这些属性指定了您需要在支付表中输入的账单和发货地址的字段，以便它们在用户授权支付请求时在支付表提供的 `PKPayment` 对象中可用。

#### requiredBillingAddressFields (PKAddressField), requiredShippingAddressFields (PKAddressField)

如果您需要在发货信息中包含姓名、电子邮件和邮政地址，可以使用类似以下代码：

[PRE3]

这些是地址字段指定常量：

+   `PKAddressFieldNone`（不需要任何字段）

+   `PKAddressFieldPostalAddress`

+   `PKAddressFieldPhone`

+   `PKAddressFieldEmail`

+   `PKAddressFieldName`

+   `PKAddressFieldAll`

### 账单和发货联系人

这些属性指定了账单和发货联系信息。您可以在显示支付表之前设置它们（然而，这并不推荐，因为用户最有可能拥有最新的联系信息）。用户也可以通过在支付表中选择或指定不同的联系人来更改它们。

#### billingContact (PKContact*), shippingContact (PKContact*)

这些是 `PKContact` 对象的组成部分：

+   `emailAddress (NSString*)`

+   `name (NSPersonNameComponents*)`

+   `phoneNumber (CNPhoneNumber*)`

+   `postalAddress (CNPostalAddress*)`

### 运输方式

运输方式是你支持的运输选项（承运人或速度）。

#### shippingMethods (NSArray<PKShippingMethod*>*)

此属性包含客户可用的运输方式集合。

每个`PKShippingMethod`对象都有以下属性：

+   `detail (NSString*)`: 这向用户描述了运输方式，例如 `"四天内送达"`。

+   `identifier (NSString*)`: 这在应用中标识运输方式。当用户选择一种运输方式时，使用此属性来识别所选择的方法。

### 运输类型

这指定了产品运输的方式，例如是否由商家发货，或者由客户从商家的店铺取货。

#### shippingType (PKShippingType)

这些是可用的运输类型：

+   `PKShippingTypeShipping` (默认): 商家使用客户选择的运输方式将产品运送给客户。

+   `PKShippingTypeDelivery`: 商家（使用车辆）将产品运送给客户。这用于食品、杂货、家具或不能运输的特殊商品。

+   `PKShippingTypeStorePickup`: 客户从商家的店铺取货。在这种情况下，将`shippingContact`设置为你的店铺地址或通过将`requiredShippingAddressFields`设置为`PKAddressFieldNone`来隐藏它从支付单中。请记住，以可靠的方式与客户确认取货地点。

+   `PKShippingTypeServicePickup`: 商家从`shippingContact`中指定的地址取货（例如，当客户支付要求送货时）。

### 应用数据

此属性存储了一个`NSData`对象的哈希值。

#### applicationData (NSData*)

当设置时，此属性包含指定数据的哈希值（然而，它不包含数据的值）。如果你的支付网关支持它，从支付网关API获取的`PKPaymentToken`对象的`paymentData`属性设置为当你设置`applicationData`时生成的相同哈希值。你可以将此哈希值作为发送支付信息到你的订单处理服务器的一部分发送。另外，你可以将实际数据发送到你的服务器。然后你可以在服务器上计算哈希值，并将两个哈希值进行比较，以确保数据在传输过程中未被更改。

## PKPaymentSummaryItem类

`PKPaymentSummaryItem` 类代表客户支付的项，例如代表产品价格、税费、运费等的总计。你可以将一个或多个项标识为“挂起”（价格必须在未来的某个时间才能确定）或“最终”。支付单不会显示“挂起”项的价格；因此，你应该将这些项的价格设置为零。当服务（如出租车行程）完成时，你可以计算实际价格并在交易完成后向客户报告。

### 创建摘要项

这些方法创建摘要项。

#### +summaryItemWithLabel:amount:

这创建了一个带有标签和金额的“最终”摘要项。

#### +summaryItemWithLabel:amount:type:

这创建了一个带有标签和金额的“挂起”或“最终”摘要项。

以下代码创建了一个“挂起”摘要项：

[PRE4]

### 摘要项组件

这些属性访问摘要项的组件。

#### 标签（NSString*）

这是摘要项的标签。

#### 金额（NSDecimalNumber*）

这是摘要项的价格（如果项目是“最终”，则显示在支付单上）。

### 挂起或最终

这个属性决定了摘要项的类型。

#### 类型（PKPaymentSummaryItemType）

这指定了项目是“挂起”还是“最终”。

这是 `type` 属性的可能值：

+   `PKPaymentSummaryItemTypeFinal`

+   `PKPaymentSummaryItemTypePending`

## `PKPaymentMethod` 类

`PKPaymentMethod` 类提供了访问在苹果支付交易中使用的支付卡信息的权限。当用户在支付单中选择支付卡时，支付单会调用其代理的 `paymentAuthorizationViewController:didSelectPaymentMethod:completion:` 方法。

### 卡名称

这个属性标识了 `payment` 方法的支付卡。

#### displayName（NSString*）

这标识了支付卡给用户。

### 卡类型

这个属性标识了支付卡的类型。

#### 类型（PKPaymentMethodType）

这标识了使用的支付卡类型（Amex、Visa 等）。旧卡可能没有这项信息；在这种情况下，此属性的值为 `PKPaymentMethodTypeUnknown`。

这些常量标识了苹果支付支持的卡片类型：

+   `PKPaymentMethodTypeUnknown`

+   `PKPaymentMethodTypeDebit`

+   `PKPaymentMethodTypeCredit`

+   `PKPaymentMethodTypePrepaid`

+   `PKPaymentMethodTypeStore`

### 支付网络

这个属性标识了支付网络。

#### 网络（NSString*）

这标识了卡的支付网络（或协会）。

这些常量标识了苹果支付支持的支付网络：

+   `PKPaymentNetworkAmex`

+   `PKPaymentNetworkDiscover`

+   `PKPaymentNetworkMasterCard`

+   `PKPaymentNetworkPrivateLabel`（这是为商店卡）

+   `PKPaymentNetworkVisa`

### 支付密码

这个属性与支付密码一起使用。

#### paymentPass（PKPaymentPass*）

你可以用这个，例如，来支持定制的品牌支付卡。查看苹果的文档以获取更多详细信息。

## `PKShippingMethod` 类

`PKShippingMethod`类代表您为向客户交付商品所支持的配送方式。

### 用户友好的描述

此属性向用户提供有关配送方式的信息。

#### 详细信息（NSString*）

这向用户描述配送方式，例如：“四天内送达”。

### 应用级标识符

此属性标识了应用内的配送方式。

#### 标识符（NSString*）

当用户更改配送方式时，使用此属性来识别所选的配送方式。

## `PKPaymentAuthorizationViewController`类

`PKPaymentAuthorizationViewController`类有两个功能：

+   确定用户是否可以使用Apple Pay进行支付

+   向用户展示付款单

此类通过`PKPaymentAuthorizationViewControllerDelegate`协议的方法与其代理（您实现）进行通信。

### 确定Apple Pay支持

以下方法确定设备是否支持Apple Pay，以及用户添加的支付卡是否支持特定的支付网络和支付处理功能。

#### +canMakePayments

这表示设备是否支持Apple Pay。

例如，要确定设备是否支持Apple Pay，可以使用类似以下代码：

[PRE5]

#### +canMakePaymentsUsingNetworks:

这表示用户是否可以通过您支持的网路使用Apple Pay。如果用户尚未将支付卡添加到设备，则此方法返回`NO`。

在调用`canMakePayments`之后，您可以调用此方法。

例如，要仅接受Visa或Amex的支付卡，可以使用类似以下代码：

[PRE6]

这些常量标识了Apple Pay支持的支付网络：

+   `PKPaymentNetworkAmex`

+   `PKPaymentNetworkDiscover`

+   `PKPaymentNetworkMasterCard`

+   `PKPaymentNetworkPrivateLabel`（这是为商店卡）

+   `PKPaymentNetworkVisa`

#### +canMakePaymentsUsingNetworks:capabilities:

这表示用户是否可以通过您支持的支付网络和支付处理能力（如3D Secure和EMV）在设备上使用Apple Pay。如果用户尚未将兼容的支付卡添加到设备，则此方法返回`NO`。

在调用`canMakePayments`之后，您可以调用此方法。

例如，要仅接受支持3D Secure和EMV协议的Visa或Amex支付卡，可以使用此代码：

[PRE7]

这些常量标识了Apple Pay支持的支付处理功能：

+   `PKMerchantCapability3DS`

+   `PKMerchantCapabilityEMV`

+   `PKMerchantCapabilityCredit`

+   `PKMerchantCapabilityDebit`

### 初始化和展示

以下方法准备并展示付款单。

#### -initWithPaymentRequest:

此方法使用支付请求初始化一个分配的`PKPaymentAuthorizationViewController`对象。

#### -initWithPaymentRequest:

如果用户可以使用Apple Pay，则此方法使用提供的`PKPaymentRequest`对象初始化一个支付授权视图控制器（付款单）。

你可以在配置好支付请求后调用这个方法。

### 支付表单代理

这个属性标识了支付表单报告支付请求更改的对象：

#### 代理 id <PKPaymentAuthorizationViewControllerDelegate>

这是采用`PKPaymentAuthorizationViewControllerDelegate`协议以响应支付表单用户事件的对象。

## `PKPayment`类

`PKPayment`类表示用户授权的支付。当用户在支付表单中授权支付请求时（通过`PKPaymentAuthorizationViewControllerDelegate`协议中的`paymentAuthorizationViewController:didAuthorizePayment:completion:`方法），你会得到一个`PKPayment`对象。`PKPayment`对象包含了支付网关和发行和收购银行处理支付、向用户收费并将资金存入你账户所需的支付信息。这同样是为了你的订单处理Web应用程序在发行银行授权交易后启动服务或向客户发货。

### 支付信息

这个属性包含了关于支付的信息，它标识了使用的支付卡、参与交易的银行以及其他细节：

#### token (PKPaymentToken*)

这是支付网关、发行银行和收购银行使用的加密支付信息。

### 账单和发货联系人

这个属性包含了在初始化支付表单时标记为“必需”的账单和发货信息：

#### billingContact (PKContact*)

这包含了交易中的账单联系信息。只有支付请求中`requiredBillingAddressFields`属性中标识的字段会被填充。如果没有请求任何字段，这个属性将是`nil`。

#### shippingContact (PKContact*)

这是交易中的发货联系人信息。只有支付请求中`requiredShippingAddressFields`属性中标识的字段会被填充。如果没有请求任何字段，这个属性将是`nil`。

#### shippingMethod

这标识了用户从支付请求中`shippingMethods`属性列出的发货方式中选择的方法。

## `PKPayment`类

一个`PKPaymentToken`对象包含了支付网关和银行使用的加密支付信息，用于资助支付所使用的支付卡类型（借记卡、信用卡或私人标签），以及交易的标识符。

注意，一些支付网关API在其工作流程中不使用这个类。相反，它们提供自己的支付令牌类。使用Stripe的示例项目使用`STPToken`类。查看`ProductCard`类中的`process_ApplePay_payment_with_Stripe:completion:`方法（`ProductCard.m`）以获取详细信息。

### 加密支付信息

这个属性仅由授权的实体解密，例如你的支付网关。

#### paymentData (NSData*)

这是交易的加密支付信息。

### 支付方法和交易标识符

这些属性提供了有关支付令牌的信息。

#### paymentMethod (PKPaymentMethod*)

这是用于交易的支付方法。

#### transactionIdentifier (NSString*)

这是您的支付网关分配的交易标识符。

# 支付表单用户事件协议

本节描述了支付表单用于与其代理通信的协议。

## `PKPaymentAuthorizationViewControllerDelegate` 协议

`PKPaymentAuthorizationViewControllerDelegate` 协议指定了支付表单用于将其事件传达给其代理（您可以实现以响应这些方法的调用）的方法集。以下各节确定了用户可以在支付表单中执行的操作，以及当用户执行这些操作时调用的代理方法。

### 用户选择支付卡

用户可以在支付表中切换支付卡（即，如果用户在设备上的Apple Pay中添加了多个支付卡）。

#### -paymentAuthorizationViewController:didSelectPaymentMethod: completion:

当用户选择支付卡时，会调用此方法。如果支付卡会影响支付请求的摘要项（例如，如果您提供使用商店卡的折扣），则必须计算支付请求的摘要项（这些是`PKPaymentSummaryItem`对象数组）。在任何情况下，您都必须将摘要项数组作为此方法完成处理程序的参数提供。

### 用户选择收货地址

用户可以从之前输入的地址列表中选择一个收货地址，或者在支付表单中输入一个新的地址。

#### -paymentAuthorizationViewController:didSelectShippingContact: completion:

用户指定收货地址后，会调用此方法。支付表单在`PKContact* contact`参数中提供收货地址。此收货地址已匿名化（即，不包括个人信息）。在美国，这些信息仅包括城市、州和邮政编码。

如果您的运输方法根据运输地址而变化，在此方法中，您应创建一个包含适用于新运输地址的运输方法的数组。您还应该使用重新计算的运输方法数组中的第一个运输方法重新计算支付请求的摘要项数组以计算运输费用（支付表单选择运输方法数组中的第一个运输方法）。

此方法的完成块接受三个参数：`status`、`shippingMethods`和`summaryItems`：

+   `status`：这表示收货地址是否有效。如果用户指定的收货地址正确，则将此参数设置为`PKPaymentAuthorizationStatusSuccess`。否则，将其值设置为`PKPaymentAuthorizationStatusInvalidShippingPostalAddress`。

+   `shippingMethods`：这是重新计算的运输方法数组。

+   `summaryItems`：这是重新计算的摘要项数组。

### 用户选择一个配送方式

用户可以从您支持的配送方式列表中选择配送方式（这存储在支付请求的`shippingMethods`属性中）。

#### -paymentAuthorizationViewController:didSelectShippingMethod: completion:

当用户选择配送方式时，会调用此方法。如果用户选择的配送方式会影响支付请求的摘要项（例如，如果配送方式有不同的价格），则必须重新计算支付请求的摘要项（`PKPaymentSummaryItem`对象数组）。

此方法的完成块接受两个参数：`status`和`summaryItems`。

+   `status`：这表示配送方式和配送地址（在调用`paymentAuthorizationViewController:didSelectShippingContact:completion:`代理方法时指定）是否有效。如果用户指定的配送地址正确，则将此参数设置为`PKPaymentAuthorizationStatusSuccess`。否则，将其值设置为`PKPaymentAuthorizationStatusInvalidShippingPostalAddress`。

+   `summaryItems`：这是重新计算的摘要项数组。

### 用户授权支付请求

用户在支付单中确认或更改信息后，用户可以授权支付请求以进行Apple Pay交易。

#### -paymentAuthorizationViewControllerWillAuthorizePayment:

在用户验证支付单之后但在调用`paymentAuthorizationViewController:didAuthorizePayment:completion:`之前调用此方法。使用此方法进行授权准备。

#### -paymentAuthorizationViewController:didAuthorizePayment: completion:

在用户授权支付请求之后调用此方法。这是您使用支付网关处理支付的地方。完成后，您必须使用状态参数的适当值调用完成处理程序块（`(void (^)(PKPaymentAuthorizationStatus status)) completion`）。

这是完成处理程序`status`参数的可能值：

+   `PKPaymentAuthorizationStatusSuccess`：这表示支付卡的发行银行成功授权支付

+   `PKPaymentAuthorizationStatusFailure`：这表示由于支付卡的发行银行授权失败而失败

+   `PKPaymentAuthorizationStatusInvalidBillingPostalAddress`：这表示由于无效的账单地址而失败

+   `PKPaymentAuthorizationStatusInvalidShippingPostalAddress`：这表示由于无效的配送地址而失败

+   `PKPaymentAuthorizationStatusInvalidShippingContact`：这表示由于无效的配送联系人而失败

### 支付单完成

用户授权支付请求并在应用处理Apple Pay交易后，或者用户取消支付单时，支付单完成。

#### -paymentAuthorizationViewControllerDidFinish:

当交易完成或用户取消支付表单时调用。您必须关闭支付表单并执行任何其他必要的更新。

# 辅助类

本节描述了您必须使用的一些附加类，以使用Apple Pay API。这些类包括：

+   `PKContact`: 这用于表示联系信息（姓名、电话号码和电子邮件）和地址

+   `CNPhoneNumber`: 这用于表示电话号码

+   `CNPostalAddress`: 这用于表示地址

## PKContact类

`PKContact`类包含表示账单或配送地址组成部分的属性。支付表单在用户选择配送地址时（`paymentAuthorizationViewController: didSelectShippingContact:completion:`）向其代理提供此类的实例，并在用户授权支付请求时（`paymentAuthorizationViewController:didAuthorizePayment:completion:`）在`PKPayment`对象中提供。

## 联系地址组成部分

这些属性包含了`PKPaymentRequest`对象中`requiredBillingAddressFields`或`requiredShippingAddressFields`属性请求的信息。对应未请求地址字段的属性值为`nil`。

### emailAddress (NSString*)

这是联系人的电子邮件地址。

### name (NSPersonNameComponents*)

这是联系人的名字。

### phoneNumber (CNPhoneNumber*)

这是联系人的电话号码。

### postalAddress (CNPostalAddress*)

这是联系人的邮政地址。

## NSPersonNameComponents类

`NSPersonNameComponents`类包含表示个人姓名组成部分的属性。

### 个人姓名组成部分

这些属性表示个人姓名的组成部分。

#### namePrefix (NSString*)

这是在人的名字前使用的称呼（或头衔），例如 `"Dr."`，`"Ms."` 或 `"Mr."`。

#### givenName (NSString*)

这是人的给定名（或名）。

#### middleName (NSString*)

这是人的第二个名字。

#### familyName (NSString*)

这是人的姓氏（通常与兄弟姐妹共享）。

#### nameSuffix (NSString*)

这些是添加到个人全名中的字母，用于提供有关个人的额外信息，例如 `"Jr."` 或 `"Ph.D."`。

#### nickname (NSString)

这是用于人的熟悉或幽默的名字。

#### phoneticRepresentation (NSPersonNameComponents*)

如果指定了，这是本类中除`phoneticRepresentation`之外其他每个属性的音标表示。

## CNPhoneNumber类

`CNPhoneNumber`类代表`PKContact`对象的电话号码组件。

### 创建电话号码

当创建`PKContact`对象时，要设置其`phoneNumber`属性，请使用此方法创建存储在属性中的`CNPhoneNumber`值。

例如，此代码设置联系人的电话号码：

[PRE8]

#### +phoneNumberWithStringValue:

这会创建一个带有提供文本的电话号码组件。

### 电话号码字符串

此属性包含表示联系人电话号码的字符串。

#### stringValue (NSString*)

这是表示电话号码的文本。

## CNPostalAddress 类

`CNPostalAddress` 类表示 `PKContact` 对象的邮政地址组件。

### 邮政地址组件

这些属性表示地址的组成部分。

#### street (NSString*)

这是联系人的街道地址。

#### city (NSString*)

这是联系人所在的城市。

#### state (NSString*)

这是联系人所在的状态。

#### postalCode (NSString*)

这是联系人所在地的邮政（或邮政编码）代码。

#### country (NSString*)

这是联系人所在的国家。

#### ISOCountryCode (NSString*)

这是国家属性的 ISO 3166 国家代码。

# 摘要

本章描述了您必须使用以处理 Apple Pay 支付的大多数类。它还描述了支付表单用于将用户操作传达给其代理的协议，以便您可以适当地做出响应。最后，本章列出了几个您必须与之合作的附加类，例如访问地址组件，如人的姓名、电话号码和邮政地址。

这标志着 *Apple Pay Essentials* 的结束，旨在帮助你向应用程序添加 Apple Pay 支持。本书介绍了在线支付的世界，描述了支付协会、支付网关和银行如何共同工作，将客户的支付卡资金转移到商家的账户。Apple Pay 通过增加便利性、隐私性和安全性来改进这一模型。

你学习了如何在客户端应用程序中使用 REST API 与提供所需信息的服务器网络应用程序（例如销售产品的详细信息和支持的运输方式）进行通信，以及处理来自应用程序的支付和订单。

本书提供了一个示例工作流程，用于向客户显示产品信息，并在适当的时候显示 **Apple** **Pay** 按钮。你学习了如何创建支付请求并将其添加到支付和订单信息中，以及展示主要的 Apple Pay 用户界面和支付表单。本书展示了如何响应用户在支付表单中做出的更改，例如选择不同的支付卡、运输地址或运输方式。

你学习了如何在用户授权支付请求后处理应用程序和后端服务器中的支付和订单信息。本书还展示了如何在交易完成后处理支付表单。

最后，本书描述了订单管理系统的关键元素和流程，使用 `Node.js` 处理订单和支付，以及 Mongoose 模块与基于文档的数据库进行交互。

通过这些知识，你现在能够为你的应用程序用户提供基于 Apple Pay 的支付便利性和安全性。
