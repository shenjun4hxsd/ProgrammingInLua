##调用静态成员

**CallStatic.lua.txt**

```lua
    --访问静态属性
    local deltaTime = CS.UnityEngine.Time.deltaTime - CS.UnityEngine.Time.deltaTime%0.001
    print("Time deltaTime :", deltaTime)
    
    --对静态属性进行赋值
    CS.UnityEngine.Time.timeScale = 0.5
    print("Time timeScale :", CS.UnityEngine.Time.timeScale)
    
    --调用静态方法
    local GameObject = CS.UnityEngine.GameObject
    local obj = GameObject.Find('GameObject')
    if nil ~= obj then
        print("Destroy GameObject")
        CS.UnityEngine.Object.Destroy(obj)
    end
```

CallStatic.cs

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
    	public class CallStatic : MonoBehaviour {
    
    		void Start () {
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'CallStatic'");
                luaEnv.Dispose();
    		}
        }
    }
```

🔚