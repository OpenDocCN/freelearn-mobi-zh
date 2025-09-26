# 第七章：添加声音和视频

你的游戏可能看起来很惊艳，但如果它是静音的，观众可能会觉得体验很无聊。幸运的是，Marmalade 通过支持声音和视频播放来解决这个问题。在本章中，我们将学习以下内容：

+   播放记录在 MP3 等格式的音频文件

+   使用声音样本添加多个同时音效

+   播放全动作视频剪辑

# Marmalade 的多媒体支持

现代智能手机和平板设备现在能够播放高质量的音乐和视频，因此 Marmalade 提供让我们利用这些能力的方法是有意义的。

Marmalade 提供了三个不同的 API 层，适用于多媒体支持。这些是 s3eSound、s3eAudio 和 s3eVideo。不出所料，后者与视频文件的播放有关，但您可能想知道为什么提供了两个与声音相关的 API。

s3eSound 和 s3eAudio 之间的区别在于，前者通常用于音效，而后者通常用于音乐。s3eSound API 允许我们同时播放多个不同的音效，但默认情况下只提供对 16 位单声道 PCM 声音样本的支持。另一方面，s3eAudio API 允许我们播放压缩格式，如 MP3，但（在大多数设备上）我们只能播放单个音频轨道。

好消息是，大多数现代设备允许我们同时使用 s3eSound 和 s3eAudio，从而让我们享受到两者的最佳效果。

在接下来的章节中，我们将探讨如何使用这三个 API，并也会看看另一个名为`SoundEngine`的模块，它使得使用 s3eSound API 变得更容易。

# s3eAudio API

让我们从最快捷、最简单的方法开始，让我们的游戏不再只是沉默的强者。

s3eAudio API 允许我们播放压缩的音乐格式，如 MP3 和 AAC。某些设备也可能允许我们播放其他格式，如 MIDI 文件。Marmalade 使用特定设备可能内置的任何音频编解码器，而不是解码音频本身，因此请确保你选择的音频格式支持你希望针对的所有设备。

### 注意

由于其普遍性，建议您使用 MP3 作为首选格式。很少有设备（如果有的话）不能播放 MP3 文件，而且该格式本身提供了广泛的比特率，这样您可以在音频质量和文件大小之间进行权衡。

现在，让我们看看如何播放音频轨道，以及 s3eAudio API 为我们提供了哪些其他功能。为了使用 s3eAudio，我们不需要在 MKB 文件中添加任何内容，因为它是 Marmalade 的低级 API 之一，始终可用。我们只需要在需要访问 s3eAudio 函数的任何源文件中包含头文件`s3eAudio.h`。

## 开始音频播放

有两种方法可以开始播放音频轨道。第一种允许我们指定要播放的音频轨道的文件名和轨道重复的次数，看起来像这样：

```swift
s3eAudioPlay("music.mp3", aRepeatCount);
```

文件名只是一个标准的 C 语言，以 null 结尾的字符串，当从 Windows 运行时相对于` data`目录，或者相对于设备上的应用程序安装目录。指定重复次数的数字将使音频轨道播放指定次数，而将其设置为零将使轨道连续循环。

另一种方法是按照以下方式从内存区域播放音频轨道：

```swift
s3eAudioPlayFromBuffer(apBuffer, aBufferLength, aRepeatCount);
```

参数` apBuffer`和` aBufferLength`提供了音频轨道所在的内存位置和音频数据字节数的长度。重复次数的指定方式与` s3eAudioPlay`相同。

在大多数情况下，我们会发现第一种方法就足够好了，因为它易于使用，不需要我们分配内存块并填充数据。如果你已经预加载了音频数据，可能会发现缓冲区方法提供了稍微快一点的初始播放，但在大多数最新设备上，这种差异是可以忽略不计的。

如果你在一个音频轨道正在播放时调用这些函数中的任何一个，该轨道将被停止，并开始播放新轨道。

## 暂停、恢复和停止播放

一旦音频轨道开始播放，我们可以通过调用` s3eAudioPause`函数来暂停播放。可以通过调用` s3eAudioResume`从暂停点重新开始播放音频。最后，要完全停止播放，只需调用` s3eAudioStop`。

这三个函数都不需要参数，在没有错误发生的情况下将返回` S3E_RESULT_SUCCESS`。如果在没有意义的情况下调用这些函数，例如在没有音频播放时调用` s3eAudioPause`，则会引发错误。

