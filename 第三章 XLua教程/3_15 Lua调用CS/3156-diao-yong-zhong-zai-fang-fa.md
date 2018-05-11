##调用重载方法

**OverloadMethod.lua.txt**

```lua
    local overloadMethod = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.OverLoadMethod))
    
    --调用无参的方法
    overloadMethod:Func()
    --调用参数为int类型的方法
    overloadMethod:Func(10)
    --调用参数为string类型的方法
    overloadMethod:Func("shenjun")
```

**OverloadMethod.cs**


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
        public class OverLoadMethod : MonoBehaviour {
    
            void Start () {
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'OverloadMethod'");
                luaEnv.Dispose();
            }
    
            void Update () {
                
            }
    
            public void Func()
            {
                Debug.Log("Func NoParam");
            }
    
            public void Func(int num)
            {
                Debug.Log("Func int param :" + num);
            }
    
            public void Func(string name)
            {
                Debug.Log("Func string param :" + name);
            }
        }
    }
```