##Coroutine

####CoroutineTest.cs

```csharp
    using UnityEngine;
    using XLua;
    
    public class CoroutineTest : MonoBehaviour {
        LuaEnv luaenv = null;
        // Use this for initialization
        void Start()
        {
            luaenv = new LuaEnv();
            luaenv.DoString("require 'coruntine_test'");
        }
    
        // Update is called once per frame
        void Update()
        {
            if (luaenv != null)
            {
                luaenv.Tick();
            }
        }
    
        void OnDestroy()
        {
            luaenv.Dispose();
        }
    }
```

####coroutine_test.lua.txt

```lua
    local util = require 'xlua.util'
    
    local yield_return = (require 'cs_coroutine').yield_return
    
    local co = coroutine.create(function()
        print('coroutine start!')
        local s = os.time()
        yield_return(CS.UnityEngine.WaitForSeconds(3))
        print('wait interval:', os.time() - s)
        
        local www = CS.UnityEngine.WWW('http://www.qq.com')
        yield_return(www)
    	if not www.error then
            print(www.bytes)
    	else
    	    print('error:', www.error)
    	end
    end)
    
    assert(coroutine.resume(co))
```

####cs_coroutine.lua.txt
```lua
    local util = require 'xlua.util'
    
    local gameobject = CS.UnityEngine.GameObject('Coroutine_Runner')
    CS.UnityEngine.Object.DontDestroyOnLoad(gameobject)
    local cs_coroutine_runner = gameobject:AddComponent(typeof(CS.Coroutine_Runner))
    
    local function async_yield_return(to_yield, cb)
        cs_coroutine_runner:YieldAndCallback(to_yield, cb)
    end
    
    return {
        yield_return = util.async_to_sync(async_yield_return)
    }
```

####Coroutine_Runner.cs
```csharp
    using UnityEngine;
    using XLua;
    using System.Collections.Generic;
    using System.Collections;
    using System;
    
    [LuaCallCSharp]
    public class Coroutine_Runner : MonoBehaviour
    {
        public void YieldAndCallback(object to_yield, Action callback)
        {
            StartCoroutine(CoBody(to_yield, callback));
        }
    
        private IEnumerator CoBody(object to_yield, Action callback)
        {
            if (to_yield is IEnumerator)
                yield return StartCoroutine((IEnumerator)to_yield);
            else
                yield return to_yield;
            callback();
        }
    }
    
    public static class CoroutineConfig
    {
        [LuaCallCSharp]
        public static List<Type> LuaCallCSharp
        {
            get
            {
                return new List<Type>()
                {
                    typeof(WaitForSeconds),
                    typeof(WWW)
                };
            }
        }
    }
```

🔚