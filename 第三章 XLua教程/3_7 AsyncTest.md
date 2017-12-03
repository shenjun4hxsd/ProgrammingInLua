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

####message_box.lua.txt

```lua
    local util = require 'xlua.util'
    
    local sync_alert = util.async_to_sync(CS.MessageBox.ShowAlertBox)
    local sync_confirm = util.async_to_sync(CS.MessageBox.ShowConfirmBox) 
    
    --构造alert和confirm函数
    return {
        alert = function(title, message)
    		if not message then
    			title, message = message, title
    		end
    		 sync_alert(message,title)
        end;
    	
    	confirm = function(title, message)
    		local ret = sync_confirm(title,message)
    		return ret == true
        end;
     }
```

####MessageBox.cs

```csharp
    using UnityEngine;
    using UnityEngine.UI;
    using XLua;
    using System.Collections.Generic;
    using System;
    using UnityEngine.Events;
    
    public class MessageBox : MonoBehaviour{
    
        public static void ShowAlertBox(string message, string title, Action onFinished = null)
        {
            var alertPanel = GameObject.Find("Canvas").transform.Find("AlertBox");
            if (alertPanel == null)
            {
                alertPanel = (Instantiate(Resources.Load("AlertBox")) as GameObject).transform;
                alertPanel.gameObject.name = "AlertBox";
                alertPanel.SetParent(GameObject.Find("Canvas").transform);
                alertPanel.localPosition = new Vector3(-6f, -6f, 0f);
            }
            alertPanel.Find("title").GetComponent<Text>().text = title;
            alertPanel.Find("message").GetComponent<Text>().text = message;
            alertPanel.gameObject.SetActive(true);
            if (onFinished != null)
            {
                var button = alertPanel.Find("alertBtn").GetComponent<Button>();
                UnityAction onclick = null;
                onclick = () =>
                {
                    onFinished();
                    alertPanel.gameObject.SetActive(false);
                    button.onClick.RemoveListener(onclick);
                };
                button.onClick.RemoveAllListeners();
                button.onClick.AddListener(onclick);
            }
        }
    
        public  static void ShowConfirmBox(string message, string title, Action<bool> onFinished = null)
        {
            var confirmPanel = GameObject.Find("Canvas").transform.Find("ConfirmBox");
            if (confirmPanel == null)
            {
                confirmPanel = (Instantiate(Resources.Load("ConfirmBox")) as GameObject).transform;
                confirmPanel.gameObject.name = "ConfirmBox";
                confirmPanel.SetParent(GameObject.Find("Canvas").transform);
                confirmPanel.localPosition = new Vector3(-8f, -18f, 0f);
            }
            confirmPanel.Find("confirmTitle").GetComponent<Text>().text = title;
            confirmPanel.Find("conmessage").GetComponent<Text>().text = message;
            confirmPanel.gameObject.SetActive(true);
            if (onFinished != null)
            {
                var confirmBtn = confirmPanel.Find("confirmBtn").GetComponent<Button>();
                var cancelBtn = confirmPanel.Find("cancelBtn").GetComponent<Button>();
                UnityAction onconfirm = null;
                UnityAction oncancel = null;
    
                Action cleanup = () =>
                {
                    confirmBtn.onClick.RemoveListener(onconfirm);
                    cancelBtn.onClick.RemoveListener(oncancel);
                    confirmPanel.gameObject.SetActive(false);
                };
    
                onconfirm = () =>
                {
                    onFinished(true);
                    cleanup();
                };
    
                oncancel = () =>
                {
                    onFinished(false);
                    cleanup();
                };
                confirmBtn.onClick.RemoveAllListeners();
                cancelBtn.onClick.RemoveAllListeners();
                confirmBtn.onClick.AddListener(onconfirm);
                cancelBtn.onClick.AddListener(oncancel);
            }
        }
    }
    
    public static class MessageBoxConfig
    {
        [CSharpCallLua]
        public static List<Type> CSharpCallLua = new List<Type>()
        {
            typeof(Action),
            typeof(Action<bool>),
            typeof(UnityAction),
        };
    }
```

🔚