# <st c="0">8</st>

# <st c="2">从网络连接和获取数据</st>

找到一个不与服务器连接的应用程序极其困难。<st c="120">大多数应用程序并非独立运行——它们需要验证用户身份、获取信息，并允许用户执行最终将同步回</st> <st c="288">服务器的操作。</st>

<st c="299">因此，了解网络的工作方式很重要——不是了解HTTP的一般工作方式，而是了解iOS应用程序如何高效且简单地与服务器交互</st> <st c="445">。</st>

<st c="456">在本章中，我们将涵盖以下主题：</st>

+   <st c="509">理解</st> <st c="524">移动网络</st>

+   <st c="541">处理HTTP请求，包括</st> <st c="576">它们的响应</st>

+   <st c="591">在应用程序流程中集成网络调用</st>

+   <st c="634">探索Combine如何与网络</st> <st c="663">一起工作</st>

<st c="678">让我们开始理解网络如何融入我们的</st> <st c="735">应用程序架构。</st>

# <st c="752">技术要求</st>

<st c="775">对于本章，您必须从Apple的App Store下载Xcode版本15.0或更高版本。</st> <st c="864">在App Store中搜索Xcode，选择并下载最新版本。</st> <st c="942">启动Xcode，并遵循系统可能提示您进行的任何其他安装说明。</st> <st c="1045">一旦Xcode完全启动，您就可以开始了</st> <st c="1089">。</st>

<st c="1095">您需要运行最新版本的macOS（Ventura</st> <st c="1152">或更高版本）。</st>

<st c="1162">您还可以从以下GitHub</st> <st c="1227">链接下载示例代码：</st> [<st c="1233">https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter%208</st>](https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter%208)

# <st c="1318">理解移动网络</st>

<st c="1350">与网络</st> <st c="1375">一起工作是在开发iOS应用程序的过程中一个至关重要的部分。</st> <st c="1422">理解网络如何融入我们的应用程序架构是至关重要的，但它究竟意味着什么？</st> <st c="1520">观看关于执行一个</st> `<st c="1565">URLSession</st>` <st c="1575">请求的简单教程是可以的，但现实世界中的应用程序并不这样工作。</st> <st c="1624">不是这样。</st>

<st c="1633">在我们深入之前，让我们回顾一下基本的应用程序架构</st> <st c="1703">看起来是什么样子：</st>

+   **<st c="1714">UI层</st>**<st c="1723">：这是负责向用户展示</st> <st c="1760">UI，包括响应用户输入。</st> <st c="1814">UI层包括SwiftUI/UIKit视图和</st> <st c="1867">视图模型。</st>

+   **<st c="1879">业务逻辑</st>**<st c="1894">：这是负责在管理基本</st> <st c="1938">应用程序逻辑的同时操作数据</st> <st c="1964">。</st>

+   **<st c="1982">数据层</st>**：这是负责存储和检索与业务逻辑相关的数据实体的。

我猜这个三层架构不会让你感到惊讶，因为大多数移动应用都采用类似的架构。

当我们开始理解网络任务的位置时，我们将不得不查看数据层，在某些特定情况下，业务逻辑层（例如，当与分析或第三方库一起工作时）也需要查看。然而，为什么我们需要查看这些层呢？为了理解为什么我们的网络活动与数据层相关，让我们回顾我们的主要网络目标：

+   **<st c="2585">同步信息</st>** <st c="2605">到和从</st> <st c="2618">我们的后端</st>

+   处理**<st c="2639">认证</st>**

+   需要服务器的逻辑活动

在大多数应用中，网络是同步数据与后端所必需的。数据层作为实体真实信息的首要存储库运行。尝试从其他层直接从网络访问实体将破坏这一原则。

*<st c="2936">图 8</st>**<st c="2945">.1</st>* 显示了一个基本的应用架构和网络位置：

![图 8.1：基本应用架构](img/B21795_08_1.jpg)

图 8.1：基本应用架构

