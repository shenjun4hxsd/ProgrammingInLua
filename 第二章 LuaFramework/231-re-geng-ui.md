##热更 UI

&emsp;

界面系统在游戏中占据重要地位。游戏界面是否友好，很大程度上决定了玩家的体验；界面开发是否便利，也影响着游戏的开发进度。Unity3D 的UGUI系统，使用户可以“可视化地”开发界面，那么怎样用Lua去调用UGUI呢？

&emsp;

####一、显示 UI 界面

下面演示如何显示一个UI界面。由于UI界面也是一种资源，使用第二篇“资源热更新”的方法即可。这个例子中，制作一个含有按钮的界面，然后组成名为Panel1的UI预设，存放到Tank目录下。

前面（第二篇）已在Packager类HandleExampleBundle方法中添加了一句“AddBuildMap("tank" + AppConst.ExtName, "*.prefab", "Assets/Tank");”（当然也可以添加到其他地方），它会把Tank目录下的所有预设打包成名为tank的资源包。故而点击“Build xxx Resource”后，Panel1也会被打包到tank资源包中。

修改Lua入口函数Main.lua中的Main方法，在加载资源后把panel1放到Canvas下（需要在场景中添加画布），然后调整它的位置和大小。

```lua
    --主入口函数。从这里开始lua逻辑
    function Main()
        LuaHelper = LuaFramework.LuaHelper;
        resMgr = LuaHelper.GetResManager();
        resMgr:LoadPrefab('tank', { 'Panel1' }, OnLoadFinish);
    end

    --加载完成后的回调--
    function OnLoadFinish(objs)
        --显示面板
        go = UnityEngine.GameObject.Instantiate(objs[0]);
        local parent = UnityEngine.GameObject.Find("Canvas")
        go.transform:SetParent(parent.transform);
        go.transform.localScale = Vector3.one;
        go.transform.localPosition = Vector3.zero;
    end
```