##UIEvent

####ButtonInteraction.lua.txt

通过LuaBehaviour执行此Lua脚本

```lua
	function start()
		print("lua start...")
	
		self:GetComponent("Button").onClick:AddListener(function()
			print("clicked, you input is '" ..input:GetComponent("InputField").text .."'")
		end)
	end
```