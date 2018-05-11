##调用重载方法

**OverloadMethod.lua.txt**

```lua
    local overloadMethod = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.OverLoadMethod))
    
    --调用无参的方法
    overloadMethod:Func()
    --调用参数为int类型的方法
    overloadMethod:Func(10)
    --调用参数为string类型的方法
    overloadMethod:Func("shenjun")
```