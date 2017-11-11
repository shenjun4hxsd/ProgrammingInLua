##资源热更新

&emsp;

热更新涉及资源热更新和代码热更新（其实lua代码也是资源），那接下来看看如何动态加载一个模型，然后热更成其他素材。这一部分涉及资源打包、动态创建资源等内容。

&emsp;


####一、创建物体

为了调试的方便，先将框架配置为本地模式，待测试热更新时再改成更新模式。

```csharp
        public const bool UpdateMode = false;  ✅                     //更新模式-默认关闭 
        public const bool LuaByteMode = false;                       //Lua字节码模式-默认关闭 
        public const bool LuaBundleMode = false;  ✅                  //Lua代码AssetBundle模式
```

先测试个简单的创建物体，新建一个名为go的物体，然后设置它的坐标为(1,1,1)。这段代码虽然不涉及资源加载，但能展示“把物体添加到场景中”的过程。Main.lua的代码如下：

```lua
    function Main()                                    
        local go = UnityEngine.GameObject ('go')
        go.transform.position = Vector3.one             
    end
```

要热更新资源，便需要制作资源。这里制作一个名为tankPrefab的坦克模型预设，然后存到Assets/Tank目录下。接下来对它做打包，然后动态加载。

&emsp;


####二、资源打包

LuaFramework在打包方面并没有做太多的工作，我们需要手动打包。打开Assets/LuaFramework/Editor/Packager.cs，按照示例的写法，加上下面这一行：将Assets/Tank目录下的所有预设（.prefab）打包成名为tank的包。

```csharp
    /// <summary>
    /// 处理框架实例包
    /// </summary>
    static void HandleExampleBundle() {
        string resPath = AppDataPath + "/" + AppConst.AssetDir + "/";
        if (!Directory.Exists(resPath)) Directory.CreateDirectory(resPath);

        AddBuildMap("prompt" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/Examples/Builds/Prompt");
        AddBuildMap("message" + AppConst.ExtName, "*.prefab", "Assets/LuaFramework/Examples/Builds/Message");

        AddBuildMap("prompt_asset" + AppConst.ExtName, "*.png", "Assets/LuaFramework/Examples/Textures/Prompt");
        AddBuildMap("shared_asset" + AppConst.ExtName, "*.png", "Assets/LuaFramework/Examples/Textures/Shared");

        // 坦克的 ✅
        AddBuildMap("tank" + AppConst.ExtName, "*.prefab", "Assets/Tank");
    }
```

点击“Build Windows Resource”，即可在StreamingAssets中看到打包好的文件。


>Unity3D资源包里面包含多个资源，就像一个压缩文件一样。在动态加载的时候，便需要有加载包文件、或取包中的资源两步操作（框架已经帮我们做好了这部分工作，直接调用API即可）。

&emsp;


####三、动态加载模型

如下图所示，Unity3D资源包里面包含多个资源，就像一个压缩文件一样。在动态加载的时候，便需要有加载包文件、或取包中的资源两步操作（框架已经帮我们做好了这部分工作，直接调用API即可）。


```lua
    --主入口函数。从这里开始lua逻辑
    function Main()                                 
        LuaHelper = LuaFramework.LuaHelper;
        resMgr = LuaHelper.GetResManager();
        resMgr:LoadPrefab('tank', { 'TankPrefab' }, OnLoadFinish);
    end
    
    --加载完成后的回调--
    function OnLoadFinish(objs)
        local go = UnityEngine.GameObject.Instantiate(objs[0]);
        LuaFramework.Util.Log("Finish");        
    end
```

完成后运行游戏，即可看到动态加载出来的模型。

&emsp;

####四、加载资源的过程

只有理解了动态加载，即LoadPrefab的过程，才能算是真正的理解了热更新。LoadPrefab为ResourceManager中定义的方法，在Assets\LuaFramework\Scripts\Manager\ResourceManager.cs中实现。

