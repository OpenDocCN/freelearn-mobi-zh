# 第五章. 跨平台音频流

在本章中，我们考虑构建交互式移动应用程序所需的最后一个非视觉组件。我们寻找的是一个真正可移植的音频播放实现，适用于 Android 和桌面 PC。我们建议使用 OpenAL 库，因为它在桌面平台上已经非常成熟。音频播放本质上是一个异步过程，因此解码并将数据提交给声音 API 应该在单独的线程上完成。我们将基于第三章的*网络编程*中的多线程代码创建一个音频流库。

原始未压缩音频可能占用大量内存，因此经常使用不同种类的压缩格式。我们将在本章考虑其中一些格式，并展示如何使用原生 C++代码和流行的第三方库在 Android 中播放它们。

# 初始化和播放

本章节我们将使用跨平台的 OpenAL 音频库。为了使所有示例保持简洁且自包含，我们从可以播放未压缩`.wav`文件的最小化示例开始。

让我们简要描述一下产生声音需要做些什么。OpenAL 的例程处理播放和录音过程中遇到的对象。`ALCdevice`对象代表音频硬件的一个单元。由于多个线程可能同时产生声音，因此引入了另一个名为`ALCcontext`的对象。首先，应用程序打开一个设备，然后创建一个上下文并将其附加到打开的设备上。每个上下文都维护着多个`Audio Source`对象，因为即使单个应用程序也可能需要同时播放多个声音。

我们越来越接近实际的声音产生。还需要一个对象作为波形容器，这称为缓冲区。音频录音可能相当长，所以我们不会将整个声音作为一个缓冲区提交。我们以小块读取样本，并使用几个缓冲区（通常是一对）将这些块提交到音频源的队列中。

以下伪代码描述了如何播放完全适合内存的声音：

1.  首先打开一个设备，创建一个上下文，并将上下文附加到设备上。

1.  创建一个音频源，分配一个声音缓冲区。

1.  将波形数据加载到缓冲区中。

1.  将缓冲区入队到音频源。

1.  等待播放完成。

1.  销毁缓冲区、源和上下文，并关闭设备。

在第 5 步有一个明显的问题。我们无法将应用程序的 UI 线程阻塞几秒钟，因此声音播放必须是异步的。幸运的是，OpenAL 调用是线程安全的，我们可以在没有自己进行任何 OpenAL 同步的情况下在单独的线程中执行播放。

让我们检查示例`1_InitOpenAL`。为了在步骤 3 中执行波形加载并尽可能保持代码简单，我们取一个`.wav`文件并将其加载到`clBlob`对象中。在步骤 2 中，我们创建一个音频源和缓冲区，其参数与`WAV`头中的参数相对应。步骤 1、4 和 6 仅包含一些 OpenAL API 调用。步骤 5 通过在原子条件变量上进行忙等待循环来完成。

这个示例的本地 C++入口点从创建一个单独的音频线程开始，该线程声明为全局对象`g_Sound`。`g_FS`对象包含`clFileSystem`类的实例，用于从文件加载音频数据：

```java
clSoundThread g_Sound;
clPtr<clFileSystem> g_FS;
int main()
{
  g_FS = make_intrusive<clFileSystem>();
  g_FS->Mount( "." );
  g_Sound.Start();
  g_Sound.Exit( true );
  return 0;
}
```

`clSoundThread`类包含一个 OpenAL 设备和上下文。音频源和缓冲区句柄也为此单一源单一缓冲区的示例而声明：

```java
class clSoundThread: public iThread
{
  ALCdevice* FDevice;
  ALCcontext* FContext;
  ALuint FSourceID;
  ALuint FBufferID;
```

`Run()`方法负责所有初始化、加载和结束工作：

```java
  virtual void Run()
  {
```

要使用 OpenAL 例程，我们应该加载库。对于 Android、Linux 和 OS X，实现很简单，我们只需使用静态链接库即可。然而，对于 Windows，我们加载`OpenAL32.dll`文件，并从动态链接库中获取所有必要的函数指针：

```java
    LoadAL();
```

首先，我们打开一个设备并创建一个上下文。`alcOpenDevice()`的`nullptr`参数意味着我们正在使用默认的音频设备：

```java
    FDevice = alcOpenDevice( nullptr );
    FContext = alcCreateContext( FDevice, nullptr );
    alcMakeContextCurrent( FContext );
```

然后我们创建一个音频源并将其音量设置为最大级别：

```java
    alGenSources( 1, &FSourceID );
    alSourcef( FSourceID, AL_GAIN, 1.0 );
```

波形的加载，对应于我们伪代码中的第 3 步，通过将整个`.wav`文件读取到`clBlob`对象中完成：

```java
    auto data = LoadFileAsBlob( g_FS, "test.wav" );
```

可以通过以下方式访问头文件：

```java
    const sWAVHeader* Header = ( const sWAVHeader* )Blob->GetData();
```

我们从`clBlob`中复制字节到声音缓冲区，跳过头文件对应大小的字节数：

```java
    const unsigned char* WaveData = ( const unsigned char* )Blob->GetData() +
      sizeof( sWAVHeader );
    PlayBuffer( WaveData, Header->DataSize,
      Header->SampleRate );
```

现在让我们忙等待声音播放完毕：

```java
    while ( IsPlaying() ) {}
```

