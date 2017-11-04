##LuaFramework

&emsp;


####Lua源码目录介绍：

&emsp;&emsp;**● 3rd**：里面是第三方的一些插件lua、实例源码文件，比如：`cjson`、`pbc`、`pblua`、`sproto`等。

&emsp;&emsp;**● Common**：公用的`lua`文件目录，如`define.lua`文件，一些变量声明，全局配置等，`functions.lua`常用函数库，通讯的`protocal.lua`协议文件。

&emsp;&emsp;**● Controller**：控制器目录，它不依赖于某一个`Lua`面板，它是独立存活在`Luavm`中的一个操作类，操作数据、控制面板显示而已。

&emsp;&emsp;**● Logic**：目录里面存放的是一些管理器类，比如`GameManager`游戏管理器、`NetworkManager`网络管理器，如果你有新的管理器可以放到里面。

&emsp;&emsp;**● System**：这个目录是`cstolua`的系统目录，里面存放都是一些常用的`lua`类，为了优化`lua`调用速度，用`lua`重写的`unity`常用类。

&emsp;&emsp;**● View**：这是面板的视图层，里面都是一些被`Unity`调用的面板的变量，走的是`Unity` `GameObject`的生命周期的事件调用。

####lua和C#的交互两个工具类：
```csharp
    LuaHelper.cs静态类    //lua调用C#
    Util.cs              //静态方法CallMethod C#调用lua
```

&emsp;

####分析图：

![](/assets/20160506154319148))

####配置文件：

![](/assets/20160506154351680)

####启动流程：

![](/assets/20160506154414941)


🔚