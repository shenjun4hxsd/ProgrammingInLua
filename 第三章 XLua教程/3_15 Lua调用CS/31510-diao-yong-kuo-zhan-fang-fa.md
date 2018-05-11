##调用扩展方法

**ExtensionMethod.lua.txt**

```lua
    local obj = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.ExtensionMethod))
    obj:ShowInfo()
```

**ExtensionMethod.cs**


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
        public class ExtensionMethod : MonoBehaviour {
    
            void Start () {
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'ExtensionMethod'");
                luaEnv.Dispose();
            }
    
            void Update () {
                
            }
        }
    
        [LuaCallCSharp]
        public static class ExtensionClass
        {
            public static void ShowInfo(this ExtensionMethod self)
            {
                Debug.Log(self.gameObject.name);
            }
        }
    }
```

🔚