最后，我们停止源，删除所有对象并卸载 OpenAL 库：

```java
    alSourceStop( FSourceID );
    alDeleteSources( 1, &FSourceID );
    alDeleteBuffers( 1, &FBufferID );
    alcDestroyContext( FContext );
    alcCloseDevice( FDevice );
    UnloadAL();
  }
```

`clSoundThread`类还包含两个辅助方法。`IsPlaying()`方法通过请求其状态来检查声音是否仍在播放：

```java
  bool IsPlaying() const
  {
    int State;
    alGetSourcei( FSourceID, AL_SOURCE_STATE, &State );
    return State == AL_PLAYING;
  }
```

`PlayBuffer()`方法创建一个缓冲区对象，用`Data`参数中的波形填充它并开始播放：

```java
  void PlayBuffer( const unsigned char* Data, int DataSize, int SampleRate )
  {
    alBufferData( FBufferID, AL_FORMAT_MONO16,
      Data, DataSize, SampleRate );
    alSourcei( FSourceID, AL_BUFFER, FBufferID );
    alSourcei( FSourceID, AL_LOOPING, AL_FALSE );
    alSourcef( FSourceID, AL_GAIN, 1.0f );
    alSourcePlay( FSourceID );
  }
```

上述代码依赖于两个全局函数。`Env_Sleep()`函数以给定的毫秒数休眠。Windows 版本的代码与 Android 和 OS X 略有不同：

```java
  void Env_Sleep( int Milliseconds )
  {
    #if defined(_WIN32)
      Sleep( Milliseconds );
    #elif defined(ANDROID)
      std::this_thread::sleep_for(
        std::chrono::milliseconds( Milliseconds ) );
    #else
      usleep( static_cast<useconds_t>( Milliseconds ) * 1000 );
    #endif
  }
```

### 注意

我们在 Windows 上使用`Sleep()`以与一些缺乏对`std::chrono`支持的 MinGW 发行版兼容。如果你想要使用 Visual Studio，只需坚持使用`std::this_thread::sleep_for()`。

`LoadFileAsBlob()`函数使用提供的`clFileSystem`对象将文件内容加载到内存块中。我们在后续的大部分代码示例中重复使用这个例程。

```java
  clPtr<clBlob> LoadFileAsBlob( const clPtr<clFileSystem>& FileSystem, const std::string& Name )
  {
    auto Input = FileSystem->CreateReader( Name );
    auto Res = make_intrusive<clBlob>();
    Res->AppendBytes( Input->MapStream(), Input->GetSize() );
    return Res;
  }
```

如果你在桌面机器上通过输入`make all`编译并运行此示例，你应该能听到一个短暂的叮当声。在我们结束 Android 应用程序之前，让我们进一步了解如何进行声音流处理。

# 流式声音

现在我们能够播放短音频样本，是时候将音频系统组织成类，并仔细查看`2_Streaming`示例了。长音频样本（如背景音乐）在解压缩形式下需要大量内存。流式传输是一种小块小块地、逐片解压缩它们的技术。`clAudioThread`类负责初始化并处理除播放声音之外的所有工作：

```java
  class clAudioThread: public iThread
  {
  public:
    clAudioThread()
    : FDevice( nullptr )
    , FContext( nullptr )
    , FInitialized( false )
    {}
    virtual void Run()
    {
      if ( !LoadAL() ) { return; }
      FDevice = alcOpenDevice( nullptr );
      FContext = alcCreateContext( FDevice, nullptr );
      alcMakeContextCurrent( FContext );
      FInitialized = true;
      while ( !IsPendingExit() ) { Env_Sleep( 100 ); }
      alcDestroyContext( FContext );
      alcCloseDevice( FDevice );
      UnloadAL();
    }
```

此方法用于将音频线程的开始与其用户同步：

```java
    virtual void WaitForInitialization() const
    {
      while ( !FInitialized ) {}
    }
  private:
    std::atomic<bool> FInitialized;
    ALCdevice* FDevice;
    ALCcontext* FContext;
  };
```

`clAudioSource`类代表单一声音产生实体。波形数据不是存储在源本身中，我们推迟对`clAudioSource`类的描述。现在，我们介绍提供下一个音频缓冲区数据的`iWaveDataProvider`接口类。对`iWaveDataProvider`实例的引用存储在`clAudioSource`类中：

```java
  class iWaveDataProvider: public iIntrusiveCounter
  {
  public:
```

音频信号属性存储在这三个字段中：

```java
    int FChannels;
    int FSamplesPerSec;
    int FBitsPerSample;
    iWaveDataProvider()
    : FChannels( 0 )
    , FSamplesPerSec( 0 )
    , FBitsPerSample( 0 ) {}
```

两个纯虚方法提供了对音频源当前播放的波形数据的访问。它们应在实际的解码器子类中实现：

```java
    virtual unsigned char* GetWaveData() = 0;
    virtual size_t GetWaveDataSize() const = 0;
```

`IsStreaming()`方法告诉我们此提供程序是否代表连续流或如前一个示例中的单个音频数据块。`StreamWaveData()`方法加载、解码或生成`GetWaveData()`函数访问的缓冲区中的值；它通常也在子类中实现。当`clAudioSource`需要更多音频数据以排队进入缓冲区时，它会调用`StreamWaveData()`方法：

