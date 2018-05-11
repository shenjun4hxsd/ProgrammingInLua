##Params参数

**Params.lua.txt**

```lua
   local params = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.Params))
   local result = params:Split(params.msg, ' ', '#')
   
   local t = {}
   for i = 1, result.Length do
      t[#t+1] = result[i-1]
   end
   
   for i,v in ipairs(t) do
       print(v)
   end
```