```csharp
     public void LoadPrefab(string abName, string[] assetNames, LuaFunction func) {
        abName = abName.ToLower();
        List<UObject> result = new List<UObject>();
        for (int i = 0; i < assetNames.Length; i++) {
            UObject go = LoadAsset<UObject>(abName, assetNames[i]);
            if (go != null) result.Add(go);
        }
        if (func != null) func.Call((object)result.ToArray());
    }
    
    /// <summary>
    /// 载入素材
    /// </summary>
    public T LoadAsset<T>(string abname, string assetname) where T : UnityEngine.Object {
        abname = abname.ToLower();
        AssetBundle bundle = LoadAssetBundle(abname);
        return bundle.LoadAsset<T>(assetname);
    }
    
    /// <summary>
    /// 载入AssetBundle
    /// </summary>
    /// <param name="abname"></param>
    /// <returns></returns>
    public AssetBundle LoadAssetBundle(string abname) {
        if (!abname.EndsWith(AppConst.ExtName)) {
            abname += AppConst.ExtName;
        }
        AssetBundle bundle = null;
        if (!bundles.ContainsKey(abname)) {
            byte[] stream = null;
            string uri = Util.DataPath + abname;
            Debug.LogWarning("LoadFile::>> " + uri);
            LoadDependencies(abname);

            stream = File.ReadAllBytes(uri);
            bundle = AssetBundle.LoadFromMemory(stream); //关联数据的素材绑定
            bundles.Add(abname, bundle);
        } else {
            bundles.TryGetValue(abname, out bundle);
        }
        return bundle;
    }

    /// <summary>
    /// 载入依赖
    /// </summary>
    /// <param name="name"></param>
    void LoadDependencies(string name) {
        if (manifest == null) {
            Debug.LogError("Please initialize AssetBundleManifest by calling AssetBundleManager.Initialize()");
            return;
        }
        // Get dependecies from the AssetBundleManifest object..
        string[] dependencies = manifest.GetAllDependencies(name);
        if (dependencies.Length == 0) return;

        for (int i = 0; i < dependencies.Length; i++)
            dependencies[i] = RemapVariantName(dependencies[i]);

        // Record and load all dependencies.
        for (int i = 0; i < dependencies.Length; i++) {
            LoadAssetBundle(dependencies[i]);
        }
    }
```

打包后，Unity3D会产生一个名为AssetBundle.manifest的文件（框架会将该文件放在StreamingAssets中），该文件包含所有包的依赖信息。所以在加载资源前需要先加载这个文件，m_AssetBundleManifest便是指向这个包的变量。相关代码如下：

```csharp
    /// <summary>
    /// 初始化
    /// </summary>
    public void Initialize() {
        byte[] stream = null;
        string uri = string.Empty;
        bundles = new Dictionary<string, AssetBundle>();
        uri = Util.DataPath + AppConst.AssetDir;
        Debug.Log("uri : " + uri);
        if (!File.Exists(uri)) return;
        stream = File.ReadAllBytes(uri);
        assetbundle = AssetBundle.LoadFromMemory(stream);
        manifest = assetbundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
    }
```

加载这个包后，便可以使用下面的语句获取某个包所依赖的所有包名，然后加载它们。

```csharp
    string[] dependencies = manifest.GetAllDependencies(name);
```

字典类型的bundles保存了所有已经加载资源包。如果某个包已经被加载过，那下次需要用到它时，直接从字典中取出即可，减少重复加载。简化后的代码如下：

```csharp
    /// <summary>
    /// 载入AssetBundle
    /// </summary>
    /// <param name="abname"></param>
    /// <returns></returns>
    public AssetBundle LoadAssetBundle(string abname) {
        if (!abname.EndsWith(AppConst.ExtName)) {
            abname += AppConst.ExtName;
        }
        AssetBundle bundle = null;
        if (!bundles.ContainsKey(abname)) {
            byte[] stream = null;
            string uri = Util.DataPath + abname;
            Debug.LogWarning("LoadFile::>> " + uri);
            LoadDependencies(abname);

            stream = File.ReadAllBytes(uri);
            bundle = AssetBundle.LoadFromMemory(stream); //关联数据的素材绑定
            bundles.Add(abname, bundle);
        } else {
            bundles.TryGetValue(abname, out bundle);
        }
        return bundle;
    }
```

&emsp;



####五、资源热更新

“资源热更新”和上一篇的“代码热更新”完全相同，开启更新模式后，将新的资源文件复制到服务器上，框架即可自动下载更新的资源。这里不再复述。


🔚
