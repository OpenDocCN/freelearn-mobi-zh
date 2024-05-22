# 第四章 组织虚拟文件系统

在本章中，我们将实现低级别的抽象，以处理操作系统无关的文件和文件系统访问。我们将展示如何实现可移植且透明地访问`.apk`文件内部打包的 Android 资源，而不依赖于任何内置 API。在桌面环境中构建可调试的多平台应用程序时，这种方法是必要的。

# 挂载点

挂载点的概念几乎在现代每一个文件系统中都可以找到。对于跨平台 C++程序来说，以一种统一的方式来访问异构存储设备中的文件非常方便。例如，在 Android 上，每个只读数据文件可以存储在`.apk`包内，开发者被迫使用特定的 Android 资产管理 API。在 OSX 和 iOS 上，访问程序束需要另一个 API，在 Windows 上，应用程序应该将其所有内容存储在其文件夹中，该文件夹的物理路径也取决于应用程序安装的位置。

为了在不同平台之间组织文件访问，我们提出了一个浅层类层次结构，它抽象了文件管理的差异，如下面的图所示：

![挂载点](img/image00221.jpeg)![挂载点](img/image00222.jpeg)

虚拟文件系统是挂载点的集合。每个挂载点都是一个文件系统文件夹的抽象。这种组织方式允许我们将实际的操作系统特定文件访问例程和文件名映射从应用程序代码中隐藏起来。本章涵盖了文件系统、挂载点和流接口的描述。

我们定义了一个`iMountPoint`接口，它可以解析虚拟文件名，并创建文件阅读对象的实例：

```java
class iMountPoint: public iIntrusiveCounter
{
public:
```

检查在这个挂载点是否存在的虚拟文件：

```java
  virtual bool FileExists( const std::string& VirtualName ) const = 0;
```

将虚拟文件名转换为绝对文件名：

```java
  virtual std::string MapName( const std::string& VirtualName ) const = 0;
```

`CreateReader()`成员函数创建一个文件阅读器对象，该对象实现了本章后续介绍的`iRawFile`接口。这个方法通常只被`clFileSystem`类使用：

```java
  virtual clPtr<iRawFile> CreateReader( const std::string& VirtualName ) const = 0;
```

最后两个成员函数获取和设置此挂载点的内部名称。这个字符串稍后会在`clFileSystem`接口中使用，以搜索和识别挂载点：

```java
  virtual void SetName( const std::string& N ) { FName = N; }
  virtual std::string GetName() const { return FName; }
private:
  std::string FName;
};
```

我们的虚拟文件系统实现为挂载点的线性集合。这里的`clFileSystem::CreateReader()`方法创建一个`iIStream`对象，该对象封装了对文件数据的访问：

```java
clPtr<iIStream> CreateReader( const std::string& FileName ) const;
```

`Mount()`方法将一个物理（这里*物理*指的是特定操作系统的路径）路径添加到挂载点列表中。如果`PhysicalPath`值表示本地文件系统的一个文件夹，则会创建一个`clPhysicalMountPoint`实例。如果`PhysicalPath`是一个`.zip`或`.apk`文件的名称，则会将`clArchiveMountPoint`实例添加到挂载点列表中。`clPhysicalMountPoint`和`ArchiveMountPoint`类的定义可以在代码包中的示例`1_ArchiveFileAccess`中找到：

```java
void Mount( const std::string& PhysicalPath );
```

`VirtualNameToPhysical()`将我们的虚拟路径转换为特定操作系统的系统文件路径：

```java
std::string VirtualNameToPhysical(
  const std::string& Path ) const;
```

`FileExists()`方法检查每个挂载点，以确定文件是否存在于其中一个挂载点中：

```java
  bool FileExists( const std::string& Name ) const;
```

`clFileSystem`类的私有部分负责管理内部挂载点列表。`FindMountPoint()`方法搜索包含名为`FileName`的文件的挂载点。`FindMountPointByName()`方法在内部使用，允许文件名称的别名。`AddMountPoint()`检查提供的挂载点是否唯一，如果是，则将其添加到`FMountPoints`容器中：

```java
private:
  clPtr<iMountPoint> FindMountPointByName( const std::string& ThePath );
  void AddMountPoint( const clPtr<iMountPoint>& MP );
  clPtr<iMountPoint> FindMountPoint( const std::string& FileName ) const;
```

最终，挂载点集合存储在`std::vector`中：

```java
  std::vector< clPtr<iMountPoint> > FMountPoints;
};
```

当我们想在应用程序代码中访问一个文件时，我们是通过文件系统对象`g_FS`来实现的：

```java
auto f = g_FS->CreateReader( "test.txt" );
```

# 挂载点与流

在 Android 上，`test.txt`文件很可能位于`.apk`包中，需要在`CreateReader()`调用中完成大量工作。`test.txt`的数据被提取出来，并创建了一个`clMemFileMapper`实例。让我们深入探究文件操作背后的隐藏管道。

`CreateReader()`的代码很简单。首先，我们将路径中的斜杠和反斜杠转换为与底层操作系统匹配的样式。然后找到一个包含名为`FileName`的文件的挂载点。最后，创建一个`clFileMapper`实例。这个类实现了`iIStream`接口。让我们仔细看看这些类：

```java
clPtr<iIStream> clFileSystem::CreateReader(
  const std::string& FileName ) const
{
  std::string Name = Arch_FixFileName( FileName );
  clPtr<iMountPoint> MountPoint = FindMountPoint( Name );
```

