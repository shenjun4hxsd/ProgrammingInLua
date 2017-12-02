##UIEvent

####ButtonInteraction.lua.txt

```lua
	function start()
		print("lua start...")
	
		self:GetComponent("Button").onClick:AddListener(function()
			print("clicked, you input is '" ..input:GetComponent("InputField").text .."'")
		end)
	end
```