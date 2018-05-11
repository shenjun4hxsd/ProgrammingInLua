##调用有默认参数方法

**DefaultValueParam.lua.txt**

```lua
    local default = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.DefaultValueParam))
    
    --调用方法，使用默认参数
    default:DefaultValueMethod()
    --调用方法，不使用默认参数
    default:DefaultValueMethod(100) 
```

**DefaultValueParam.cs**

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
        public class DefaultValueParam : MonoBehaviour {
    
            void Start () {
    
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'DefaultValueParam'");
                luaEnv.Dispose();
            }
    
            void Update () {
                
            }
    
            public void DefaultValueMethod(int num = 0)
            {
                Debug.Log("num :" + num);
            }
        }
    }
```