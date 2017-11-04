####TestString.cs

```csharp
    using UnityEngine;
    using System.Collections;
    using LuaInterface;
    using System;
    using System.Reflection;
    using System.Text;
    
    public class TestString : LuaClient
    {
        string script =
    @"           
        function Test()
            local str = System.String.New('男儿当自强')
            local index = str:IndexOfAny('儿自')
            print('and index is: '..index)
            local buffer = str:ToCharArray()
            print('str type is: '..type(str)..' buffer[0] is ' .. buffer[0])
            local luastr = tolua.tolstring(buffer)
            print('lua string is: '..luastr..' type is: '..type(luastr))
            luastr = tolua.tolstring(str)
            print('lua string is: '..luastr)                    
        end
    ";
    
        protected override LuaFileUtils InitLoader()
        {
            return new LuaResLoader();
        }
    
        //屏蔽，例子不需要运行
        protected override void CallMain() { }
    
        protected override void OnLoadFinished()
        {
            base.OnLoadFinished();
            luaState.DoString(script);
            LuaFunction func = luaState.GetFunction("Test");
            func.Call();
            func.Dispose();
            func = null;
        }
    }
```

🔚