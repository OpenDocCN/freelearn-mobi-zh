# 多媒体

在本章中，我们将涵盖以下主题：

+   使用 `SoundPool` 播放声音效果

+   使用 `MediaPlayer` 播放音频

+   在您的应用中响应用户硬件媒体控件

+   使用默认相机应用拍照

+   使用 Camera2 API 拍照

# 简介

在前几章中我们已经探讨了图形和动画，现在是时候看看 Android 中可用的声音选项了。播放声音最常用的两个选项如下：

+   `SoundPool`：这是用于短声音片段的

+   `MediaPlayer`：这是为较大的声音文件（如音乐）和视频文件设计的

前两个教程将探讨如何使用这些库。我们还将探讨如何使用与声音相关的硬件，例如音量控制和媒体播放控制（播放、暂停、下一曲和上一曲，通常在耳机上可用）。

本章的其余部分将专注于使用相机，无论是通过 Intents（将相机请求传递到默认相机应用）还是直接使用相机 API。我们将展示一个使用 Android 5.0 Lollipop（API 21）发布的 Camera2 API 的完整示例。

# 使用 SoundPool 播放声音效果

当您需要在您的应用程序中使用声音效果时，`SoundPool` 通常是一个好的起点。

`SoundPool` 很有趣，因为它允许我们通过改变播放速率和允许同时播放多个声音来创建声音的特殊效果。

支持的流行音频文件类型包括：

+   3GPP (`.3gp`)

+   3GPP (`.3gp`)

+   FLAC (`.flac`)

+   MP3 (`.mp3`)

+   MIDI 类型 0 和 1 (`.mid`, `.xmf`, 和 `.mxmf`)

+   Ogg (`.ogg`)

+   WAVE (`.wav`)

查看支持的媒体格式链接，以获取完整的列表，包括网络协议。

如同 Android 中的常见做法，新版本的操作系统发布会带来 API 的变化。`SoundPool` 也不例外，原始的 `SoundPool` 构造函数在 Lollipop（API 21）中被弃用。我们不会将我们的最小 API 设置为 21 或依赖弃用的代码（这可能在某个时刻停止工作），我们将实现旧的和新的方法，并在运行时检查操作系统版本以使用适当的方法。

本教程将演示如何使用 Android 的 `SoundPool` 库播放声音效果。为了演示同时播放声音，我们将创建两个按钮，每个按钮在被按下时都会播放一个声音。

# 准备工作

在 Android Studio 中创建一个新的项目，命名为 `SoundPool`。使用默认的 Phone & Tablet 选项，并在提示活动类型时选择 Empty Activity。

