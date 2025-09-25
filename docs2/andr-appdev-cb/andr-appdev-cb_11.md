# 第十一章. 多媒体

在本章中，我们将涵盖以下主题：

+   使用 SoundPool 播放声音效果

+   使用 MediaPlayer 播放音频

+   在您的应用程序中响应用户媒体控制

+   使用默认相机应用程序拍照

+   使用（旧）Camera API 拍照

+   使用 Camera2（新）API 拍照

# 简介

现在我们已经在前几章中探讨了图形和动画，现在是时候看看 Android 中可用的声音选项了。播放声音最流行的两种选项包括：

+   **SoundPool**：这是用于短声音剪辑的

+   **MediaPlayer**：这是为较大的声音文件（如音乐）和视频文件设计的

前两个食谱将探讨如何使用这些库。我们还将探讨如何使用与声音相关的硬件，例如音量控制媒体播放控制（通常在耳机上找到的播放、暂停等）。

本章的其余部分将专注于使用相机，无论是通过 Intents 间接（将相机请求传递到默认相机应用程序）还是直接使用相机 API。我们将检查与 Android 5.0 Lollipop（API 21）一起发布的新的 Camera2 API，但也会查看原始的 Camera API，因为大约 75%的市场还没有 Lollipop。（为了帮助您利用 Camera2 API 提供的新功能，我们将展示一种使用旧 Camera API 的新方法，以便更容易地在您的应用程序中使用这两个 Camera API。）

# 使用 SoundPool 播放声音效果

当您在应用程序中需要声音效果时，SoundPool 通常是一个好的起点。

SoundPool 很有趣，因为它允许我们通过改变播放速率以及允许同时播放多个声音来创建特殊效果。

支持的流行音频文件类型包括：

+   3GPP (`.3gp`)

+   3GPP (`.3gp`)

+   FLAC (`.flac`)

+   MP3 (`.mp3`)

+   MIDI 类型 0 和 1（`.mid`，`.xmf`和`.mxmf`）

+   Ogg (`.ogg`)

+   WAVE (`.wav`)

请参阅**支持的媒体格式**链接以获取完整列表，包括网络协议。

在 Android 中很常见，新版本的操作系统会带来 API 的变化。`SoundPool`也不例外，原始的`SoundPool`构造函数在 Lollipop（API 21）中被弃用。我们不会将我们的最小 API 设置为 21 或依赖于弃用的代码（这可能在某个时刻停止工作），我们将实现旧的和新的方法，并在运行时检查操作系统版本以使用适当的方法。

本食谱将演示如何使用 Android `SoundPool`库播放声音效果。为了演示同时播放声音，我们将创建两个按钮，每个按钮在被按下时都会播放一个声音。

## 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为：`SoundPool`。使用默认的**手机和平板电脑**选项，并在提示**活动类型**时选择**空活动**。