## 改变音量

与 Marmalade 中的大多数低级 API 一样，s3eAudio 具有一对函数` s3eAudioGetInt`和` s3eAudioSetInt`，用于更改与该 API 相关的属性。在 s3eAudio 中，我们使用这些函数之一来更改音频播放的音量。

要设置播放音量，我们可以进行以下调用：

```swift
s3eAudioSetInt(S3E_AUDIO_VOLUME, S3E_AUDIO_MAX_VOLUME / 2);
```

在上述示例中，我们将音量设置为` S3E_AUDIO_MAX_VOLUME`的一半，这是允许的最大音量。

要确定当前音量，我们使用以下代码：

```swift
int32 lVolume = s3eAudioGetInt(S3E_AUDIO_VOLUME);
```

我们还可以通过传入值` S3E_AUDIO_VOLUME_DEFAULT`来请求音频的默认音量。这是播放音频的默认音量级别，由 Marmalade SDK 选择，以便在所有设备上提供相当一致的音量级别。

## 其他音频查询

` s3eAudioGetInt`函数允许我们进行关于音频播放的几个其他查询。以下表格显示了可以指定的属性：

| 属性 | 描述 |
| --- | --- |
| `S3E_AUDIO_STATUS` | 返回当前音频状态——`S3E_AUDIO_STOPPED`、`S3E_AUDIO_PLAYING`、`S3E_AUDIO_PAUSED`或`S3E_AUDIO_FAILED`之一。 |
| `S3E_AUDIO_POSITION` | 返回音频轨道中的当前位置，单位为毫秒，如果没有轨道正在播放，则返回`0`。请注意，并非所有平台都支持此功能。 |
| `S3E_AUDIO_CHANNEL` | 返回当前选定的音频通道。此属性也可以在`s3eAudioSetInt`中使用，以选择将应用于未来音频命令的音频通道。有关音频通道的更多信息，请参阅以下属性。 |
| `S3E_AUDIO_NUM_CHANNELS` | 返回可用的音频通道数。在大多数平台上，这将返回`1`，因为大多数设备在任何时候只允许播放一个音频轨道。一些设备提供多个通道，这意味着可以同时播放多个音频轨道。 |
| `S3E_AUDIO_MUTES_S3ESOUND` | 如果硬件无法同时通过 s3eAudio 和 s3eSound 输出声音，则返回`1`。在这种情况下，播放音频轨道将继续 s3eSound 处理，但不会实际产生任何输出。 |
| `S3E_AUDIO_DURATION` | 返回当前播放的轨道长度，单位为毫秒。 |
| `S3E_AUDIO_PLAYBACK_FROM_HTTP_AVAILABLE` | 如果硬件能够通过从 URL 流式传输播放音频轨道，则返回`1`。 |

## 轨道结束通知

我们可以使用两种方法来确定音频轨道何时完成。一种方法是使用轮询方法，另一种方法是使用回调。

要轮询音频轨道是否已完成，我们可以执行以下操作：

```swift
if (s3eAudioIsPlaying() == S3E_FALSE)
{
  // Audio is not playing!
}
```

此函数返回`S3E_TRUE`，如果音频正在播放，或者返回`S3E_FALSE`，如果它已停止或暂停。实际上，这个函数只是调用`s3eAudioGetInt`并带有属性`S3E_AUDIO_STATUS`的一个快捷方式。

回调方法也非常简单易用，如下面的代码片段所示：

```swift
int32 AudioFinished(s3eAudioCallbackData* apAudioData,
void* apUserData)
{
  // apAudioData->m_ChannelID identifies the audio channel that
  // has completed.
  // s3eCallback functions must return a value, but in case of
  // audio callback the value returned does not matter.
  return 0;
}

// Use the following line to set up the audio callback
s3eAudioRegister(S3E_AUDIO_STOP, (s3eCallback) AudioFinished, NULL);

// And this line to remove the callback function
s3eAudioUnRegister(S3E_AUDIO_STOP, (s3eCallback) AudioFinished);
```

当音频轨道完成时，回调函数将被调用，并将通过`apUserData`参数传递的用户数据指针传递。如果我们要求音频轨道循环播放，除非是最后一次重复，否则不会调用该函数。如果由于错误（如损坏的轨道）而停止音频，该函数也会被调用。我们可以通过调用返回枚举类型`s3eAudioError`的错误代码的`s3eAudioGetError`函数来确定完成是否由错误引起。错误代码的完整列表可以在`s3eAudio.h`中找到。

