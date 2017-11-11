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