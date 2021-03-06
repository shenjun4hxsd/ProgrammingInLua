##编写Lua逻辑

&emsp;

为实现代码热更新，在Unity3D中使用lua，然而为此也需付出不少代价。其一，使代码结构混乱（尽管可以优化），其二降低了运行速度，其三增加学习成本（还要多学一门语言）。为了热更新，所有的逻辑都要用lua编写，那么怎样用lua编写游戏逻辑呢？

&emsp;

####一、Lua 的 Update 方法

第一篇“代码热更新”演示了用lua打印HelloWorld的方法，第二篇“资源热更新”演示了加载坦克模型的方法。这一篇要把两者结合起来，用lua实现“用键盘控制坦克移动”的功能。用Lua和用c#编写的Unity3D程序大同小异，只需正确使用API即可。

**1）、Update 方法**

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

**2）、控制坦克**

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

框架中提供了数十个可供lua调用的c#类，但这些往往不够用，需要自己添加，本节将介绍添加自定义API的方法。

**1）、编写C#类**

例如，编写TestLuaFun.类，它包含一个静态方法Log，会打印出两行文本。

```csharp
    using UnityEngine;
    using System.Collections;
 
    public class TestLuaFun 
    {
        public static void Log() 
        {
            Debug.Log("《Unity3D网络游戏实战》是一本好书！");
            Debug.Log("《手把手教你用c#制作rpg游戏》也是一本好书！");
        }
    }
```

**2）、修改CustomSetting**

打开CustomSetting.cs，在customTypeList中添加一句“_GT(typeof(TestLuaFun))”。

```csharp
    //在这里添加你要导出注册到lua的类型列表
    public static BindType[] customTypeList =
    {
        // ...
        
        _GT(typeof(GameManager)),
        _GT(typeof(LuaManager)),
        _GT(typeof(PanelManager)),
        _GT(typeof(SoundManager)),
        _GT(typeof(TimerManager)),
        _GT(typeof(ThreadManager)),
        _GT(typeof(NetworkManager)),
        _GT(typeof(ResourceManager)),

        // 在这里添加导出注册到lua的类型类标
        _GT(typeof(TestLuaFunc)),
    }
```

**3）、生成wrap文件**

点击菜单栏的Lua→Clear wrap files和Lua→Generate All，重新生成wrap文件。由于刚刚在customTypeList添加了类，所以会生成TestLuaFun类的wrap文件TestLuaFunWrap.cs。


打开TestLuaFunWrap.cs，可以看到TestLuaFun注册了Log方法。

```csharp
    //this source code was auto-generated by tolua#, do not modify it
    using System;
    using LuaInterface;

    public class TestLuaFuncWrap
    {
        public static void Register(LuaState L)
        {
            L.BeginClass(typeof(TestLuaFunc), typeof(System.Object));
            L.RegFunction("Log", Log);
            L.RegFunction("New", _CreateTestLuaFunc);
            L.RegFunction("__tostring", ToLua.op_ToString);
            L.EndClass();
        }

        [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
        static int _CreateTestLuaFunc(IntPtr L)
        {
            try
            {
                int count = LuaDLL.lua_gettop(L);

                if (count == 0)
                {
                    TestLuaFunc obj = new TestLuaFunc();
                    ToLua.PushObject(L, obj);
                    return 1;
                }
                else
                {
                    return LuaDLL.luaL_throw(L, "invalid arguments to ctor method: TestLuaFunc.New");
                }
            }
            catch (Exception e)
            {
                return LuaDLL.toluaL_exception(L, e);
            }
        }

        [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
        static int Log(IntPtr L)
        {
            try
            {
                ToLua.CheckArgsCount(L, 0);
                TestLuaFunc.Log();
                return 0;
            }
            catch (Exception e)
            {
                return LuaDLL.toluaL_exception(L, e);
            }
        }
    }
```

4）、测试API


修改main.lua，调用TestLuaFun.Log() ，即可看到效果。

```lua
    --主入口函数。从这里开始lua逻辑
    function Main()                                 
        TestLuaFun.Log()  
    end
```

&emsp;

####三、原理

tolua实现了LuaInterface，抛开luaFramework，只需创建lua虚拟机，便能在c#中调用lua代码，如下所示。

```csharp
    using UnityEngine;
    using System.Collections;
    using LuaInterface;
 
    public class test : MonoBehaviour
    {
        void Start () 
        {
            //初始化
            LuaState lua = new LuaState();
            LuaBinder.Bind(lua);
            //lua代码
            string luaStr =
                @"                
                    print('hello tolua#, 广告招租')
                    LuaFramework.Util.Log('HelloWorld');
                    TestLuaFun.Log()                
                ";
            //执行lua脚本
            lua.DoString(luaStr);
        }
    } 
```

实际上LuaFramework也是用了相似的方法，框架启动后，会创建LuaManager、LuaLooper的实例。LuaManager创建lua虚拟机并调用Main.lua的Main方法，LuaLooper处理了UpdateBeat相关的事情。如下所示：

```csharp
    private LuaState lua;
    private LuaLoader loader;
    private LuaLooper loop = null;
    
    void Awake() {
        loader = new LuaLoader();
        lua = new LuaState();
        this.OpenLibs();
        lua.LuaSetTop(0);

        LuaBinder.Bind(lua);
        DelegateFactory.Init();
        LuaCoroutine.Register(lua, this);
    }

    public void InitStart() {
        InitLuaPath();
        InitLuaBundle();
        this.lua.Start();    //启动LUAVM
        this.StartMain();
        this.StartLooper();
    }
    
    void StartMain() {
        lua.DoFile("Main.lua");

        LuaFunction main = lua.GetFunction("Main");
        main.Call();
        main.Dispose();
        main = null;    
    }
    
    void StartLooper() {
        loop = gameObject.AddComponent<LuaLooper>();
        loop.luaState = lua;
    }
```

🔚