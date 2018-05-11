##调用枚举类型

**CallCSEnum.lua.txt**

```lua
    local callEnum = CS.UnityEngine.Object.FindObjectOfType(typeof(CS.shenjun.CallEnum))
    callEnum:EnemyAttack(callEnum.myType)
    
    --直接访问枚举
    local boss = CS.shenjun.EnemyType.Boss
    print("===========", boss)  -- Boss : 1
    callEnum:EnemyAttack(2)
    
    --把字符串转换成枚举
    local npc = CS.shenjun.EnemyType.__CastFrom('NPC')
    print("===========", npc)
    callEnum:EnemyAttack(npc)
    
    --把数字转换成枚举
    local mon = CS.shenjun.EnemyType.__CastFrom(0)
    print("===========", mon)
    callEnum:EnemyAttack(mon)
```