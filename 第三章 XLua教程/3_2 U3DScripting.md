## U3DScripting


####LuaBehaviour.cs

```csharp
    using UnityEngine;
    using System.Collections;
    using System.Collections.Generic;
    using XLua;
    using System;

    [System.Serializable]
    public class Injection
    {
        public string name;
        public GameObject value;
    }

    [LuaCallCSharp]
    public class LuaBehaviour : MonoBehaviour {
        public TextAsset luaScript;
        public Injection[] injections;
        
        //all lua behaviour shared one luaenv only!
        internal static LuaEnv luaEnv = new LuaEnv();
        internal static float lastGCTime = 0;
        internal const float GCInterval = 1;//1 second 

        private Action luaStart;
        private Action luaUpdate;
        private Action luaOnDestroy;

        private LuaTable scriptEnv;

        void Awake()
        {
            scriptEnv = luaEnv.NewTable();// luaEnv.NewTable()构造一个table

            LuaTable meta = luaEnv.NewTable();
            // meta 添加键值对: meta = { __Index = luaEnv.Global }
            meta.Set("__index", luaEnv.Global);
            scriptEnv.SetMetaTable(meta);
            // 销毁meta
            meta.Dispose();

            // 将当前对象保存到table中： scriptEnv = { self = this }
            scriptEnv.Set("self", this);
            // 将植入的对象保存到table中
            foreach (var injection in injections)
            {
                scriptEnv.Set(injection.name, injection.value);
            }

            // 第一个参数：执行的lua字符串
            // 第二个参数：发生error时的debug显示信息中使用，指明某某代码块的某行错误；
            // 第三个参数：为这个代码块；
            luaEnv.DoString(luaScript.text, "LuaBehaviour", scriptEnv);

            Action luaAwake = scriptEnv.Get<Action>("awake");
            scriptEnv.Get("start", out luaStart);
            scriptEnv.Get("update", out luaUpdate);
            scriptEnv.Get("ondestroy", out luaOnDestroy);

            if (luaAwake != null)
            {
                luaAwake();
            }
        }

        void Start ()
        {
            if (luaStart != null)
            {
                luaStart();
            }
        }

        void Update ()
        {
            if (luaUpdate != null)
            {
                luaUpdate();
            }
            if (Time.time - LuaBehaviour.lastGCTime > GCInterval)
            {
                luaEnv.Tick();
                LuaBehaviour.lastGCTime = Time.time;
            }
        }

        void OnDestroy()
        {
            if (luaOnDestroy != null)
            {
                luaOnDestroy();
            }
            luaOnDestroy = null;
            luaUpdate = null;
            luaStart = null;
            scriptEnv.Dispose();
            injections = null;
        }
    }
```

&emsp;

#### LuaTestScript.lua.txt

```lua
    local speed = 10

    function start()
        print("lua start...")
    end

    function update()
        local r = CS.UnityEngine.Vector3.up * CS.UnityEngine.Time.deltaTime * speed
        self.transform:Rotate(r)
    end

    function ondestroy()
        print("lua destroy")
    end
```


🔚