是否使用轮询或基于回调的方法取决于你的应用程序，实际上在游戏中我们通常并不特别关心音频轨道何时结束，因为我们通常只是希望同一轨道无限循环，直到需要新的音频片段。如果你只是在启动画面期间等待铃声结束，轮询方法可能就足够了，但如果你想要将几个轨道依次连接起来，回调方法可能带来一个干净的解决方案。

# s3eSound API

如果你想要在你的游戏中添加点音效，比如激光束和爆炸声，你需要使用 s3eSound API。这个 API 允许同时以不同的音量和音调播放多个声音样本，通过将它们混合成一个单一的输出。

要使用 s3eSound API，只需在你的源代码中包含文件`s3eSound.h`。

API 期望所有音效都作为未压缩的 16 位有符号 PCM 提供。API 不支持如 WAV 这样的文件格式，因此你必须编写自己的代码来加载和从这些文件中提取样本数据。

随着你阅读这一部分，你可能会开始认为要播放一些音效有很多事情要做。虽然这看起来可能是这样，但 s3eSound 实际上是一个非常底层的 API，提供了足够的灵活性，允许你编写自己的复杂音效程序。

在本章的后面部分，我们将介绍 Marmalade 提供的`SoundEngine`模块，它为 s3eSound API 提供了一个包装器。`SoundEngine`模块为我们处理了使用 s3eSound API 的大部分繁重工作，并且还包括直接从 GROUP 文件中加载 WAV 文件的能力。

## 开始播放声音

为了使用 s3eSound 播放声音样本，我们首先必须分配一个空闲的声音通道。s3eSound API 提供了一定数量的通道（我们稍后会看到如何确定确切的数量），允许我们指定声音样本、音量和播放速率。然后，所有当前活动通道的声音数据在 s3eSound 的内部工作中被混合成一个单一的波形，这就是通过设备的音频硬件播放的内容。为了分配一个空闲通道，我们进行以下函数调用：

```swift
int32 lChannel = s3eSoundGetFreeChannel();
```

这将返回一个空闲通道的 ID 号，如果没有可用通道，则返回`-1`。大多数情况下，不太可能没有空闲通道，但如果我们正在播放很多音效，我们可能想要考虑给每个音效标记一个优先级值，并维护一个当前活动音效的列表。当我们用完通道时，我们可以检查音效列表，并回收被优先级最低的音效使用的通道，当然前提是它比我们希望启动的音效优先级低！

假设有一个可用的通道，我们必须设置样本数据的播放速率，这可以通过以下方式完成：

```swift
s3eSoundChannelSetInt(lChannel, S3E_CHANNEL_RATE, lFrequency);
```

第一个参数是我们刚刚分配的声音通道 ID。第二个参数表示我们想要设置该通道的播放速率，第三个参数是实际期望的播放速率，单位为赫兹（Hz）。可以设置的最大频率由定义`S3E_SOUND_MAX_FREQ`指定。

我们还应该设置我们想要播放的声音的音量，这同样是通过使用`s3eSoundChannelSetInt`函数来完成的：

```swift
s3eSoundChannelSetInt(lChannel, S3E_CHANNEL_VOLUME, lVolume);
```

`lVolume`参数的有效值从`0`到定义`S3E_SOUND_MAX_VOLUME`。

### 注意

一旦声音开始播放，就可以随时更改音量和播放速率。这使得可以实现音量淡入淡出或音调转换等效果。

现在我们可以开始播放我们的声音样本。我们通过以下调用来完成：

```swift
s3eSoundChannelPlay(lChannel, lSampleData, lNumSamples, lRepeatCount,
lLoopIndex);
```

毫不奇怪，我们首先传入我们正在使用的通道 ID，然后是`lSampleData`参数中内存地址，可以在其中找到 16 位 PCM 样本数据。`lNumSamples`参数是我们波形中的实际声音样本数量（不是字节数），而`lRepeatCount`表示我们希望声音重复的次数。值为`0`将使声音永远播放。最后，`lLoopIndex`参数允许我们指定如果声音重复，则从哪个样本开始。这使得可以使用只需要重复部分样本数据的声音。

## 暂停、恢复和停止播放

