##UI事件

**Editor目录下MyGenericTypesStaticList.cs**

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using System;
    using System.Linq;
    using System.Reflection;
    using XLua;
    
    [LuaCallCSharp]
    public static class MyGenericTypeStaticList
    {
    
        [CSharpCallLua]
        public static List<Type> mymodule_lua_call_cs_list = new List<Type>()
            {
                typeof(UnityEngine.EventSystems.IPointerDownHandler),
            };
    
        [Hotfix]
        public static List<Type> by_property
        {
            get
            {
                return (from type in Assembly.GetExecutingAssembly().GetTypes()
                        where type.Namespace == "UnityEngine.EventSystems"
                        select type).ToList();
            }
        }
    
    }
```