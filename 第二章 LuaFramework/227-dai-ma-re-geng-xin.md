##代码热更新

####一、新建场景

在任意物体上添加Main组件。其实Main组件里面只是调用了`AppFacade.Instance.StartUp()`，这是框架的起点。框架将会自动完成资源加载、热更新等等事项。

####二、删掉事例的调用

现在不需要框架自带的示例了，需要删掉一些代码，使框架只运行我们编写的lua文件。打开`Assets\LuaFramework\Scripts\Manager\GameManager.cs`，将`OnInitalize`修改成下图这个样子。这是lua的入口，框架会调用Main.lua的Main方法。

```csharp
    void OnInitialize1() {
        LuaManager.InitStart();
            LuaManager.DoFile("Logic/Game");         //加载游戏
            LuaManager.DoFile("Logic/Network");      //加载网络
            NetManager.OnInit();                     //初始化网络
            Util.CallMethod("Game", "OnInitOK");     //初始化完成

            initialize = true;
    }
```

