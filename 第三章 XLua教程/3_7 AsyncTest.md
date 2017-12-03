##AsyncTest

####AsyncTest.cs

```csharp
    using UnityEngine;
    using XLua;
    using System.Collections.Generic;
    using System;
    
    public class AsyncTest : MonoBehaviour
    {
        LuaEnv luaenv = null;
        
        void Start()
        {
            luaenv = new LuaEnv();
            luaenv.DoString("require 'async_test'");
        }
    
        // Update is called once per frame
        void Update()
        {
            if (luaenv != null)
            {
                luaenv.Tick();
            }
        }
    }
```

####async_test.lua.txt

```lua
    local util = require 'xlua.util'
    local message_box = require 'message_box'
    
    -------------------------async_recharge-----------------------------
    local function async_recharge(num, cb) --模拟的异步充值
        print('requst server...')
        cb(true, num)
    end
    
    local recharge = util.async_to_sync(async_recharge)
    -------------------------async_recharge end----------------------------
    local buy = function()
        message_box.alert("余额提醒","您余额不足，请充值！")
    	if message_box.confirm("确认充值10元吗？","确认框" ) then
    		local r1, r2 = recharge(10)
    		print('recharge result', r1, r2)
    		message_box.alert("提示","充值成功！")
    	else
    	    print('cancel')
    	    message_box.alert("提示","取消充值！")
    	end
    end
    --将按钮监听点击事件，绑定buy方法
    CS.UnityEngine.GameObject.Find("Button"):GetComponent("Button").onClick:AddListener(util.coroutine_call(buy))
```