一旦声音通道开始播放声音样本，我们可能想要暂时暂停其播放或完全停止它。要暂停声音通道，我们使用`s3eSoundChannelPause`函数，并且我们可以从暂停位置重新开始播放，使用`s3eSoundChannelResume`。要完全停止声音通道，我们调用`s3eSoundChannelStop`。每个这些函数都接受一个单一参数，即我们想要影响的声音通道 ID。

要确定特定声音通道的当前播放状态，我们可以使用`s3eSoundChannelGetInt`函数，如下所示：

```swift
if (s3eSoundChannelGetInt(lChannel, S3E_CHANNEL_STATUS) == 1)
{
  // Sound channel is currently playing
}

if (s3eSoundChannelGetInt(lChannel, S3E_CHANNEL_PAUSED) == 1)
{
  // Sound channel is currently active, but paused
}
```

注意，此函数也可以与`S3E_CHANNEL_RATE`和`S3E_CHANNEL_VOLUME`属性一起使用，以发现特定通道的当前采样率和音量。

最后，还可以使用`s3eSoundPauseAllChannels`、`s3eSoundResumeAllChannels`和`s3eSoundStopAllChannels`函数一次性影响所有当前活动的声音通道。这些函数不接受任何输入，对于处理进入和退出暂停模式，或从游戏的一个部分切换到另一个部分（例如，退出标题屏幕并进入主游戏）等情况非常有用。

## 全局声音设置

除了能够按通道读写设置外，我们还可以设置影响全局声音支持的设置。为此，我们使用`s3eSoundSetInt`和`s3eSoundGetInt`函数，如下所示：

```swift
// To read a global sound setting
int32 lValue = s3eSoundGetInt(lProperty);

// To change a global sound setting
s3eSoundSetInt(lProperty, lValue);
```

这里是一些 `lProperty` 参数的更有用的值：

| 属性 | 描述 |
| --- | --- |
| `S3E_SOUND_VOLUME` | 可以用来读取或写入当前主音量。这将适当地缩放每个独立通道的音量。最大值由定义 `S3E_SOUND_MAX_VOLUME` 决定。 |
| `S3E_SOUND_DEFAULT_FREQ` | 这是启动声音通道播放时将使用的默认频率。如果我们的所有声音波形具有相同的采样率，则可以写入此属性一次，而无需在播放每个单独的声音时显式设置采样率。 |
| `S3E_SOUND_NUM_CHANNELS` | 一个只读值，指示可以同时播放的最大声音数量。 |
| `S3E_SOUND_USED_CHANNELS` | 一个只读值，显示当前正在使用的声音通道。这作为一个位掩码返回，最低有效位与声音通道 0 相关。此值可以用来确定一个可用的声音通道，但为了未来的兼容性，建议使用 `s3eSoundGetFreeChannel` 来执行此操作。 |
| `S3E_SOUND_AVAILABLE` | 一个只读值，如果设备上可用 s3eSound，则返回 `1`。 |
| `S3E_SOUND_VOLUME_DEFAULT` | 一个只读值，用作全局音量的默认值。它可能因设备而异，旨在允许所有设备上的声音输出具有相似的音量。 |

Marmalade 文档中描述了其他值，但在这里我们不会介绍它们，因为它们用于诸如自定义声音流生成等目的，这些目的超出了本书的范围。

## 声音通知

我们已经看到了如何使用轮询方法来检测声音通道是否正在播放，但有时知道声音样本何时播放完毕也是有用的，例如，这样我们就可以立即开始播放新的声音效果。

s3eSound API 允许我们在每个通道上设置几个不同的回调函数，我们使用 `s3eSoundChannelRegister` 和 `s3eSoundChannelUnRegister` 函数来启用和禁用它们，如下所示：

```swift
// To set up a sound channel callback
s3eSoundChannelRegister(lChannel, lCallbackType, (s3eCallback)
              CallbackFunction, lpUserData);

// To disable a sound channel callback
s3eSoundChannelUnRegister(lChannel, lCallbackType);
```

与所有其他 Marmalade 回调一样，我们通过传递函数本身的指针来指定回调函数的代码，我们还可以注册一个用户数据块，当它被触发时，此数据块将被传递到该函数中。有四种不同的回调类型，称为 `S3E_CHANNEL_END_SAMPLE`、`S3E_CHANNEL_STOP_AUDIO`、`S3E_CHANNEL_GEN_AUDIO` 和 `S3E_CHANNEL_GEN_AUDIO_STEREO`。在这里，我们只会查看前两种，因为后两种与生成自定义音频流有关，超出了本书的范围。有关如何使用这些回调类型的示例，请查看 `SoundEngine` 模块的源代码，我们将在下一节中介绍。

