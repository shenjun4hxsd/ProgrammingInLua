##HelloWorld

执行Lua代码
1、引入命名空间 using XLua;

2、创建LuaEnv对象luaenv

3、通过luaenv对象调用DoString方法，并传入Lua代码作为参数

4、CS.UnityEngine.Debug.Log('hello world')是通过Lua调用C#中的Debug.Log方法。


```csharp
    using UnityEngine;
    using XLua;

    public class Helloworld : MonoBehaviour {
        // Use this for initialization
        void Start () {
        LuaEnv luaenv = new LuaEnv();
        luaenv.DoString("CS.UnityEngine.Debug.Log('hello world')");
        luaenv.Dispose();
    }
	
        // Update is called once per frame
        void Update () {
	
        }
    }
```

🔚