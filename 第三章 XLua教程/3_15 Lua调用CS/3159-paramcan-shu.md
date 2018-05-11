##Params参数

**Params.lua.txt**

```lua
   local params = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.Params))
   local result = params:Split(params.msg, ' ', '#')
   
   local t = {}
   for i = 1, result.Length do
      t[#t+1] = result[i-1]
   end
   
   for i,v in ipairs(t) do
       print(v)
   end
```

**Params.cs**

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
        public class Params : MonoBehaviour
        {
    
            public string msg = "a b C#d";
    
            void Start()
            {
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'Params'");
                luaEnv.Dispose();
            }
    
            void Update()
            {
    
            }
    
            public string[] Split(string s, params string[] chs)
            {
                return s.Split(chs, System.StringSplitOptions.RemoveEmptyEntries);
            }
        }
    }
```

🔚