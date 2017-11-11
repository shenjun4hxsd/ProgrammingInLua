##编写Lua逻辑

&emsp;

为实现代码热更新，在Unity3D中使用lua，然而为此也需付出不少代价。其一，使代码结构混乱（尽管可以优化），其二降低了运行速度，其三增加学习成本（还要多学一门语言）。为了热更新，所有的逻辑都要用lua编写，那么怎样用lua编写游戏逻辑呢？

&emsp;

####一、Lua 的 Update 方法

第一篇“代码热更新”演示了用lua打印HelloWorld的方法，第二篇“资源热更新”演示了加载坦克模型的方法。这一篇要把两者结合起来，用lua实现“用键盘控制坦克移动”的功能。用Lua和用c#编写的Unity3D程序大同小异，只需正确使用API即可。

1）、Update 方法

出于效率的考虑，tolua提供了名为UpdateBeat的对象，在LuaFramework中，只需给UpdateBeat添加回调函数，该函数便会每帧执行，相当于Monobehaviour的Update方法。Lua代码如下所示：

```lua
    function Main()                                 
        UpdateBeat:Add(Update, self)
    end
  
    function Update()
        LuaFramework.Util.Log("每帧执行一次");
    end
```

function Main()                                 
    UpdateBeat:Add(Update, self)
end
 
function Update()
        LuaFramework.Util.Log("每帧执行一次");
end