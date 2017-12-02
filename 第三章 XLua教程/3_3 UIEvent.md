##UIEvent

####ButtonInteraction.lua.txt

通过`LuaBehaviour`执行此`Lua`脚本，并通过Injections数组中添加一组`input`

	Name:input
	Value:InputField

```lua
	function start()
		print("lua start...")
	
		self:GetComponent("Button").onClick:AddListener(function()
			print("clicked, you input is '" ..input:GetComponent("InputField").text .."'")
		end)
	end
```

🔚