在 *<st c="3091">图 8</st>**<st c="3099">.1</st>*<st c="3101"> 中，我们可以看到网络是构建数据层的一个组件。大多数应用的基本思想是网络成为数据源，并填充应用的数据存储。

例如，在一个音乐应用中，网络层可能会连接到后端，获取专辑和歌曲，并将它们存储在本地存储中，例如**<st c="3431">Core Data</st>**。网络层也是建立在不同的组件之上以正确运行的。

我们可以将网络操作想象成一个工厂生产线。我们请求一条信息，并处理返回的数据包，将其通过几个阶段传输，直到我们将其正确地存储在我们的本地存储中或展示出来。

在我们回顾数据包可能经历的各个阶段之前，让我们尝试一起构建一个网络请求。我们将从回顾基本的 HTTP 请求方法开始。

# 处理 HTTP 请求

<st c="3950">HTTP请求是客户端发送到服务器以请求信息和执行操作的</st> <st c="3979">消息。</st> <st c="4055">服务器处理该请求并向客户端返回响应。</st> <st c="4127">客户端确实会异步执行HTTP请求以释放主线程。</st> <st c="4210">然而，客户端与服务器之间的连接是</st> **<st c="4267">同步</st>** <st c="4278">的，因为客户端等待服务器的响应以完成</st> <st c="4341">请求操作。</st>

<st c="4359">HTTP请求的主要组件是请求方法，它指示请求的主要目标。</st> <st c="4459">现在让我们来了解一下一些基本的HTTP</st> <st c="4496">方法。</st>

## <st c="4508">基本的HTTP请求方法</st>

<st c="4535">REST API基于请求-响应风格的理念，并且是与后端单向通信。</st> <st c="4557">REST API在与后端通信时共有八种方法可供使用。</st> <st c="4658">然而，在大多数情况下，我们将使用以下</st> <st c="4783">四种方法：</st>

+   `<st c="4796">GET</st>`<st c="4800">：这用于仅从服务器检索信息。</st> <st c="4839">它应该是一个安全的调用，这意味着执行一个</st> `<st c="4914">GET</st>` <st c="4917">请求不应该影响</st> <st c="4947">后端数据。</st>

+   `<st c="4960">POST</st>`<st c="4965">：通常使用</st> `<st c="4972">POST</st>` <st c="4976">方法向后端提交</st> <st c="5007">数据。</st> <st c="5029">在许多情况下，</st> `<st c="5048">POST</st>` <st c="5052">方法在后台数据存储中执行更改或更改</st> <st c="5116">用户状态。</st>

+   `<st c="5127">PUT</st>`<st c="5131">：我们使用</st> `<st c="5141">PUT</st>` <st c="5144">来创建或更新</st> <st c="5164">对象。</st> <st c="5174">与</st> `<st c="5185">POST</st>` <st c="5189">方法不同，</st> `<st c="5198">PUT</st>` <st c="5201">被认为是幂等的。</st> <st c="5228">我们可以发送多个相同的</st> `<st c="5259">PUT</st>` <st c="5262">请求，并期望与发送一个请求相同的效果。</st>

+   `<st c="5322">DELETE</st>`<st c="5329">：正如其名所示，我们使用</st> `<st c="5359">DELETE</st>` <st c="5365">来删除</st> <st c="5375">对象。</st> <st c="5385">显然，我们可以使用</st> `<st c="5407">POST</st>` <st c="5411">来做这件事，但使用</st> `<st c="5433">DELETE</st>`<st c="5439">，我们与</st> <st c="5461">标准保持一致。</st>

值得一提的是，技术上我们甚至可以使用<st c="5475">以下内容</st> `<st c="5534">GET</st>` <st c="5537">来更改服务器。</st> <st c="5569">然而，正确的方法确保了可预测性和可靠性，并且与</st> <st c="5659">REST原则</st>相一致。

<st c="5675">要执行基本的HTTP请求，我们首先应该熟悉</st> `<st c="5746">URLSession</st>` <st c="5756">类。</st>

## <st c="5763">使用URLSession</st>

