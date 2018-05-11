##调用委托

**CallCSDelegate.lua.txt**

```lua
    local callDel = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.CallDelegate))
    
    -- 对委托进行初始化赋值
    callDel.del = callDel.del2
    
    -- 对委托添加成员方法 错误
    -- callDel.del = callDel.Test
    
    -- 在使用前定义
    function Test()
        print("Lua Func")
    end
    
    -- 右操作数可以是同类型的C# delegate/静态方法或者是lua函数。
    callDel.del = callDel.del + Test
    
    -- 删除一个委托
    --callDel.del = callDel.del - callDel.del2
    
    -- 判断委托是否为空
    if nil ~= callDel.del then
        -- 调用委托
        callDel:del()
    end
    
    --把委托变为空
    callDel.del = nil
```

**CallCSDelegate.cs**

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
        public class CallDelegate : MonoBehaviour {
    
            public System.Action del = null;
    
            public System.Action del2 = null;
    
    		LuaEnv luaEnv = new LuaEnv();
            void Start () {
                del2 = Test;
                luaEnv.DoString("require 'CallCSDelegate'");
            }
    
            void Update () {
                
            }
    
            private void OnDestroy()
            {
                del = null;
                del2 = null;
                luaEnv.Dispose();
            }
    
            public void Test()
            {
                Debug.Log("CS Func");
            }
        }
    }
```

🔚