##调用事件

**CallCSEvent.lua.txt**

```lua
    local callEvent = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.CallEvent))
    
    -- 注册静态方法
    callEvent:onClick('+', CS.shenjun.CallEvent.StaticTest)
    callEvent:onValueChanged('+', CS.shenjun.CallEvent.StaticTest)
    
    -- 取消注册的静态方法
    callEvent:onClick('-', CS.shenjun.CallEvent.StaticTest)
    callEvent:onValueChanged('-', CS.shenjun.CallEvent.StaticTest)
    
    -- 注册一个实例方法 错误
    -- callEvent:onClick('+', callEvent.Test)
    
    function test1()
        print('lua no param')
    end
    
    function test2(n)
        print('lua have param :', n)
    end
    
    -- 注册lua方法
    callEvent:onClick('+', test1)
    callEvent:onValueChanged('+', test2)
    
    -- 调用事件执行
    callEvent:Invoke(10)
```


**CallCSEvent.cs**

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
        public class CallEvent : MonoBehaviour {
    
            public event System.Action onClick;
    
            [CSharpCallLua]
            public delegate void Del(int n);
            public event Del onValueChanged;
    
            void Start () {
    
                LuaEnv luaEnv = new LuaEnv();
                luaEnv.DoString("require 'CallCSEvent'");
    
                onClick = null;
                onValueChanged = null;
                luaEnv.Dispose();
            }
    
            void Update () {
                
            }
    
            public void Test()
            {
                Debug.Log("CS Func");
            }
    
            public static void StaticTest(int value = 0)
            {
                Debug.Log("CS Static Func :" + value);
            }
    
            public void Invoke(int value = 0)
            {
                if(onClick != null)
                {
                    onClick.Invoke();
                }
    
                if(onValueChanged != null)
                {
                    onValueChanged(value);
                }
            }
        }
    }
```

🔚