首先，让我们看看`S3E_CHANNEL_END_SAMPLE`回调，它允许我们循环声音并将不同的声音作为一个序列连接起来。注册的回调函数将其第一个参数传递给一个指向`s3eSoundEndSampleInfo`结构的指针。该结构通过其`m_Channel`成员指示哪个声音通道已结束。

如果我们想在通道上开始播放全新的声音，我们可以将`s3eSoundEndSampleInfo`结构的`m_NewData`成员设置为新的样本数据的起始地址，将`m_NumSamples`成员设置为新波形中的样本数量。

该结构还包含一个名为`m_RepsRemaining`的成员，它允许我们更改我们希望在声音通道上重复的样本数据的次数。请注意，尽管如此，每次达到样本数据的末尾时，此回调仍然会被触发。

如果我们希望通道继续播放样本数据，无论是原始数据还是使用`s3eSoundEndSampleInfo`结构的`m_NewData`和`m_NumSamples`成员指定的新样本，我们必须从回调函数返回一个非零值。如果返回零，则声音通道将停止播放。

以下代码示例将之前描述的功能付诸实践：

```swift
// Simple structure used to indicate the next sound sample to play
typedef struct
{
  void* mSampleData;
  uint32 mSampleCount;
} NewSoundData;

// Sample callback function that will start a new sound effect
// playing if one has been specified when registering the
// callback function
int32 SoundEndCallback(s3eSoundEndSampleInfo* apInfo,
   NewSoundData* apSound)
{
  if (apSound)
  {
     apInfo->m_NewData = apSound->mSampleData;
     apInfo->m_NumSamples = apSound->mSampleCount;
     apInfo->m_RepsRemaining = 1;
  }
  return apInfo->m_RepsRemaining;
}

// Register the callback function to play a new sound when
// current sound completes
s3eSoundChannelRegister(lChannel, S3E_CHANNEL_END_SAMPLE,
   (s3eCallback) SoundEndCallback,
   &lNewSoundDataInstance);
```

我们将要考虑的第二种回调类型是`S3E_CHANNEL_STOP_AUDIO`。当声音通道完全播放完声音时（例如，如果我们设置了`S3E_CHANNEL_END_SAMPLE`回调并从中返回零以结束所有播放）将发生此回调。它传递一个指向`s3eSoundEndSampleInfo`结构的指针，但唯一有效的字段是`m_Channel`成员。

# `SoundEngine`模块

如本章前面的部分所示，使用 s3eSound 的基本方法实际上相当简单。作为开发者，我们必须处理的主要问题是 s3eSound 只能支持原始未压缩的 16 位 PCM 样本，这意味着将声音数据放入内存以便播放是我们的责任。

存储声音样本最常见的文件格式之一是 WAV 文件，所以如果我们能够使用这种格式来存储我们的声音效果，那岂不是很好？如果我们能够使用与纹理和 3D 模型相同的资源管理器代码将这些文件加载到内存中，岂不是也很好？

我们所祈求的答案是`SoundEngine`模块，这是一个位于 s3eSound 之上的层，它允许我们通过资源管理器轻松地加载和访问声音效果。

`SoundEngine`模块不仅仅如此。它还封装了我们在本章中学到的 s3eSound 调用，并允许我们支持一种可以存储在 WAV 文件中的进一步的声音格式——即压缩的 IMA ADPCM 类型。鉴于声音样本数据可能相当大，这种格式特别有用；因此，这种格式可以帮助我们在略微降低音质的情况下回收一些内存空间。

以下部分简要介绍了如何使用此模块，但为了获取完整细节，你应该参考源代码和头文件，以查看 `SoundEngine` 提供的所有功能。本章附带的音频示例项目也使用了此模块，因此请查看该部分以了解更多信息。

## 将 SoundEngine 模块添加到项目中

`SoundEngine` 模块实际上随 Marmalade SDK 一起提供，但它尴尬地位于 `examples` 目录中。解决这个问题的最简单方法是将整个 `SoundEngine` 目录复制到你的项目所在的目录，然后通过将 `SoundEngine` 添加到你的 MKB 文件子项目中来引用它。这与我们在上一章的示例代码中引入的 GUI 和 `Localise` 模块所采用的方法相同。

