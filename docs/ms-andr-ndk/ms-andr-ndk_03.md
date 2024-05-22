# 第三章：网络功能

在本章中，我们将学习如何从本地 C/C++代码处理与网络相关的功能。网络任务是异步的，就时间而言是不可预测的。即使底层连接是通过 TCP 协议建立的，也不能保证交付时间，应用程序在等待数据时完全有可能冻结。在 Android SDK 中，这一点被大量的类和设施所隐藏。而在 Android NDK 中，*相反地*，你必须自己克服这些困难，没有来自任何特定平台帮助者的协助。为了开发响应迅速且安全的应用程序，必须解决许多问题：我们需要完全控制下载过程，限制下载数据的大小，并优雅地处理发生的错误。不过，我们不会深入探讨 HTTP 和 SSL 协议实现的细节，我们将使用 libcurl 和 OpenSSL 库，专注于与应用程序开发相关的高级任务。然而，我们将会更详细地了解如何以可移植的方式实现基本异步机制。本章的前几个例子仅适用于桌面，其目的是展示如何实现跨平台同步原语。但是，在本章的最后，我们将看到如何将这些部分整合到一个移动应用程序中。

# 侵入式智能指针

在多线程环境中跟踪所有本地内存分配是一个出了名困难的流程，特别是在涉及在不同线程间传递对象所有权时。在 C++中，可以使用智能指针自动化内存管理。标准的`std::shared_ptr`类是个不错的起点。然而，我们想要关注更有趣且轻量级的技术。我们也不会使用 Boost 库，因为我们在编译时间上真的想要保持精简。

### 注意

最新版本的 Android NDK 已完全支持 C++ 11 标准库。如果你对`std::shared_ptr`或 Boost 库中的侵入式指针感到更熟悉，可以自由使用这些库中的智能指针。

如其名所示，侵入式智能指针中，引用计数被嵌入到对象中。实现这一点的最简单方式是通过继承以下基类：

```java
class iIntrusiveCounter
{
private:
  std::atomic<long> m_RefCounter;
public:
  iIntrusiveCounter( ) : m_RefCounter( 0 ) {}
  virtual ~iIntrusiveCounter( ) {}
  long GetReferenceCounter( ) const volatile 
  { return m_RefCounter; }
```

它使用标准原子变量来保存计数器的值。在 C++ 11 标准库被广泛采用之前，实现一个可移植的原子计数器需要使用特定平台的原子操作，比如 POSIX 或 Windows。如今，使用 C++ 11 可以编写适用于所有平台的干净代码；无论是 Android、Windows、Linux、OS X、iOS，甚至是黑莓 10，如果你愿意的话。以下是我们可以如何增加计数器的示例：

```java
  void IncRefCount( )
  {
    m_RefCounter.fetch_add( 1, std::memory_order_relaxed );
  }
```

使用 `++` 运算符替代 `fetch_add()` 是完全可行的。然而，编译器要求以这种方式递增原子整数变量需要是顺序一致的，这可能在生成的汇编代码中插入冗余的内存屏障。由于我们不对递增值进行任何决策，这里的内存屏障是不必要的，可以放宽内存排序，只要求变量的原子性。这正是 `fetch_add()` 使用 `std::memory_order_relaxed` 标志所做的，在一些非 x86 平台上可以生成更快的代码。递减要更复杂一些。确实，我们需要决定何时移除对象，只有在引用计数递减到零时才这样做。

这是正确执行操作的代码：

```java
  void DecRefCount()
  {
    if ( m_RefCounter.fetch_sub( 1, std::memory_order_release ) == 1 )
    {
```

`std::memory_order_release` 标志意味着对内存位置的运算需要所有先前的内存写入对所有执行相同位置获取操作的线程可见。进入 `if` 块后，我们将通过插入适当的内存屏障来执行获取操作：

```java
      std::atomic_thread_fence( std::memory_order_acquire );
```

在这一点之后，我们现在可以允许对象执行自杀操作：

```java
      delete this;
    }
  }
};
```

