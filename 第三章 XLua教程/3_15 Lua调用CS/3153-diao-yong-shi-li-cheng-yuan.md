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