### 注意

`SoundEngine` 模块位于 `examples` 文件夹中，这意味着它实际上并不被认为是 Marmalade SDK 的主要部分。实际上，`SoundEngine` 代码突然从 SDK 中消失的可能性非常低，因为 s3eSound API 很可能不会发生剧烈变化；所以你不需要担心直接在自己的项目中使用它。如果你更喜欢编写自己的代码，`SoundEngine` 至少提供了一个很好的示例，说明了如何使用 s3eSound API。

在将模块添加到我们的项目中后，我们可以在代码中包含 `IwSound.h` 文件以使用它。需要调用 `IwSoundInit` 来设置一切，并在程序结束时调用 `IwSoundTerminate` 来清理。

我们还必须添加一个自定义资源处理程序，以便资源管理器能够加载 WAV 文件。以下代码片段将完成这项工作：

```swift
IwGetResManager()->AddHandler(new CIwResHandlerWAV);
```

最后，有一个管理类负责处理所有与声音相关的事件，我们必须确保在主游戏循环的某个地方调用这个类的 `Update` 方法。我们通过以下代码行来实现：

```swift
IwGetSoundManager()->Update();
```

## 加载和访问声音资源

要加载一个 WAV 文件，我们只需在 GROUP 文件中添加对其文件名的引用即可，尽管我们还需要做更多的工作才能播放声音。我们需要做的是声明 `CIwSoundSpec` 类的一个实例。

这个类允许我们通过名称引用特定的声音样本，并让我们设置音量和音调来播放声音。我们还可以指定是否希望声音循环（请注意，`SoundEngine` 目前没有提供指定循环次数的方法；我们只能表示连续循环）。以下是一个示例定义：

```swift
CIwSoundSpec
{
  name gun1
  data gun_shot1

  // Play at the default pitch for the sample
  pitch 1.0

  // Play at half volume
  vol 0.5

  // Do we want this sound to loop?
  looping false
}
```

`pitch` 和 `vol`（音量）参数被指定为分数比例，其中 `1.0` 表示声音的默认音调或音量级别。我们还可以为这两个参数指定一个范围，以便在开始声音时选择一个随机值。为音调指定一个范围可以非常有用，可以在不添加大量略有不同声音样本的情况下，为你的游戏中的音效添加一些变化。

下面的示例显示了如何指定音量和音调的范围：

```swift
CIwSoundSpec
{
  name gun2
  data gun_shot2

  // Choose a random pitch when playing this sound
  pitchMin 0.9
  pitchMax 1.1

  // Choose a random volume when playing this sound
  volMin 0.9
  volMax 1.1

  // Do we want this sound to loop?
  looping false
}
```

另一个我们可以访问的有用类是 `CIwSoundGroup`。这个类允许我们将多个不同的音效组合在一起，并且可以同时暂停、恢复、停止，或者改变正在播放的任何音效的音量或音调。请注意，音效组只允许指定一个音量或音调值，而不是一个随机范围：

```swift
CIwSoundGroup
{
   name guns

   // Reduce volume of all gun sounds by a half
   vol 0.5

   // Include the gun1 sound in this group
   addSpec gun1
}
```

可以使用 `addSpec` 关键字将声音添加到组中，或者你也可以在定义时使用 `group` 关键字后跟组名，将 `CIwSoundSpec` 添加到组中。我们可以使用这两种方法中的任何一种，但组或声音必须在引用它之前被声明。

要访问声音规范或组，我们只需加载 GROUP 文件，并使用资源管理器以正常方式检索它们。以下是一个示例：

```swift
IwGetResManager()->LoadGroup("sounds.group");
CIwSoundSpec* lpGunSpec = static_cast<CIwSoundSpec*>(
  IwGetResManager()->GetResNamed("gun1", "CIwSoundSpec"));
CIwSoundGroup* lpGunsGroup = static_cast<CIwSoundGroup*>(
  IwGetResManager()->GetResNamed("guns", "CIwSoundGroup"));
```

## 播放、停止和改变声音参数

一旦我们获得了指向 `CIwSoundSpec` 的指针，我们就可以通过调用 `Play` 方法开始播放它，这将执行所有幕后操作，如分配一个空闲通道和设置音量和播放速度。`Play` 方法可以传递一个可选参数，它是一个 `CIwSoundParams` 类的实例，允许在开始声音时修改音量和音调。

