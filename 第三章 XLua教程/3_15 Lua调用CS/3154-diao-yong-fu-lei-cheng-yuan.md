##调用父类成员

**CallCSParent.lua.txt**

```lua
    local camera = CS.UnityEngine.Camera.main
    if camera ~= nil then
        print("Find Camera")
    end
    
    local callParent = camera.transform:GetComponent(typeof(CS.shenjun.CallParent))
    if nil ~= callParent then
        print("GetComponent CallParent")
    else
        print("Not Find Component CallParent")
    end
    
    
    --调用父类的静态字段
    print(CS.shenjun.CallParent.index)
    
    --调用父类的非静态字段
    print(callParent.tag)
    
    --调用父类的静态方法
    CS.shenjun.CallParent.SetIndex()
    
    --调用父类的非静态方法
    print(callParent:CompareTag("Player"))
```

**CallParent.cs**

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
        public class CallParent : Parent {
    
    		void Start () {
    
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'CallCSParent'");
                luaEnv.Dispose();
    		}
    	}
    
        public class Parent : MonoBehaviour
        {
            public static int index = 0;
    
            public static void SetIndex()
            {
                index++;
                Debug.Log("index : " + index);
            }
        }
    }
```

🔚