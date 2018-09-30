##调用泛型方法

**Generic.lua.txt**

```lua
    local obj = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.Generic))
    local cam = obj:GetCamera()
    local cam1 = obj:GetComponent();
    
    if cam1 == cam then
        print("the same camera")
    end
```

**Generic.cs**

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
        public sealed class Generic : MonoBehaviour {
    
    
            void Start () {
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'Generic'");
                luaEnv.Dispose();
            }
    
            public Camera GetCamera()
            {
                return this.GetComponent<Camera>();
            }
        }
    
        [LuaCallCSharp]
        public static class GenericExtension
        {
            public static Camera GetComponent(this Generic self)
            {
                return self.GetComponent<Camera>();
            }
        }
    }
```

🔚