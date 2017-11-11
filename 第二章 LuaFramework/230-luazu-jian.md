##Lua 组件

&emsp;

基于组件的编程模式是Unity3D的核心思想之一，然而使用纯lua编程，基本就破坏了这一模式。那么有没有办法做一些封装，让Lua脚本也能挂载到游戏物体上，作为组件呢？

&emsp;

####一、设计思想

在需要添加Lua组件的游戏物体上添加一个LuaComponent组件，LuaComponent引用一个lua表，这个lua表包含lua组件的各种属性以及Awake、Start等函数，由LuaComponent适时调用Lua表所包含的函数。

```csharp
    /*
     *  created by shenjun
     */

    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using LuaInterface;
    using LuaFramework;

    public class LuaComponent : MonoBehaviour {

        // Lua 表
        public LuaTable table;
    
        // 添加Lua组件
        public static LuaTable Add(GameObject go, LuaTable tableClass)
        {
            LuaFunction fun = tableClass.GetLuaFunction("New");
            if (fun == null) return null;
    
            object[] rets = fun.LazyCall(tableClass);
            if (rets.Length != 1) return null;
    
            LuaComponent cmp = go.AddComponent<LuaComponent>();
            cmp.table = (LuaTable)rets[0];
            cmp.CallAwake();
            return cmp.table;
        }
    
        public static LuaTable Get(GameObject go, LuaTable table)
        {
            LuaComponent[] cmps = go.GetComponents<LuaComponent>();
            foreach (LuaComponent cmp in cmps)
            {
                string mat1 = table.ToString();
                string mat2 = cmp.table.GetMetaTable().ToString();
                if (mat1 == mat2) return cmp.table;
            }
            return null;
        }
    
        void CallAwake()
        {
            LuaFunction fun = table.GetLuaFunction("Awake");
    
            if (fun != null)
                fun.Call(table, gameObject);
        }
    
        void Start () {
            LuaFunction fun = table.GetLuaFunction("Start");
    
            if (fun != null)
                fun.Call(table, gameObject);
        }
    	
        void Update () {
            // 效率问题有待测试和优化
            // 可在lua中调用UpdateBeat替代
            LuaFunction fun = table.GetLuaFunction("Update");
    
            if (fun != null)
                fun.Call(table, gameObject);
    	}
    }
```

下面列举lua组件的文件格式，它包含一个表（如Component），这个表包含property1 、property2 等属性，包含Awake、Start等方法。表中必须包含用于派生对象的New方法，它会创建一个继承自Component的表o，供LuaComponent调用。

```lua
    Component=    --组件表
    ​{
        property1 = 100,
        property2 = “helloWorld”
    }
    
    function Component:Awake() 
        print("TankCmp Awake name = "..self.name );
    end
    
    function Component:Start() 
        print("TankCmp Start name = "..self.name );
    End
    
    --更多方法略
    
    function Component:New(obj) 
        local o = {} 
        setmetatable(o, self)  
        self.__index = self  
        return o
    end  
```

&emsp;

####二、LuaComponent 组件

LuaComponent主要有Get和Add两个静态方法，其中Get相当于UnityEngine中的GetComponent方法，Add相当于AddComponent方法，只不过这里添加的是lua组件不是c#组件。每个LuaComponent拥有一个LuaTable（lua表）类型的变量table，它既引用上述的Component表。

Add方法使用AddComponent添加LuaComponent，调用参数中lua表的New方法，将其返回的表赋予table。

Get方法使用GetComponents获取游戏对象上的所有LuaComponent（一个游戏对象可能包含多个lua组件，由参数table决定需要获取哪一个），通过元表地址找到对应的LuaComponent，返回lua表。

&emsp;

####三、调试LuaComponent

现在编写名为TankCmp的lua组件，测试LuaCompomemt的功能，TankCmp会在Awake、Start和Update打印出属性name。TankCmp.lua的代码如下：

```lua
    TankCmp =
    {
        --里面可以放一些属性
        Hp = 100,
        att = 50,
        name = "good tank",
    }

    function TankCmp:Awake()
        print("TankCmp Awake name = "..self.name );

    end

    function TankCmp:Start()
        print("TankCmp Start name = "..self.name );
    end

    function TankCmp:Update()
        print("TankCmp Update name = "..self.name );
    end

    --创建对象
    function TankCmp:New(obj)
        local o = {}
        setmetatable(o, self)  
        self.__index = self  
        return o
    end  
```

编写Main.lua，给游戏对象添加lua组件。

```lua
    require "TankCmp"

    --主入口函数。从这里开始lua逻辑
    function Main()
        --组件1
        local go = UnityEngine.GameObject ('go')
        local tankCmp1 = LuaComponent.Add(go,TankCmp)
        tankCmp1.name = "Tank1"

        --组件2
        local go2 = UnityEngine.GameObject ('go2')
        LuaComponent.Add(go2,TankCmp)
        local tankCmp2 = LuaComponent.Get(go2,TankCmp)
        tankCmp2.name = "Tank2"
    end
```