`delete this` 习惯用法在 [`isocpp.org/wiki/faq/freestore-mgmt#delete-this`](https://isocpp.org/wiki/faq/freestore-mgmt#delete-this) 有解释。

### 注意

`iIntrusiveCounter` 类是我们引用计数机制的核心。代码可能看起来非常简单；然而，这个实现的背后逻辑比看起来要复杂得多。有关所有详细细节，请参考 Herb Sutter 的 *C++ and Beyond 2012: Herb Sutter - atomic<> Weapons, 1 of 2* 演讲：

[`channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Herb-Sutter-atomic-Weapons-1-of-2`](http://channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Herb-Sutter-atomic-Weapons-1-of-2)

[`channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Herb-Sutter-atomic-Weapons-2-of-2`](http://channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Herb-Sutter-atomic-Weapons-2-of-2)

现在，我们可以实现一个轻量级的 RAII 泛型智能指针类，它使用我们刚刚编写的计数器基类：

```java
template <class T> class clPtr
{
public:
  /// default constructor
  clPtr(): FObject( 0 ) {}
  /// copy constructor
  clPtr( const clPtr& Ptr ): FObject( Ptr.FObject )
  {
    LPtr::IncRef( FObject );
  }
```

在这里，复制构造函数没有直接调用 `FObject->IncRefCount()` 方法。而是调用一个辅助函数 `LPtr::IncRef()`，它接受 `void*` 并将对象作为参数传递给该函数。这样做是为了允许我们的侵入式智能指针与那些已声明但尚未定义的类一起使用：

```java
  /// move constructor
  clPtr( clPtr&& Ptr :): FObject( Ptr.FObject )
  {
    Ptr.FObject = nullptr;
  }
  template <typename U> clPtr( const clPtr<U>& Ptr )): FObject( Ptr.GetInternalPtr() )
  {
    LPtr::IncRef( FObject );
  }
```

从 `T*` 的隐式构造函数很有用：

```java
  clPtr( T* const Object ): FObject( Object )
  {
    LPtr::IncRef( FObject );
  }
```

与构造函数类似，析构函数使用辅助函数来递减引用计数：

```java
  ~clPtr()
  {
    LPtr::DecRef( FObject );
  }
```

若干个命名辅助函数可用于检查智能指针的状态：

```java
  /// check consistency
  inline bool IsValid() const
  {
    return FObject != nullptr;
  }
  inline bool IsNull() const
  {
    return FObject == nullptr;
  }
```

与其他方法相比，赋值运算相当慢：

```java
  /// assignment of clPtr
  clPtr& operator = ( const clPtr& Ptr )
  {
    T* Temp = FObject;
    FObject = Ptr.FObject;
    LPtr::IncRef( Ptr.FObject );
    LPtr::DecRef( Temp );
    return *this;
  }
```

但是不包括 `move` 赋值运算符：

```java
  clPtr& operator = ( clPtr&& Ptr )
  {
    FObject = Ptr.FObject;
    Ptr.FObject = nullptr;
    return *this;
  }
```

`->` 运算符对于每个智能指针类都是必不可少的：

```java
  inline T* operator -> () const
  {
    return FObject;
  }
```

这是一个有点棘手的问题：一个自动类型转换运算符，用于将智能指针转换为私有类 `clProtector` 的实例：

```java
  inline operator clProtector* () const
  {
    if ( !FObject ) return nullptr;
    static clProtector Protector;
    return &Protector;
  }
```

这种类型转换用于允许像`if ( clPtr )`这样的安全空指针检查。这是安全的，因为您不能对生成的指针执行任何操作。内部的私有类`clProtector`没有实现`delete()`运算符，因此使用它将产生编译错误：

```java
private:
  class clProtector
  {
private:
    void operator delete( void* ) = delete;
  };
```

### 注意事项

本书的源代码包没有使用 C++ 11 的`= delete`表示法来删除函数，只是让它未实现。这是为了与旧编译器保持兼容性。如果你针对的是最新版本的 GCC/Clang 和 Visual Studio，使用`= delete`将是非常好的。

让我们回到我们的`clPtr`类。不幸的是，标准`dynamic_cast<>`运算符不能以原始方式使用，因此我们需要进行替换：

```java
public:
  /// cast
  template <typename U> inline clPtr<U> DynamicCast() const
  {
    return clPtr<U>( dynamic_cast<U*>( FObject ) );
  }
```

这是我们的智能指针在语法上与原始指针唯一不同的地方。此外，我们需要一组比较运算符，以使我们的类在不同的容器中更有用：

```java
  template <typename U> inline bool operator == ( const clPtr<U>&Ptr1 ) const
  {
    return FObject == Ptr1.GetInternalPtr();
  }
  template <typename U> inline bool operator == ( const U* Ptr1 )const
  {
    return FObject == Ptr1;
  }
  template <typename U> inline bool operator != ( const clPtr<U>&Ptr1 ) const
  {
    return FObject != Ptr1.GetInternalPtr();
  }
```

这是一个函数，用于简化智能指针与接受原始指针的 API 之间的连接。到基础`T*`类型的转换应该是显式的：

```java
  inline T* GetInternalPtr() const
  {
    return FObject;
  }
```

当处理低级指针问题时，一些辅助函数可能很有用。删除对象，不要释放它：

```java
  inline void Drop()
  {
    FObject = nullptr;
  }
```

清除对象，减少引用计数，类似于将其赋值为`nullptr`：

```java
  inline void Clear()
  {
    *this = clPtr<T>();
  }
```

最后但同样重要的是，指针本身：

```java
private:
  T* FObject;
};
```

从此，我们可移植的侵入式智能指针是自包含的，可以用于实际应用中。还有一件事要做，那就是一种语法糖。C++ 11 典型的使用`auto`关键字，这样可以在表达式中只写一次类型名称。但是，下面的实例化将不起作用，因为当我们希望`p`的类型是`clPtr< clSomeObject>`时，推导出的`p`的类型将是`clSomeObject*`：

```java
auto p = new clSomeObject( a, b, c );
```

使用标准共享指针时，通过使用`std::make_shared()`模板辅助函数来解决此问题，该函数返回正确的类型（并在幕后进行一些有用的计数器存储优化）：

```java
auto p = std::make_shared<clSomeObject>( a, b, c );
```

在这里，`p`的推导类型是`std::shared_ptr<clSomeObject>`，最终符合我们的预期。我们可以使用 C++ 11 提供的完美转发机制和`std::forward()`函数创建一个类似的辅助函数：

```java
template< class T, class... Args > clPtr<T> make_intrusive( Args&&... args )
{
  return clPtr<T>( new T( std::forward<Args>( args )... ) );
}
```

这种用法是 C++11 风格的，很自然：

```java
auto p = make_intrusive<clSomeObject>( a, b, c );
```

智能指针的完整源代码可以在`1_IntrusivePtr`示例中找到。现在，我们可以进一步使用这个类作为我们多线程内存管理的基石。

# 可移植的多线程原语

在撰写本文时，C++11 标准中期待已久的`std::thread`在 MinGW 工具链中尚不可用，并且它不具备调整线程优先级的能力，这对于网络来说很重要。因此，我们实现了一个简单的类`iThread`，带有虚拟方法`Run()`，以允许在我们的代码中进行可移植的多线程：

```java
class iThread
{
```

内部`LPriority`枚举定义了线程优先级类：

```java
public:
  enum LPriority
  {
    Priority_Idle         = 0,
    Priority_Lowest       = 1,
    Priority_Low          = 2,
    Priority_Normal       = 3,
    Priority_High         = 4,
    Priority_Highest      = 5,
    Priority_TimeCritical = 6
  };
```

构造函数和析构函数的代码很简单：

```java
  iThread(): FThreadHandle( 0 ), FPendingExit( false )
  {}
  virtual ~iThread()
  {}
```

`Start()`方法创建一个特定于操作系统的线程句柄并开始执行。在这本书的所有示例中，我们不需要推迟线程执行；我们只需使用默认参数调用`_beginthreadex()`和`pthread_create()`系统例程。`EntryPoint()`方法稍后定义：

```java
  void Start()
  {
    void* ThreadParam = reinterpret_cast<void*>( this );
    #ifdef _WIN32
      unsigned int ThreadID = 0;
      FThreadHandle = ( uintptr_t )_beginthreadex( nullptr, 0, &EntryPoint, ThreadParam, 0, &ThreadID );
    #else
      pthread_create( &FThreadHandle, nullptr, EntryPoint, ThreadParam );
      pthread_detach( FThreadHandle );
    #endif
  }
```

系统相关的线程句柄和布尔原子变量（指示此线程是否应停止执行）在类的私有部分中声明：

```java
private:
  thread_handle_t FThreadHandle;
  std::atomic<bool> FpendingExit;
```

本地线程 API 仅支持 C 函数，因此我们必须声明一个静态包装方法`EntryPoint()`，该方法将`void*`参数转换为`iThread`并调用类的`Run()`方法。线程函数的调用约定和结果类型在 POSIX 和 Windows 上有所不同：

```java
  #ifdef _WIN32
    #define THREAD_CALL unsigned int __stdcall
  #else
    #define THREAD_CALL void*
  #endif
    static THREAD_CALL EntryPoint( void* Ptr );
```

受保护的部分定义了`Run()`和`NotifyExit()`虚拟方法，这些方法在子类中被重写。`GetHandle()`方法允许子类访问特定平台的线程句柄：

```java
protected:
  virtual void Run() = 0;
  virtual void NotifyExit() {};
  thread_handle_t GetHandle() { return FThreadHandle; }
```

要停止线程，我们将设置`FPendingExit`标志并调用`NotifyExit()`方法通知线程所有者。可选的`Wait`参数强制该方法等待线程的实际终止：

```java
  void Exit( bool Wait )
  {
    FPendingExit = true;
    NotifyExit();
    if ( !Wait ) { return; }
```

我们必须确保`Exit()`不要从同一线程的`Run()`方法中调用，以避免死锁，因此我们将调用`GetCurrentThread()`并将结果与我们的句柄进行比较：

```java
    if ( GetCurrentThread() != FThreadHandle )
    {
```

对于 Windows，我们将通过调用`WaitForSingleObject()`来模拟`join`操作，然后通过`CloseHandle()`终止线程：

```java
      #ifdef _WIN32
        WaitForSingleObject(( HANDLE )FThreadHandle, INFINITE );
        CloseHandle( ( HANDLE )FThreadHandle );
      #else
        pthread_join( FThreadHandle, nullptr );
      #endif
    }
  }
```

在 Android 上，`GetCurrentThread()`方法的实现与典型的 POSIX 版本略有不同。因此，这个方法包含了一个三重的`#ifdef`子句：

```java
  native_thread_handle_t iThread::GetCurrentThread()
  {
    #if defined( _WIN32)
      return GetCurrentThreadId();
    #elif defined( ANDROID )
      return gettid();
    #else
      return pthread_self();
    #endif
  }
```

`EntryPoint()`方法是将我们面向对象的`iThread`包装类与特定平台的 C 风格线程 API 联系在一起的粘合剂：

```java
  THREAD_CALL iThread::EntryPoint( void* Ptr )
  {
    iThread* Thread = reinterpret_cast<iThread*>( Ptr );
    if ( Thread )
    {
      Thread->Run();
    }
    #ifdef _WIN32
      _endthreadex( 0 );
      return 0;
    #else
      pthread_exit( 0 );
    return nullptr;
    #endif
  }
```

最后一个细节是`SetPriority()`方法，该方法用于控制线程的 CPU 时间分配。在 Windows 中，该方法的主要部分是将我们的`LPriority`枚举转换为`windows.h`头文件中定义的数值：

```java
  void iThread::SetPriority( LPriority Priority )
  {
    #ifdef _WIN32
      int P = THREAD_PRIORITY_IDLE;
      switch(Priority)
      {
        case Priority_Lowest:
          P = THREAD_PRIORITY_LOWEST; break;
        case Priority_Low:
          P = THREAD_PRIORITY_BELOW_NORMAL; break;
        case Priority_Normal:
          P = THREAD_PRIORITY_NORMAL; break;
        case Priority_High:
          P = THREAD_PRIORITY_ABOVE_NORMAL; break;
        case Priority_Highest:
          P = THREAD_PRIORITY_HIGHEST; break;
        case Priority_TimeCritical:
          P = THREAD_PRIORITY_TIME_CRITICAL; break;
      }
      SetThreadPriority( ( HANDLE )FThreadHandle, P );
    #else
```

对于 POSIX，我们将我们的优先级值重新缩放到操作系统中可用的最小和最大优先级之间的整数：

```java
      int SchedPolicy = SCHED_OTHER;
      int MaxP = sched_get_priority_max( SchedPolicy );
      int MinP = sched_get_priority_min( SchedPolicy );
      sched_param SchedParam;
      SchedParam.sched_priority = MinP + (MaxP - MinP) / (Priority_TimeCritical - Priority + 1);
      pthread_setschedparam( FThreadHandle, SchedPolicy, &SchedParam );
    #endif
  }
```

现在，我们可以使用`iThread`类来构建更有用的高级线程原语。为了实现类似`std::mutex`的跨平台轻量级对象，我们将使用 Marcus Geelnard 的 TinyThread 库，该库可以在[`tinythreadpp.bitsnbites.eu`](http://tinythreadpp.bitsnbites.eu)下载。但是，如果你不需要与旧编译器兼容，也可以自由使用标准互斥锁。

让我们继续处理任务队列。

# 任务队列

为了处理逻辑工作单元，我们将声明具有`Run()`方法的`iTask`类，该方法可以执行耗时的操作。类的声明在视觉上与`iThread`有些相似。然而，其实例实现了一些相对简短的操作，并且可以在不同的线程中执行：

```java
  class iTask: public iIntrusiveCounter
  {
  public:
    iTask()
    : FIsPendingExit( false )
    , FTaskID( 0 )
    , FPriority( 0 )
    {};
```

纯虚方法`Run()`应该在子类中被重写以执行实际工作：

```java
    virtual void Run() = 0;
```

下面的方法可选择性地取消任务，与`iThread`类中的方法类似。它们的作用是通知宿主线程应取消此任务：

```java
    virtual void Exit()
    {
      FIsPendingExit = true;
    }
    virtual bool IsPendingExit() const volatile
    {
      return FIsPendingExit;
    }
```

`GetTaskID()`和`SetTaskID()`方法访问任务的内部唯一标识符，用于取消执行：

```java
    virtual void SetTaskID( size_t ID )
    { FTaskID = ID; };
    virtual size_t GetTaskID() const
    { return FTaskID; };
```

`GetPriority()`和`SetPriority()`方法由任务调度程序使用，以确定执行任务的顺序：

```java
    virtual void SetPriority( int P )
    {
      FPriority = P;
    };
    virtual int GetPriority() const
    {
      return FPriority;
    };
```

类的私有部分包含一个原子退出标志，任务 ID 值和任务优先级：

```java
  private:
    std::atomic<bool> FIsPendingExit;
    size_t FTaskID;
    int FPriority;
  };
```

任务的管理由`clWorkerThread`类完成。基本上，它是一组`iTask`实例的集合，通过`AddTask()`方法进行输入。类的私有部分包含`iTask`的`std::list`和几个同步基元：

```java
  class clWorkerThread: public iThread
  {
  private:
    std::list< clPtr<iTask> >   FPendingTasks;
    clPtr<iTask>                FCurrentTask;
    mutable tthread::mutex      FTasksMutex;
    tthread::condition_variable FCondition;
```

`FCurrentTask`字段在内部用于跟踪正在进行的任务。`FTasksMutex`字段是一个互斥锁，用于确保对`FPendingTasks`的线程安全访问。`FCondition`条件变量用于通知列表中任务的可可用性。

`AddTask()`方法将新任务插入列表中，并通知`Run`方法任务已可用：

```java
    virtual void   AddTask( const clPtr<iTask>& Task )
    {
      tthread::lock_guard<tthread::mutex> Lock( FTasksMutex );
      FPendingTasks.push_back( Task );
      FCondition.notify_all();
    }
```

为了检查是否有未完成的任务，我们将定义`GetQueueSize()`方法。该方法使用`std::list.size()`，并在当前有活动任务正在运行时增加返回的值：

```java
    virtual size_t GetQueueSize() const
    {
      tthread::lock_guard<tthread::mutex> Lock( FTasksMutex );
      return FPendingTasks.size() + ( FCurrentTask ? 1 : 0 );
    }
```

有一个`CancelTask()`方法来取消单个任务，以及一个`CancelAll()`方法来一次性取消所有任务：

```java
    virtual bool   CancelTask( size_t ID )
    {
      if ( !ID ) { return false; }
      tthread::lock_guard<tthread::mutex> Lock( FTasksMutex );
```

首先，我们检查是否有正在运行的任务，并且其 ID 与我们想要取消的 ID 匹配：

```java
      if ( FCurrentTask && FCurrentTask->GetTaskID() == ID )
        FCurrentTask->Exit();
```

然后，我们将遍历任务列表，并请求给定 ID 的任务退出，从待处理任务列表中移除它们。这可以通过使用简单的 lambda 表达式来完成：

```java
      FPendingTasks.remove_if(
        ID
        {
          if ( T->GetTaskID() == ID )
          {
            T->Exit();
            return true;
          }
          return false;
        }
      );
```

最后，我们通知所有人列表已更改：

```java
      FCondition.notify_all();
      return true;
    }
```

`CancelAll()`方法要简单得多。迭代任务列表，请求每个项目终止；这之后，清空容器并发送通知：

```java
    virtual void CancelAll()
    {
      tthread::lock_guard<tthread::mutex> Lock( FTasksMutex );
      if ( FcurrentTask )
      {
        FcurrentTask->Exit();
      }
      for ( auto& Task: FpendingTasks )
      {
        Task->Exit();
      }
      FpendingTasks.clear();
      Fcondition.notify_all();
    }
```

主要工作在`Run()`方法中完成，该方法等待下一个任务到达并执行它：

```java
    virtual void Run()
    {
```

外层循环使用`iThread::IsPendingExit()`例程检查我们是否需要停止这个工作线程：

```java
    while ( !IsPendingExit() )
    {
```

`ExtractTask()`方法从列表中提取下一个任务。它会等待条件变量直到任务实际可用：

```java
      FCurrentTask = ExtractTask();
```

如果任务有效且未请求取消，我们可以开始执行任务：

```java
      if ( FCurrentTask &&
        !FCurrentTask->IsPendingExit())
      FCurrentTask->Run();
```

任务完成工作后，我们将清除状态以确保正确的`GetQueueSize()`操作：

```java
      FCurrentTask = nullptr;
    }
  }
```

`ExtractTask()`方法在`FPendingTasks`列表中实现了一个线程安全的线性搜索，以选择具有最高优先级的`iTask`实例：

```java
  clPtr<iTask> ExtractTask()
  {
    tthread::lock_guard<tthread::mutex> Lock( FTasksMutex );
```

为了避免进行忙等（spinlock）并耗尽 CPU 周期，将检查条件变量：

```java
    while ( FPendingTasks.empty() && !IsPendingExit() )
      FCondition.wait( FTasksMutex );
```

如果列表为空，将返回空智能指针：

```java
    if ( FPendingTasks.empty() )
      return clPtr<iTask>();
```

`Best`变量存储了要执行的选择任务：

```java
    auto Best = FPendingTasks.begin();
```

遍历`FPendingTask`列表，并将优先级值与`Best`变量中的值进行比较，我们将选择任务：

```java
    for ( auto& Task : FPendingTasks )
    {
      if ( Task->GetPriority() >
        ( *Best )->GetPriority() ) *Best = Task;
    }
```

最后，我们将从容器中删除选定的任务并返回结果。需要临时变量以确保我们的智能指针不会将引用计数减为零：

```java
    clPtr<iTask> Result = *Best;
    FPendingTasks.erase( Best );
    Return Result;
  }
```

现在，我们已经有了处理异步任务的类。在我们可以继续实际的异步网络连接——异步回调之前，还有一件至关重要的事情要做。

# 消息泵和异步回调

在上一节中，我们定义了`clWorkerThread`和`iTask`类，它们允许我们在 C++代码中在 UI 线程之外执行耗时操作。为了组织一个响应式界面，我们最后需要的能力是在不同线程之间传递事件。为此，我们需要一个可调用的接口，它可以封装传递给方法的参数，以及一个线程安全的机制来传递这样的胶囊。

一个很好的候选胶囊是`std::packaged_task`，但它在最新的 MinGW 工具链中不受支持。因此，我们将定义自己的轻量级引用计数抽象类`iAsyncCapsule`，它实现了一个单一的方法，`Invoke()`：

```java
  class iAsyncCapsule: public iIntrusiveCounter
  {
  public:
    virtual void Invoke() = 0;
  };
```

我们将包裹在`clPtr`中的`iAsyncCapsule`实例的优先级集合称为*异步队列*。`clAsyncQueue`类实现了`DemultiplexEvents()`方法，该方法将在处理传入事件的线程中调用。

### 注意

这被称为反应器模式。其文档可以在[`en.wikipedia.org/wiki/Reactor_pattern`](http://en.wikipedia.org/wiki/Reactor_pattern)找到。

解复用包括调用所有通过`EnqueueCapsule()`方法从其他线程添加的累积`iAsyncCapsule`。这两种方法应该是线程安全的，实际上也是。然而，`DemultiplexEvents()`在意义上不是可重入的，也就是说，两个线程不应当对同一对象调用`DemultiplexEvents()`。这一限制是性能优化的一部分，我们将在后面看到。我们使用两个`iAsyncCapsule`容器，并在每次调用`DemultiplexEvents()`时切换它们。这使得`EnqueueCapsule()`执行更快，因为我们不需要复制队列内容以确保线程安全。否则，由于在互斥锁锁定时我们不应该调用`Invoke()`，所以进行复制是必要的。

类的私有部分包含当前使用的队列索引`FCurrentQueue`，两个`iAsyncCapsule`容器，指向当前队列的指针以及用于防止同时访问`FAsyncQueues`数组的互斥锁：

```java
  class clAsyncQueue
  {
  private:
    using CallQueue = std::vector< clPtr<iAsyncCapsule> >;
    size_t FCurrentQueue;
    std::array<CallQueue, 2> FAsyncQueues;
    /// switched for shared non-locked access
    CallQueue* FAsyncQueue;
    tthread::mutex FDemultiplexerMutex;
```

构造函数初始化当前队列指针和索引：

```java
  public:
    clAsyncQueue()
    : FDemultiplexerMutex()
    , FCurrentQueue( 0 )
    , FAsyncQueues()
    , FAsyncQueue( &FAsyncQueues[0] )
    {}
```

`EnqueueCapsule()`方法与`WorkerThread::AddTask()`类似。首先，我们创建一个作用域内的`lock_guard`对象，然后调用`push_back()`以将`iAsyncCapsule`对象入队：

```java
    virtual void EnqueueCapsule(
      const clPtr<iAsyncCapsule>& Capsule )
    {
      tthread::lock_guard<tthread::mutex>
        Lock( FDemultiplexerMutex );
      FAsyncQueue->push_back( Capsule );
    }
```

`DemultiplexEvents()`方法保存对当前队列的引用：

```java
    virtual void DemultiplexEvents()
    {
```

`DemultiplexEvents()`被设计为只在单个线程上运行。此时不需要加锁：

```java
      CallQueue& LocalQueue = FAsyncQueues[ FCurrentQueue ];
```

然后，交换当前队列指针。这是一个原子操作，因此我们使用互斥锁来防止访问`FAsyncQueue`指针和索引：

```java
      {
        tthread::lock_guard<tthread::mutex>
          Lock( FDemultiplexerMutex );
        FCurrentQueue = ( FCurrentQueue + 1 ) % 2;
        FAsyncQueue = &FAsyncQueues[ FCurrentQueue ];
      }
```

最后，当前队列中的每个`iAsyncCapsule`都会被调用，并且`LocalQueue`会被清空：

```java
      for ( auto& i: LocalQueue ) i->Invoke();
      LocalQueue.clear();
    }
  };
```

典型的使用场景是在一个线程向另一个线程发布回调。这里考虑的一个小示例使用了`clResponseThread`类，该类有一个无尽循环作为主线程：

```java
  class clResponseThread: public iThread, public clAsyncQueue
  {
  public:
    virtual void Run()
    {
      for (;;) DemultiplexEvents();
    }
  };
```

示例`clRequestThread`类每秒产生两次事件：

```java
  class clRequestThread: public iThread
  {
  public:
    explicit clRequestThread( clAsyncQueue* Target )
    : FTarget(Target)
    {}
    virtual void Run()
    {
      int id = 0;
      for (;;)
      {
        FTarget->EnqueueCapsule( make_intrusive<clTestCall>( id++ ) );
        OS_Sleep( 500 );
      }
    }
  private:
    clAsyncQueue* FTarget;
  };
```

测试调用仅打印带有`clTestCall` ID 的消息：

```java
  class clTestCall: public iAsyncCapsule
  {
  private:
    int id;
  public:
    explicit clTestCall( int i ): id(i) {}
    virtual void Invoke()
    {
      std::cout "Test " << id << std::endl;
    }
  };
```

在`main()`函数中，我们创建两个线程并开始一个无限循环：

```java
  clResponseThread Responder;
  clRequestThread Requester( &Responder );
  Responder.Start();
  Requester.Start();
  for (;;) {}
```

在下一节中，我们将使用类似的方法通知主线程下载结果。`clResponseThread`类成为 UI 线程，而`clRequestThread`是一个`WorkerThread`方法，其中每个执行的下载任务一旦下载完成就会触发一个事件。

# 使用 libcurl 进行异步网络操作

在第二章 *本地库* 中展示了 libcurl 的简单使用。现在，我们使用之前提到的多线程原语来扩展代码，以允许异步下载。

这里引入的`clDownloadTask`类跟踪下载过程，并在过程完成时调用回调函数：

```java
  class clDownloadTask: public iTask
  {
  public:
```

构造函数接受要下载资源的 URL、唯一的任务标识符、回调函数以及指向 downer 实例的指针：

```java
    clDownloadTask( const std::string& URL,
      size_t TaskID, 
      const clPtr<clDownloadCompleteCallback>& CB,
      clDownloader* Downloader );
```

我们将关注`Run()`、`Progress()`和`InvokeCallback()`方法，因为它们构成了此类的主要逻辑：

```java
    virtual void Run() override;
  private:
    void Progress( double TotalToDownload,
      double NowDownloaded,
      double TotalToUpload,
      double NowUploaded );
    void InvokeCallback();
  };
```

`Run()`方法在下载线程上运行；它初始化并使用 libcurl 实际执行资源的下载：

```java
  void clDownloadTask::Run()
  {
```

此硬引用是必需的，以防止任务在外部被销毁（如果任务被取消）：

```java
    clPtr<clDownloadTask> Guard( this );
    CURL* Curl = curl_easy_init_P();
```

libcurl 的初始化代码在这里。所有可能的参数可以在官方文档中找到，地址为[`curl.haxx.se/libcurl/c/curl_easy_setopt.html`](http://curl.haxx.se/libcurl/c/curl_easy_setopt.html)：

```java
    curl_easy_setopt_P( Curl, CURLOPT_URL, FURL.c_str() );
    curl_easy_setopt_P( Curl, CURLOPT_FOLLOWLOCATION, 1 );
    curl_easy_setopt_P( Curl, CURLOPT_NOPROGRESS, false );
    curl_easy_setopt_P( Curl, CURLOPT_FAILONERROR, true );
    curl_easy_setopt_P( Curl, CURLOPT_MAXCONNECTS, 10 );
    curl_easy_setopt_P( Curl, CURLOPT_MAXFILESIZE, DownloadSizeLimit );
    curl_easy_setopt_P( Curl, CURLOPT_WRITEFUNCTION,
      &MemoryCallback );
    curl_easy_setopt_P( Curl, CURLOPT_WRITEDATA, this );
    curl_easy_setopt_P( Curl, CURLOPT_PROGRESSFUNCTION, &ProgressCallback );
    curl_easy_setopt_P( Curl, CURLOPT_PROGRESSDATA, this );
```

以下行设置尝试连接时要等待的秒数。使用零值表示无限期等待：

```java
    curl_easy_setopt_P( Curl, CURLOPT_CONNECTTIMEOUT, 30 );
```

在这里，我们设置允许 libcurl 函数执行的最大秒数：

```java
    curl_easy_setopt_P( Curl, CURLOPT_TIMEOUT, 600 );
```

禁用 OpenSSL 对证书的验证，这将允许访问具有自签名证书的站点。然而，在生产代码中，你可能想要删除此模式，以减少中间人攻击的可能性：

```java
    curl_easy_setopt_P( Curl, CURLOPT_SSL_VERIFYPEER, 0 );
    curl_easy_setopt_P( Curl, CURLOPT_SSL_VERIFYHOST, 0 );
    curl_easy_setopt_P( Curl, CURLOPT_HTTPGET, 1 );
```

### 注意

在协商 SSL 连接时，服务器会发送一个证书来标识其身份。Curl 验证证书是否真实——也就是说，你可以信任服务器就是证书所说的那个实体。这种信任基于一系列数字签名，根植于你提供的认证机构（CA）证书。

你可以在以下 URL 找到文档：

[`curl.haxx.se/libcurl/c/CURLOPT_SSL_VERIFYPEER.html`](http://curl.haxx.se/libcurl/c/CURLOPT_SSL_VERIFYPEER.html)

[`curl.haxx.se/libcurl/c/CURLOPT_SSL_VERIFYHOST.html`](http://curl.haxx.se/libcurl/c/CURLOPT_SSL_VERIFYHOST.html)

执行实际下载：

```java
    FCurlCode = curl_easy_perform_P( Curl );
    curl_easy_getinfo_P( Curl, CURLINFO_RESPONSE_CODE, &FRespCode );
    curl_easy_cleanup_P( Curl );
```

让下载器处理此任务的结果。我们很快就会跟随这段代码：

```java
    if ( FDownloader ) { FDownloader->CompleteTask( this ); }
  }
```

私有的`InvokeCallback()`成员函数可以被友类`clDownloader`访问：

```java
  void clDownloadTask::InvokeCallback()
  {
    tthread::lock_guard<tthread::mutex> Lock( FExitingMutex );
```

本质上，这只是对`FCallback->Invoke()`的调用，并增加了两个运行时检查。第一个检查任务是是否没有被取消：

```java
    if ( !IsPendingExit() )
    {
      if ( FCurlCode != 0 )
      {
        FResult = nullptr;
      }
```

第二个检查回调的可用性并准备所有参数：

```java
      if ( FCallback )
      {
        FCallback->FTaskID = GetTaskID();
        FCallback->FResult = FResult;
        FCallback->FTask = clPtr<clDownloadTask>( this );
        FCallback->FCurlCode = FCurlCode;
        FCallback->Invoke();
        FCallback = nullptr;
      }
    }
  }
```

需要注意的是，回调的调用是在互斥锁锁定的情况下进行的。这样做是为了确保正确的取消行为。然而，`InvokeCallback()`并不是直接从`clDownloadTask`中调用的。相反，是通过`Run()`方法中的`FDownloader->CompleteTask( this )`进行间接调用。让我们看看它里面的内容，以及`clDownloader`类的核心部分：

```java
  class clDownloader: public iIntrusiveCounter
  {
  public:
    explicit clDownloader( const clPtr<clAsyncQueue>& Queue );
    virtual ~clDownloader();
```

这个方法是我们公共下载 API 最重要的部分：

```java
    virtual clPtr<clDownloadTask> DownloadURL(
      const std::string& URL, size_t TaskID,
      const clPtr<clDownloadCompleteCallback>& CB );
    virtual bool CancelLoad( size_t TaskID );
    virtual void CancelAll();
    virtual size_t GetNumDownloads() const;
```

下面是处理间接调用的代码：

```java
  private:
    void CompleteTask( clPtr<clDownloadTask> Task );
    friend class clDownloadTask;
```

这是在其中运行`clDownloadTask`的线程：

```java
    clPtr<clWorkerThread> FDownloadThread;
```

外部事件队列通过构造函数参数进行初始化：

```java
    clPtr<clAsyncQueue> FEventQueue;
  };
```

然而，`DownloadURL()`方法是关键的，其实现却出奇地简单：

```java
  clPtr<clDownloadTask> DownloadURL( const std::string& URL,size_t TaskID,const clPtr<clDownloadCompleteCallback>& CB )
  {
    if ( !TaskID || !CB ) { return clPtr<clDownloadTask>(); }
    auto Task = make_intrusive<clDownloadTask>(URL, TaskID, CB, this );
    FDownloadThread->AddTask( Task );
    return Task;
  }
```

实际上，所有繁重的工作都是在前面提到的方法`clDownloadTask::Run()`中完成的。在这里，我们只是将新构建的任务排入工作线程队列中。最有趣的事情发生在`CompleteTask()`内部：

```java
  void clDownloader::CompleteTask( clPtr<clDownloadTask> Task )
  {
    if ( !Task->IsPendingExit() )
    {
      if ( FEventQueue )
      {
```

这里，一个回调包装器被插入到事件队列中：

```java
        FEventQueue->EnqueueCapsule(
          make_intrusive<clCallbackWrapper>(Task) );
      }
    }
  }
```

辅助类调用了`FTask->InvokeCallback()`方法。记住，该方法是在正确的线程上被调用的，它是由事件队列分派的：

```java
  class clCallbackWrapper: public iAsyncCapsule
  {
  public:
    explicit clCallbackWrapper(
      const clPtr<clDownloadTask> T ):FTask(T) {}
    virtual void Invoke() override
    {
      FTask->InvokeCallback();
    }
  private:
    clPtr<clDownloadTask> FTask;
  };
```

使用示例可以在源代码包的`3_Downloader`文件夹中找到。它就像这段代码一样简单：

```java
  int main()
  {
    Curl_Load();
```

这个队列将处理下载结果：

```java
    auto Events = make_intrusive<clAsyncQueue>();
    auto Downloader = make_intrusive<clDownloader>( Events );
    clPtr<clDownloadTask> Task = Downloader->DownloadURL(
      http://downloads.sourceforge.net/freeimage/FreeImage3160.zip,
      1, make_intrusive<clTestCallback>() );
    while ( !g_ShouldExit ) { Events->DemultiplexEvents(); }
    return 0;
  }
```

`clTestCallback`类打印下载进度并将结果保存到文件中，在我们的示例中是一个`.zip`文件。

### 注意

我们使用`LUrlParser`库从给定的 URL 中提取文件名，[`github.com/corporateshark/LUrlParser`](https://github.com/corporateshark/LUrlParser)。

示例代码可以通过输入 `make all` 使用 MinGW 编译。同样的代码可以在 Android 上运行，无需更改，使用从第二章，*原生库*编译的 Curl 库。我们建议您在 Android 上尝试此代码，并直接从 C++代码进行一些下载操作。

# 原生应用中的 Android 授权

本章的大部分内容已经致力于 C++中的低级网络功能，这对于编写多平台代码至关重要。然而，在本章中省略一些 Android 特定的事项是不公平的。让我们通过授权机制来学习如何将其移入 C++代码。为此，我们将需要大量与 Java 代码交互，因为所有授权功能都是 Java 独有的。

### 注意

在这里，我们假设您已经熟悉如何在 Java 中进行授权检查。官方 Google 文档可以在这里找到：

[`developer.android.com/google/play/licensing/setting-up.html`](http://developer.android.com/google/play/licensing/setting-up.html)

[`developer.android.com/google/play/licensing/adding-licensing.html`](http://developer.android.com/google/play/licensing/adding-licensing.html)

本示例的源代码位于 `4_Licensing` 文件夹中。首先，让我们定义基本常量，这些值应该与 Android SDK 中的匹配。请查看 `License.h` 文件：

```java
  constexpr int LICENSED = 0x0100;
  constexpr int NOT_LICENSED = 0x0231;
  constexpr int RETRY = 0x0123;
  constexpr int ERROR_INVALID_PACKAGE_NAME = 1;
  constexpr int ERROR_NON_MATCHING_UID = 2;
  constexpr int ERROR_NOT_MARKET_MANAGED = 3;
  constexpr int ERROR_CHECK_IN_PROGRESS = 4;
  constexpr int ERROR_INVALID_PUBLIC_KEY = 5;
  constexpr int ERROR_MISSING_PERMISSION = 6;
```

然后，`Callbacks.h` 声明了从授权检查器调用的回调：

```java
  void OnStart();
  void OnLicensed( int Reason );
  void OnLicenseError( int ErrorCode );
```

主源文件包含那些回调的实现：

```java
  #include <stdlib.h>
  #include "Callbacks.h"
  #include "License.h"
  #include "Log.h"
  void OnStart()
  {
    LOGI( "Hello Android NDK!" );
  }
  void OnLicensed( int Reason )
  {
    LOGI( "OnLicensed: %i", Reason );
```

在这里，只有当我们确实未获得授权时才终止应用程序：

```java
    if ( Reason == NOT_LICENSED )
    {
      exit( 255 );
    }
  }
  void OnLicenseError( int ErrorCode )
  {
    LOGI( "ApplicationError: %i", ErrorCode );
  }
```

让我们深入到 JNI 和 Java 代码中，看看这些回调是如何被调用的。`LicenseChecker.cpp` 文件包含了对前面提到的回调的静态 Java 方法的一对一映射：

```java
  extern "C"
  {
    JNIEXPORT void JNICALL Java_com_packtpub_ndkmastering_AppActivity_Allow(
      JNIEnv* env, jobject obj, int Reason )
    {
      OnLicensed( Reason );
    }
    JNIEXPORT void JNICALL Java_com_packtpub_ndkmastering_AppActivity_DontAllow(
      JNIEnv* env, jobject obj, int Reason )
    {
      OnLicensed( Reason );
    }
    JNIEXPORT void JNICALL Java_com_packtpub_ndkmastering_AppActivity_ApplicationError(
      JNIEnv* env, jobject obj, int ErrorCode )
    {
      OnLicenseError( ErrorCode );
    } 
  }
```

我们跟随代码进入 `AppActivity.java` 文件，该文件声明了 `CheckLicense()`：

```java
  public void CheckLicense( String BASE64_PUBLIC_KEY,
    byte[] SALT )
  {
    String deviceId = Secure.getString( getContentResolver(), Secure.ANDROID_ID );
```

构造 `LicenseCheckerCallback` 对象。Google 授权库在完成后会调用它：

```java
    m_LicenseCheckerCallback = new AppLicenseChecker();
```

使用 `Policy` 构造 `LicenseChecker`：

```java
    m_Checker = new LicenseChecker( this,
      new ServerManagedPolicy(this,
        new AESObfuscator( SALT,
        getPackageName(), deviceId) ),
      BASE64_PUBLIC_KEY);
    m_Checker.checkAccess( m_LicenseCheckerCallback );
  }
```

回调的 Java 部分就在这里，位于类声明的底部：

```java
  public static native void Allow( int reason );
  public static native void DontAllow( int reason );
  public static native void ApplicationError( int errorCode );
```

`AppLicenseChecker` 类只是调用这些静态方法，将事件路由到 JNI 代码。多么简单！现在，您可以在 C++代码中以可移植的方式处理（和测试）对授权检查事件的反应。使用以下命令为 Android 构建示例，亲自看看吧：

```java
>ndk-build
>ant debug

```

运行时日志可以通过 `logcat` 访问。桌面版本可以通过 `make all` 命令构建，正如本书中的所有示例一样。

# Flurry 分析

让我们再接触一个与 Java 相关的事项及其与原生 C++代码的绑定。Flurry.com 是一个流行的应用内分析服务。通过向 Flurry.com 发送信息，可以完成对应用中最常用功能的确定，之后可以通过他们的网页访问收集到的统计数据。

例如，您的应用程序中有几种不同的游戏模式选项：战役、单级别或在线。用户选择其中一种模式，就会生成并发送一个事件到 Flurry.com。我们希望从 C++代码发送这些事件。

请查看 `5_Flurry` 文件夹中的示例应用程序。`main.cpp` 文件包含了一个典型的使用示例：

```java
  void OnStart()
  {
    TrackEvent( "FlurryTestEvent" );
  }
```

`TrackEvent()` 的定义以及 Android 与桌面实现的区别位于 `Callbacks.cpp` 文件中：

```java
  extern "C"
  {
    void Android_TrackEvent( const char* EventID );
  };
  void TrackEvent( const char* EventID )
  {
    #if defined(ANDROID)
      Android_TrackEvent( EventID );
    #else
      printf( "TrackEvent: %s\n", EventID );
    #endif
  }
```

Android 实现需要一些 JNI 代码才能工作。请查看以下 `jni/JNI.c` 文件：

```java
  void Android_TrackEvent( const char* EventID )
  {
    JAVA_ENTER();
    jstring jstr = (*env)->NewStringUTF( env, EventID );
    FindJavaStaticMethod( env, &Class, &Method,
      "com/packtpub/ndkmastering/AppActivity",
      "Callback_TrackEvent", "(Ljava/lang/String;)V" );
    (*env)->CallStaticVoidMethod( env, Class, Method, jstr );
    JAVA_LEAVE();
  }
```

`Callback_TrackEvent()` 在主活动中定义如下：

```java
  public static void Callback_TrackEvent( String EventID )
  {
    if ( m_Activity == null ) return;
    m_Activity.TrackEvent( EventID );
  }
  public void TrackEvent( String EventID )
  {
    FlurryAgent.logEvent( EventID );
  }
```

Flurry 分析 API 的其他部分也可以通过类似的方式从 C++路由到 Java，反之亦然。我们建议您在 Flurry 上注册一个账户，获取应用密钥，并尝试自己运行示例。只需替换 `FlurryAgent.init()` 和 `FlurryAgent.onStartSession()` 的应用密钥，即可在 Android 上运行应用程序。构建过程很简单，只需使用 `ndk-build` 和 `ant debug`。

# 总结

在本章中，我们学习了如何实现精简且可移植的多线程原语，例如引用计数侵入式智能指针、工作线程和消息泵，并使用它们创建简单的可移植 C++网络访问框架。我们还稍微涉及了 Java，以展示如何在本地代码中处理许可和用量分析。在下一章中，我们将从网络方面抽身，学习如何使用虚拟文件系统抽象来处理异构文件系统。