在这里，我们使用空对象模式（[`en.wikipedia.org/wiki/Null_Object_pattern`](http://en.wikipedia.org/wiki/Null_Object_pattern)）来定义非存在文件的中性行为。`clNullRawFile`类表示一个不与任何实际设备关联的空文件：

```java
  if ( !MountPoint ) { return make_intrusive<clFileMapper>( make_intrusive<clNullRawFile>() ); }
  return make_intrusive<clFileMapper>( MountPoint->CreateReader( Name ) );
}
```

`FindMountPoint()`方法遍历挂载点集合，以找到包含给定名称文件的挂载点：

```java
clPtr<iMountPoint> clFileSystem::FindMountPoint( const std::string& FileName ) const
{
  if ( FMountPoints.empty() )
  {
    return nullptr;
  }
  if ( ( *FMountPoints.begin() )->FileExists( FileName ) )
  {
    return ( *FMountPoints.begin() );
  }
```

反向迭代挂载点，以便首先检查最近挂载的路径：

```java
  for ( auto i = FMountPoints.rbegin();
    i != FMountPoints.rend(); ++i )
  {
    if ( ( *i )->FileExists( FileName ) )
    {
      return ( *i );
    }
  }
  return *( FMountPoints.begin() );
}
```

`clFileSystem`类将大部分工作委托给各个`iMountPoint`实例。例如，检查文件是否存在是通过找到适当的`iMountPoint`对象并询问该点是否存在文件来执行的：

```java
bool clFileSystem::FileExists( const std::string& Name ) const
{
  if ( Name.empty() || Name == "." ) { return false; }
  clPtr<iMountPoint> MP = FindMountPoint( Name );
  return MP ? MPD->FileExists( Name ) : false;
}
```

也可以通过适当的`iMountPoint`实例找到物理文件名：

```java
std::string clFileSystem::VirtualNameToPhysical(
  const std::string& Path ) const
{
  if ( FS_IsFullPath( Path ) ) { return Path; }
  clPtr<iMountPoint> MP = FindMountPoint( Path );
  return ( !MP ) ? Path : MP->MapName( Path );
}
```

物理文件名不直接用于访问文件。例如，如果挂载了一个存档，并且我们想要访问存档中的文件，那么该文件的物理路径对操作系统来说是没有意义的。相反，一切都由挂载点抽象化，物理文件名只在我们应用程序中作为标识符使用。

只有当新的挂载点是唯一的时候，它才会被添加到集合中；没有理由允许重复。

```java
void clFileSystem::AddMountPoint( const clPtr<iMountPoint>& MP )
{
  if ( !MP ) { return; }
  if ( std::find( FMountPoints.begin(), FMountPoints.end(), MP ) == FMountPoints.end() )
  {
    FMountPoints.push_back( MP );
  }
}
```

`clFileSystem::Mount()`的代码选择要实例化的挂载点类型：

```java
void clFileSystem::Mount( const std::string& PhysicalPath )
{
  clPtr<iMountPoint> MP;
```

我们在这里使用了一个简单的硬编码逻辑。如果路径以`.zip`或`.apk`子字符串结尾，我们将实例化`clArchiveMountPoint`：

```java
  if ( Str::EndsWith( PhysicalPath, ".apk" ) || Str::EndsWith( PhysicalPath, ".zip" ) )
  {
    auto Reader = make_intrusive<clArchiveReader>();
    bool Result = Reader->OpenArchive( CreateReader( PhysicalPath ) );
    MP = make_intrusive<clArchiveMountPoint>( Reader );
  }
  else
```

否则，我们将检查`clPhysicalPath`是否存在，然后创建`clPhysicalMountPoint`：

```java
  {
    #if !defined( OS_ANDROID )
      if ( !FS_FileExistsPhys( PhysicalPath ) )
      return;
    #endif
      MP = make_intrusive<clPhysicalMountPoint>(PhysicalPath );
  }
```

如果创建挂载点成功，我们设置其名称并将其添加到集合中：

```java
  MP->SetName( PhysicalPath );
  AddMountPoint( MP );
}
```

我们稍后会回到挂载点的实现。现在，我们转向流。对文件的实际读取访问是通过`iIStream`接口完成的：

```java
class iIStream: public iIntrusiveCounter
{
public:
```

接下来的两个方法分别获取虚拟和物理文件名：

```java
  virtual std::string GetVirtualFileName() const = 0;
  virtual std::string GetFileName() const = 0;
```

`Seek()`方法设置绝对读取位置；`GetSize()`和`GetPos()`确定大小和当前的读取位置，而`Eof()`检查是否已达到文件末尾：

```java
  virtual void   Seek( const uint64 Position ) = 0;
  virtual uint64 GetSize() const = 0;
  virtual uint64 GetPos() const = 0;
  virtual bool   Eof() const = 0;
```

`Read()`方法将指定`Size`的数据块读取到无类型内存缓冲区`Buf`中：

```java
  virtual uint64 Read( void* Buf, const uint64 Size ) = 0;
```

最后两个方法使用内存映射实现对文件数据的数组式访问。第一个返回与此文件对应的共享内存的指针：

```java
  virtual const ubyte* MapStream() const = 0;
```

第二个方法返回从当前文件位置开始的内存指针。这对于在块和内存映射访问样式之间无缝切换非常方便：

```java
  virtual const ubyte* MapStreamFromCurrentPos() const = 0;
};
```

为了避免 UI 线程阻塞，这些方法通常应该在工作者线程上调用。

所有访问物理文件的工作都在`clFileMapper`类中完成。它是`iIStream`接口的一个实现，将所有 I/O 操作委托给实现`iRawFile`接口的对象。`iRawFile`本身在应用程序代码中不直接使用，所以让我们先看看`clFileMapper`类：

```java
class clFileMapper: public iIStream
{
public:
```

构造函数只是存储了对`iRawFile`实例的引用，并重置了读取指针：

```java
  explicit FileMapper( clPtr<iRawFile> File ):
    FFile( File ), FPosition( 0 ) {}
  virtual ~FileMapper() {}
```

`GetVirtualFileName()`和`GetFileName()`方法使用`iRawFile`的实例分别获取虚拟和物理文件名：

```java
  virtual std::string GetVirtualFileName() const
  { return FFile->GetVirtualFileName(); }
  virtual std::string GetFileName() const
  { return FFile->GetFileName(); }
```

`Read()`方法模拟了`std::ifstream.read`和`libc`中的`read()`例程。它可能看起来不寻常，但读取是通过访问内存映射文件的`memcpy`调用完成的。`iRawFile::GetFileData()`的描述将澄清这些问题：

```java
  virtual uint64 Read( void* Buf, uint64 Size )
  {
    uint64 RealSize = ( Size > GetBytesLeft() ) ? GetBytesLeft() : Size;
    if ( !RealSize ) { return 0; }
    memcpy( Buf, ( FFile->GetFileData() + FPosition ),static_cast<size_t>( RealSize ) );
    FPosition += RealSize;
    return RealSize;
  }
```

定位和内存映射都委托给底层的`iRawFile`实例：

```java
  virtual void Seek( const uint64 Position)
  { FPosition = Position; }
  virtual uint64 GetSize() const
  { return FFile->GetFileSize(); }
  virtual bool Eof() const
  { return ( FPosition >= FFile->GetFileSize() ); }
  virtual const ubyte* MapStream() const
  { return FFile->GetFileData(); }
  virtual const ubyte* MapStreamFromCurrentPos() const
  { return ( FFile->GetFileData() + FPosition ); }
```

私有部分包含了对`iRawFile`的引用和当前的读取位置：

```java
private:
  clPtr<iRawFile> FFile;
  uint64 FPosition;
};
```

现在我们可以声明`iRawFile`接口，它非常简单：

```java
class iRawFile: public iIntrusiveCounter
{
public:
  iRawFile() {}
  virtual ~iRawFile() {}
```

前四个方法获取和设置虚拟和物理文件名：

```java
  std::string GetVirtualFileName() const
  { return FVirtualFileName; }
  std::string  GetFileName() const
  { return FFileName; }
    void SetVirtualFileName( const std::string& VFName )
    { FVirtualFileName = VFName; }
    void SetFileName( const std::string& FName )
    { FFileName = FName; }
```

这个接口的实质在于以下两个方法，它们获取文件数据的原始指针和文件的大小：

```java
    virtual const ubyte* GetFileData() const = 0;
    virtual uint64 GetFileSize() const = 0;
```

私有部分包含文件名的字符串：

```java
  private:
    std::string    FFileName;
    std::string    FVirtualFileName;
  };
```

声明完所有接口后，我们可以继续进行它们的实现。

# 访问宿主文件系统中的文件

我们从`clRawFile`类开始，它使用特定于操作系统的内存映射例程将文件映射到内存中：

```java
class clRawFile: public iRawFile
{
public:
  RawFile() {}
  virtual ~RawFile() { Close(); }
```

`Open()`成员函数完成了大部分繁重的工作。它存储物理和虚拟文件名，打开文件句柄并创建文件的映射视图：

```java
  bool Open( const std::string& FileName,
    const std::string& VirtualFileName )
  {
    SetFileName( FileName );
    SetVirtualFileName( VirtualFileName );
    FSize = 0;
    FFileData = nullptr;
```

在 Windows 上，我们使用`CreateFileA()`来打开文件。像往常一样，我们将特定于操作系统的部分用`#ifdef`块括起来。：

```java
    #ifdef _WIN32
      FMapFile = CreateFileA( FFileName.c_str(), GENERIC_READ,
        FILE_SHARE_READ, nullptr, OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL | FILE_FLAG_RANDOM_ACCESS,
        nullptr );
```

打开文件后，我们创建一个映射对象，并使用 `MapViewOfFile()` 系统调用获取指向文件数据的指针：

```java
      FMapHandle = CreateFileMapping( FMapFile,
        nullptr, PAGE_READONLY, 0, 0, nullptr );
      FFileData = ( ubyte* )MapViewOfFile( FMapHandle, FILE_MAP_READ, 0, 0, 0 );
```

如果出现错误，请关闭句柄并取消操作：

```java
      if ( !FFileData )
      {
        CloseHandle( ( HANDLE )FMapHandle );
        return false;
      }
```

为了防止读取超出文件末尾，我们应该获取文件的大小。在 Windows 中是这样完成的：

```java
      DWORD dwSizeLow = 0, dwSizeHigh = 0;
      dwSizeLow = ::GetFileSize( FMapFile, &dwSizeHigh );
      FSize = ( ( uint64 )dwSizeHigh << 32 )
        | ( uint64 )dwSizeLow;
```

在 Android 中，我们使用 `open()` 初始化文件句柄，并使用 `fstat()` 获取其大小：

```java
    #else
      FFileHandle = open( FileName.c_str(), O_RDONLY );
      struct stat FileInfo;
```

如果 `fstat()` 成功，我们可以获取其大小。如果文件大小非零，我们调用 `mmap()` 函数将文件映射到内存中：

```java
      if ( !fstat( FFileHandle, &FileInfo ) )
      {
        FSize = static_cast<uint64_t>( FileInfo.st_size );
```

确保对于大小为零的文件不调用 `mmap()`：

```java
      if ( FSize )
        FFileData = ( uint8_t* )( mmap( nullptr, FSize, PROT_READ, MAP_PRIVATE, FFileHandle, 0 ) );
      }
```

一旦我们有了 `mmap`-ed 的内存块，就可以立即关闭文件句柄。这是标准做法：

```java
      close( FFileHandle );
    #endif
      return true;
  }
```

`Close()` 方法取消内存块映射并关闭文件句柄：

```java
  void Close()
  {
```

在 Windows 中，我们使用 `UnmapViewOfFile()` 和 `CloseHandle()` 系统调用：

```java
    #ifdef _WIN32
      if ( FFileData  ) { UnmapViewOfFile( FFileData ); }
      if ( FMapHandle ) { CloseHandle( (HANDLE)FMapHandle ); }
      CloseHandle( ( HANDLE )FMapFile );
```

在 Android 中，我们调用 `munmap()` 函数：

```java
    #else
      if ( FFileData )
      {
        munmap( reinterpret_cast<void*>( FFileData ), FSize );
      }
    #endif
  }
```

`clRawFile` 类的其余部分包含两个简单的方法，返回文件数据指针和文件大小。私有部分声明文件句柄、文件大小和数据指针：

```java
  virtual const ubyte* GetFileData() const { return FFileData; }
  virtual uint64       GetFileSize() const { return FSize; }
private:
  #ifdef _WIN32
    HANDLE    FMapFile;
    HANDLE    FMapHandle;
  #else
    int       FFileHandle;
  #endif
    ubyte*    FFileData;
    uint64    FSize;
};
```

要使用 `clFileSystem` 类访问虚拟文件系统中的物理文件夹，我们声明了 `clPhysicalMountPoint` 类，代表宿主文件系统上的单个文件夹：

```java
class clPhysicalMountPoint: public iMountPoint
{
public:
```

`clPhysicalMountPoint` 的构造函数通过添加一个路径分隔符（根据底层操作系统的约定是斜杠或反斜杠）来修复物理文件夹路径：

```java
  clPhysicalMountPoint( const std::string& PhysicalName ):FPhysicalName( PhysicalName )
  {
    Str_AddTrailingChar( &FPhysicalName, PATH_SEPARATOR );
  }
  virtual ~PhysicalMountPoint() {}
```

`FileExists()` 方法使用依赖于操作系统的例程来检查文件是否存在：

```java
  virtual bool FileExists( const std::string& VirtualName ) const override
  {
    return FS_FileExistsPhys( MapName( VirtualName ) );
  }

```

`MapName()` 方法通过添加 `FPhysicalName` 前缀将虚拟文件转换为物理文件名。`FS_IsFullPath()` 例程在以下代码中定义：

```java
  virtual std::string  MapName( const std::string& VirtualName )const override
  {
    return FS_IsFullPath( VirtualName ) ? VirtualName : ( FPhysicalName + VirtualName );
  }
```

`clRawFile` 实例是在 `clPhysicalMountPoint::CreateReader()` 方法中创建的：

```java
  virtual clPtr<iRawFile> CreateReader(
    const std::string& VirtualName ) const override
  {
    std::string PhysName = MapName( VirtualName );
    auto File = make_intrusive<clRawFile>();
    if ( File->Open( FS_ValidatePath( PhysName ), VirtualName ) ) { return File; }
    return make_intrusive<clNullRawFile>();
  }
```

类的私有部分包含文件夹的物理名称：

```java
private:
  std::string FPhysicalName;
};
```

为了完成此代码，我们必须实现一些服务例程。第一个是 `FS_IsFullPath()`，它检查路径是否为绝对路径。对于 Android，这意味着路径以 `/` 字符开始，对于 Windows，完整路径必须以 `<drive>:\` 子字符串开始，其中 `<drive>` 是驱动器字母：

```java
inline bool FS_IsFullPath( const std::string& Path )
{
  return ( Path.find( ":\\" ) != std::string::npos ||
  #if !defined( _WIN32 )
    ( Path.length() && Path[0] == '/' ) ||
  #endif
    Path.find( ":/" )  != std::string::npos ||
    Path.find( ".\\" ) != std::string::npos );
}
```

`FS_ValidatePath()` 方法将每个斜杠或反斜杠字符替换为特定于平台的 `PATH_SEPARATOR`：

```java
inline std::string FS_ValidatePath( const std::string& PathName )
{
  std::string Result = PathName;
  for ( size_t i = 0; i != Result.length(); ++i )
    if ( Result[i] == '/' || Result[i] == '\\' )
    {
      Result[i] = PATH_SEPARATOR;
    }
  return Result;
}
```

要检查文件是否存在，我们使用 `stat()` 例程，其语法在 Windows 和 Android 上略有不同：

```java
inline bool FS_FileExistsPhys( const std::string& PhysicalName )
{
  #ifdef _WIN32
    struct _stat buf;
    int Result = _stat( FS_ValidatePath( PhysicalName ).c_str(),
      &buf );
  #else
    struct stat buf;
    int Result = stat( FS_ValidatePath( PhysicalName ).c_str(),
      &buf );
  #endif
    return Result == 0;
}
```

`PATH_SEPARATOR` 是一个特定于平台的字符常量：

```java
#if defined( _WIN32 )
  const char PATH_SEPARATOR = '\\';
#else
  const char PATH_SEPARATOR = '/';
#endif
```

上述代码足以访问直接存储在宿主文件系统上的文件。接下来，我们继续了解其他抽象概念以获取 Android `.apk` 包。

# 内存文件

以下 `iRawFile` 接口的实现封装了对未类型化内存块的访问作为文件访问。我们将使用此类来访问存档中的未压缩数据。

```java
class clMemRawFile: public iRawFile
{
public:
```

参数化构造函数用于初始化指向数据缓冲区的指针及其大小：

```java
  clMemRawFile( const uint8_t* BufPtr, size_t BufSize, bool OwnsBuffer )
  : FOwnsBuffer( OwnsBuffer )
  , FBuffer( BufPtr )
  , FBufferSize( BufSize )
  {}
```

对于一个内存块来说，内存映射是微不足道的，我们只需返回存储的原始指针：

```java
  virtual const uint8_t* GetFileData() const override
  { return FBuffer; }
  virtual uint64_t GetFileSize() const override
  { return FBufferSize; }
private:
  const uint8_t* FBuffer;
  size_t FBufferSize;
};
```

当我们处理归档文件读取时，将回到这个类。现在，让我们熟悉一个更多重要的概念，这是透明访问`.apk`包所必需的。

# 别名

前一节提到的文件抽象非常强大。它们可以用来创建嵌套的挂载点，以访问其他文件中打包的文件。让我们通过定义`clAliasMountPoint`来展示这种方法的灵活性，它类似于 Unix 或 NTFS 文件系统中的符号链接。

该实现将每个`iMountPoint::`方法调用重定向到另一个挂载点实例，同时在运行时通过为我们想要访问的每个虚拟文件名添加一个特定的`FAlias`前缀来转换文件名：

```java
class clAliasMountPoint: public iMountPoint
{
public:
  explicit clAliasMountPoint( const clPtr<iMountPoint>& Src )
  : Falias(), FMP( Src )
  {}
  virtual bool FileExists( const std::string& VirtualName ) const { return FMP->FileExists( FAlias + VirtualName ); }
  virtual std::string MapName( const std::string& VirtualName ) const { return FMP->MapName( FAlias + VirtualName ); }
  virtual clPtr<iRawFile> CreateReader( const std::string& VirtualName ) const { return FMP->CreateReader( FAlias + VirtualName ); }
private:
  std::string FAlias;
  clPtr<iMountPoint> FMP;
};
```

我们添加了`FileSystem::AddAlias()`成员函数，它通过将它们与`FAlias`前缀连接起来，来装饰现有挂载点的文件名：

```java
void clFileSystem::AddAlias( const std::string& SrcPath, const std::string& Alias )
{
  if (clPtr<iMountPoint> MP = FindMountPointByName( SrcPath ) ) AddMountPoint(new AliasMountPoint( MP, Alias ) );
}
```

这种机制可以用来将路径（如`assets/`）透明地重映射到我们文件系统的根目录，这对于 Android 上的应用程序功能至关重要。

# 写文件

在开始更复杂的归档解包工作之前，让我们先休息一下，看看如何写入文件。我们使用`iOStream`接口，它只声明了四个纯虚方法。`GetFileName()`方法返回虚拟文件名。`Seek()`方法设置写入位置，`GetFilePos()`返回它。`Write()`方法接受一个无类型的内存缓冲区并将其写入输出流：

```java
class iOStream: public iIntrusiveCounter
{
public:
  iOStream() {};
  virtual ~iOStream() {};
  virtual std::string GetFileName() const = 0;
  virtual void   Seek( const uint64 Position ) = 0;
  virtual uint64 GetFilePos() const = 0;
  virtual uint64 Write(const void* Buf, const uint64 Size) = 0;
};
```

我们在这里提供的`iOStream`的唯一实现是`clMemFileWriter`，它将一个无类型的内存块视为输出流。这个类用于访问`.zip`文件中的数据。首先，数据被解包，然后使用`clMemRawFile`进行包装：

```java
class clMemFileWriter: public iOStream
{
public:
```

实际的底层内存块由存储在此类中的`clBlob`对象通过 RAII 管理（[`en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization`](https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization)）：

```java
  clMemFileWriter()
  : FBlob( make_intrusive<clBlob>() )
  , FFileName()
  , FPosition( 0 )
  {}
  explicit clMemFileWriter( const clPtr<clBlob>& Blob )
  : FBlob( Blob )
  , FFileName()
  , FPosition( 0 )
  {}
```

`Seek()`方法增加当前的写入位置：

```java
  virtual void Seek( const uint64 Position )
  {
    FPosition = ( Position > FBlob->GetSize() ) ? FBlob->GetSize() - 1 : Position;
  }
```

`Write()`方法重定向到`clBlob`对象：

```java
  virtual uint64_t Write( const void* Buf, uint64_t Size ) override
  {
    return FBlob->AppendBytes( Buf,static_cast<size_t>( Size ) );
  }
```

伴随的源代码包含了`clFileWriter`类的实现，其中包含了类似于`clRawFile::Open()`的`Open()`方法。`Write()`方法使用系统 I/O 例程将数据写入物理文件。

现在，我们有足够的脚手架代码可以进一步处理`.zip`归档。

# 访问归档文件

由于`.apk`实际上就是一个花哨的`.zip`压缩包，我们使用了 Jean-loup Gailly 的 ZLib 库结合 MiniZIP 库来从中获取压缩文件。完整的源代码大约有 500 千字节大小，因此我们提供了两个文件，`libcompress.c`和`libcompress.h`，它们可以轻松地集成到任何构建过程中。我们的目标是实现`clArchiveMountPoint`，它枚举归档中的文件，为特定文件解压缩数据，并创建一个`clMemFileMapper`来读取其数据。为此，我们需要引入一个辅助类，`clArchiveReader`，它读取和解压缩`.zip`归档文件：

```java
class clArchiveReader: public iIntrusiveCounter
{
private:
```

`clArchiveReader`类中定义的私有的`sFileInfo`结构体封装了一组有用的文件属性以及指向压缩文件数据的指针：

```java
  struct sFileInfo
  {
    /// offset to the file
    uint64 FOffset;
    /// uncompressed file size
    uint64 FSize;
    /// compressed file size
    uint64 FCompressedSize;
    /// Compressed data
    void* FSourceData;
  };
```

`clArchiveReader`类的私有部分包含一个`sFileInfo`结构的集合，在`FFileInfos`字段中，一个包含大写文件名的`FFileNames`向量，一个包含归档内文件名的`FReadFileNames`向量，以及一个`std::map`对象，它将每个文件名映射到解压文件向量`FExtractedFromArchive`中的索引：

```java
  std::vector<sFileInfo> FFileInfos;
  std::vector<std::string> FFileNames;
  std::vector<std::string> FRealFileNames;
  mutable std::map<std::string, int> FFileInfoIdx;
  std::map<int, const void*> FExtractedFromArchive;
```

`FSourceFile`字段保存指向`.apk`文件的源文件流的指针：

```java
  clPtr<iIStream> FSourceFile;
public:
  clArchiveReader()
  : FFileInfos()
  , FRealFileNames()
  , FFileInfoIdx()
  , FSourceFile()
  {}
  virtual ~clArchiveReader()
  { CloseArchive(); }
```

`OpenArchive()`成员函数调用`Enumerate_ZIP()`来填充`FFileInfos`容器。`CloseArchive()`执行一些必要的清理工作：

```java
  bool OpenArchive( const clPtr<iIStream>& Source )
  {
   if ( !Source ) { return false; }
   if ( !CloseArchive() ) { return false; }
   if ( !Source->GetSize() ) { return false ; }
   FSourceFile = Source;
   return Enumerate_ZIP();
  }
  bool CloseArchive()
  {
    FFileInfos.clear();
    FFileInfoIdx.clear();
    FFileNames.clear();
    FRealFileNames.clear();
    ClearExtracted();
    FSourceFile = nullptr;
    return true;
  }
```

下面将详细描述长的`ExtractSingleFile()`方法。它接受来自归档的压缩文件名和一个包含文件数据的`iOStream`对象。`AbortFlag`是指向原子布尔标志的指针，用于多线程解压缩。解压缩器会不时地轮询它。如果值设置为`true`，则内部解压缩循环会提前终止，`ExtractSingleFile()`返回`false`。

`Progress`指针用于更新解压缩进程的进度，这也应该是原子操作。如果归档文件已加密，可以提供一个可选的`Password`参数：

```java
  bool ExtractSingleFile( const std::string& FileName,
    const std::string& Password, std::atomic<int>* AbortFlag,
    std::atomic<float>* Progress, const clPtr<iOStream>& Out );
```

接下来的两个方法使用`FFileInfos`向量来检查此归档中是否存在文件并获取其解压缩的大小：

```java
  bool FileExists( const std::string& FileName ) const
  {
    return GetFileIdx( FileName ) > -1;
  }
  uint64 GetFileSizeIdx( const std::string& FileName ) const
  {
    return ( Idx > -1 ) ? FFileInfos[ Idx ].FSize : 0;
  }
```

`GetFileDataIdx()`方法首先检查文件是否已经解压缩。在这种情况下，返回来自`FExtractedFromArchive`的指针：

```java
  const void* GetFileDataIdx( int Idx )
  {
    if ( Idx <= -1 ) { return nullptr; }
    if ( FExtractedFromArchive.count( Idx ) > 0 )
    {
      return FExtractedFromArchive[Idx]->GetDataConst();
    }
```

如果文件尚未解压缩，将调用`GetFileData_ZIP()`函数，并从`clBlob`返回一个已解包的内存块：

```java
    auto Blob = GetFileData_ZIP( Idx );
    if ( Blob )
    {
      FExtractedFromArchive[Idx] = Blob;
      return Blob->GetDataConst();
    }
    return nullptr;
  }
```

`GetFileIdx()`方法将`FileName`映射到`FFileInfos`向量内部的索引。它使用辅助的`FFileInfoIdx`对象来存储字符串到索引的对应关系：

```java
  int GetFileIdx( const std::string& FileName ) const
  {
    return ( FFileInfoIdx.count( FileName ) > 0 ) ? FFileInfoIdx[ FileName ] : -1;
  }
```

最后两个公共函数返回归档中的文件数量和每个文件的名称：

```java
  size_t GetNumFiles() const { return FFileInfos.size(); }
  std::string GetFileName( int Idx ) const
  { return FFileNames[Idx]; }
```

`clArchiveReader`类的私有部分声明了用于解压缩数据管理的内部方法。`Enumerate_ZIP()`方法通过读取归档头填充`FFileInfos`容器。`GetFileData_ZIP()`成员函数从归档中提取文件数据：

```java
private:
  bool Enumerate_ZIP();
  const void* GetFileData_ZIP( size_t Idx );
```

`ClearExtracted()`方法是从`CloseArchive()`中调用的。它会释放每个解压文件所分配的内存。这里的一切都是通过`clBlob`类使用 RAII 管理的：

```java
  void ClearExtracted()
  {
    FExtractedFromArchive.clear();
  }
```

让我们看看使用`ExtractSingleFile()`方法的`GetFileData_ZIP()`方法的实现：

```java
  clPtr<clBlob> clArchiveReader::GetFileData_ZIP( int Idx )
  {
    if ( FExtractedFromArchive.count( Idx ) > 0 )
    {
      return FExtractedFromArchive[ Idx ];
    }
```

创建包含解压缩数据的`clMemFileWriter`对象：

```java
    clPtr<clMemFileWriter> Out =
      clFileSystem::CreateMemWriter( "mem_blob",
        FFileInfos[ Idx ].FSize );
```

`ExtractSingleFile()`处理解压缩。在这里我们使用了一个阻塞调用（`AbortFlag`参数为`nullptr`）和一个空密码：

```java
    if ( ExtractSingleFile( FRealFileNames[ Idx ], "",
      nullptr, nullptr, Out ) )
    {
```

如果调用成功，我们从`clMemFileWriter`对象返回解压缩的内容：

```java
      return Out->GetBlob();
    }
    return make_intrusive<clBlob>();
  }
```

`ExtractSingleFile()`方法创建`zlib`读取对象，将读取器定位在压缩文件数据的开头，并调用`ExtractCurrentFile_ZIP()`方法以执行实际解压缩：

```java
  bool clArchiveReader::ExtractSingleFile(
    const std::string& FileName, const std::string& Password,
    std::atomic<int>* AbortFlag, std::atomic<float>* Progress,
    const clPtr<iOStream>& Out )
  {
    std::string ZipName = FileName;
    std::replace( ZipName.begin(), ZipName.end(), '\\', '/' );
    clPtr<iIStream> TheSource = FSourceFile;
    FSourceFile->Seek( 0 );
```

我们创建内部结构，允许`zlib`从我们的`iIStream`对象中读取。稍后在`Enumerate_ZIP()`中也会进行同样的操作。`fill_functions()`例程以及与此相关的所有回调都在本节下面描述：

```java
    zlib_filefunc64_def ffunc;
    fill_functions( TheSource.GetInternalPtr(), &ffunc );
    unzFile UnzipFile = unzOpen2_64( "", &ffunc );
    if ( unzLocateFile(UnzipFile, ZipName.c_str(), 0) != UNZ_OK )
    {
```

如果在归档中没有找到文件，则返回`false`：

```java
      return false;
    }
```

一旦定位了读取器，我们调用`ExtractCurrentFile_ZIP()`方法：

```java
    int ErrorCode = ExtractCurrentFile_ZIP( UnzipFile,
      Password.empty() ? nullptr : Password.c_str(),
      AbortFlag, Progress, Out );
    unzClose( UnzipFile );
    return ErrorCode == UNZ_OK;
  }
```

我们解压缩器的核心在于`ExtractCurrentFile_Zip`()。该方法接收一个内存块作为输入，读取文件的解压缩字节，并将其写入输出流：

```java
  int ExtractCurrentFile_ZIP( unzFile UnzipFile,
    const char* Password, std::atomic<int>* AbortFlag,
    std::atomic<float>* Progress, const clPtr<iOStream>& Out )
  {
    char FilenameInzip[1024];
    unz_file_info64 FileInfo;
```

`unzGetCurrentFileInfo64()`函数检索未压缩的文件大小。我们用它来计算总进度并将其写入`Progress`参数：

```java
    int ErrorCode = unzGetCurrentFileInfo64( UnzipFile,
      &FileInfo, FilenameInzip, sizeof( FilenameInzip ),
      nullptr, 0, nullptr, 0 );
    if ( ErrorCode != UNZ_OK ) { return ErrorCode; }
```

`unzOpenCurrentFilePassword()`调用初始化了解压缩过程：

```java
    ErrorCode = unzOpenCurrentFilePassword( uf, password );
    if ( ErrorCode != UNZ_OK ) { return err; }
```

方法的最后部分是一个循环，该循环读取一包解压缩的字节，并调用`Out`对象的`iOStream::Write`方法：

```java
    uint64_t FileSize = ( uint64_t )FileInfo.uncompressed_size;
```

在基于内存映射文件的示例实现中，我们将 64 位文件大小转换为`size_t`。这实际上在 32 位目标上打破了大于 2Gb 文件的支持。然而，这种权衡在大多数实际移动应用中是可以接受的，除非你正在编写通用的`.zip`解压缩器，当然：

```java
    Out->Reserve( ( size_t )FileSize );
    unsigned char Buffer[ WRITEBUFFERSIZE ];
    uint64_t TotalBytes = 0;
    int BytesRead = 0;
    do
    {
```

如果需要，我们可以通过检查`AbortFlag`指针（由另一个线程设置）来决定是否跳出循环：

```java
      if ( AbortFlag && *AbortFlag ) break;

```

`unzReadCurrentFile()`函数执行到输出流的解压缩：

```java
      BytesRead = unzReadCurrentFile( UnzipFile, Buffer, WRITEBUFFERSIZE );
      if ( BytesRead < 0 ) { break; }
      if ( BytesRead > 0 )
      {
        TotalBytes += BytesRead;
        Out->Write( Buffer, BytesRead );
      }
```

写入解压缩数据后，我们相应地更新`Progress`计数器：

```java
      if ( Progress )
      {
        *Progress = (float)TotalBytes / (float)FileSize;
      }
    }
    while ( BytesRead > 0 );
```

最后，我们关闭`UnzipFile`读取器对象：

```java
    ErrorCode = unzCloseCurrentFile( UnzipFile );
    return ErrorCode;
  }
```

归档中文件的枚举是通过另一个名为`Enumerate_ZIP()`的成员函数完成的：

```java
  bool Enumerate_ZIP()
  {
    clPtr<iIStream> TheSource = FSourceFile;
    FSourceFile->Seek( 0 );
```

首先，我们填充`zlib`所需的回调以读取自定义文件流，在本例中是我们的`iIStream`对象：

```java
    zlib_filefunc64_def ffunc;
    fill_functions( TheSource.GetInternalPtr(), &ffunc );
    unzFile UnzipFile = unzOpen2_64( "", &ffunc );
```

然后，读取归档的头部以确定压缩文件的数量：

```java
    unz_global_info64 gi;
    int ErrorCode = unzGetGlobalInfo64( uf, &gi );
```

对于每个压缩文件，我们提取稍后用于解压缩的信息：

```java
    for ( uLong i = 0; i < gi.number_entry; i++ )
    {
      if ( ErrorCode != UNZ_OK ) { break; }
      char filename_inzip[256];
      unz_file_info64 file_info;
      ErrorCode = unzGetCurrentFileInfo64( UnzipFile, &file_info, filename_inzip, sizeof(filename_inzip), nullptr, 0, nullptr, 0 );
      if ( ErrorCode != UNZ_OK ) { break; }
      if ( ( i + 1 ) < gi.number_entry )
      {
        ErrorCode = unzGoToNextFile( UnzipFile );
        if ( ErrorCode != UNZ_OK ) { break; }
      }
```

在每次迭代中，我们填充`sFileInfo`结构并将其存储在`FFileInfos`向量中：

```java
      sFileInfo Info;
      Info.FOffset = 0;
      Info.FCompressedSize = file_info.compressed_size;
      Info.FSize = file_info.uncompressed_size;
      FFileInfos.push_back( Info );
```

文件名中的所有反斜杠都被转换为在归档内路径元素之间起分隔作用的字符。`FFileInfoIdx`映射被填充，以便快速查找文件索引：

```java
      std::string TheName = Arch_FixFileName(filename_inzip);
      FFileInfoIdx[ TheName ] = ( int )FFileNames.size();
      FFileNames.emplace_back( TheName );
      FRealFileNames.emplace_back( filename_inzip );
    }
```

最后，我们清理`zlib`读取器对象并返回成功代码：

```java
    unzClose( UnzipFile );
    return true;
  }
```

让我们仔细看看`fill_functions()`方法。内存块包含在`iIStream`中，因此我们实现了一组`zlib`需要的回调，以便与我们的流类一起工作。第一个方法`zip_fopen()`对`iIStream`进行准备：

```java
  static voidpf ZCALLBACK zip_fopen ( voidpf opaque, const void* filename, int mode )
  {
    ( ( iIStream* )opaque )->Seek( 0 );
    return opaque;
  }
```

从`iIStream`读取字节的操作在`zip_fread()`中实现：

```java
  static uLong ZCALLBACK zip_fread ( voidpf opaque, voidpf stream, void* buf, uLong size )
  {
    iIStream* S = ( iIStream* )stream;
    int64 CanRead = ( int64 )size;
    int64 Sz = S->GetSize();
    int64 Ps = S->GetPos();
    if ( CanRead + Ps >= Sz ) { CanRead = Sz - Ps; }
    if ( CanRead > 0 ) {  S->Read( buf, ( uint64 )CanRead ); }
    else { CanRead = 0; }
    return ( uLong )CanRead;
  }
```

`zip_ftell()`函数告诉`iIStream`中的当前位置：

```java
  static ZPOS64_T ZCALLBACK zip_ftell(voidpf opaque, voidpf stream)
  {
    return ( ZPOS64_T )( ( iIStream* )stream )->GetPos();
  }
```

`zip_fseek()`例程设置读取指针，就像`libc`的`fseek()`一样：

```java
  static long ZCALLBACK zip_fseek ( voidpf  opaque, voidpf stream, ZPOS64_T offset, int origin )
  {
    iIStream* S = ( iIStream* )stream;
    int64 NewPos = ( int64 )offset;
    int64 Sz = ( int64 )S->GetSize();
    switch ( origin )
    {
      case ZLIB_FILEFUNC_SEEK_CUR:
        NewPos += ( int64 )S->GetPos(); break;
      case ZLIB_FILEFUNC_SEEK_END:
        NewPos = Sz - 1 - NewPos; break;
      case ZLIB_FILEFUNC_SEEK_SET: break;
      default:  return -1;
    }
    if ( NewPos >= 0 && ( NewPos < Sz ) )
    {
      S->Seek( ( uint64 )NewPos );
    }
    else
    {
      return -1;
    }
    return 0;
  }
```

对于`iIstream`类，`fclose()`和`ferror()`的类似操作是微不足道的：

```java
  static int ZCALLBACK zip_fclose( voidpf opaque, voidpf stream )
  {
    return 0;
  }
  static int ZCALLBACK zip_ferror( voidpf opaque, voidpf stream )
  {
    return 0;
  }
```

辅助`fill_functions()`例程填充了`zlib`使用的回调结构：

```java
  void fill_functions( iIStream* Stream, zlib_filefunc64_def* pzlib_filefunc_def )
  {
    pzlib_filefunc_def->zopen64_file = zip_fopen;
    pzlib_filefunc_def->zread_file = zip_fread;
    pzlib_filefunc_def->zwrite_file = NULL;
    pzlib_filefunc_def->ztell64_file = zip_ftell;
    pzlib_filefunc_def->zseek64_file = zip_fseek;
    pzlib_filefunc_def->zclose_file = zip_fclose;
    pzlib_filefunc_def->zerror_file = zip_ferror;
    pzlib_filefunc_def->opaque = Stream;
  }
```

这就是关于低级解压缩细节的全部内容。让我们进入更友好的抽象和包装领域。`clArchiveMountPoint`类包装了`clArchiveReader`的一个实例，并实现了`CreateReader()`、`FileExists()`和`MapName()`方法：

```java
  class clArchiveMountPoint: public iMountPoint
  {
  public:
    explicit clArchiveMountPoint( const clPtr<ArchiveReader>& R )
    : FReader(R) {}
```

`CreateReader()`方法实例化`clMemRawFile`类并附加一个提取的内存块：

```java
    virtual clPtr<iRawFile> CreateReader(
      const std::string& VirtualName ) const
    {
      std::string Name = Arch_FixFileName( VirtualName );
      const void* DataPtr  = FReader->GetFileData( Name );
      size_t FileSize = static_cast<size_t>( FReader->GetFileSize( Name ) );
      auto File = clMemRawFile::CreateFromManagedBuffer( DataPtr, FileSize );
      File->SetFileName( VirtualName );
      File->SetVirtualFileName( VirtualName );
      return File;
    }
```

`FileExists()`方法是对`clArchiveReader::FileExists()`的间接调用：

```java
    virtual bool FileExists( const std::string& VirtualName )const
    {
      return FReader->FileExists( Arch_FixFileName( VirtualName ) );
    }
```

对于此类挂载点，`MapName()`的实现是微不足道的：

```java
    virtual std::string MapName( const std::string& VirtualName ) const
    { return VirtualName; }
```

私有部分只包含对`clArchiveReader`对象的引用：

```java
  private:
    clPtr<clArchiveReader> FReader;
  };
```

显而易见，简单的`clArchiveMountPoint`的缺点在于其非异步阻塞实现。构造函数接受一个完全初始化的`clArchiveReader`对象，这意味着我们需要阻塞直到`clArchiveReader::OpenArchive()`完成其工作。克服此问题的一种方法是在不同的线程上运行`OpenArchive()`，在任务队列中，并在解析归档后创建挂载点。当然，所有后续调用`CreateReader()`以期望从此挂载点获取数据的操作应该推迟，直到收到信号。我们鼓励读者使用前一章讨论的`clWorkerThread`类实现这种异步机制。更复杂的归档挂载点实现可以接受构建的`clArchiveReader`并自行调用`OpenArchive()`。这需要更复杂的架构，因为`clFileSystem`和/或`clArchiveMountPoint`类应该能够访问专用的工人线程。然而，它本质上将所有耗时的解压缩操作复杂性隐藏在简洁的接口背后。

# 访问应用程序资产

要在 Android 上的 C++代码中访问`.apk`包内的数据，我们需要使用 Java 代码获取`.apk`的路径，并使用 JNI 将结果传递给我们的 C++代码。

在`onCreate()`方法中，将来自`getApplication().getApplicationInfo().sourceDir`的值传递给我们的本地代码：

```java
  @Override protected void onCreate( Bundle icicle )
  {
    onCreateNative( getApplication().getApplicationInfo().sourceDir );
  }
  public static native void onCreateNative( String APKName );
```

`onCreateNative()`的实现可以在`1_ArchiveFileAccess\jni\Wrappers.cpp`中找到，如下所示：

```java
  extern "C"
  {
    JNIEXPORT void JNICALL
    Java_com_packtpub_ndkmastering_AppActivity_onCreateNative( JNIEnv* env, jobject obj, jstring APKName )
    {
      g_APKName = ConvertJString( env, APKName );
      LOGI( "APKName = %s", g_APKName.c_str() );
      OnStart( g_APKName );
    }
  }
```

我们使用`ConvertJString()`函数将`jstring`转换为`std::string`。JNI 方法`GetStringUTFChars()`和`ReleaseStringUTFChars()`获取和释放指向字符串的 UTF8 编码字符数组的指针：

```java
  std::string ConvertJString( JNIEnv* env, jstring str )
  {
    if ( !str ) { return std::string(); }
    const jsize len = env->GetStringUTFLength( str );
    const char* strChars = env->GetStringUTFChars( str, ( jboolean* )0 );
    std::string Result( strChars, len );
    env->ReleaseStringUTFChars( str, strChars );
    return Result;
  }
```

在`main.cpp`文件中的`OnStart()`回调中实现了简单的使用示例。它挂载路径，在 Android 上创建归档挂载点，打开归档`test.zip`并列出其内容。在桌面上，此代码运行并读取存储在`assets/test.zip`的`test.zip`：

```java
  void OnStart( const std::string& RootPath )
  {
    auto FS = make_intrusive<clFileSystem>();
    FS->Mount( "" );
    FS->Mount( RootPath );
    FS->AddAliasMountPoint( RootPath, "assets" );
    const char* ArchiveName = "test.zip";
    auto File = FS->CreateReader( ArchiveName );
    auto Reader = make_intrusive<clArchiveReader>();
    if ( !Reader->OpenArchive( File ) )
    {
      LOGI( "Bad archive: %s", ArchiveName );
      return;
    }
```

遍历此归档中的所有文件并打印它们的名字和内容：

```java
    for ( size_t i = 0; i != Reader->GetNumFiles(); i++ )
    {
      LOGI( "File[%i]: %s", i,
      Reader->GetFileName( i ).c_str() );
      const char* Data = reinterpret_cast<const char*>( Reader->GetFileDataIdx( i ) );
      LOGI( "Data: %s", std::string( Data,
        static_cast<size_t>(
          Reader->GetFileSizeIdx( i ) ) ).c_str() );
    }
  }
```

查看并尝试`1_ArchiveFileAccess`示例。它为在桌面上调试 Android 文件访问代码提供了很好的体验。使用`make all`构建桌面环境，使用`ndk-build & ant debug`构建 Android。

# 概述

在本章中，我们学习了如何以与平台无关的方式通过 C++处理文件和`.apk`归档。我们将在后续章节中使用此功能来访问文件。