```java
    virtual bool IsStreaming() const { return false; }
    virtual int StreamWaveData( int Size ) { return 0; }
```

最后一个辅助函数返回 OpenAL 使用的内部数据格式。这里我们只支持每样本 8 位或 16 位的立体声和单声道信号：

```java
    ALuint GetALFormat() const
    {
      if ( FBitsPerSample == 8 )
        return ( FChannels == 2 ) ?
          AL_FORMAT_STEREO8 : AL_FORMAT_MONO8;
      if ( FBitsPerSample == 16 )
        return ( FChannels == 2 ) ?
          AL_FORMAT_STEREO16 : AL_FORMAT_MONO16;
      return AL_FORMAT_MONO8;
    }
  };
```

我们的基本声音解码是在`clStreamingWaveDataProvider`类中完成的。它包含`FBuffer`数据向量和其中的有用字节数：

```java
  class clStreamingWaveDataProvider: public iWaveDataProvider
  {
  public:
    clStreamingWaveDataProvider()
    : FBufferUsed( 0 )
    {}
    virtual bool IsStreaming() const override
    { return true; }
    virtual unsigned char* GetWaveData() override
    { return ( unsigned char* )&FBuffer[0]; }
    virtual size_t GetWaveDataSize() const override
    { return FBufferUsed; }
    std::vector<char> FBuffer;
    size_t FBufferUsed;
  };
```

我们准备描述实际执行繁重任务的`clAudioSource`类。构造函数创建一个 OpenAL 音频源对象，设置音量级别并禁用循环：

```java
  class clAudioSource: public iIntrusiveCounter
  {
  public:
    clAudioSource()
    : FWaveDataProvider( nullptr )
    , FBuffersCount( 0 )
    {
      alGenSources( 1, &FSourceID );
      alSourcef( FSourceID, AL_GAIN, 1.0 );
      alSourcei( FSourceID, AL_LOOPING, AL_FALSE );
    }
```

我们有两种不同的使用场景。如果附加的`iWaveDataProvider`支持流式传输，我们需要创建并维护至少两个声音缓冲区。这两个缓冲区都被加入到 OpenAL 播放队列中，并在其中一个缓冲区播放完成后进行交换。在每次交换事件中，我们调用`iWaveDataProvider`的`StreamWaveData()`方法将数据流式传输到下一个音频缓冲区。如果`iWaveDataProvider`不支持流式传输，我们只需要一个在开始时初始化的单个缓冲区。

`Play()`方法用解码后的数据填充两个缓冲区，并调用`alSourcePlay()`开始播放：

```java
    void Play()
    {
      if ( IsPlaying() ) { return; }
      if ( !FWaveDataProvider ) { return; }
      int State;
      alGetSourcei( FSourceID, AL_SOURCE_STATE, &State );
      if ( State != AL_PAUSED && FWaveDataProvider->IsStreaming() )
      {
        UnqueueAll();
        StreamBuffer( FBufferID[0], BUFFER_SIZE );
        StreamBuffer( FBufferID[1], BUFFER_SIZE );
        alSourceQueueBuffers( FSourceID, 2, &FBufferID[0] );
      }
      alSourcePlay( FSourceID );
    }
```

`Stop()`和`Pause()`方法分别调用适当的 OpenAL 例程来停止和暂停播放：

```java
    void Stop()
    {
      alSourceStop( FSourceID );
    }
    void Pause()
    {
      alSourcePause( FSourceID );
      UnqueueAll();
    }
```

`LoopSound()`和`SetVolume()`方法控制播放参数：

```java
    void LoopSound( bool Loop )
    {
    alSourcei( FSourceID, AL_LOOPING, Loop ? 1 : 0 );
    }
    void SetVolume( float Volume )
    {
      alSourcef( FSourceID, AL_GAIN, Volume );
    }
```

`IsPlaying()`方法是从上一个示例中复制而来的：

```java
    bool IsPlaying() const
    {
      int State;
      alGetSourcei( FSourceID, AL_SOURCE_STATE, &State );
      return State == AL_PLAYING;
    }
```

`StreamBuffer()`方法将新产生的音频数据写入其中一个缓冲区：

```java
    int StreamBuffer( unsigned int BufferID, int Size )
    {
      int ActualSize = FWaveDataProvider->StreamWaveData( Size );
      alBufferData( BufferID,
        FWaveDataProvider->GetALFormat(),
        FWaveDataProvider->GetWaveData(),
        ( int )FWaveDataProvider->GetWaveDataSize(),
        FWaveDataProvider->FSamplesPerSec );
      return ActualSize;
    }
```

`Update()` 方法应该足够频繁地被调用，以防止音频缓冲区出现下溢。然而，只有当附加的 `iWaveDataProvider` 表示音频流时，此方法才重要：

```java
    void Update( float DeltaSeconds )
    {
      if ( !FWaveDataProvider ) { return; }
      if ( !IsPlaying() ) { return; }
      if ( FWaveDataProvider->IsStreaming() )
      {
```

我们询问 OpenAL 已经处理了多少个缓冲区：

```java
      int Processed;
      alGetSourcei( FSourceID, AL_BUFFERS_PROCESSED, &Processed );
```

我们从队列中移除每个已处理的缓冲区，并调用 `StreamBuffer()` 来解码更多数据。最后，我们将缓冲区重新加入播放队列：