<st c="5787">我们可以使用一个名为</st> <st c="5806">的类</st> `<st c="5814">URLSession</st>` <st c="5824">来执行和管理</st> <st c="5846">网络请求。</st> `<st c="5865">URLSession</st>` <st c="5875">是苹果所说的</st> **<st c="5908">URL 加载系统</st>**<st c="5926">的一部分，而</st> <st c="5941">URL 加载系统</st> <st c="5926">又是</st> **<st c="5957">Foundation</st>** <st c="5967">框架的一部分。</st>

<st c="5978">`<st c="5983">URLSession</st>` <st c="5993">类负责协调我们应用中的不同URL请求。</st> <st c="6067">让我们看看如何使用</st> `<st c="6100">URLSession</st>` <st c="6113">执行基本的</st> `<st c="6103">GET</st>` <st c="6103">响应：</st>

[PRE0]

<st c="6433">此代码示例创建了一个名为</st> `<st c="6477">URLRequest</st>` <st c="6487">的对象，基于特定的URL。</st> <st c="6515">`<st c="6519">URLRequest</st>` <st c="6529">类封装了执行特定URL请求所需的信息。</st> <st c="6608">它通常由以下信息组成：</st>

+   <st c="6657">请求</st> <st c="6670">基本URL</st>

+   <st c="6678">请求方法 –</st> `<st c="6700">GET</st>`<st c="6703">,</st> `<st c="6705">POST</st>`<st c="6709">,</st> `<st c="6711">PUT</st>`<st c="6714">,</st> <st c="6716">或</st> `<st c="6719">DELETE</st>`

+   <st c="6725">请求</st> <st c="6738">HTTP头</st>

<st c="6750">请注意，</st> `<st c="6767">URLRequest</st>` <st c="6777">结构</st> <st c="6787">不执行实际的HTTP请求</st> <st c="6827">或包含其响应信息。</st> <st c="6865">`<st c="6869">URLSession</st>` <st c="6879">类负责执行和管理不同的</st> <st c="6943">HTTP请求。</st>

<st c="6957">初始化一个</st> `<st c="6993">URLSession</st>` <st c="7003">实例</st> <st c="6976">有两种方式：</st>

+   <st c="7013">我们可以调用静态的</st> `<st c="7037">shared</st>` <st c="7043">属性</st> <st c="7052">并将其用作一个</st> **<st c="7069">单例</st>**<st c="7078">。如果我们想简化我们的实现，而不需要自定义处理请求的方式，或者在不同区域有不同的要求，我们就这样做：</st>

    [PRE1]

+   <st c="7279">如果我们需要更多的灵活性，我们可以创建一个</st> `<st c="7338">URLSession</st>` <st c="7348">实例</st> <st c="7408">（就像上一个代码示例中那样）并用我们自己的配置初始化它。</st>

<st c="7426">一个配置对象允许我们更好地调整我们的请求。</st> <st c="7494">例如，我们可以定义每个请求包含额外的头信息，设置超时和缓存，甚至接受cookie</st> <st c="7616">接受策略。</st>

<st c="7636">以下是一个设置具有特定超时时间和无缓存的</st> `<st c="7676">URLSession</st>` <st c="7686">类代码示例：</st> <st c="7730">：</st>

[PRE2]

