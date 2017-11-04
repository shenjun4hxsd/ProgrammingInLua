####HelloWorld.cs

```csharp
    using UnityEngine;
    using LuaInterface;
    using System;
    
    public class HelloWorld : MonoBehaviour
    {
        /*
            这个类及其的简单，就是教你怎么启动luastate，然后将字符串lua代码执行进lua虚拟机里面，
            因为并没有作用域，所以当DoString的时候，就直接打印出字符串来了。然后将luastate析构掉，置空完毕。
        */
        void Awake()
        {
            LuaState lua = new LuaState();
            lua.Start();
            string hello =
                @"                
                    print('hello tolua#')                                  
                ";
            
            lua.DoString(hello, "HelloWorld.cs");
            lua.CheckTop();
            lua.Dispose();
            lua = null;
        }
    }
```

🔚