```java
      while ( Processed-- )
      {
        unsigned int BufID;
        alSourceUnqueueBuffers( FSourceID, 1, &BufID );
        StreamBuffer( BufID, BUFFER_SIZE );
        alSourceQueueBuffers( FSourceID, 1, &BufID );
      }
    }
  }
```

析构函数会停止播放并销毁 OpenAL 音频源和缓冲区：

```java
  virtual ~clAudioSource()
  {
    Stop();
    alDeleteSources( 1, &FSourceID );
    alDeleteBuffers( FBuffersCount, &FBufferID[0] );
  }
```

`BindWaveform()` 方法将一个新的 `iWaveDataProvider` 附加到这个音频源实例：

```java
  void BindWaveform( clPtr<iWaveDataProvider> Wave )
  {
    FWaveDataProvider = Wave;
    if ( !Wave ) { return; }
```

对于流式的 `iWaveDataProvider`，我们需要两个缓冲区。一个正在播放，另一个正在更新：

```java
    if ( FWaveDataProvider->IsStreaming() )
    {
      FBuffersCount = 2;
      alGenBuffers( FBuffersCount, &FBufferID[0] );
    }
    else
```

如果附加的波形不是流式，或者更具体地说，它不是压缩的，我们会创建一个单一缓冲区并将所有数据复制到其中：

```java
    {
      FBuffersCount = 1;
      alGenBuffers( FBuffersCount, &FBufferID[0] );
      alBufferData( FBufferID[0],
        FWaveDataProvider->GetALFormat(),
        FWaveDataProvider->GetWaveData(),
        ( int )FWaveDataProvider->GetWaveDataSize(),
        FWaveDataProvider->FSamplesPerSec );
      alSourcei( FSourceID, AL_BUFFER, FBufferID[0] );
    }
  }
```

私有方法 `UnqueueAll()` 使用 `alSourceUnqueueBuffers()` 来清除 OpenAL 播放队列：

```java
private:
  void UnqueueAll()
  {
    int Queued;
    alGetSourcei( FSourceID, AL_BUFFERS_QUEUED, &Queued );
    if ( Queued > 0 )
    {
      alSourceUnqueueBuffers( FSourceID, Queued, &FBufferID[0] );
    }
  }
```

类的尾部部分定义了附加的 `iWaveDataProvider` 的引用，OpenAL 对象的内部句柄以及已分配缓冲区的数量：

```java
  clPtr<iWaveDataProvider> FWaveDataProvider;
  unsigned int FSourceID;
  unsigned int FBufferID[2];
  int FBuffersCount;
};
```

为了展示一些基本的流式处理能力，我们更改了 `1_InitOpenAL` 的示例代码，并创建了一个带有附加音调发生器的音频源，如下代码所示：

```java
class clSoundThread: public iThread
{
  virtual void Run()
  {
    g_Audio.WaitForInitialization();
    auto Src = make_intrusive<clAudioSource>();
    Src->BindWaveform( make_intrusive<clToneGenerator>() );
    Src->Play();
    double Seconds = Env_GetSeconds();
    while ( !IsPendingExit() )
    {
      float DeltaSeconds = static_cast<float>( Env_GetSeconds() - Seconds );
      Src->Update( DeltaSeconds );
      Seconds = Env_GetSeconds();
    }
  }
};
```

在此示例中，我们故意避免了解压缩声音的问题，以便专注于流式处理逻辑。因此，我们从程序生成的声音开始。`clToneGenerator` 类重写了 `StreamWaveData()` 方法并生成正弦波，即纯音调。为了避免可听见的故障，我们必须仔细采样正弦函数并记住最后一个生成样本的整数索引。这个索引存储在 `FLastOffset` 字段中，并在每次迭代中的计算中使用。

类的构造函数将音频参数设置为 16 位 44.1kHz，并在 `FBuffer` 容器中分配一些空间。这个音调的基本频率设置为 440 Hz：

```java
class clToneGenerator : public clStreamingWaveDataProvider
{
public:
  clToneGenerator()
  : FFrequency( 440.0f )
  , FAmplitude( 350.0f )
  , FLastOffset( 0 )
  {
    FBufferUsed = 100000;
    FBuffer.resize( 100000 );
    FChannels = 2;
    FSamplesPerSec = 44100;
    FBitsPerSample = 16;
  }
```

在 `StreamWaveData()` 中，我们检查 `FBuffer` 向量中是否有可用空间，并在必要时重新分配它：

```java
  virtual int StreamWaveData( int Size )
  {
    if ( Size > static_cast<int>( FBuffer.size() ) )
    {
      FBuffer.resize( Size );
      LastOffset = 0;
    }
```

最后，我们计算音频样本。频率会根据样本数量重新计算：

```java
    const float TwoPI = 2.0f * 3.141592654f;
    float Freq = TwoPI * FFrequency /
      static_cast<float>( FSamplesPerSec );
```

由于我们需要 `Size` 字节，并且我们的信号包含两个声道，每个声道 16 位样本，因此我们需要总共 `Size/4` 个样本：

```java
    for ( int i = 0 ; i < Size / 4 ; i++ )
    {
      float t = Freq * static_cast<float>( i + LastOffset );
      float val = FAmplitude * std::sin( t );
```

我们将浮点数值转换为 16 位有符号整数，并将此整数的低字节和高字节放入 `FBuffer` 中。对于每个声道，我们存储两个字节：

