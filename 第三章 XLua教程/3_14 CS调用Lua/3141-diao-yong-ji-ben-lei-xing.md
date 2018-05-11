##一、调用Lua基本类型

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    
    using XLua;
    
    namespace shenjun
    {
    	public class CSCallLuaBasicType : MonoBehaviour {
    
            LuaEnv luaEnv = new LuaEnv();
    
    		void Start () {
    
                luaEnv.DoString("require 'BasicLua'");
    
                int a = luaEnv.Global.Get<int>("a");
                //int a;
                //luaEnv.Global.Get("a", out a);
    
                float b = luaEnv.Global.Get<float>("b");
    
                string c = luaEnv.Global.Get<string>("c");
    
                bool d = luaEnv.Global.Get<bool>("d");
    
                string n = luaEnv.Global.GetInPath<string>("e.f.name");
    
                Debug.Log(string.Format("a :{0}, b :{1}, c :{2}, d :{3}, name :{4}", a, b, c, d, n));
            }
    		
    		void Update () {
                if(luaEnv != null)
                {
                    luaEnv.Tick();
                }
    		}
    
            void OnDestroy()
            {
                luaEnv.Dispose();
            }
        }
    }
```

**BasicLua.lua.txt**

```lua
    a = 1
    b = 1.5
    c = 'hello world'
    d = true
    
    e = {
        ["f"] = { ["name"] = "shenjun" },
        "unity"
    }
    
    --f = { 2, 3 }
```

🔚