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





