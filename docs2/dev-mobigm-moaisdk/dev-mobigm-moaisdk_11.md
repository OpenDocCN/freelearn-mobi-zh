# 第十一章。让正确的音乐进来！

音频和音乐总是留给最后阶段。这本书也发生了同样的事情，这真是太尴尬了。如果你对你的游戏体验认真负责，从一开始就要考虑音乐和声音。即使你的游戏不是以音频为中心，你也应该将音乐和声音视为反馈系统，并将它们与图形同等重要。

在本章中，我们将学习如何在游戏中使用内置的 Untz 声音系统播放音频文件。我们将添加背景音乐和音效。

# 音频管理器

为了在我们的游戏中添加一些音乐和效果，我们将创建一个名为`AudioManager`的模块，该模块将负责加载和播放声音。

1.  首先，我们需要创建一个名为`audio_manager.lua`的文件。这个文件将负责管理我们所有的声音需求。在这种情况下，我们将使用 Untz，但你可以轻松地修改它以使用 FMOD，如下所示：

    ```swift
    module ( "AudioManager", package.seeall )
    local audio_definitions = {
      backgroundMusic = {
        type = RESOURCE_TYPE_SOUND, 
        fileName = 'sounds/music.mp3', 
        loop = true,
        volume = 1
      },
      jump = {
        type = RESOURCE_TYPE_SOUND, 
        fileName = 'sounds/jump.wav', 
        loop = false,
        volume = 1
      }
    }
    ```

1.  与我们为游戏定义资源的方式一样，我们将使用音频文件的资源定义来定义所有声音。我们之前在第六章中看到了这一点，*资源管理器*。如果你不明白我们在做什么，我建议你回过头去阅读关于声音定义的内容。第六章。`sounds`表将作为我们声音的缓存。它将如下所示：

    ```swift
    sounds = {}
    ```

1.  在初始化方法中，我们首先加载所有添加的定义。我们通过从`ResourceDefinitions`调用`setDefinitions`方法来完成此操作，如下所示：

    ```swift
    function AudioManager:initialize ()
        ResourceDefinitions:setDefinitions ( audio_definitions )
    ```

1.  然后，由于我们正在使用 Untz，我们需要按照以下方式初始化它：

    ```swift
        MOAIUntzSystem.initialize ()
    end
    ```

    ### 小贴士

    您可以使用两个可选参数调用`MOAIUntzSystem.initialize`：采样率（可以更改以匹配您的音频文件）和每缓冲区帧数。

1.  为了播放我们的声音，我们将定义一个方法，将它们加载到本地的`sounds`表中，如下所示：

    ```swift
    function AudioManager:get ( name )
        local audio = self.sounds[name]

        if not audio then 
            audio = ResourceManager:get ( name )
            self.sounds[name] = audio
        end

        return audio
    end
    ```

    此方法检查音频是否已被预先加载；如果没有加载，则加载并返回它。我们将此与我们将要编写的`play`方法分开，因为我们可能希望在播放之前预加载声音。

1.  下一步是创建`play`方法：

    ```swift
    function AudioManager:play ( name, loop )
        local audio = AudioManager:get ( name )
    ```

    1.  这里我们获取实际的音频。如果它已被预加载，则可以播放而无需任何加载延迟。

        ```swift
            if loop ~= nil then
                audio:setLooping ( loop )
            end
        ```

    1.  在这里，我们可以使用`loop`参数来覆盖默认的`loop`属性。

        ```swift
            audio:play ()
        end
        ```

    1.  最后，我们播放它。

        ### 小贴士

        值得注意的是，多次播放声音会导致之前的播放中断。

1.  现在，我们将编写一个停止音频的函数：

    ```swift
    function AudioManager:stop ( name )
        local audio = AudioManager:get ( name )
        audio:stop ()
    end
    ```

1.  我们需要在`main.lua`文件中所需`game.lua`之上添加此文件。

# 背景音乐

现在，让我们按照以下步骤播放一些背景音乐：

1.  在`Game:initialize`的底部，您应该添加以下方法调用：

    ```swift
        AudioManager:initialize ()
    ```

1.  我们使用`AudioManager:initialize`（我们在本章早些时候创建的方法）初始化音频：

    ```swift
        AudioManager:play ( 'backgroundMusic' )
    ```

1.  然后我们播放背景音乐。记住，我们在`audio_manager.lua`文件中的`audio_definitions`表中定义了它。

应该就是这样了。运行游戏，你应该正在听 Milhouse Palacios 的一首精彩歌曲。

# 声音效果

我们将添加一个在角色跳跃时播放的声音。

打开`character.lua`文件，并在`Character:jump`方法内部调用`AudioManager:play`：

```swift
function Character:jump ( keyDown )
    if keyDown and not self.jumping then
        AudioManager:play ( 'jump' )
        self.physics.body:applyForce ( 0, 8000 )
        self.jumping = true
        self:startAnimation ( 'jump' )
    end
end
```

就这些了。运行它并听你的跳跃。

# 摘要

在本章中，我们创建了一个新的模块来处理声音。我们添加了一些背景音乐和一个跳跃声音效果。

我们已经覆盖了很多内容，你现在可以使用我们所学到的知识来创建一个完整游戏。下一章将指导你如何将我们的*集中注意力*游戏部署到 iOS 上。
