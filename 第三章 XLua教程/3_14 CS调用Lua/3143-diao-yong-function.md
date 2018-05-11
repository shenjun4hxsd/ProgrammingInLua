##调用Function

FunctionLua.lua.txt

```lua
    function NoParam()
        print('LuaFunction NoParam')
    end
    
    function HaveParam(a, b)
        print('LuaFunction HaveParam : ', 'a: ', a, 'b: ', b)
    end
    
    function HaveMultiParam(...)
        print(...)
    end
            
    function ReturnString()
        print('LuaFunction ReturnString')
        return "i am the return string!"
    end
    
    function ReturnFunction()
        print('LuaFunction ReturnFunction')
        return NoParam
    end
    
    function ReturnMultiValue()
    	print("LuaFunction ReturnMultiValue")
    	return true, "ReturnMultiValue", 1, "num"
    end
```

####一、映射到Delegate