let task = session.dataTask(with: request) { (data,

response, error)

[PRE3]

if let error = error as? URLError {

    switch error.code {

    case .cannotFindHost:

        // 通知用户。default:

        print("错误：\(error)")

    }

    return

}

[PRE4]

if let httpResponse = response as? HTTPURLResponse {

            switch httpResponse.statusCode {

            case 200..<300:

                print("成功：

                \(httpResponse.statusCode)")

            case 400..<500:

                print("客户端错误：

                \(httpResponse.statusCode)")

            case 500..<600:

                print("服务器错误：

                \(httpResponse.statusCode)")

            default:

                print("其他状态码：

                \(httpResponse.statusCode)")

            }

        } else {

            print("无效的HTTP响应")

        }

[PRE5]

func handleResponse(data: Data?, response: URLResponse?, error: Error?) throws {

    if let error = error {

        throw error

    }

    guard let httpResponse = response as? HTTPURLResponse

    else {

        throw NetworkingError.invalidResponse

    }

    switch httpResponse.statusCode {

    case 200..<300:

        if let responseData = data {

            if let errorData = try? JSONDecoder().decode(ErrorResponse.self,

            from: responseData) {

                throw NetworkingError.dataError

            }

        }

    case 400..<500:

        throw NetworkingError.clientError(statusCode:

        httpResponse.statusCode)

    case 500..<600:

        throw NetworkingError.serverError(statusCode:

        httpResponse.statusCode)

    default:

        throw NetworkingError.otherError

    }

}

[PRE6]

let task = session.dataTask(with: request) { (data,

response, error) in

    do {

        try handleResponse(data: data, response: response,

        error: error)

    } catch let error {

        print("错误：\(error)")

    }

}

[PRE7]

{

"id": 1,

"name": "John Doe",

"email": "john@example.com"

}

[PRE8]

struct Contact: Codable {

    let id: Int

    let fullName: String

    let userEmail: String

    // 定义自定义编码键以匹配JSON键

    private enum CodingKeys: String, CodingKey {

        case id

        case fullName = "name"

        case userEmail = "email"

    }

}

[PRE9]

let errorData = try? JSONDecoder().decode(Contact.self,

from: responseData)

[PRE10]

struct ServerResponse: Codable {

    let responseID: String

    let timestamp: String

    let orgID: String

    let contact: Contact

}

let jsonString = """

{

"responseID": "12345",

"timestamp": "2024-03-25T12:00:00Z",

"orgID": "5678",

"contact": {

    "id": 1,

    "fullName": "John Doe",

    "userEmail": "john@example.com"

}

}

"""

let jsonData = jsonString.data(using: .utf8)! let response = try

JSONDecoder().decode(ServerResponse.self, from: jsonData)

[PRE11]

class DataStore {

    private var contacts: [Contact] = []

    func updateContacts(with newContacts: [Contact]) {

        contacts = newContacts

    }

    func getAllContacts() -> [Contact] {

        return contacts

    }

}

[PRE12]

class SyncManager {

    private let dataStore: DataStore

    init(dataStore: DataStore) {

        self.dataStore = dataStore

    }

    func syncData() {

        NetworkHandler.fetchData { result in

            switch result {

            case .success(let data):

                do {

                    let contacts = try JSONDecoder().decode([Contact].self, from: data)

                    self.dataStore.updateContacts(with: contacts)

                    print("数据同步成功")

                } catch {

                    print("JSON解码数据错误:", error)

                }

            case .failure(let error):

                print("获取数据错误:", error)

            }

        }

    }

}

[PRE13]

import SwiftUI

struct ContactsView: View {

    @State private var contacts: [Contact] = []

    @State private var isLoading = false

    var body: some View {

        NavigationView {

            List(contacts) { contact in

                VStack(alignment: .leading) {

                    Text(contact.name).font(.headline)

                    Text(contact.phoneNumber).font(.subheadline)

                }

            }

            .navigationTitle("联系人")

            .onAppear {

                fetchContacts()

            }

            .overlay {

                if isLoading {

                    ProgressView("加载中...")

                }

            }

        }

    }

    private func fetchContacts() {

        isLoading = true

        NetworkHandler().fetchData { fetchedContacts in

            contacts = fetchedContacts

            isLoading = false

        }

    }

}

[PRE14]

import SwiftUI

struct ContactsView: View {

    @State private var contacts: [Contact] = []

    var body: some View {

        NavigationView {

            List(contacts) { contact in

                VStack(alignment: .leading) {

                    Text(contact.name).font(.headline)

                    Text(contact.phoneNumber).font(.subheadline)

                }

            }

            .navigationTitle("联系人")

            .onAppear {

                loadContacts()

            }

        }

    }

    private func loadContacts() {

        contacts = loadFromCache()

        NetworkHandler().fetchData { fetchedContacts in

            contacts = fetchedContacts

            saveToCache(contacts: fetchedContacts)

        }

    }

}

[PRE15]

class IncrementalLoader {

    var currentPage = 1

    let itemsPerPage = 10

    var contacts = [Contact]()

    func loadNextPage() {

        guard let url = URL(string:

"https://api.example.com/contacts?page=\(currentPage)&limit=\(itemsPerPage)") else {

            print("无效的URL")

            return

        }

        let task = URLSession.shared.dataTask(with: url) { [weak self] (data, response, error) in

            guard let self = self else { return }

            do {

                let newContacts = try JSONDecoder().decode([Contact].self, from: data)

                DispatchQueue.main.async {

                    self.contacts.append(contentsOf: newContacts)

                    print("获取联系人: \(newContacts)")

                    self.currentPage += 1

                }

            } catch {

                print("JSON解码错误: \(error)")

            }

        }

        task.resume()

    }

}

[PRE16]

class ContactsSyncManager {

    let userDefaults = UserDefaults.standard

    let lastUpdatedKey = "lastUpdatedTime"

    let syncEndpoint = URL(string:

    "https://example.com/api/sync/contacts")! func syncContacts() {

        var request = URLRequest(url: syncEndpoint)

        request.httpMethod = "POST"

        request.addValue("application/json",

        forHTTPHeaderField: "Content-Type")

        let lastUpdatedTime = userDefaults.double(forKey:

        lastUpdatedKey)

        let requestBody = ["lastUpdatedTime":

        lastUpdatedTime]

        request.httpBody = try? JSONSerialization.data(withJSONObject:

        requestBody)

        URLSession.shared.dataTask(with: request) { [weak

        self] data, response, error in

            self?.processDeltaUpdates(response: response)

        }.resume()

    }

[PRE17]

private func processDeltaUpdates(response:

    ContactsDeltaUpdateResponse) {

        // 这里可以根据需要处理新增、删除和更新的联系人

        print("新联系人:

        \(response.newContacts.count)")

        print("Deleted Contacts:

        \(response.deletedContacts.count)")

        print("Updated Contacts:

        \(response.updatedContacts.count)")

        userDefaults.set(response.lastUpdated, forKey:

        lastUpdatedKey)

    }

}

[PRE18]

class ContactRequest {

    func fetchData() -> AnyPublisher<[Contact], Error> {

        let url = URL(string:

        "https://api.example.com/contacts")! return URLSession.shared.dataTaskPublisher(for:

        url)

            .map { $0.data }

            .decode(type: [Contact].self, decoder:

            JSONDecoder())

            .eraseToAnyPublisher()

    }

}

[PRE19]

class DataStore {

    @Published var contacts: [Contact] = []

}

[PRE20]

class ContactsSync {

    let contactRequest = ContactRequest()

    let dataStore = DataStore()

    func syncContacts() { <st c="35679">contactRequest.fetchData()</st> .sink(receiveCompletion: { completion in

                switch completion {

                case .finished:

                    print("Data fetch completed

                    successfully")

                case .failure(let error):

                    print("Error fetching data: \(error)")

                }

            }, receiveValue: { [weak self] contacts in <st c="35935">self?.dataStore.contacts = contacts</st> })

            .store(in: &cancellables)

    }

    private var cancellables = Set<AnyCancellable>()

}

let contactsSync = ContactsSync()

contactsSync.syncContacts()

[PRE21]

URLSession.shared.dataTaskPublisher(for: request)

            .tryMap { output in

                guard let response = output.response as? HTTPURLResponse, response.statusCode ==

                200 else {

                    throw URLError(.badServerResponse)

                }

                return output.data

            }

            .decode(type: ContactsDeltaUpdateResponse.self,

            decoder: JSONDecoder())

            .receive(on: DispatchQueue.main)

            .sink(receiveCompletion: { completion in

                switch completion {

                case .finished:

                    break

                case .failure(let error):

                    print("Error during sync:

                    \(error.localizedDescription)")

                }

            }, receiveValue: { [weak self] response in

                self?.processDeltaUpdates(response:

                response)

            })

            .store(in: &cancellables)

[PRE22]