`Play` 方法返回一个指向 `CIwSoundInst` 类的指针，该类提供了修改单个声音实例的音量或音调的方法，并且还提供了名为 `Pause`、`Resume` 和 `Stop` 的方法，这些方法应该很容易理解！如果没有可用的空闲音效通道，`Play` 方法将返回 `NULL`。

如果我们有一个指向 `CIwSoundGroup` 的指针，我们可以影响其中所有当前正在播放的声音实例。同样，有 `Pause`、`Resume` 和 `Stop` 方法，它们会执行你预期的操作，此外还有 `SetVol` 和 `SetPitch` 方法，这些方法将调整声音的当前音量和音调。这些方法使用值 `IW_GEOM_ONE`（4096）来表示一个比例为一的缩放。

# s3eVideo API

我们将通过快速浏览 Marmalade 的多媒体支持来结束对 Marmalade 的多媒体支持的探讨，我们将简要了解使用 s3eVideo API 播放视频剪辑的支持。为了使用它提供的功能，我们只需要将 `s3eVideo.h` 文件包含到我们的源代码中。

在我们开始之前，使用游戏中的视频片段时需要考虑两件事。首先，虽然可以指定视频片段在屏幕上的显示位置，但它总是会覆盖在所有其他图形之上。第二个问题是，由于许多移动设备硬件的限制，s3eVideo API 不能与 s3eAudio 和 s3eSound API 同时使用。在 s3eAudio 的情况下，任何当前正在播放的曲目将被停止（这也适用于相反的情况——开始音频曲目将停止当前正在播放的视频片段）。当视频片段播放时，s3eSound API 将继续处理其事件，但其声音输出将被静音，直到视频片段播放完毕。对于大多数游戏，我们可能会决定在开始视频片段之前明确停止所有 s3eSound 播放，尤其是如果我们正在执行任何高级操作，比如使用回调系统将声音样本合并在一起。

## 开始视频播放

s3eVideo API 的工作方式与 s3eAudio API 类似。要开始播放视频片段，我们使用 `s3eVideoPlay` 函数，指定视频片段的文件名、我们希望它循环的次数、屏幕位置以及我们想要显示的大小，如下所示：

```swift
s3eVideoPlay(lFileName, lRepeatCount, lX, lY, lWidth, lHeight);
```

视频片段将自动调整大小以适应矩形，但不会尝试保持正确的宽高比。

在可能的情况下，通常最好尝试使视频片段的分辨率与您想要显示它们的矩形区域相同。这将避免不必要的图像拉伸（这可能会看起来相当难看！）并可能导致性能略有提升，尽管在大多数现代设备上，缩放将在硬件中完成，并且不会有明显的差异。

视频文件本身的实际大小也值得考虑，因为我们通常希望最小化最终安装包的大小。最终，我们需要进行一些尝试和错误，直到我们得到一个在可接受的质量、性能和文件大小方面都符合要求的结果。

## 确定视频编解码器支持

s3eVideo API 利用设备的内置视频解码功能，因此并非所有视频格式都能在所有设备上播放。为了确定是否支持特定的编解码器，有一个名为 `s3eVideoIsCodecSupported` 的函数，它接受来自 `s3eVideoCodec` 枚举的值。查看 `s3eVideo.h` 文件或 Marmalade 文档以获取可能的值的完整列表。

## 暂停、恢复和停止视频播放

当涉及到控制视频播放时，与 s3eAudio API 的相似之处显而易见。函数 `s3eVideoPause`、`s3eVideoResume` 和 `s3eVideoStop` 都不接受任何参数，并分别用于暂停、恢复和结束视频片段播放。

## 视频结束通知

我们再次有选择轮询或回调来检测视频播放结束。让我们从涉及调用函数 `s3eVideoIsPlaying` 的轮询方法开始，该函数如果视频正在播放，则返回 `S3E_TRUE`，如果视频已暂停或停止，则返回 `S3E_FALSE`。实际上非常简单！

如果我们想使用回调方法，以下代码片段说明了应该做什么：

```swift
int32 VideoFinished(void* apSystemData, void* apUserData)
{
  // apSystemData will always be NULL as there is no data associated
  // with this callback.
  // Return value is unimportant.
  return 0;
}

// To set up the callback function
s3eVideoRegister(S3E_VIDEO_STOP, (s3eCallback) VideoFinished, NULL);

// And to cancel it again...
s3eVideoUnRegister(S3E_VIDEO_STOP, (s3eCallback) VideoFinished);
```

