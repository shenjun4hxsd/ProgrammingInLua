##调用实例成员

**CallMember.lua.txt**

```lua
    local GameObject = CS.UnityEngine.GameObject
    local obj = GameObject.Find('GameObject')
    
    --访问成员属性
    local transform = obj.transform
    print(transform.position)
    
    --对成员属性进行赋值
    transform.position = CS.UnityEngine.Vector3(1,1,1)
    
    --调用成员方法
    local camera = CS.UnityEngine.Camera.main
    camera.transform:LookAt(transform)
```

**CallMember.cs**

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
    	public class CallMember : MonoBehaviour {
            
            LuaEnv luaEnv = new LuaEnv();
    
    		void Start () {
                luaEnv.DoString("require 'CallMember'");
    
    		}
    
    		void Update () {
                if(luaEnv != null)
                {
                    luaEnv.Tick();
                }
    		}
    
            private void OnDestroy()
            {
                luaEnv.Dispose();
            }
        }
    }
```

