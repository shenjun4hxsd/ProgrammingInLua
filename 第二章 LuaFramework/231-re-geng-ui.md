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

运行游戏，即可看到加载出来的界面。

&emsp;

####二、事件响应

c#中可以使用事件监听的方法给UI组件添加事件。例如，添加按钮点击事件的方法如下：

```csharp
    Button btn = go.GetComponent ();
    btn.onClick.AddListener(()=>{this.OnClick(go);});
```

然而在LuaFramework的API中，没能找到合适的方法，只能根据第三篇中“自定义API”的方法，自己编写一套了。编写UIEvent类，它包含用于添加监听事件的AdonClick和清除监听事件的ClearButtonClick方法，代码如下所示（完成后记得要“修改CustomSetting”和“生成wrap文件”）。

```csharp
    using UnityEngine;
    using System.Collections;
    using LuaInterface;
    using UnityEngine.UI;

    public class UIEvent 
    {

        //添加监听
        public static void AdonClick(GameObject go, LuaFunction luafunc) 
        {
            if (go == null || luafunc == null) return;
            Button btn = go.GetComponent ();
            if (btn == null) return;
            btn.onClick.AddListener(()=>{luafunc.Call(go);});
        }

        //清除监听
        public static void ClearButtonClick(GameObject go) 
        {
            if (go == null) return;
            Button btn = go.GetComponent<Button> ();
            if (btn == null) return;
            btn.onClick.RemoveAllListeners();
        }
    }
```

接下来测试下这套API，修改Main.lua，代码如下：

```lua
    --主入口函数。从这里开始lua逻辑
    function Main()                                        
        略
    end

    --加载完成后的回调--
    function OnLoadFinish(objs)
        --显示面板
        略
        --事件处理
        local btn = go.transform:FindChild("Button").gameObject
        UIEvent.AdonClick(btn, OnClick)
    end

    function OnClick()

        print("触发按钮事件")

    end
```