当视频播放停止时，回调将被触发，无论是我们明确调用 `s3eVideoStop`，播放过程中发生错误（如损坏的视频文件）发生，还是使用 `s3eAudioPlay` 开始音频轨道。请注意，如果我们正在循环视频剪辑，则回调在视频剪辑重复之间不会被触发。

对于大多数游戏，视频剪辑可能仅在开场序列或教程中使用，因为在使用游戏本身中的视频可能并不实用。考虑到这一点，检测视频剪辑何时完成的轮询方法通常足够。

## 其他视频查询

s3eVideo API，就像 s3eSound 和 s3eAudio API 一样，也有一对用于读取和写入全局视频参数的函数。它们被称为 `s3eVideoGetInt` 和 `s3eVideoSetInt`。它们如下所示：

```swift
int32 lValue = s3eVideoGetInt(lProperty);
s3eVideoSetInt(lProperty, lValue);
```

以下表格显示了可用于 `lProperty` 参数的值：

| 属性 | 描述 |
| --- | --- |
| `S3E_VIDEO_VOLUME` | 此属性用于查找与视频剪辑相关的声音的当前音量级别，也可以设置新的音量。最大音量级别由 `S3E_VIDEO_MAX_VOLUME` 的值定义。 |
| `S3E_VIDEO_DEFAULT_VOLUME` | 这是一个只读属性，显示用于在视频剪辑中播放声音的默认音量。其值旨在为所有设备类型提供类似的音量水平。 |
| `S3E_VIDEO_STATUS` | 这是一个只读参数，显示视频播放的当前状态。它将返回以下值之一：`S3E_VIDEO_STOPPED`、`S3E_VIDEO_PLAYING`、`S3E_VIDEO_PAUSED` 或 `S3E_VIDEO_FAILED`。 |
| `S3E_VIDEO_POSITION` | 此属性返回视频当前播放位置（以毫秒为单位），如果没有视频正在播放，则返回 `0`。此参数不能写入，因此无法跳转到视频剪辑的特定点。 |

# 示例代码

本章与三个示例项目相关联，它们将在以下部分中描述。这些项目中使用的声音、音频和视频剪辑是从几个提供大量免费库存媒体的优秀网站上获取的！以下提供了这些网站的链接：

[`www.royalty-free-music-room.com`](http://www.royalty-free-music-room.com)

[`www.partnersinrhyme.com`](http://www.partnersinrhyme.com)

## 声音项目

此项目演示了使用 s3eAudio API 和 `SoundEngine` 模块（它反过来又使用 s3eSound）。

运行示例时，你会看到三个可点击的按钮，这些按钮是使用上一章中引入的 GUI 模块实现的。第一个按钮使用 s3eAudio 切换 MP3 轨道的开启和关闭，而其他两个按钮则使用`SoundEngine`播放一些音效。

## 视频项目

这是一个简单的例子，展示了如何使用 s3eVideo API 来启动和停止视频片段。屏幕底部的按钮可以启动和停止一个视频片段，该片段会连续循环播放。

## 滑雪项目

最后，我们再次回到滑雪项目，它通过添加一些音乐和音效得到了增强，这并不会让人感到惊讶。

现在主菜单在等待玩家按按钮时会播放 MP3 音频轨道。按下按钮时，会播放一个确认音效。

在游戏本身中，增加了几个声音。当滑雪者移动时，会通过循环样本产生呼啸声，随着玩家转向，这个样本的音调会降低，使声音听起来更加动态。

新增的其他声音包括当玩家通过一个门时播放的庆祝声音，当玩家与障碍物碰撞时发出的痛苦呼喊声，以及当玩家与旗帜杆碰撞并使其晃动时播放的弹簧声。

# 摘要

随着本章的结束，我们对 Marmalade 的多媒体支持的研究现在使我们能够播放音效、音乐轨道，以及播放视频片段。

几乎没有游戏不包含某种形式的声音或音乐，添加一些音效可以使你的游戏产生巨大的变化。虽然并非所有游戏都需要使用视频，但知道我们随时可以使用它，这感觉很好。

在下一章中，我们将探讨 Marmalade 如何帮助我们尽可能多地针对各种设备，从入门级手机到高端手机。
