##代码热更新

####一、新建场景

在任意物体上添加Main组件。其实Main组件里面只是调用了`AppFacade.Instance.StartUp()`，这是框架的起点。框架将会自动完成资源加载、热更新等等事项。

####二、删掉事例的调用

现在不需要框架自带的示例了，需要删掉一些代码，使框架只运行我们编写的lua文件。打开`Assets\LuaFramework\Scripts\Manager\GameManager.cs`，将`OnInitalize`修改成下图这个样子。这是lua的入口，框架会调用`Main.lua`的`Main`方法。

```csharp
    void OnInitialize() {
        LuaManager.InitStart();
        // LuaManager.DoFile("Logic/Game");         //加载游戏
        // LuaManager.DoFile("Logic/Network");      //加载网络
        // NetManager.OnInit();                     //初始化网络
        // Util.CallMethod("Game", "OnInitOK");     //初始化完成
        // ...
            
        initialize = true;
    }
```

####三、编写lua代码

打开Assets\LuaFramework\Lua\main.lua，编写lua代码。这里只添加一句“LuaFramework.Util.Log("HelloWorld");”（如下所示），它的功能相当于Debug.Log("HelloWorld")。

```csharp
    --主入口函数。从这里开始lua逻辑
    function Main()

        LuaFramework.Util.Log("HelloWorld");

    end
```


“LuaFramework.Util.Log("HelloWorld")”中的Util是c#里定义的类，供lua中调用。可以打开Assets\LuaFramework\Editor\CustomSettings.cs看到所有可以供lua调用的类，如下图是CustomSettings.cs的部分语句。

```csharp
        //for LuaFramework
        _GT(typeof(RectTransform)),
        _GT(typeof(Text)),

        _GT(typeof(Util)),
        _GT(typeof(AppConst)),
        _GT(typeof(LuaHelper)),
        _GT(typeof(ByteBuffer)),
        _GT(typeof(LuaBehaviour)),

        _GT(typeof(GameManager)),
        _GT(typeof(LuaManager)),
        _GT(typeof(PanelManager)),
        _GT(typeof(SoundManager)),
        _GT(typeof(TimerManager)),
        _GT(typeof(ThreadManager)),
        _GT(typeof(NetworkManager)),
        _GT(typeof(ResourceManager)),
```

再由具体的类可以查找所有的API（参见下面两个图），如下图是Util类的部分语句。

```csharp
        public static void Log(string str) {
            Debug.Log(str);
        }

        public static void LogWarning(string str) {
            Debug.LogWarning(str);
        }

        public static void LogError(string str) {
            Debug.LogError(str);
        }
```

####四、运行游戏

点击菜单栏中LuaFramework→Build Windows Resource，生成资源文件。然后运行游戏，即可在控制台中看到打印出的HelloWorld。

按照默认的设置，每更改一次lua代码，都需要执行Build XXX Resource才能生效。读者可以将Assets\LuaFramework\Scripts\ConstDefine\AppConst.cs中的LuaBundleMode修改为false，这样代码文件便不会以AssetBundle模式读取，会直接生效，以方便调试。

```csharp
    //Lua代码AssetBundle模式
    public const bool LuaBundleMode = false;
```

####五、热更新原理

接下来便要尝试代码热更新，让程序下载服务器上的lua文件，然后运行它。在说明热更新之前，需要先看看Unity3D热更新的一般方法。如下图所示，Unity3D的热更新会涉及3个目录。

![](/assets/屏幕快照 2017-11-11 下午1.44.14.png)


**游戏资源目录**：里面包含Unity3D工程中StreamingAssets文件夹下的文件。安装游戏之后，这些文件将会被一字不差地复制到目标机器上的特定文件夹里，不同平台的文件夹不同，如下所示（上图以windows平台为例）

```csharp
    Mac OS或Windows：Application.dataPath + "/StreamingAssets";

    IOS： Application.dataPath + "/Raw";

    Android：jar:file://" + Application.dataPath + "!/assets/";
```

**数据目录**：由于“游戏资源目录”在Android和IOS上是只读的，不能把网上的下载的资源放到里面，所以需要建立一个“数据目录”，该目录可读可写。第一次开启游戏后，程序将“游戏资源目录”的内容复制到“数据目录中”（步骤1，这个步骤只会执行一次，下次再打开游戏就不复制了）。游戏过程中的资源加载，都是从“数据目录”中获取、解包（步骤3）。不同平台下，“数据目录”的地址也不同，LuaFramework的定义如下：

```csharp
    Android或IOS：Application.persistentDataPath + "/LuaFramework"    

    Mac OS或Windows：c:/LuaFramework/

    调试模式下：Application.dataPath + "/StreamingAssets/"
```
>注：”LuaFramework”和”StreamingAssets”由配置决定，这里取默认值

**网络资源地址**：存放游戏资源的网址，游戏开启后，程序会从网络资源地址下载一些更新的文件到数据目录。

这些目录包含着不同版本的资源文件，以及用于版本控制的files.txt。Files.txt的内容如下图所示，里面存放着资源文件的名称和md5码。程序会先下载“网络资源地址”上的files.txt，然后与“数据目录”中文件的md5码做比较，更新有变化的文件（步骤2）。

```csharp
    StreamingAssets|1738a512698f18f78bfca1591edcf5ae
    StreamingAssets.manifest|1994f53ee73e2badeddd65601d587f38
    message.unity3d|1ab91c2ccd6a03232c0255c1895c6435
    message.unity3d.manifest|9f46935cf1de80e8fab9927c6a25803f
    prompt.unity3d|01b68a164fa653bf853bedbda3a5f3d0
    prompt.unity3d.manifest|9ebfa552ca1e8a84360944c0dee65bb3
    prompt_asset.unity3d|1655812ce2260a4747a7fdb8e39fc007
    prompt_asset.unity3d.manifest|ed9d66a485b1d96d504f7002c52ef9a4
    shared_asset.unity3d|17ce04b42949f651895d634f852dd972
    shared_asset.unity3d.manifest|71721e8dbf69055affcf2a2e0b551f01
    tank.unity3d|a5f10487d490d13642d40fce937c195c
    tank.unity3d.manifest|202261f1e418ab2bbb8384ed6f54f958
    lua/Build.bat|4cfc2f258ecdb9bb43756a2e737c7a1d
```