##资源热更新

&emsp;

热更新涉及资源热更新和代码热更新（其实lua代码也是资源），那接下来看看如何动态加载一个模型，然后热更成其他素材。这一部分涉及资源打包、动态创建资源等内容。

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

        // 坦克的
        AddBuildMap("tank" + AppConst.ExtName, "*.prefab", "Assets/Tank");
    }
```