为了演示同时播放声音，我们需要在项目中至少包含两个音频文件。我们去了 SoundBible.com([`soundbible.com/royalty-free-sounds-5.html`](http://soundbible.com/royalty-free-sounds-5.html) ) 并找到了两个免费公共领域声音，包括在下载项目文件中：

第一个声音是较长时间播放的声音：

[`soundbible.com/2032-Water.html`](http://soundbible.com/2032-Water.html)

第二个声音较短：

[`soundbible.com/1615-Metal-Drop.html`](http://soundbible.com/1615-Metal-Drop.html)

## 如何做到...

如前所述，我们需要在项目中包含两个音频文件。一旦你准备好了音效文件，请按照以下步骤操作：

1.  创建一个新的原始文件夹（**文件** | **新建** | **Android 资源目录**）并在 **资源类型** 下拉菜单中选择 `raw`。

1.  将你的音效文件复制到 `res/raw` 目录下，命名为 `sound_1` 和 `sound_2`。 (保留它们的原始扩展名。)

1.  打开 `activity_main.xml` 并将现有的 `TextView` 替换为以下按钮：

    ```java
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Play Sound 1"
        android:id="@+id/button1"
        android:layout_centerInParent="true"
        android:onClick="playSound1"/>
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Play Sound 2"
        android:id="@+id/button2"
        android:layout_below="@+id/button1"
        android:layout_centerHorizontal="true"
        android:onClick="playSound2"/>
    ```

1.  现在打开 `ActivityMain.java` 并添加以下全局变量：

    ```java
    HashMap<Integer, Integer> mHashMap= null;
    SoundPool mSoundPool;
    ```

1.  修改现有的 `onCreate()` 方法，如下所示：

    ```java
    final Button button1=(Button)findViewById(R.id.button1);
    button1.setEnabled(false);
    final Button button2=(Button)findViewById(R.id.button2);
    button2.setEnabled(false);

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        createSoundPoolNew();
    }else{
        createSoundPooolOld();
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

1.  添加 `createSoundPoolNew()` 方法：

    ```java
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

1.  添加 `createSoundPooolOld()` 方法：

    ```java
    @SuppressWarnings("deprecation")
    private void createSoundPooolOld(){
        mSoundPool = new SoundPool(2, AudioManager.STREAM_MUSIC, 0);
    }
    ```

1.  添加按钮的 `onClick()` 方法：

    ```java
    public void playSound1(View view){
        mSoundPool.play(mHashMap.get(1), 0.1f, 0.1f, 1, 0, 1.0f);
    }

    public void playSound2(View view){
        mSoundPool.play(mHashMap.get(2), 0.9f, 0.9f, 1, 1, 1.0f);
    }
    ```

1.  重写 `onStop()` 回调，如下所示：

    ```java
    protected void onStop() {
        super.onStop();
        mSoundPool.release();
    }
    ```

1.  在设备或模拟器上运行应用程序。

## 它是如何工作的...

首先要注意的细节是如何构建这个对象本身。正如我们在介绍中提到的，SoundPool 构造函数在 Lollipop (API 21) 中发生了变化。旧的构造函数已被弃用，转而使用 `SoundPool.Builder()`。在像 Android 这样不断变化的环境中，API 的变化非常常见，因此学习如何应对这些变化是个好主意。正如你所见，在这种情况下并不困难。我们只需检查当前的操作系统版本并调用相应的方法。值得注意的是方法注释：

```java
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
```

以及：

```java
@SuppressWarnings("deprecation")
```

在创建 SoundPool 之后，我们设置了一个 `setOnLoadCompleteListener()` 监听器。启用按钮主要是为了演示目的，以说明 SoundPool 在可用之前需要加载音效资源。

在使用 SoundPool 时，最后一点是调用 `play()`。我们需要传递 `soundID`，这是我们在使用 `load()` 加载声音时返回的。`Play()` 给我们一些选项，包括声音音量（左右）、循环次数和播放速率。为了展示灵活性，我们将第一个声音（较长的声音）以较低的音量播放，以创造更多的背景效果。第二个声音以较高的音量播放，并且我们播放了两次。

## 还有更多...

如果你只需要一个基本的音效，例如点击声，你可以使用 AudioManager 的 `playSoundEffect()` 方法。以下是一个示例：

```java
AudioManager audioManager =(AudioManager) 
this.getSystemService(Context.AUDIO_SERVICE);
audioManager.playSoundEffect(SoundEffectConstants.CLICK);
```

你只能指定来自 `SoundEffectConstants` 的声音；你不能使用自己的音效文件。

## 参见

+   **开发者文档：SoundPool**

    [`developer.android.com/reference/android/media/SoundPool.html`](https://developer.android.com/reference/android/media/SoundPool.html)

+   **开发者文档：**

    [`developer.android.com/reference/android/media/AudioManager.html`](https://developer.android.com/reference/android/media/AudioManager.html)

# 使用 MediaPlayer 播放音频

MediaPlayer 可能是为你的应用程序添加多媒体功能最重要的类之一。它支持以下媒体源：

+   项目资源

+   本地文件

+   外部资源（例如 URL，包括流媒体）

MediaPlayer 支持以下流行的音频文件：

+   3GPP (`.3gp`)

+   3GPP (`.3gp`)

+   FLAC (`.flac`)

+   MP3 (`.mp3`)

+   MIDI 类型 0 和 1 (`.mid`, `.xmf`, 和 `.mxmf`)

+   Ogg (`.ogg`)

+   WAVE (`.wav`)

以及以下流行的文件类型：

+   3GPP (`.3gp`)

+   Matroska (`.mkv`)

+   WebM (`.webm`)

+   MPEG-4 (`.mp4`, `.m4a`)

请参阅*支持的媒体格式*链接，获取完整列表，包括网络协议。

本教程将演示如何在你的应用程序中设置 MediaPlayer 以播放项目中的声音。（有关 MediaPlayer 提供的完整功能的详细说明，请参阅本教程末尾的开发者文档链接。）

## 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为：`MediaPlayer`。使用默认的**手机和平板电脑**选项，并在提示**活动类型**时选择**空活动**。

我们还需要为这个教程准备一个声音文件，并将使用之前教程中使用的相同较长的“水”声音。

第一个声音是一个较长的播放声音：[`soundbible.com/2032-Water.html`](http://soundbible.com/2032-Water.html)

## 如何实现...

如前所述，我们需要一个声音文件包含在项目中。一旦你准备好了你的声音文件，请按照以下步骤操作：

1.  创建一个新的 raw 文件夹（**文件** | **新建** | **Android 资源目录**），并在**资源类型**下拉菜单中选择`raw`。

1.  将你的声音文件复制到`res/raw`目录下，命名为`sound_1`。（保留原始扩展名。）

1.  打开`activity_main.xml`文件，并用以下按钮替换现有的`TextView`：

    ```java
    <Button
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Play"
        android:id="@+id/buttonPlay"
        android:layout_above="@+id/buttonPause"
        android:layout_centerHorizontal="true"
        android:onClick="buttonPlay" />
    <Button
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Pause"
        android:id="@+id/buttonPause"
        android:layout_centerInParent="true"
        android:onClick="buttonPause"/>
    <Button
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Stop"
        android:id="@+id/buttonStop"
        android:layout_below="@+id/buttonPause"
        android:layout_centerHorizontal="true"
        android:onClick="buttonStop"/>
    ```

1.  现在打开`ActivityMain.java`文件，并添加以下全局变量：

    ```java
    MediaPlayer mMediaPlayer;
    ```

1.  添加`buttonPlay()`方法：

    ```java
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

1.  添加`buttonPause()`方法：

    ```java
    public void buttonPause(View view){
        if (mMediaPlayer!=null && mMediaPlayer.isPlaying()) {
            mMediaPlayer.pause();
        }
    }
    ```

1.  添加`buttonStop()`方法：

    ```java
    public void buttonStop(View view){
        if (mMediaPlayer!=null) {
            mMediaPlayer.stop();
            mMediaPlayer.release();
            mMediaPlayer = null;
        }
    }
    ```

1.  最后，使用以下代码覆盖`onStop()`回调：

    ```java
    protected void onStop() {
        super.onStop();
        if (mMediaPlayer!=null) {
            mMediaPlayer.release();
            mMediaPlayer = null;
        }
    }
    ```

1.  你现在可以在设备或模拟器上运行应用程序了。

## 工作原理...

这里的代码相当简单。我们使用我们的声音创建 MediaPlayer 并开始播放声音。按钮将依次重播、暂停和停止。

即使这个基本示例也说明了关于 MediaPlayer 的一个非常重要的概念，那就是*状态*。如果你正在认真使用 MediaPlayer，请查看以下提供的链接以获取详细信息。

## 更多内容...

为了使我们的演示更容易理解，我们使用 UI 线程进行所有操作。对于这个例子，使用项目附带的一个短音频文件，我们不太可能遇到任何 UI 延迟。通常，在准备 MediaPlayer 时使用后台线程是一个好主意。为了使这个常见任务更容易，MediaPlayer 已经包含了一个异步准备方法，称为`prepareAsync()`。以下代码将创建一个`OnPreparedListener()`监听器并使用`prepareAsync()`方法：

```java
mMediaPlayer = new MediaPlayer();
mMediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
    @Override
    public void onPrepared(MediaPlayer mp) {
        mMediaPlayer.start();
    }
});
try {
    mMediaPlayer.setDataSource(*//*URI, URL or path here*//*));
} catch (IOException e) {
    e.printStackTrace();
}
mMediaPlayer.prepareAsync();
```

### 在后台播放音乐

我们的例子旨在在应用处于前台时播放音频，并在`onStop()`回调中释放 MediaPlayer 资源。如果你正在创建音乐播放器，并想在用户使用其他应用时在后台播放音乐，那会怎样？在这种情况下，你将需要在服务中使用 MediaPlayer 而不是 Activity。你将以相同的方式使用 MediaPlayer 库；你只需要从 UI 传递信息（如声音选择）到你的服务。

### 注意

注意，由于服务在同一个 UI 线程中运行，因此你仍然不希望在服务中执行可能阻塞的操作。MediaPlayer 可以处理后台线程以防止阻塞 UI 线程，否则，你可能需要自己进行线程处理。（有关线程和选项的更多信息，请参阅第十四章，*为应用商店准备你的应用*。）

### 使用硬件音量键控制你应用的音频音量

如果你希望音量控制控制你应用中的音量，请使用`setVolumeControlStream()`方法指定你应用的音频流，如下所示：

```java
setVolumeControlStream(AudioManager.STREAM_MUSIC);
```

参见以下`AudioManager`链接以获取其他流选项。

## 参见

+   支持的媒体格式[`developer.android.com/guide/appendix/media-formats.html`](https://developer.android.com/guide/appendix/media-formats.html)

+   **开发者文档：MediaPlayer** [`developer.android.com/reference/android/media/MediaPlayer.html`](http://developer.android.com/reference/android/media/MediaPlayer.html)

+   **开发者文档：AudioManager** [`developer.android.com/reference/android/media/AudioManager.html`](https://developer.android.com/reference/android/media/AudioManager.html)

# 在你的应用中响应用户媒体控制

你的应用响应用户媒体控制，如播放、暂停、跳过等，是用户会欣赏的贴心功能。

Android 通过媒体库实现了这一点。与之前提到的*使用 SoundPool 播放声音效果*配方一样，Lollipop 版本改变了这样做的方式。与`SoundPool`示例不同，这个配方能够利用另一种方法——兼容性库。

这个菜谱将向你展示如何设置`MediaSession`以响应硬件按钮，这将适用于 Lollipop 及以后的版本，以及使用`MediaSessionCompat`库的以前的`Lollilop`版本。（兼容性库将负责检查操作系统版本并自动使用正确的 API 调用。）

## 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为：`HardwareMediaControls`。使用默认的**手机和平板电脑**选项，并在提示**活动类型**时选择**空活动**。

## 如何操作...

我们将仅使用 Toast 消息来响应硬件事件，因此不需要对活动布局进行任何更改。要开始，请打开`ActivityMain.java`并按照以下步骤操作：

1.  创建以下`mMediaSessionCallback`以响应媒体按钮：

    ```java
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

1.  将以下代码添加到现有的`onCreate()`回调中：

    ```java
    MediaSessionCompat mediaSession = new MediaSessionCompat(this, getApplication().getPackageName());
    mediaSession.setCallback(mMediaSessionCallback);
    mediaSession.setFlags(MediaSessionCompat.FLAG_HANDLES_MEDIA_BUTTONS);
    mediaSession.setActive(true);
    PlaybackStateCompat state = new PlaybackStateCompat.Builder()
      .setActions(
        PlaybackStateCompat.ACTION_PLAY | PlaybackStateCompat.ACTION_PLAY_PAUSE | PlaybackStateCompat.ACTION_PAUSE | PlaybackStateCompat.ACTION_SKIP_TO_NEXT | PlaybackStateCompat.ACTION_SKIP_TO_PREVIOUS).build();
    mediaSession.setPlaybackState(state);
    ```

1.  在具有媒体控制功能（如耳机）的设备或模拟器上运行应用程序，以查看 Toast 消息。

## 它是如何工作的...

设置此功能有四个步骤：

1.  创建一个`MediaSession.Callback`并将其附加到 MediaSession

1.  将媒体会话标志设置为指示我们想要媒体按钮

1.  将`SessionState`设置为`active`

1.  使用我们将要处理的操作设置`PlayBackState`

步骤 4 和 1 协同工作，因为回调将只获取在`PlayBackState`中设置的的事件。

由于我们在这个菜谱中实际上并没有控制任何播放，所以我们只是演示如何响应硬件事件。你将需要在`PlayBackState`中实现实际的功能，并在调用`setActions()`之后调用`setState()`。

这是一个非常棒的示例，说明了 API 的更改如何使事情变得更容易。由于新的`MediaSession`和`PlaybackState`被整合到兼容性库中，我们可以在旧版本的操作系统上利用这些新 API。

## 更多...

### 检查正在使用的硬件

如果你想你的应用程序根据当前输出硬件以不同的方式响应，你可以使用`AudioManager`来检查。以下是一个示例：

```java
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

## 参见

+   **开发者文档：MediaSession**

    [`developer.android.com/reference/android/media/session/MediaSession.html`](https://developer.android.com/reference/android/media/session/MediaSession.html)

+   **开发者文档：MediaSessionCompat**

    [`developer.android.com/reference/android/support/v4/media/session/MediaSessionCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/MediaSessionCompat.html)

+   **开发者文档：PlaybackState**

    [`developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html)

+   **开发者文档：PlaybackStateCompat**

    [`developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html)

# 使用默认相机应用拍照

如果你的应用程序需要从相机获取图像，但不是相机替换应用，可能最好允许“默认”相机应用拍照。这也尊重了用户对首选相机应用的偏好。

当你拍照时，除非它仅针对你的应用程序，否则将其公开是良好的实践。（这允许它被包含在用户的照片库中。）这个菜谱将演示使用默认的拍照应用拍照，将其保存到公共文件夹，并显示图像。

## 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为：`UsingTheDefaultCameraApp`。使用默认的 **Phone & Tablet** 选项，并在提示 **Activity Type** 时选择 **Empty Activity**。

## 如何操作...

我们将创建一个包含 ImageView 和按钮的布局。按钮将创建一个 Intent 来启动默认的相机应用。当相机应用完成后，我们的应用将收到回调。首先打开 Android Manifest 并按照以下步骤操作：

1.  添加以下权限：

    ```java
    <uses-permission
    android:name="android.permission.READ_EXTERNAL_STORAGE" />
    ```

1.  打开 `activity_main.xml` 文件，并用以下视图替换现有的 `TextView`：

    ```java
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/imageView"
        android:src="img/ic_launcher"
        android:layout_centerInParent="true"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Take Picture"
        android:id="@+id/button"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:onClick="takePicture"/>
    ```

1.  打开 `MainActivity.java` 并将以下全局变量添加到 `MainActivity` 类中：

    ```java
    final int PHOTO_RESULT=1;
    private Uri mLastPhotoURI=null;
    ```

1.  添加以下方法来创建照片的 URI：

    ```java
    private Uri createFileURI() {
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(System.currentTimeMillis());
        String fileName = "PHOTO_" + timeStamp + ".jpg";
        return Uri.fromFile(new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),fileName));
    }
    ```

1.  添加以下方法来处理按钮点击：

    ```java
    public void takePicture(View view) {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_ CAPTURE);
        if (takePictureIntent.resolveActivity(getPackageManager()) != 
            null) {
            mLastPhotoURI = createFileURI();
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, mLastPhotoURI);
             startActivityForResult(takePictureIntent, PHOTO_RESULT);
        }
    }
    ```

1.  添加一个新的方法来覆盖 `onActivityResult()`，如下所示：

    ```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == PHOTO_RESULT && resultCode == RESULT_OK ) {
            mImageView.setImageBitmap(BitmapFactory.decodeFile(mLastPhotoURI.getPath()));
        }
    }
    ```

1.  你现在可以开始在设备或模拟器上运行应用程序了。

## 它是如何工作的...

与默认相机应用一起工作的有两部分。第一部分是设置启动应用的意图。我们使用 `MediaStore.ACTION_IMAGE_CAPTURE` 创建 Intent，表示我们想要一个拍照应用。通过检查 `resolveActivity()` 的结果来验证是否存在默认应用。只要它不为 null，我们就知道有一个应用程序可以处理这个意图。（否则，我们的应用将会崩溃。）我们创建一个文件名并将其添加到意图中：`putExtra(MediaStore.EXTRA_OUTPUT, mLastPhotoURI)`。

当我们在 `onActivityResult()` 中收到回调时，我们首先确保它是 `PHOTO_RESULT` 和 `RESULT_OK`（用户可能已取消），然后我们在 `ImageView` 中加载照片。

## 还有更多...

如果你不在乎图片存储在哪里，你可以不使用 `MediaStore.EXTRA_OUTPUT` 额外信息来调用 intent。如果你没有指定输出文件，`onActivityResult()` 将在数据 Intent 中包含图像的缩略图。以下是显示缩略图的方法：

```java
if (data != null) {
    imageView
.setImageBitmap((Bitmap) data.getExtras().get("data"));
}
```

这是加载全分辨率图像的代码，使用 `data Intent` 返回的 URI：

```java
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

### 调用默认视频应用

如果你想调用默认的视频捕获应用，过程是相同的。只需在步骤 5 中更改 intent，如下所示：

```java
Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
```

你可以在 `onActivityResult()` 中获取视频的 URI，如下所示：

```java
Uri videoUri = intent.getData();
```

## 参考内容

+   在 第九章 的 *将大图像缩小以避免内存不足异常* 菜谱中，*图形和动画*。

# 使用（旧版）Camera API 拍照

之前的菜谱演示了如何使用 intent 调用默认的图片应用程序。如果您只需要快速拍照，intent 可能是理想的解决方案。如果不是，并且您需要更多对摄像头的控制，这个菜谱将向您展示如何直接使用 Camera API 使用摄像头。

实际上有两种使用 Camera API 的方法——一种是 Android 1.0（API 1）中发布的原始 Camera API 和 Android 5.0（API 21）中发布的 Camera2 API。我们将涵盖新旧两种 API。理想情况下，您可能希望编写适用于最新和最佳 API 的应用程序，但在撰写本文时，Android 5.0（API 21）的市场份额只有大约 23%。如果您只使用 Camera2 API，您将排除超过 75% 的市场。

编写您的应用程序以使用 Camera2 API 来利用新功能，同时仍然使用原始 Camera API 为您的其他用户提供一个功能性的应用程序。为了帮助使用两者，这个菜谱将利用 Android 的新功能，特别是 Android 4.0（API 14）中引入的 `TextureView`。我们将使用 `TextureView` 来代替更传统的 `SurfaceView`，用于显示摄像头预览。这将允许您使用与较新的 Camera2 API 相同的布局，因为它也使用 `TextureView`。（将最小 API 设置为 Android 4.0（API 14）及以上，这占市场份额超过 96%，并不会太多地限制您的用户基础。）

## 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为 `CameraAPI`。在 **目标 Android 设备** 对话框中，选择 **手机和平板** 选项，并为 **最小 SDK** 选择 API 14（或更高）。当提示 **活动类型** 时，选择 **空活动**。

## 如何操作...

首先打开 AndroidManifest.xml 并按照以下步骤操作：

1.  添加以下两个权限：

    ```java
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission
    android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ```

1.  现在打开 `activity_main.xml` 并将现有的 TextView 替换为以下视图：

    ```java
    <TextureView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/textureView"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Take Picture"
        android:id="@+id/button"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:onClick="takePicture"/>
    ```

1.  打开 `MainActivity.java` 并修改 `MainActivity` 类声明以实现 `SurfaceTextureListener`，如下所示：

    ```java
    public class MainActivity extends AppCompatActivity
            implements TextureView.SurfaceTextureListener {
    ```

1.  将以下全局声明添加到 `MainActivity` 中：

    ```java
    @Deprecated
    private Camera mCamera;
    private TextureView mTextureView;
    ```

1.  创建以下 `PictureCallback` 以处理保存照片：

    ```java
    Camera.PictureCallback pictureCallback = new Camera.PictureCallback() {
        @Override
        public void onPictureTaken(byte[] data, Camera camera) {
            try {
                String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(System.currentTimeMillis());
                String fileName = "PHOTO_" + timeStamp + ".jpg";
                File pictureFile = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),fileName);

                FileOutputStream fileOutputStream =new FileOutputStream(pictureFile.getPath());
                fileOutputStream.write(data);
                fileOutputStream.close();
                Toast.makeText(MainActivity.this, "Picture Taken", Toast.LENGTH_SHORT).show();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    };
    ```

1.  将以下代码添加到现有的 `onCreate()` 回调中：

    ```java
    mTextureView = (TextureView)findViewById(R.id.textureView);
    mTextureView.setSurfaceTextureListener(this);
    ```

1.  添加以下方法以实现 `SurfaceTextureListener` 接口：

    ```java
    public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
        mCamera = Camera.open();
        if (mCamera!=null) {
            try {
                mCamera.setPreviewTexture(surface);
                mCamera.startPreview();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
        if (mCamera!=null) {
            mCamera.stopPreview();
            mCamera.release();
        }
        return true;
    }
    public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {
        // Unused
    }
    public void onSurfaceTextureUpdated(SurfaceTexture surface) {
        // Unused
    }
    ```

1.  添加以下方法来处理按钮点击：

    ```java
    public void takePicture(View view) {
        if (mCamera!=null) {
            mCamera.takePicture(null, null, pictureCallback);
        }
    }
    ```

1.  在带有摄像头的设备或模拟器上运行应用程序。

## 它是如何工作的...

首先，请注意，当您在 Android Studio 中查看此代码时，您将看到很多带有以下警告的删除线代码：

```java
'android.hardware.Camera' is deprecated
```

如介绍中所述，`android.hardware.camera2` API 在 Android 5.0 (API 19) 中引入，并取代了 `android.hardware.camera` API。

### 小贴士

你可以添加以下注释来抑制弃用警告：

```java
@SuppressWarnings("deprecation")
```

使用 Camera API 时有两个主要步骤：

+   设置预览

+   捕获图像

我们从布局中获取`TextureView`，然后使用以下代码将我们的活动（实现了`SurfaceTextureListener`）作为监听器分配：

```java
mTextureView.setSurfaceTextureListener(this);
```

当`TextureView`表面准备好时，我们会在我们创建的`Camera.PictureCallback`类中收到`onSurfaceTextureAvailable`回调，其中我们使用以下代码设置预览表面：

```java
mCamera.setPreviewTexture(surface);
mCamera.startPreview();
```

下一步是在按钮按下时拍照。我们使用以下代码来完成这个操作：

```java
mCamera.takePicture(null, null, pictureCallback);
```

当图片准备好时，我们会在我们创建的`Camera.PictureCallback`类中收到`onPictureTaken()`回调。

## 更多内容...

请记住，这段代码的目的是向您展示它是如何工作的，而不是创建一个完整的商业应用程序。正如大多数开发者所知，编码中的真正挑战是处理所有的问题情况。一些需要改进的领域包括添加切换相机的功能，因为当前应用程序使用的是默认相机。此外，请查看预览和保存图片时的设备方向。一个更复杂的应用程序会在后台线程上处理一些工作，以避免 UI 线程上的延迟。（请查看下一个菜谱，了解我们如何在后台线程上执行一些相机处理。） 

### 设置相机参数

Camera API 包括参数，允许我们调整相机设置。在这个例子中，我们可以更改预览的大小：

```java
Camera.Parameters parameters = mCamera.getParameters();
parameters.setPreviewSize(mPreviewSize.width, 
mPreviewSize.height);
mCamera.setParameters(parameters);
```

请记住，硬件也必须支持我们想要的设置。在这个例子中，我们首先想要查询硬件以获取所有可用的预览模式，然后设置符合我们要求的那个。 （在下一个菜谱中设置图片分辨率时，请参见此示例。）请参阅 Camera 文档链接中的`getParameters()`。

## 参见

+   下一个菜谱：*使用 Camera2（新）API 拍照*

+   在第八章中，*读取设备方向*的菜谱，*使用触摸屏和传感器*提供了检测当前设备方向的示例

+   **开发者文档：构建相机应用**，请访问：[`developer.android.com/guide/topics/media/camera.html#custom-camera`](https://developer.android.com/guide/topics/media/camera.html#custom-camera)

+   [`developer.android.com/reference/android/hardware/Camera.html`](https://developer.android.com/reference/android/hardware/Camera.html)

# 使用 Camera2（新）API 拍照

现在我们已经了解了旧的 Camera API，是时候学习新的 Camera2 API 了。不幸的是，由于 API 的异步特性，它稍微复杂一些。幸运的是，整体概念与之前的 Camera API 相同。

## 准备工作

在 Android Studio 中创建一个新的项目，命名为`Camera2API`。在**目标 Android 设备**对话框中，选择**手机和平板电脑**选项，并将**最小 SDK**设置为 API 21（或更高）。当提示**活动类型**时，选择**空活动**。

## 如何操作...

正如您将看到的，这个菜谱有很多代码。首先，打开 AndroidManifest 文件，按照以下步骤操作：

1.  添加以下两个权限：

    ```java
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ```

1.  现在打开`activity_main.xml`，并用以下视图替换现有的 TextView：

    ```java
    <TextureView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/textureView"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Take Picture"
        android:id="@+id/button"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:onClick="takePicture"/>
    ```

1.  现在打开`MainActivity.java`文件，并将以下全局变量添加到`MainActivity`类中：

    ```java
    private CameraDevice mCameraDevice = null;
    private CaptureRequest.Builder mCaptureRequestBuilder = null;
    private CameraCaptureSession mCameraCaptureSession  = null;
    private TextureView mTextureView = null;
    private Size mPreviewSize = null;
    ```

1.  添加以下`Comparator`类：

    ```java
    static class CompareSizesByArea implements Comparator<Size> {
        @Override
        public int compare(Size lhs, Size rhs) {
            return Long.signum((long) lhs.getWidth() * lhs.getHeight() - (long) rhs.getWidth() * rhs.getHeight());
        }
    }
    ```

1.  添加以下`CameraDevice.StateCallback`：

    ```java
    private CameraDevice.StateCallback mStateCallback = new CameraDevice.StateCallback() {
        @Override
        public void onOpened(CameraDevice camera) {
            mCameraDevice = camera;
            SurfaceTexture texture = mTextureView.getSurfaceTexture();
            if (texture == null) {
                return;
            }
            texture.setDefaultBufferSize(mPreviewSize.getWidth(), mPreviewSize.getHeight());
            Surface surface = new Surface(texture);
            try {
                mCaptureRequestBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
            } catch (CameraAccessException e){
                e.printStackTrace();
            }
            mCaptureRequestBuilder.addTarget(surface);
            try {
                mCameraDevice.createCaptureSession(Arrays.asList(surface), mPreviewStateCallback, null);
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

1.  添加以下`SurfaceTextureListener`：

    ```java
    private TextureView.SurfaceTextureListener mSurfaceTextureListener =     new TextureView.SurfaceTextureListener() {
        @Override
        public void onSurfaceTextureUpdated(SurfaceTexture surface) {}
        @Override
        public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {}
        @Override
        public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
                return false;
        }
        @Override
        public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
                openCamera();
        }
    };
    ```

1.  添加以下`CameraCaptureSession.StateCallback`：

    ```java
    private CameraCaptureSession.StateCallback mPreviewStateCallback = new CameraCaptureSession.StateCallback() {
        @Override
        public void onConfigured(CameraCaptureSession session) {
            startPreview(session);
        }

        @Override
        public void onConfigureFailed(CameraCaptureSession session) {}
    };
    ```

1.  将以下代码添加到现有的`onCreate()`回调中：

    ```java
    mTextureView = (TextureView) findViewById(R.id.textureView);
    mTextureView.setSurfaceTextureListener(mSurfaceTextureListener);
    ```

1.  添加以下方法以重写`onPause()`和`onResume()`：

    ```java
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
            mTextureView.setSurfaceTextureListener(mSurfaceTextureListener);
        }
    }
    ```

1.  添加`openCamera()`方法：

    ```java
    private void openCamera() {
        CameraManager manager = (CameraManager) getSystemService(CAMERA_SERVICE);
        try{
            String cameraId = manager.getCameraIdList()[0];
            CameraCharacteristics characteristics = manager.getCameraCharacteristics(cameraId);
            StreamConfigurationMap map = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP); 
            mPreviewSize = map.getOutputSizes(SurfaceTexture.class) [0];
            manager.openCamera(cameraId, mStateCallback, null);
        } catch(CameraAccessException e) {
            e.printStackTrace();
        } catch (SecurityException e) {
            e.printStackTrace();
        }
    }
    ```

1.  添加`startPreview()`方法：

    ```java
    private void startPreview(CameraCaptureSession session) { 
        mCameraCaptureSession = session; 
        mCaptureRequestBuilder.set(CaptureRequest.CONTROL_MODE,CameraMetadata.CONTROL_MODE_AUTO); 
        HandlerThread backgroundThread = new HandlerThread("CameraPreview"); 
        backgroundThread.start();
        Handler backgroundHandler = new Handler(backgroundThread. getLooper());
        try {
            mCameraCaptureSession.setRepeatingRequest(mCaptureRequestBuilder.build(), null, backgroundHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
    ```

1.  添加`getPictureFile()`方法：

    ```java
    private File getPictureFile() {
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss"). format(System.currentTimeMillis());
        String fileName = "PHOTO_" + timeStamp + ".jpg";
        return new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),fileName);
    }
    ```

1.  添加保存图像文件的`takePicture()`方法：

    ```java
    protected void takePicture(View view) {
        if (null == mCameraDevice) {
            return;
        }
        CameraManager manager = (CameraManager)
        getSystemService(Context.CAMERA_SERVICE);
        try {
            CameraCharacteristics characteristics = manager.getCameraCharacteristics(mCameraDevice.getId());
            StreamConfigurationMap configurationMap = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
            if (configurationMap == null) return;
            Size largest = Collections.max(
                Arrays.asList(configurationMap.getOutputSizes(ImageFormat.JPEG)),
                new CompareSizesByArea());
            ImageReader reader = ImageReader.newInstance(largest.getWidth(), largest.getHeight(), ImageFormat.JPEG, 1);
            List < Surface > outputSurfaces = new ArrayList < Surface > (2);
            outputSurfaces.add(reader.getSurface());
            outputSurfaces.add(new Surface(mTextureView.getSurfaceTexture()));
            final CaptureRequest.Builder captureBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_STILL_ CAPTURE);
            captureBuilder.addTarget(reader.getSurface());
            captureBuilder.set(CaptureRequest.CONTROL_MODE,
                CameraMetadata.CONTROL_MODE_AUTO);
            ImageReader.OnImageAvailableListener readerListener = new ImageReader.OnImageAvailableListener() {
                @Override
                public void onImageAvailable(ImageReader reader) {
                    Image image = null;
                    try {
                        image = reader.acquireLatestImage();
                        ByteBuffer buffer = image.getPlanes()[0].getBuffer();
                        byte[] bytes = new byte[buffer.capacity()];
                        buffer.get(bytes);
                        OutputStream output = new FileOutputStream( get PictureFile());
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
            };
            HandlerThread thread = new HandlerThread("CameraPicture");
            thread.start();
            final Handler backgroudHandler = new Handler(thread.getLooper());
            reader.setOnImageAvailableListener(readerListener, backgroudHandler);
            final CameraCaptureSession.CaptureCallback captureCallback = new CameraCaptureSession.CaptureCallback() {
                @Override
                public void onCaptureCompleted(
                CameraCaptureSession session, CaptureRequest request, TotalCaptureResult result) {
                        super.onCaptureCompleted(session, request, result);
                        Toast.makeText(MainActivity.this, "Picture Saved", Toast.LENGTH_SHORT).show();
                        startPreview(session);
                }
            };
            mCameraDevice.createCaptureSession(outputSurfaces, new CameraCaptureSession.StateCallback() {
                @Override
                public vod onConfigured(CameraCaptureSession session) {
                    try {
                        session.capture(captureBuilder.build(), captureCallback, backgroudHandler);
                    } catch (CameraAccessException e) {
                        e.printStackTrace();
                    }
                }
                @Override
                public void onConfigureFailed(CameraCaptureSession session) { }
            }, backgroudHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
    ```

1.  在具有摄像头的设备或模拟器上运行应用程序。

## 工作原理...

由于我们在前面的菜谱中学习了 TextureView，我们可以跳转到新的 Camera2 API 信息。

尽管涉及更多的类，就像旧的 Camera API 一样，这里有两个基本步骤：

+   设置预览

+   捕获图像

### 设置预览

下面是代码如何设置预览的概述：

1.  首先，我们在`onCreate()`中使用`setSurfaceTextureListener()`方法设置`TextureView.SurfaceTextureListener`。

1.  当我们收到`onSurfaceTextureAvailable()`回调时，我们打开相机。

1.  我们将我们的`CameraDevice.StateCallback`类传递给`openCamera()`方法，它最终调用`onOpened()`回调。

1.  `onOpened()`通过调用`getSurfaceTexture()`获取预览的表面，并通过调用`createCaptureSession()`将其传递给 CameraDevice。

1.  最后，当`CameraCaptureSession.StateCallback onConfigured()`被调用时，我们使用`setRepeatingRequest()`方法开始预览。

### 捕获图像

尽管`takePicture()`方法看起来像是过程性的，捕获图像也涉及到多个类并依赖于回调。以下是代码如何捕获图像的分解：

1.  用户点击**拍照**按钮。

1.  然后查询相机以找到最大的可用图像大小。

1.  然后创建一个`ImageReader`。

1.  接下来，他/她设置了`OnImageAvailableListener`，并在`onImageAvailable()`回调中保存图像。

1.  然后，创建`CaptureRequest.Builder`并包含`ImageReader`表面。

1.  接下来，创建`CameraCaptureSession.CaptureCallback`，它定义了`onCaptureCompleted()`回调。捕获完成后，它重新开始预览。

1.  然后，调用`createCaptureSession()`方法，创建一个`CameraCaptureSession.StateCallback`。这是调用`capture()`方法的地方，传入之前创建的`CameraCaptureSession.CaptureCallback`。

## 更多内容...

与之前的 Camera 示例一样，我们只是创建了基础代码来演示一个可工作的 Camera 应用程序。同样，还有改进的空间。首先，你应该处理设备方向，无论是预览还是保存图像时。（参见之前的配方以获取链接。）此外，随着 Android 6.0（API 23）的现在可用，现在是开始使用新的权限模型的好时机。而不是像我们在`openCamera()`方法中所做的那样只是检查异常，最好是检查所需的权限。

## 参见

+   之前的配方：*使用（旧）Camera API 拍照*

+   *第十四章中新的 Android 6.0 运行时权限模型* 第十四章*，为 Play 商店准备你的应用*

+   开发者文档：Camera2 API

+   [`developer.android.com/reference/android/hardware/camera2/package-summary.html`](https://developer.android.com/reference/android/hardware/camera2/package-summary.html)