```java
      short V = static_cast<short>( val );
      FBuffer[i * 4 + 0] = V & 0xFF;
      FBuffer[i * 4 + 1] = V >> 8;
      FBuffer[i * 4 + 2] = V & 0xFF;
      FBuffer[i * 4 + 3] = V >> 8;
    }
```

计算后，我们增加样本计数并取余数，以避免计数器中的整数溢出：

```java
    LastOffset += Size / 4;
    LastOffset %= FSamplesPerSec;
    return ( FBufferUsed = Size );
  }
  float FFrequency;
  float FAmplitude;
private:
  int LastOffset;
};
```

编译后的示例将产生一个 440 Hz 的纯音调。我们鼓励您更改 `clToneGenerator::FFrequency` 的值，看看它是如何工作的。您甚至可以使用此示例为您的乐器创建一个简单的音叉应用程序。至于乐器，让我们生成一些模仿弦乐器的音频数据。

# 弦乐器的音乐模型

让我们使用前一个示例的代码来实现一个简单的弦乐器物理模型。稍后你可以使用这些例程为 Android 创建一个小型的交互式合成器。

弦被建模为一系列垂直振动的点质量。严格来说，我们求解具有特定初始和边界条件的线性一维波动方程。声音是通过在声音接收位置取得解的值来产生的。

我们需要`clGString`类来存储所有的模型值和最终结果。`GenerateSound()`方法会预先计算字符串参数，并相应地调整数据容器的大小：

```java
class clGString
{
public:
  void GenerateSound()
  {
    // 4 seconds, 1 channel, 16 bit
    FSoundLen  = 44100 * 4 * 2;
    FStringLen = 200;
```

`Frc`值是声音的规范化基频。泛音是由物理模型隐式创建的：

```java
    float Frc = 0.5f;
    InitString( Frc );
    FSamples.resize( FsoundLen );
    FSound.resize( FsoundLen );
    float MaxS = 0;
```

在初始化阶段之后，我们通过在循环中调用`Step()`方法来执行波动方程的积分。`Step()`成员函数返回弦在接收位置处的位移：

```java
    for ( int i = 0; i < FSoundLen; i++ )
    {
      FSamples[i] = Step();
```

在每一步，我们将值限制在最大值：

```java
      if ( MaxS < fabs(FSamples[i]) )
      MaxS = fabs( FSamples[i] );
    }
```

最后，我们将浮点数值转换为有符号短整型。为了避免溢出，每个样本都要除以`MaxS`的值：

```java
    const float SignedShortMax = 32767.0f;
    float k = SignedShortMax / MaxS;
    for ( int i = 0; i < FSoundLen; i++ )
    {
      FSound [i] = FSamples [i] * k;
    }
  }
  std::vector<short int> FSound;
private:
  int FPickPos;
  int FSoundLen;
  std::vector<float> FSamples;
  std::vector<float> FForce;
  std::vector<float> FVel;
  std::vector<float> FPos;
  float k1, k2;
  int FStringLen;
  void InitString(float Freq)
  {
    FPos.resize(FStringLen);
    FVel.resize(FStringLen);
    FForce.resize(FStringLen);
    const float Damping = 1.0f / 512.0f;
    k1 = 1 - Damping;
    k2 = Damping / 2.0f;
```

我们将声音接收器放置在靠近末尾的位置：

```java
    FPickPos = FStringLen * 5 / 100;
    for ( int i = 0 ; i < FStringLen ; i++ )
    {
      FVel[i] = FPos[i] = 0;
    }
```

为了获得更好的结果，我们在弦元素的质地上产生轻微的变化：

```java
    for ( int i = 1 ; i < FStringLen - 1 ; i++ )
    {
      float m = 1.0f + 0.5f * (frand() - 0.5f);
      FForce[i] = Freq / m;
    }
```

在开始时，我们为弦的第二部分设置非零速度：

```java
    for ( int i = FStringLen/2; i < FStringLen - 1; i++ )
    {
      FVel[i] = 1;
    }
  }
```

`frand()`成员函数返回 0..1 范围内的伪随机浮点值：

```java
  inline float frand()
  {
    return static_cast<float>( rand() ) / static_cast<float>( RAND_MAX );
  }
```

### 注意

如果你的编译器支持，使用`std::random`是获取伪随机数的首选方式。

这是使用新的 C++11 标准库生成 0…1 范围内均匀分布的伪随机浮点数的方法：

```java
  std::random_device rd;
  std::mt19937 gen( rd() );
  std::uniform_real_distribution<> dis( 0.0, 1.0 );
  float frand()
  {
    return static_cast<float>( dis( gen ) );
  }
```

尽管这段简短的代码片段在我们的源代码包中未使用，但它可能对你有用。让我们回到我们示例的代码。

`Step()`方法进行单步操作并整合弦运动的方程。在步骤结束时，从`FPos`向量在`FPickPos`位置的值作为声音的下一个样本。对于熟悉数值方法的读者来说，可能看起来很奇怪，因为没有指定时间步长，它是隐式为 1/44100 秒的：

```java
  float Step()
  {
```

首先，我们强制施加边界条件，即弦两端的固定端点：

```java
    FPos[0] = FPos[FStringLen - 1] = 0;
    FVel[0] = FVel[FStringLen - 1] = 0;
```