为了演示同时播放声音，项目中至少需要两个音频文件。我们访问了 SoundBible.com ([`soundbible.com/royalty-free-sounds-5.html`](http://soundbible.com/royalty-free-sounds-5.html)) 并找到了两个免费、公有领域的声音，以包含在下载的项目文件中。

第一个声音是一个较长的播放声音：[`soundbible.com/2032-Water.html`](http://soundbible.com/2032-Water.html)

第二个声音较短：[`soundbible.com/1615-Metal-Drop.html`](http://soundbible.com/1615-Metal-Drop.html)

# 如何做到这一点...

如前所述，我们需要在项目中包含两个音频文件。一旦你的音频文件准备就绪，请按照以下步骤操作：

1.  创建一个新的原始文件夹（文件 | 新建 | Android 资源目录）并在资源类型下拉菜单中选择原始。

1.  将你的音频文件复制到`res/raw`作为`sound_1`和`sound_2`。 （保留它们的原始扩展名。）

1.  打开`activity_main.xml`并用以下按钮替换现有的`TextView`：

```kt
<Button
    android:id="@+id/button1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Button"
    android:onClick="playSound1"/>
<Button
    android:id="@+id/button2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Button"
    android:onClick="playSound2"
    app:layout_constraintTop_toBottomOf="@+id/button1"/>
```

1.  现在，打开`ActivityMain.java`并添加以下全局变量：

```kt
HashMap<Integer, Integer> mHashMap= null;
SoundPool mSoundPool;
```

1.  修改现有的`onCreate()`方法如下：

```kt
final Button button1 = findViewById(R.id.button1);
button1.setEnabled(false);
final Button button2 = findViewById(R.id.button2);
button2.setEnabled(false);

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    createSoundPoolNew();
} else {
    createSoundPoolOld();
}
mSoundPool.setOnLoadCompleteListener(new SoundPool.OnLoadCompleteListener() {
    @Override
    public void onLoadComplete(SoundPool soundPool, int sampleId, int status) {
        button1.setEnabled(true);
        button2.setEnabled(true);
    }
});
mHashMap = new HashMap<>();
mHashMap.put(1, mSoundPool.load(this, R.raw.sound_1, 1));
mHashMap.put(2, mSoundPool.load(this, R.raw.sound_2, 1));
```

1.  添加`createSoundPoolNew()`方法：

```kt
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
private void createSoundPoolNew() {
    AudioAttributes audioAttributes = new AudioAttributes.Builder()
            .setUsage(AudioAttributes.USAGE_MEDIA)
            .setContentType(AudioAttributes.CONTENT_TYPE_SONIFICATION)
            .build();
    mSoundPool = new SoundPool.Builder()
            .setAudioAttributes(audioAttributes)
            .setMaxStreams(2)
            .build();
}
```

1.  添加`createSoundPoolOld()`方法：

```kt
@SuppressWarnings("deprecation")
private void createSoundPoolOld(){
    mSoundPool = new SoundPool(2, AudioManager.STREAM_MUSIC, 0);
}
```

1.  添加按钮的`onClick()`方法：

```kt
public void playSound1(View view){
    mSoundPool.play(mHashMap.get(1), 0.1f, 0.1f, 1, 0, 1.0f);
}
public void playSound2(View view){
    mSoundPool.play(mHashMap.get(2), 0.9f, 0.9f, 1, 1, 1.0f);
}
```

1.  如此覆盖`onStop()`回调：

```kt
@Override
protected void onStop() {
    mSoundPool.release();
    super.onStop();
}
```

1.  在设备或模拟器上运行应用程序。

# 它是如何工作的...

首先要注意的细节是如何构建对象本身。正如我们在介绍中提到的，`SoundPool`构造函数在 Lollipop（API 21）中发生了变化。旧的构造函数已被弃用，转而使用`SoundPool.Builder()`。在像 Android 这样的不断变化的环境中，API 的变化非常常见，因此学习如何处理这些变化是个好主意。正如你所看到的，在这种情况下并不困难。我们只需检查当前的操作系统版本并调用相应的方法。值得注意的是这两个方法注解。第一个指定了目标 API：

`@TargetApi(Build.VERSION_CODES.LOLLIPOP)`

并且第二个抑制了弃用警告：

`@SuppressWarnings("deprecation")`

创建`SoundPool`后，我们设置一个`setOnLoadCompleteListener()`监听器。启用按钮主要是为了演示目的，以说明`SoundPool`在可用之前需要加载音效资源。

使用`SoundPool`的最后一个要点是调用`play()`。我们需要传递`soundID`，这是我们在使用`load()`加载声音时返回的。`play()`给我们一些选项，包括音量（左右）、循环次数和播放速率。为了展示其灵活性，我们将第一个声音（较长的声音）以较低的音量播放，以创造更多的背景效果。第二个声音以较高的音量播放，并且我们播放它两次。

# 还有更多...

如果你只需要一个基本的音效，例如点击声，你可以使用`AudioManager`的`playSoundEffect()`方法。以下是一个示例：

```kt
AudioManager audioManager =(AudioManager)
this.getSystemService(Context.AUDIO_SERVICE);
audioManager.playSoundEffect(SoundEffectConstants.CLICK);
```

你只能指定来自`SoundEffectConstants`的声音；你不能使用你自己的音频文件。

# 参见

+   `SoundPool`开发者文档：[`developer.android.com/reference/android/media/SoundPool.html`](https://developer.android.com/reference/android/media/SoundPool.html)

+   `AudioManager`开发者文档：[`developer.android.com/reference/android/media/AudioManager.html`](https://developer.android.com/reference/android/media/AudioManager.html)

# 使用 MediaPlayer 播放音频

`MediaPlayer` 可能是向您的应用程序添加多媒体功能最重要的类之一。它支持以下媒体源：

+   项目资源

+   本地文件

+   外部资源（例如 URL，包括流媒体）

`MediaPlayer` 支持以下流行的音频文件：

+   3GPP (`.3gp`)

+   3GPP (`.3gp`)

+   FLAC (`.flac`)

+   MP3 (`.mp3`)

+   MIDI 类型 0 和 1 (`.mid`, `.xmf`, 和 `.mxmf`)

+   Ogg (`.ogg`)

+   WAVE (`.wav`)

它还支持以下流行的文件类型：

+   3GPP (`.3gp`)

+   Matroska (`.mkv`)

+   WebM (`.webm`)

+   MPEG-4 (`.mp4`, `.m4a`)

有关完整列表，包括网络协议，请参阅支持的媒体格式链接。

本食谱将演示如何在您的应用程序中设置 `MediaPlayer` 以播放项目中的声音。（有关 `MediaPlayer` 提供的完整功能的详细审查，请参阅本食谱末尾的开发者文档链接。）

# 准备工作

在 Android Studio 中创建一个新的项目，命名为 `MediaPlayer`。使用默认的 Phone & Tablet 选项，并在提示活动类型时选择 Empty Activity。

我们还需要为这个食谱准备一个声音，并将使用之前食谱中使用的相同较长的“水”声音：

[`soundbible.com/2032-Water.html`](http://soundbible.com/2032-Water.html)

# 如何操作...

如前所述，我们需要一个声音文件包含在项目中。一旦您准备好了声音文件，请按照以下步骤操作：

1.  创建一个新的 raw 文件夹（文件 | 新 | Android 资源目录），并在资源类型下拉菜单中选择 raw。

1.  将您的声音文件复制到 `res/raw` 目录下，命名为 `sound_1`。（保留原始扩展名。）

1.  打开 `activity_main.xml` 并将现有的 `TextView` 替换为以下按钮：

```kt
<Button
    android:id="@+id/buttonPlay"
    android:layout_width="100dp"
    android:layout_height="wrap_content"
    android:text="Play"
    android:onClick="buttonPlay" />
<Button
    android:text="Pause"
    android:layout_width="100dp"
    android:layout_height="wrap_content"
    android:id="@+id/buttonPause"
    android:onClick="buttonPause"
    app:layout_constraintTop_toBottomOf="@+id/buttonPlay"/>
<Button
    android:text="Stop"
    android:layout_width="100dp"
    android:layout_height="wrap_content"
    android:id="@+id/buttonStop"
    android:onClick="buttonStop"
    app:layout_constraintTop_toBottomOf="@+id/buttonPause"/>
```

1.  现在，打开 `ActivityMain.java` 并添加以下全局变量：

```kt
MediaPlayer mMediaPlayer;
```

1.  添加 `buttonPlay()` 方法：

```kt
public void buttonPlay(View view){
    if (mMediaPlayer==null) {
        mMediaPlayer = MediaPlayer.create(this, R.raw.sound_1);
        mMediaPlayer.setLooping(true);
        mMediaPlayer.start();
    } else  {
        mMediaPlayer.start();
    }
}
```

1.  添加 `buttonPause()` 方法：

```kt
public void buttonPause(View view){
    if (mMediaPlayer!=null && mMediaPlayer.isPlaying()) {
        mMediaPlayer.pause();
    }
}
```

1.  添加 `buttonStop()` 方法：

```kt
public void buttonStop(View view){
    if (mMediaPlayer!=null) {
        mMediaPlayer.stop();
        mMediaPlayer.release();
        mMediaPlayer = null;
    }
}
```

1.  最后，使用以下代码覆盖 `onStop()` 回调：

```kt
@Override
protected void onStop() {
    super.onStop();
    if (mMediaPlayer!=null) {
        mMediaPlayer.release();
        mMediaPlayer = null;
    }
}
```

1.  您现在可以运行应用程序在设备或模拟器上。

# 工作原理...

这里的代码相当简单。我们使用我们的声音创建 `MediaPlayer` 并开始播放声音。按钮将相应地重播、暂停和停止。

即使这个基本示例也说明了关于 `MediaPlayer` 的一个非常重要的概念，那就是状态。如果您正在认真使用 `MediaPlayer`，请查看稍后提供的链接以获取详细信息。

# 更多内容...

为了使我们的演示更容易理解，我们使用 UI 线程进行所有操作。对于此示例，使用项目中的短音频文件，我们不太可能遇到任何 UI 延迟。通常，在准备 `MediaPlayer` 时使用后台线程是一个好主意。为了使这个常见任务更容易，`MediaPlayer` 已经包含了一个名为 `prepareAsync()` 的异步准备方法。以下代码将创建一个 `OnPreparedListener()` 监听器并使用 `prepareAsync()` 方法：

```kt
mMediaPlayer = new MediaPlayer();
mMediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
    @Override
    public void onPrepared(MediaPlayer mp) {
        mMediaPlayer.start();
    }
});
try {
    mMediaPlayer.setDataSource(/*URI, URL or path here*/));
} catch (IOException e) {
    e.printStackTrace();
}
mMediaPlayer.prepareAsync();
```

# 在后台播放音乐

我们的示例旨在当应用处于前台时播放音频，并在 `onStop()` 回调中释放 `MediaPlayer` 资源。如果你正在创建音乐播放器，并希望在用户使用其他应用时也能在后台播放音乐，那会怎样？在这种情况下，你将需要在服务中使用 `MediaPlayer` 而不是活动。你将以相同的方式使用 `MediaPlayer` 库；你只需要从 UI 传递信息（如声音选择）到你的服务。

注意，由于服务在同一个 UI 线程上运行，因此你仍然不希望在服务中执行可能阻塞的操作。MediaPlayer 可以处理后台线程以防止阻塞你的 UI 线程；否则，你可能需要自己进行线程处理。（有关线程和选项的更多信息，请参阅第十五章，*为应用商店准备你的应用*。）

# 使用硬件音量键控制你应用的音频音量

如果你希望音量控制可以控制你应用中的音量，请使用 `setVolumeControlStream()` 方法指定你的应用程序的音频流，如下所示：

```kt
setVolumeControlStream(AudioManager.STREAM_MUSIC);
```

参见下方的 `AudioManager` 链接以获取其他流选项。

# 参见

+   支持的媒体格式：[`developer.android.com/guide/appendix/media-formats.html`](https://developer.android.com/guide/appendix/media-formats.html)

+   `MediaPlayer` 开发者文档：[`developer.android.com/reference/android/media/MediaPlayer.html`](http://developer.android.com/reference/android/media/MediaPlayer.html)

+   `AudioManager` 开发者文档：[`developer.android.com/reference/android/media/AudioManager.html`](https://developer.android.com/reference/android/media/AudioManager.html)

# 在你的应用中响应用户硬件媒体控制

让你的应用响应用户媒体控制（如耳机上的控制），例如播放、暂停、跳过等，这是一个用户会喜欢的贴心功能。Android 通过媒体库实现了这一点。与之前提到的 *使用 SoundPool 播放声音效果* 的配方一样，Lollipop 版本改变了这种操作方式。与 `SoundPool` 示例不同，这个配方能够利用另一种方法，即兼容性库。

本配方将向你展示如何设置 `MediaSession` 以响应用户硬件按钮，这将适用于 Lollipop 及更高版本，以及使用 `MediaSessionCompat` 库的早期 Lollipop 版本。（兼容性库将负责检查操作系统版本并自动使用正确的 API 调用。）

# 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为 `HardwareMediaControls`。使用默认的 Phone & Tablet 选项，并在“添加活动到移动”对话框中选择 Empty Activity。

# 如何操作...

我们将只使用 Toast 消息来响应硬件事件，因此不需要对活动布局进行任何更改。第一步是将 V13 支持库添加到项目中。首先打开 `build.gradle (Module: app)` 并执行以下步骤：

1.  在依赖关系部分添加以下库：

```kt
implementation 'com.android.support:support-v13:28.0.0-rc02'
```

1.  接下来，打开 `ActivityMain.java` 并将以下 `mMediaSessionCallback` 添加到类声明中：

```kt
MediaSessionCompat.Callback mMediaSessionCallback = new MediaSessionCompat.Callback() {
    @Override
    public void onPlay() {
        super.onPlay();
        Toast.makeText(MainActivity.this, "onPlay()", Toast.LENGTH_SHORT).show();
    }
    @Override
    public void onPause() {
        super.onPause();
        Toast.makeText(MainActivity.this, "onPause()", Toast.LENGTH_SHORT).show();
    }
    @Override
    public void onSkipToNext() {
        super.onSkipToNext();
        Toast.makeText(MainActivity.this, "onSkipToNext()", Toast.LENGTH_SHORT).show();
    }
    @Override
    public void onSkipToPrevious() {
        super.onSkipToPrevious();
        Toast.makeText(MainActivity.this, "onSkipToPrevious()", Toast.LENGTH_SHORT).show();
    }
};
```

1.  将以下代码添加到现有的 `onCreate()` 回调中：

```kt
MediaSessionCompat mediaSession = 
        new MediaSessionCompat(this, getApplication().getPackageName());
mediaSession.setCallback(mMediaSessionCallback);
mediaSession.setFlags(MediaSessionCompat.FLAG_HANDLES_MEDIA_BUTTONS);
mediaSession.setActive(true);
PlaybackStateCompat state = new PlaybackStateCompat.Builder()
        .setActions(PlaybackStateCompat.ACTION_PLAY |
                PlaybackStateCompat.ACTION_PLAY_PAUSE |
                PlaybackStateCompat.ACTION_PAUSE |
                PlaybackStateCompat.ACTION_SKIP_TO_NEXT |
                PlaybackStateCompat.ACTION_SKIP_TO_PREVIOUS).build();
mediaSession.setPlaybackState(state);
```

1.  在具有媒体控制（如耳机）的设备或模拟器上运行应用程序以查看 Toast 消息。

# 它是如何工作的...

设置此功能有四个步骤：

1.  创建一个 `MediaSession.Callback` 并将其附加到 `MediaSession`

1.  设置 `MediaSession` 标志以指示我们想要媒体按钮

1.  将 `SessionState` 设置为活动状态

1.  使用我们将要处理的操作设置 `PlayBackState`

步骤 4 和步骤 1 一起工作，因为回调函数只会获取在 `PlayBackState` 中设置的 `events`。

由于我们在这个菜谱中实际上并没有控制任何播放，所以我们只是演示如何响应硬件事件。你将需要在 `PlayBackState` 中实现实际的功能，并在调用 `setActions()` 之后调用 `setState()`。

这是一个很好的例子，说明了 API 的更改可以使事情变得更容易。由于新的 `MediaSession` 和 `PlaybackState` 被整合到 `Compatibility` 库中，我们可以在旧版本的操作系统上利用这些新 API。

# 还有更多...

市场上有各种各样的硬件，你的应用如何检查正在使用什么？

# 检查硬件类型

如果你想让你的应用根据当前输出硬件的不同而以不同的方式响应，你可以使用 `AudioManager` 来检查。以下是一个示例：

```kt
AudioManager audioManager =(AudioManager) this.getSystemService(Context.AUDIO_SERVICE);
if (audioManager.isBluetoothA2dpOn()) {
    // Adjust output for Bluetooth.
} else if (audioManager.isSpeakerphoneOn()) {
    // Adjust output for Speakerphone.
} else if (audioManager.isWiredHeadsetOn()) {
    //Only checks if a wired headset is plugged in
    //May not be the audio output
} else {
    // Regular speakers?
}
```

# 参见

+   `MediaSession` 开发者文档：[`developer.android.com/reference/android/media/session/MediaSession.html`](https://developer.android.com/reference/android/media/session/MediaSession.html)

+   `MediaSessionCompat` 开发者文档：[`developer.android.com/reference/android/support/v4/media/session/MediaSessionCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/MediaSessionCompat.html)

+   `PlaybackState` 开发者文档：[`developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html)

+   `PlaybackStateCompat` 开发者文档：[`developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html)

# 使用默认相机应用拍照

如果你的应用需要从相机获取图片，但不是相机替换应用，可能允许默认的相机应用拍照会更好。这也尊重了用户的首选相机应用。

当你拍照时，除非它特定于你的应用，否则将其公开可用是一种良好的实践。（这允许它被包含在用户的照片库中。）本食谱将演示使用默认的照片应用拍照，将其保存到公共文件夹，并显示图片。

# 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为 `UsingTheDefaultCameraApp`。使用默认的 Phone & Tablet 选项，并在“添加到移动设备”对话框中选择 Empty Activity。

# 如何操作...

我们将创建一个包含 ImageView 和按钮的布局。按钮将创建一个 Intent 以启动默认的相机应用。当相机应用完成后，我们的应用将收到回调。我们将检查结果并在有照片的情况下显示图片。首先打开 Android Manifest 并按照以下步骤操作：

1.  添加以下权限：

```kt
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

1.  打开 `activity_main.xml` 并用以下视图替换现有的 TextView：

```kt
<android.support.v7.widget.AppCompatImageView
 android:id="@+id/imageView"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:src="img/ic_launcher"
 app:layout_constraintTop_toTopOf="parent"
 app:layout_constraintLeft_toLeftOf="parent"
 app:layout_constraintRight_toRightOf="parent" />
<android.support.v7.widget.AppCompatButton
 android:id="@+id/button"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:text="Take Picture"
 android:onClick="takePicture"
 app:layout_constraintBottom_toBottomOf="parent"
 app:layout_constraintLeft_toLeftOf="parent"
 app:layout_constraintRight_toRightOf="parent"/>
```

1.  打开 `MainActivity.java` 并将以下全局变量添加到 `MainActivity` 类中：

```kt
final int PHOTO_RESULT=1;
private Uri mLastPhotoURI=null;
```

1.  添加以下方法以创建照片的 URI：

```kt
private Uri createFileURI() {
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss")
            .format(System.currentTimeMillis());
    String fileName = "PHOTO_" + timeStamp + ".jpg";
    return Uri.fromFile(new File(Environment
            .getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),fileName));
}
```

1.  添加以下方法以处理按钮点击：

```kt
public void takePicture(View view) {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        mLastPhotoURI = createFileURI();
        takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, mLastPhotoURI);
        startActivityForResult(takePictureIntent, PHOTO_RESULT);
    }
}
```

1.  添加一个新方法以覆盖 `onActivityResult()`，如下所示：

```kt
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == PHOTO_RESULT && resultCode == RESULT_OK ) {
        AppCompatImageView imageView = findViewById(R.id.imageView);
        imageView.setImageBitmap(BitmapFactory.decodeFile(mLastPhotoURI.getPath()));
    }
}
```

7. 将以下代码添加到现有的 `onCreate()` 方法末尾：

```kt
StrictMode.VmPolicy.Builder builder = new StrictMode.VmPolicy.Builder();
StrictMode.setVmPolicy(builder.build());

if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) 
        != PackageManager.PERMISSION_GRANTED ) {
    ActivityCompat.requestPermissions(this, 
            new String[] {Manifest.permission.READ_EXTERNAL_STORAGE},0);
}
```

8. 你可以准备在设备或模拟器上运行应用程序了。

# 它是如何工作的...

与默认相机应用一起工作有两个部分。第一部分是设置启动应用的 Intent。我们使用 `MediaStore.ACTION_IMAGE_CAPTURE` 创建 Intent 以指示我们想要一个照片应用。我们通过检查 `resolveActivity()` 的结果来验证是否存在默认应用。只要它不为 null，我们就知道有一个应用可以处理该 Intent。（否则，我们的应用会崩溃。）我们创建一个文件名并将其添加到 Intent 中，使用 `putExtra(MediaStore.EXTRA_OUTPUT, mLastPhotoURI)`。

当我们在 `onActivityResult()` 中收到回调时，我们首先确保它是 `PHOTO_RESULT` 和 `RESULT_OK`（用户可能已取消），然后我们在 ImageView 中加载照片。

你可能想知道 `onCreate()` 中的 `StrictMode` 调用有什么用。基本上，这些代码行禁用了操作系统进行的额外安全检查。如果我们不禁用 StrictMode，当创建文件 URI 时应用会因 `FileUriExposedException` 异常而崩溃。对于生产应用，一个解决方案是创建一个 FileProvider，就像我们在第七章 *数据存储* 的 *使用有作用域目录访问外部存储* 食谱中所做的那样。有关其他选项，请参阅 *另请参阅* 部分。

# 还有更多...

如果你不在乎图片存储在哪里，你可以不使用 `MediaStore.EXTRA_OUTPUT` 额外信息调用 Intent。如果你不指定输出文件，`onActivityResult()` 将在数据 Intent 中包含图像的缩略图。以下是如何显示缩略图的方法：

```kt
if (data != null) {
    imageView.setImageBitmap((Bitmap) data.getExtras().get(“data”));
}
```

这里是加载全分辨率图像的代码，使用在 data Intent 中返回的 URI：

```kt
if (data != null) {
    try {
        imageView.setImageBitmap(
            MediaStore.Images.Media. getBitmap(getContentResolver(),
            Uri.parse(data.toUri(Intent.URI_ALLOW_UNSAFE))));
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

# 调用默认视频应用程序

如果你想调用默认的视频捕获应用程序，过程是相同的。只需在步骤 5 中更改 Intent，如下所示：

```kt
Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
```

你可以在 `onActivityResult()` 中获取视频的 URI，如下所示：

```kt
Uri videoUri = intent.getData();
```

# 参见

+   在 第十章 的 *将大图像缩小以避免内存不足异常* 配方中，*图形和动画*

+   在 第七章 的 *使用作用域目录访问外部存储* 配方中，*数据存储*

# 使用 Camera2 API 拍照

之前的配方演示了如何使用 Intent 调用默认的拍照应用程序。如果你只需要快速拍照，Intent 可能是理想的解决方案。如果不是，并且你需要对相机有更多控制，这个配方将展示如何直接使用 Camera2 API 使用相机。

由于现在 85% 的设备都在使用 Android 5.0 或更高版本，这个配方只关注 Camera2 API。 （Google 已经弃用了原始的 Camera API。）

# 准备中

在 Android Studio 中创建一个新的项目，并将其命名为 Camera2API。在目标 Android 设备对话框中，选择手机和平板选项，并选择 API 21：Android 5.0 (Lollipop) 或更高版本作为最小 SDK。在“添加到移动”对话框中选择“Empty Activity”。

# 如何做到这一点...

如您将看到的，这个配方有很多代码。首先打开 AndroidManifest.xml 并按照以下步骤操作：

1.  添加以下两个权限：

```kt
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

1.  现在，打开 `activity_main.xml` 并将现有的 TextView 替换为以下视图：

```kt
<TextureView
    android:id="@+id/textureView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toTopOf="@+id/button"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent" />
<android.support.v7.widget.AppCompatButton
    android:id="@+id/button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Take Picture"
    android:onClick="takePictureClick"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"/>
```

1.  现在，打开 `MainActivity.java` 并将以下全局变量添加到 `MainActivity` 类中：

```kt
private CameraDevice mCameraDevice = null;
private CaptureRequest.Builder mCaptureRequestBuilder = null;
private CameraCaptureSession mCameraCaptureSession  = null;
private TextureView mTextureView = null;
private Size mPreviewSize = null;
```

1.  将以下 `Comparator` 类添加到 `MainActivity` 类中：

```kt
static class CompareSizesByArea implements Comparator<Size> {
    @Override
    public int compare(Size lhs, Size rhs) {
        return Long.signum((long) lhs.getWidth() * lhs.getHeight() 
                - (long) rhs.getWidth() * rhs.getHeight());
    }
}
```

1.  添加以下 `CameraCaptureSession.StateCallback`：

```kt
private CameraCaptureSession.StateCallback mPreviewStateCallback = new CameraCaptureSession.StateCallback() {
    @Override
    public void onConfigured(CameraCaptureSession session) {
        startPreview(session);
    }
    @Override
    public void onConfigureFailed(CameraCaptureSession session) {}
};
```

1.  添加以下 `SurfaceTextureListener`：

```kt
private TextureView.SurfaceTextureListener mSurfaceTextureListener =
        new TextureView.SurfaceTextureListener() {
            @Override
            public void onSurfaceTextureUpdated(SurfaceTexture     
            surface)                        
            {
            }
            @Override
            public void onSurfaceTextureSizeChanged(
                    SurfaceTexture surface, int width, int height) {
            }
            @Override
            public boolean onSurfaceTextureDestroyed(SurfaceTexture 
            surface) {
                return false;
            }
            @Override
            public void onSurfaceTextureAvailable(
                    SurfaceTexture surface, int width, int height) {
                openCamera();
            }
        };
```

1.  添加 `CameraDevice.StateCallback` 如下：

```kt
private CameraDevice.StateCallback mStateCallback = new CameraDevice.StateCallback() {
    @Override
    public void onOpened(CameraDevice camera) {
        mCameraDevice = camera;
        SurfaceTexture texture = mTextureView.getSurfaceTexture();
        if (texture == null) {
            return;
        }
        texture.setDefaultBufferSize(mPreviewSize.getWidth(), 
        mPreviewSize.getHeight());
        Surface surface = new Surface(texture);
        try {
            mCaptureRequestBuilder = mCameraDevice

        .createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
        } catch (CameraAccessException e){
            e.printStackTrace();
        }
        mCaptureRequestBuilder.addTarget(surface);
        try {
            mCameraDevice.createCaptureSession(Arrays
                    .asList(surface), mPreviewStateCallback, null);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
    @Override
    public void onError(CameraDevice camera, int error) {}
    @Override
    public void onDisconnected(CameraDevice camera) {}
};
```

1.  添加以下 `CaptureCallback` 类以接收捕获完成事件：

```kt
final CameraCaptureSession.CaptureCallback mCaptureCallback = 
        new CameraCaptureSession.CaptureCallback() {
    @Override
    public void onCaptureCompleted(CameraCaptureSession session,  
     CaptureRequest request,                                  
     TotalCaptureResult result) {
        super.onCaptureCompleted(session, request, result);
        Toast.makeText(MainActivity.this, "Picture Saved", 
        Toast.LENGTH_SHORT).show();
        startPreview(session);
    }
};
```

1.  将以下代码添加到现有的 `onCreate()` 回调中：

```kt
mTextureView = findViewById(R.id.textureView);
mTextureView.setSurfaceTextureListener(mSurfaceTextureListener);

if(ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) 
        != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CAMERA}, 1);
}
```

1.  将以下方法添加以重写 `onPause()` 和 `onResume()`：

```kt
@Override
protected void onPause() {
    super.onPause();
    if (mCameraDevice != null) {
        mCameraDevice.close();
        mCameraDevice = null;
    }
}
@Override
public void onResume() {
    super.onResume();
    if (mTextureView.isAvailable()) {
        openCamera();
    } else {
        mTextureView.setSurfaceTextureListener(
             mSurfaceTextureListener);
    }
}
```

1.  添加 `openCamera()` 方法：

```kt
private void openCamera() {
    CameraManager manager = (CameraManager) getSystemService(CAMERA_SERVICE);
    try{
        String cameraId = manager.getCameraIdList()[0];
        CameraCharacteristics characteristics = manager.getCameraCharacteristics(cameraId);
        StreamConfigurationMap map = characteristics
                .get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
        mPreviewSize = map.getOutputSizes(SurfaceTexture.class) [0];
        manager.openCamera(cameraId, mStateCallback, null);
    } catch(CameraAccessException e) {
        e.printStackTrace();
    } catch (SecurityException e) {
        e.printStackTrace();
    }
}
```

1.  添加 `startPreview()` 方法：

```kt
private void startPreview(CameraCaptureSession session) {
    mCameraCaptureSession = session;
    mCaptureRequestBuilder.set(CaptureRequest.CONTROL_MODE, 
    CameraMetadata.CONTROL_MODE_AUTO);
    HandlerThread backgroundThread = new 
    HandlerThread("CameraPreview");
    backgroundThread.start();
    Handler backgroundHandler = new Handler(backgroundThread. 
    getLooper());
    try {
        mCameraCaptureSession
                .setRepeatingRequest(mCaptureRequestBuilder.build(), 
    null, backgroundHandler);
    } catch (CameraAccessException e) {
        e.printStackTrace();
    }
}
```

1.  添加 `getPictureFile()` 方法：

```kt
private File getPictureFile() {
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss")
            .format(System.currentTimeMillis());
    String fileName = "PHOTO_" + timeStamp + ".jpg";
    return new File(Environment
            .getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),fileName);
}
```

1.  添加以下方法以保存图像文件：

```kt
private void saveImage(ImageReader reader) {
    Image image = null;
    try {
        image = reader.acquireLatestImage();
        ByteBuffer buffer = image.getPlanes()[0].getBuffer();
        byte[] bytes = new byte[buffer.capacity()];
        buffer.get(bytes);
        OutputStream output = new FileOutputStream(getPictureFile());
        output.write(bytes);
        output.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (image != null) {
            image.close();
        }
    }
}
```

1.  添加以下方法以处理按钮点击：

```kt
public void takePictureClick(View view) {
    if (null == mCameraDevice) {
        return;
    }
    takePicture();
}
```

1.  添加最终代码以实际设置相机并拍照：

```kt
private void takePicture() {
    CameraManager manager = (CameraManager) getSystemService(Context.CAMERA_SERVICE);
    try {
        CameraCharacteristics characteristics = manager
                .getCameraCharacteristics(mCameraDevice.getId());
        StreamConfigurationMap configurationMap = characteristics

       .get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
        if (configurationMap == null) return;
        Size largest = Collections.max(Arrays.asList(configurationMap
                .getOutputSizes(ImageFormat.JPEG)), new 
        CompareSizesByArea());
        ImageReader reader = ImageReader
                .newInstance(largest.getWidth(), largest.getHeight(), 
        ImageFormat.JPEG, 1);
        List<Surface> outputSurfaces = new ArrayList<>(2);
        outputSurfaces.add(reader.getSurface());
        outputSurfaces.add(new 
        Surface(mTextureView.getSurfaceTexture()));
        final CaptureRequest.Builder captureBuilder = mCameraDevice

        .createCaptureRequest(CameraDevice.TEMPLATE_STILL_CAPTURE);
        captureBuilder.addTarget(reader.getSurface());
        captureBuilder.set(CaptureRequest.CONTROL_MODE, 
        CameraMetadata.CONTROL_MODE_AUTO);
        ImageReader.OnImageAvailableListener readerListener =
                new ImageReader.OnImageAvailableListener() {
            @Override
            public void onImageAvailable(ImageReader reader) {
                saveImage(reader);
            }
        };
        HandlerThread thread = new HandlerThread("CameraPicture");
        thread.start();
        final Handler backgroundHandler = new 
        Handler(thread.getLooper());
        reader.setOnImageAvailableListener(readerListener, 
        backgroundHandler);
        mCameraDevice.createCaptureSession(outputSurfaces,
                new CameraCaptureSession.StateCallback() {
                    @Override
                    public void onConfigured(CameraCaptureSession 
                    session) {
                        try {
                            session.capture(captureBuilder.build(),
                                    mCaptureCallback, 
                       backgroundHandler);
                        } catch (CameraAccessException e) {
                            e.printStackTrace();
                        }
                    }
                    @Override
                    public void onConfigureFailed(CameraCaptureSession 
                   session) { }
                }, backgroundHandler);
    } catch (CameraAccessException e) {
        e.printStackTrace();
    }
```

1.  在带有相机的设备或模拟器上运行应用程序。

# 它是如何工作的...

如您所见，这个配方有很多步骤，但从高层次来看，它相当简单：

+   设置相机预览

+   捕获图像

现在，我们将逐一详细说明。

# 设置相机预览

下面是代码如何设置预览的概述：

1.  首先，我们使用 `setSurfaceTextureListener()` 方法在 `onCreate()` 中设置 `TextureView.SurfaceTextureListener`

1.  当我们收到 `onSurfaceTextureAvailable()` 回调时，我们打开相机

1.  我们将我们的`CameraDevice.StateCallback`类传递给`openCamera()`方法，它最终会调用`onOpened()`回调

1.  `onOpened()`通过调用`getSurfaceTexture()`获取预览的表面，并通过调用`createCaptureSession()`将其传递给`CameraDevice`

1.  最后，当`CameraCaptureSession.StateCallback`的`onConfigured()`被调用时，我们使用`setRepeatingRequest()`方法开始预览

# 捕获图片

尽管`takePicture()`方法可能看起来是过程性的，捕获图片也涉及到几个类并依赖于回调。以下是代码工作原理的分解：

1.  当点击拍照按钮时，这个过程开始。

1.  然后代码查询相机以找到最大的可用图像大小

1.  然后创建一个 ImageReader。

1.  接下来，代码设置`OnImageAvailableListener`，并在`onImageAvailable()`回调中保存图片。

1.  然后它创建`CaptureRequest.Builder`并包含`ImageReader`表面。

1.  接下来，它创建`CameraCaptureSession.CaptureCallback`，这定义了

    `onCaptureCompleted()`回调。当捕获完成后，它重新开始预览。

1.  最后，调用`createCaptureSession()`方法，创建一个`CameraCaptureSession.StateCallback`。这是调用之前创建的`capture()`方法的地方，传递`CameraCaptureSession.CaptureCallback`。

# 还有更多...

我们刚刚创建了基础代码来演示一个可工作的相机应用程序。有许多改进的地方。首先，你应该处理设备的方向，无论是预览还是保存图片时。（见以下链接。）此外，由于 Android 6.0（API 23）拥有超过 60%的市场份额，你的应用程序应该已经使用新的权限模型。而不是像我们在`openCamera()`方法中那样只是检查异常，最好是检查所需的权限。

# 参见

+   Camera2 API 开发者文档 [`developer.android.com/reference/android/hardware/camera2/package-summary.html`](https://developer.android.com/reference/android/hardware/camera2/package-summary.html)

+   对于检测当前设备方向的示例，请参阅第九章，*使用触摸屏和传感器*

+   在第十五章的*为 Play 商店准备你的应用程序*中，*Android 6.0 运行时权限模型*配方
