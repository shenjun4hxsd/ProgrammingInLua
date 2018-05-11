##Ref、Out、Param参数

**RefOutParam.lua.txt**

```lua
    local RefOutParam = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.RefOutParam))
    
    -- 1、out参数不需要传入实参
    -- 2、返回值顺序：如果函数有返回值那么第一个是函数返回值
    -- 其他的都是ref，out参数的传出的，按顺序传出
    -- 3、如果函数没有返回值，那么按顺序返回ref，out参数
    local a, b, c = RefOutParam:RefOutParamFunc("shenjun", 10, 30)
    print("Lua Return :" ,a, b, c)
```