根据胡克定律（[`en.wikipedia.org/wiki/Hooke's_law`](http://en.wikipedia.org/wiki/Hooke's_law)），力与伸长量成正比：

```java
    for ( int i = 1 ; i < FStringLen - 1 ; i++ )
    {
      float d = (FPos[i - 1] + FPos[i + 1]) * 0.5f - FPos[i];
      FVel[i] += d * FForce[i];
    }
```

为了确保数值稳定性，我们应用一些人工阻尼，并取相邻速度的平均值。如果不这样做，会产生一些不想要的声音：

```java
    for ( int i = 1 ; i < FStringLen - 1 ; i++ )
    {
      FVel[i] = FVel[i] * k1 +
        (FVel[i - 1] + FVel[i + 1]) * k2;
    }
```

最后，我们更新位置：

```java
    for ( int i = 1 ; i < FStringLen ; i++ )
    {
      FPos[i] += FVel[i];
    }
```

为了记录我们的声音，我们只取弦的一个位置：

```java
    return FPos[FPickPos];
    }
  };
```

`1_InitOpenAL`示例可以轻松修改，以生成字符串声音，而不是加载`.wav`文件。我们创建`clGString`实例并调用`GenerateSound()`方法。之后，我们获取`FSound`向量并将其提交给音频源的`PlayBuffer()`方法：

```java
  clGString String;
  String.GenerateSound();
  const unsigned char* Data = (const unsigned char*)&String.FSound[0];
  PlayBuffer( Data, (int)String.FSound.size() );
```

在这里，采样率被硬编码为 44100 Hz。尝试`3_GuitarStringSound`示例以获取完整代码并亲自聆听。请注意，由于在播放声音之前需要进行大量预计算，启动时间可能会稍长。然而，代码非常简单，我们将其作为一个练习留给读者，让他们为 Android 编译，并从后续示例中获取所有必要的 makefile 和包装器。同时，我们将处理那些可以立即在 Android 上运行的内容。

# 解码压缩音频

现在我们已经实现了基本的音频流系统，是时候使用几个第三方库来读取压缩的音频文件了。基本上，我们需要做的是覆盖`clStreamingWaveDataProvider`类中的`StreamWaveData()`函数。这个函数反过来调用`ReadFromFile()`方法，实际解码就在这里完成。解码器的初始化在构造函数中进行，对于抽象的`iDecodingProvider`类，我们只存储对数据块引用。文件的所有压缩数据都存储在`clBlob`对象中：

```java
  class iDecodingProvider: public StreamingWaveDataProvider
  {
  protected:
    virtual int ReadFromFile( int Size, int BytesRead ) = 0;
    clPtr<clBlob> FRawData;
  public:
    bool FLoop;
    bool FEof;
    iDecodingProvider( const clPtr<clBlob>& Blob )
    : FRawData( Blob )
    , FLoop( false )
    , FEof( false )
    {}
    virtual bool IsEOF() const { return FEof; }
```

`StreamWaveData()`方法负责解码工作。前几行确保`FBuffer`有足够的空间来包含解码后的数据：

```java
    virtual int StreamWaveData( int Size ) override
    {
      int OldSize = ( int )FBuffer.size();
      if ( Size > OldSize )
      {
```

重新分配缓冲区后，我们用零填充新字节，因为非零值可能会产生意外的噪音：

```java
        FBuffer.resize( Size, 0 );
      }
      if ( FEof ) { return 0; }
```

由于`ReadFromFile()`可能会返回不充分的数据，我们以循环的方式调用它，并增加读取的字节数：

```java
      int BytesRead = 0;
      while ( BytesRead < Size )
      {
        int Ret = ReadFromFile( Size, BytesRead );
        if ( Ret > 0 ) BytesRead += Ret;

```

`ReadFromFile()`返回零意味着我们已达到流末尾：

```java
        else if ( Ret == 0 )
        {
          FEof = true;
```

通过调用`Seek()`并设置`FEof`标志来实现循环：

```java
          if ( FLoop )
          {
            Seek( 0 );
            FEof = false;
            continue;
          }
          break;
        }
```

`Ret`中的负值表示发生了读取错误。在这种情况下，我们停止解码：

```java
        else
        {
          Seek( 0 );
          FEof = true;
          break;
        }
      }
      return ( FBufferUsed = BytesRead );
    }
  };
```

接下来的两节将展示如何使用流行的第三方库解码不同格式的音频文件。

## 使用 ModPlug 库解码跟踪器音乐

我们将要处理的第一个用于解码音频文件的库是 Olivier Lapicque 的 ModPlug 库。大多数流行的跟踪器音乐文件格式[`en.wikipedia.org/wiki/Module_file`](http://en.wikipedia.org/wiki/Module_file)可以使用 ModPlug 解码并转换为适合 OpenAL 的波形。我们将介绍实现`ReadFromFile()`例程的`clModPlugProvider`类。该类的构造函数将内存块加载到`ModPlugFile`对象中，并分配默认的音频参数：

```java
  class clModPlugProvider: public iDecodingProvider
  {
  private:
    ModPlugFile* FModFile;
  public:
    ModPlugProvider( const clPtr<clBlob>& Blob ):
    {
      DecodingProvider( Blob )
      FChannels = 2;
      FSamplesPerSec = 44100;
      FBitsPerSample = 16;
      FModFile = ModPlug_Load_P(
        ( const void* ) FRawData->GetDataConst(), ( int )FRawData->GetSize()
      );
    }
```

析构函数清理 ModPlug：

```java
    virtual ~ModPlugProvider() { ModPlug_Unload_P( FModFile ); }
```

`ReadFromFile()`方法调用`ModPlug_Read()`来填充`FBuffer`：

```java
    virtual int ReadFromFile( int Size, int BytesRead )
    {
      return ModPlug_Read_P( FModFile,
        &FBuffer[0] + BytesRead, Size - BytesRead );
    }
```

流定位是通过使用`ModPlug_Seek()`例程完成的。在 ModPlug API 内部，所有的时间计算都是以毫秒为单位的：

```java
    virtual void Seek( float Time )
    {
      FEof = false;
      ModPlug_Seek_P( FModFile, ( int )( Time * 1000.0f ) );
    }
  };
```

要使用这个波形数据提供者，我们将其实例附加到`clAudioSource`对象：

```java
  Src->BindWaveform( make_intrusive<clModPlugProvider>( LoadFileAsBlob( g_FS, "augmented_emotions.xm" ) 
    )
   );
```

其他细节是从我们之前的示例中复用的。`4_ModPlug`文件夹可以在 Android 和 Windows 上构建和运行。使用`ndk-build`和`ant debug`为 Android 创建`.apk`，使用`make all`创建 Windows 可执行文件。

## 解码 MP3 文件

MPEG-1 Layer 3 格式的多数专利在 2015 年底到期，因此值得提及 Fabrice Bellard 的 MiniMP3 库。使用这个库不会比 ModPlug 更难，因为我们已经在`iDecodingProvider`中完成了所有繁重的工作。让我们看看`5_MiniMP3`示例。`clMP3Provider`类创建了解码器实例，并通过读取开头的几帧来读取流参数：

```java
  class clMP3Provider: public iDecodingProvider
  {
  public:
    clMP3Provider( const clPtr<clBlob>& Blob )
    : iDecodingProvider( Blob )
    {
      FBuffer.resize(MP3_MAX_SAMPLES_PER_FRAME * 8);
      FBufferUsed = 0;
      FBitsPerSample = 16;
      mp3 = mp3_create();
      bytes_left = ( int )FRawData->GetSize();
```

一开始，我们将流位置设置为`clBlob`对象的开始处：

```java
      stream_pos = 0;
      byte_count = mp3_decode((mp3_decoder_t*)mp3,
        ( void* )FRawData->GetData(), bytes_left,
        (signed short*)&FBuffer[0], &info);
      bytes_left -= byte_count;
```

我们需要关于音频数据的信息，因此我们从`info`结构中获取它：

```java
      FSamplesPerSec = info.sample_rate;
      FChannels = info.channels;
    }
```

析构函数中没有特别之处，以下是它的样子：

```java
    virtual ~MP3Provider()
    {
      mp3_done( &mp3 );
    }
```

`ReadFromFile()`方法跟踪源流中剩余的字节数，并填充`FBuffer`容器。构造函数和这个方法都使用`bytes_left`和`stream_pos`字段来保持当前的流位置和剩余的字节数：

```java
    virtual int ReadFromFile( int Size, int BytesRead )
    {
      byte_count = mp3_decode( (mp3_decoder_t*)mp3, (( char* )FRawData->GetData()) + stream_pos, bytes_left, (signed short *)(&FBuffer[0] + BytesRead), &info);
      bytes_left -= byte_count;
      stream_pos += byte_count;
      return info.audio_bytes;
    }
```

对于可变比特率的流，寻道并不是那么明显，因此我们将这个实现留给感兴趣的读者作为一个练习。在固定比特率的最简单情况下，只需从秒重新计算`Time`到采样率单位，然后设置`stream_pos`变量：

```java
    virtual void Seek( float Time ) override
    {
      FEof = false;
    }
  private:
    mp3_decoder_t mp3;
    mp3_info_t info;
    int stream_pos;
    int bytes_left;
    int byte_count;
  };
```

要使用它，我们将提供者附加到`clAudioSource`对象，就像使用 ModPlug 一样：

```java
  Src->BindWaveform( make_intrusive<clMP3Provider>( LoadFileAsBlob( g_FS, "test.mp3" ) ) );
```

同样，这个示例可以在 Android 上运行，去试试吧。

### 注意

这段代码没有正确处理一些 ID3 标签。如果你想基于我们的代码编写一个通用的音乐播放器，可以参考作者编写的这个开源项目：[`github.com/corporateshark/PortAMP`](https://github.com/corporateshark/PortAMP)。

## 解码 OGG 文件

还有一个值得提及的流行音频格式。Ogg Vorbis 是一种完全开放、无专利、专业的音频编码和流媒体技术，具有开源的所有好处[`www.vorbis.com`](http://www.vorbis.com)。OGG 解码和播放过程的大致流程与 MP3 类似。让我们看看示例`6_OGG`。`Decoders.cpp`文件用 OGG Vorbis 函数的定义进行了扩展，包括`OGG_clear_func()`、`OGG_open_callbacks_func()`、`OGG_time_seek_func()`、`OGG_read_func()`、`OGG_info_func()`和`OGG_comment_func()`。这些函数在 Android 上链接到一个静态库，或者在 Windows 上从`.dll`文件加载。与 MiniMP3 API 的主要区别在于向 OGG 解码器提供一组数据读取回调。这些回调在`OGG_Callbacks.inc`文件中实现。`OGG_ReadFunc()`回调将数据读取到解码器中：

```java
  static size_t OGG_ReadFunc( void* Ptr, size_t Size, size_t NMemB, void* DataSource )
  {
    clOggProvider* OGG = static_cast<clOggProvider*>( DataSource );
    size_t DataSize = OGG->FRawData->GetSize();
    ogg_int64_t BytesRead = DataSize - OGG->FOGGRawPosition;
    ogg_int64_t BytesSize = Size * NMemB;
    if ( BytesSize < BytesRead ) { BytesRead = BytesSize; }
```

它基于我们的文件系统抽象和内存映射文件：

```java
    memcpy(Ptr, ( unsigned char* )OGG->FRawData->GetDataConst() +
      OGG->FOGGRawPosition, ( size_t )BytesRead );
    OGG->FOGGRawPosition += BytesRead;
    return ( size_t )BytesRead;
  }
```

`OGG_SeekFunc()` 回调使用不同的相对定位模式来查找输入流：

```java
  static int OGG_SeekFunc( void* DataSource, ogg_int64_t Offset, int Whence )
  {
    clOggProvider* OGG = static_cast<clOggProvider*>( DataSource );
    size_t DataSize = OGG->FRawData->GetSize();
    if ( Whence == SEEK_SET )
    {
      OGG->FOGGRawPosition = Offset;
    }
    else if ( Whence == SEEK_CUR )
    {
      OGG->FOGGRawPosition += Offset;
    }
    else if ( Whence == SEEK_END )
    {
      OGG->FOGGRawPosition = DataSize + Offset;
    }
    if ( OGG->FOGGRawPosition > ( ogg_int64_t )DataSize )
    {
      OGG->FOGGRawPosition = ( ogg_int64_t )DataSize;
    }
    return static_cast<int>( OGG->FOGGRawPosition );
  }
```

`OGG_CloseFunc()` 和 `OGG_TellFunc()` 函数非常简单：

```java
  static int OGG_CloseFunc( void* DataSource )
  {
    return 0;
  }
   static long OGG_TellFunc( void* DataSource )
  {
   return static_cast<int>(
     (( clOggProvider* )DataSource )->FOGGRawPosition );
  }
```

这些回调在 `clOggProvider` 的构造函数中使用，以设置解码器：

```java
  clOggProvider( const clPtr<clBlob>& Blob )
  : iDecodingProvider( Blob )
  , FOGGRawPosition( 0 )
  {
    ov_callbacks Callbacks;
    Callbacks.read_func  = OGG_ReadFunc;
    Callbacks.seek_func  = OGG_SeekFunc;
    Callbacks.close_func = OGG_CloseFunc;
    Callbacks.tell_func  = OGG_TellFunc;
    OGG_ov_open_callbacks( this, &FVorbisFile, nullptr, -1, Callbacks );
```

流参数（如通道数、采样率和每样本位数）在这里获取：

```java
    vorbis_info* VorbisInfo = OGG_ov_info ( &FVorbisFile, -1 );
    FChannels = VorbisInfo->channels;
    FSamplesPerSec = VorbisInfo->rate;
    FBitsPerSample = 16;
  }
```

析构函数非常简单：

```java
  virtual ~clOggProvider()
  {
    OGG_ov_clear( &FVorbisFile );
  }
```

`ReadFromFile()` 和 `Seek()` 方法在精神上与我们处理 MiniMP3 时所做的非常相似：

```java
  virtual int ReadFromFile( int Size, int BytesRead ) override
  {
    return ( int )OGG_ov_read( &FVorbisFile, &FBuffer[0] + BytesRead, Size - BytesRead, 0, FBitsPerSample / 8, 1, &FOGGCurrentSection );
  }
  virtual void Seek( float Time ) override
  {
    FEof = false;
    OGG_ov_time_seek( &FVorbisFile, Time );
  }
private:
```

这是在前面章节提到的回调函数定义的地方。当然，它们可以在原地定义，而不必将它们移到单独的文件中。然而，我们认为这种分离对于本例来说在逻辑上更为清晰；将数据提供者概念和 `OGG Vorbis` 相关 API 逻辑上分开：

```java
  #include "OGG_Callbacks.inc"
  OggVorbis_File FVorbisFile;
  ogg_int64_t FOGGRawPosition;
  int FOGGCurrentSection;
};
```

这个示例也开箱即用，支持 Android。运行以下命令以在您的设备上获取 `.apk`：

```java
>ndk-build
>ant debug
>adb install -r bin/App1-debug.apk

```

现在启动活动，享受音乐吧！在后续章节中，我们将在本章内容的基础上添加更多有趣的音频内容。

# 总结

在本章中，我们学习了如何使用可移植的 C++ 代码和开源第三方库在 Android 上播放音频。提供的示例能够播放 `.mp3` 和 `.ogg` 音频文件以及 `.it`、`.xm`、`.mod` 和 `.s3m` 模块。我们还学习了如何生成自己的波形来模拟乐器。代码可以在许多系统间移植，并且可以在 Android 和 Windows 上运行和调试。现在，我们已经完成了音频部分，是时候进入下一章，使用 OpenGL 渲染一些图形了。
