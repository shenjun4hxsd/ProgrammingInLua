##使用运算符重载

**LuaCallCSOperator.lua.txt**

```lua
    local operator = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.LuaCallCSOperator))
    local Quaternion = CS.UnityEngine.Quaternion
    local Vector3 = CS.UnityEngine.Vector3
    operator.transform.rotation = operator.transform.rotation * Quaternion.AngleAxis(45, Vector3.up)
    
    operator.transform.position = operator.transform.position + Vector3.up * 10
```

**LuaCallCSOperator.cs**

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
        public class LuaCallCSOperator : MonoBehaviour {
    
            LuaEnv luaEnv = new LuaEnv();
    
            void Start () {
                luaEnv.DoString("require 'LuaCallCSOperator'");
            }
    
            void Update () {
                
                if(luaEnv != null)
                {
                    luaEnv.Tick();
                }
            }
    
            private void OnDestroy()
            {
                luaEnv.Dispose();
            }
        }
    }
```

🔚