##编写Lua逻辑

&emsp;

为实现代码热更新，在Unity3D中使用lua，然而为此也需付出不少代价。其一，使代码结构混乱（尽管可以优化），其二降低了运行速度，其三增加学习成本（还要多学一门语言）。为了热更新，所有的逻辑都要用lua编写，那么怎样用lua编写游戏逻辑呢？

&emsp;

####一、Lua 的 Update 方法

第一篇“代码热更新”演示了用lua打印HelloWorld的方法，第二篇“资源热更新”演示了加载坦克模型的方法。这一篇要把两者结合起来，用lua实现“用键盘控制坦克移动”的功能。用Lua和用c#编写的Unity3D程序大同小异，只需正确使用API即可。

1）、Update 方法

出于效率的考虑，tolua提供了名为UpdateBeat的对象，在LuaFramework中，只需给UpdateBeat添加回调函数，该函数便会每帧执行，相当于Monobehaviour的Update方法。Lua代码如下所示：

```lua
    function Main()                                 
        UpdateBeat:Add(Update, self)
    end
  
    function Update()
        LuaFramework.Util.Log("每帧执行一次");
    end
```

除了UpdateBeat，tolua还提供了LateUpdateBeat和FixedUpdateBeat，对应于Monobehaviour中的LateUpdate和FixedUpdate。

&emsp;

2）、控制坦克

现在编写“用键盘控制坦克移动”的lua代码，加载坦克模型后，使用UpdateBeat注册每帧执行的Update方法，然后在Update方法中调用UnityEngine.Input等API实现功能。代码如下：

```lua
    local go; --加载的坦克模型
    
    --主入口函数。从这里开始lua逻辑
    function Main()       
            LuaHelper = LuaFramework.LuaHelper;
            resMgr = LuaHelper.GetResManager();
            resMgr:LoadPrefab('tank', { 'TankPrefab' }, OnLoadFinish);
    end
     
    --加载完成后的回调--
    function OnLoadFinish(objs)
            go = UnityEngine.GameObject.Instantiate(objs[0]);
            LuaFramework.Util.Log("LoadFinish");
            
            UpdateBeat:Add(Update, self)
    end
     
    --每帧执行
    function Update()
            LuaFramework.Util.Log("每帧执行");
            
            local Input = UnityEngine.Input;
            local horizontal = Input.GetAxis("Horizontal");
            local verticla = Input.GetAxis("Vertical");
            
            local x = go.transform.position.x + horizontal
            local z = go.transform.position.z + verticla
            go.transform.position = Vector3.New(x,0,z)
    end
```

运行游戏，即可用键盘的控制坦克移动。

&emsp;


####二、自定义API

