##HelloWorld

**执行Lua代码**

1、引入命名空间 `using XLua;`

2、创建`LuaEnv`实例`luaenv`

3、通过`luaenv`实例调用`DoString`方法，并传入`Lua`代码作为参数

4、`CS.UnityEngine.Debug.Log('hello world')`是通过`Lua`调用`C#`中的`Debug.Log`方法。

5、销毁`luaenv`对象 `luaenv.Dispose()`

```csharp
    using UnityEngine;
    using XLua;

    public class Helloworld : MonoBehaviour {
        
        void Start () {
            LuaEnv luaenv = new LuaEnv();
            luaenv.DoString("CS.UnityEngine.Debug.Log('hello world')");
            
            // DoString返回将包含两个object， 一个是double类型的1， 一个是string类型的“hello”
            // object[] ret = luaenv.DoString("print(‘hello’)\r\nreturn 1")
            luaenv.Dispose();
        }
    }
```

🔚