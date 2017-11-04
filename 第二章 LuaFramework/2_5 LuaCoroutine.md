##Lua协同

####TestCoroutine.cs

```csharp
    using UnityEngine;
    using System;
    using System.Collections;
    using LuaInterface;
    
    //例子5和6展示的两套协同系统勿交叉使用，此为推荐方案
    public class TestCoroutine : MonoBehaviour 
    {
        public TextAsset luaFile = null;
        private LuaState lua = null;
        private LuaLooper looper = null;
    
    	void Awake () 
        {        
            lua  = new LuaState();
            lua.Start();
            LuaBinder.Bind(lua);                
            looper = gameObject.AddComponent<LuaLooper>();
            looper.luaState = lua;
    
            lua.DoString(luaFile.text, "TestCoroutine.cs");
            LuaFunction f = lua.GetFunction("TestCortinue");
            f.Call();
            f.Dispose();
            f = null;        
        }
    
        void OnDestroy()
        {
            looper.Destroy();
            lua.Dispose();
            lua = null;
        }
    
        void OnGUI()
        {
            if (GUI.Button(new Rect(50, 50, 120, 45), "Start Counter"))
            {
                LuaFunction func = lua.GetFunction("StartDelay");
                func.Call();
                func.Dispose();
            }
            else if (GUI.Button(new Rect(50, 150, 120, 45), "Stop Counter"))
            {
                LuaFunction func = lua.GetFunction("StopDelay");
                func.Call();
                func.Dispose();
            }
            else if (GUI.Button(new Rect(50, 250, 120, 45), "GC"))
            {
                lua.DoString("collectgarbage('collect')", "TestCoroutine.cs");
                Resources.UnloadUnusedAssets();
            }
        }
    }
```

&emsp;


####TestLuaCoroutine.lua.bytes

```lua
    function fib(n)
        local a, b = 0, 1
        while n > 0 do
            a, b = b, a + b
            n = n - 1
        end
    
        return a
    end
    
    function CoFunc()
        print('Coroutine started')
        local i = 0
        for i = 0, 10, 1 do
            print(fib(i))                    
            coroutine.wait(0.1)						
        end	
    	print("current frameCount: "..Time.frameCount)
    	coroutine.step()
    	print("yield frameCount: "..Time.frameCount)
    
    	local www = UnityEngine.WWW("http://www.baidu.com")
    	coroutine.www(www)
    	local s = tolua.tolstring(www.bytes)
    	print(s:sub(1, 128))
        print('Coroutine ended')
    end
    
    function TestCortinue()	
        coroutine.start(CoFunc)
    end
    
    local coDelay = nil
    
    function Delay()
    	local c = 1
    
    	while true do
    		coroutine.wait(1) 
    		print("Count: "..c)
    		c = c + 1
    	end
    end
    
    function StartDelay()
    	coDelay = coroutine.start(Delay)
    end
    
    function StopDelay()
    	coroutine.stop(coDelay)
    end
```

🔚