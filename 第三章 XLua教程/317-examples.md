##UI事件

**Editor目录下MyGenericTypesStaticList.cs**

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using System;
    using System.Linq;
    using System.Reflection;
    using XLua;
    
    [LuaCallCSharp]
    public static class MyGenericTypeStaticList
    {
    
        [CSharpCallLua]
        public static List<Type> mymodule_lua_call_cs_list = new List<Type>()
            {
                typeof(UnityEngine.EventSystems.IPointerDownHandler),
            };
    
        [Hotfix]
        public static List<Type> by_property
        {
            get
            {
                return (from type in Assembly.GetExecutingAssembly().GetTypes()
                        where type.Namespace == "UnityEngine.EventSystems"
                        select type).ToList();
            }
        }
    
    }
```

**Resources目录下LuaUIEventLocal.lua.txt**

```lua
    LuaPanel = {}
    local this = LuaPanel
    
    function LuaPanel.Awake(self, csObj)
        print("LuaPanel Awake")
        this.self = csObj
        this.Button = this.self.transform:GetComponentInChildren(typeof(CS.UnityEngine.UI.Button));
        this.Button.onClick:AddListener(this.OnBtnClick)
    
        this.Text = this.self.transform:Find("Text"):GetComponent("Text");
        this.Text.text = "LuaPanel"
    end
    
    function LuaPanel.OnDestroy()
        print("LuaPanel OnDestroy")
        this.Button.onClick:RemoveListener(this.OnBtnClick)
        this.Button.onClick:Invoke()
    end
    
    function LuaPanel.OnBtnClick()
        print("LuaPanel OnBtnClick")
        this.Text.text = "Button Clicked"
    end
    
    function LuaPanel.OnPanelClick(eventData)
        print("LuaPanel OnPanelClick")
        this.Text.text = "LuaPanel"
    end
    
    function LuaPanel.OnPointerDown(eventData)
        this.OnPanelClick(eventData)
    end
```

**Resources目录下UIEventGlobal.lua.txt**

```lua
    this = {}
    
    function Awake(luaEvent)
        this.self = luaEvent
        this.Button = this.self.transform:GetComponentInChildren(typeof(CS.UnityEngine.UI.Button))
        this.Button.onClick:AddListener(OnBtnClick)
        this.Text = this.self.transform:Find("Text"):GetComponent("Text")
        print("Lua Awake!")
    end
    
    function Start()
        print("Lua Start!")
    end
    
    function Update()
        print("Lua Update!")
    end
    
    function OnDestroy()
        this.Button.onClick:RemoveListener(OnBtnClick)
        this.Button.onClick:Invoke()
        print("Lua OnDestroy!")
    end
    
    function OnBtnClick()
        this.Text.text = "Lua OnBtnClick"
        print("Lua OnBtnClick!")
    end
    
    function OnPanelClick(eventData)
        this.Text.text = "Lua OnPanelClick"
        print("Lua OnPanelClick!")
    end
    
    function OnPointerDown(eventData)
        OnPanelClick(eventData)
    end
```

**LuaUIEventGlobal.cs**

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using UnityEngine.EventSystems;
    using System;
    using XLua;
    
    namespace shenjun
    {
        [CSharpCallLua]
        public delegate void AwakeSet(LuaUIEventGlobal self);
    
        [CSharpCallLua]
        public delegate void OnPanelClick(PointerEventData eventData);
    
        public class LuaUIEventGlobal : MonoBehaviour, IPointerDownHandler {
    
            LuaEnv luaEnv = new LuaEnv();
    
            AwakeSet luaAwake;
            Action luaStart;
            Action luaUpdate;
            Action luaOnDestroy;
    
    
            OnPanelClick onPanelClickDel;
    
            private void Awake()
            {
                luaEnv.DoString("require 'UIEventGlobal'");
    
                luaAwake = luaEnv.Global.Get<AwakeSet>("Awake");
                luaAwake(this);
    
                luaStart = luaEnv.Global.Get<Action>("Start");
                luaUpdate = luaEnv.Global.Get<Action>("Update");
                luaOnDestroy = luaEnv.Global.Get<Action>("OnDestroy");
    
                onPanelClickDel = luaEnv.Global.Get<OnPanelClick>("OnPanelClick");
            }
    
            void Start () {
                luaStart();
            }
    
            void Update () {
                luaUpdate();
            }
    
            private void OnDestroy()
            {
                luaAwake = null;
                luaStart = null;
                luaUpdate = null;
                luaOnDestroy();
    			luaOnDestroy = null;
                onPanelClickDel = null;
    
                luaEnv.Dispose();
            }
    
            public void OnPointerDown(PointerEventData eventData)
            {
                if (onPanelClickDel != null)
                onPanelClickDel(eventData);
    
            }
        }
    }
```

**LuaUIEventLocal.cs**


```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using UnityEngine.EventSystems;
    using XLua;
    
    namespace shenjun
    {
        //[CSharpCallLua]
        //public delegate void OnPanelClickDel(PointerEventData eventData);
    
        [LuaCallCSharp]
        public class LuaUIEventLocal : MonoBehaviour, IPointerDownHandler {
    
            LuaEnv luaEnv = new LuaEnv();
            ILuaPanel panel;
    
            IPointerDownHandler panelPointDown;
    
            public void OnPointerDown(PointerEventData eventData)
            {
                //if (panel.OnPanelClick != null)
                //panel.OnPanelClick(eventData);
    
                if (panelPointDown != null)
                    panelPointDown.OnPointerDown(eventData);
            }
    
            void Awake()
            {
                luaEnv.DoString("require 'LuaUIEventLocal'");
    
                panel = luaEnv.Global.Get<ILuaPanel>("LuaPanel");
                panelPointDown = luaEnv.Global.Get<IPointerDownHandler>("LuaPanel");
                //panel.self = this; // lua中也会有self的键值
    
                panel.Awake(this);
            }
    
            void Start () {
                
            }
    
            void Update () {
    
                if(luaEnv != null)
                {
                    luaEnv.Tick();
                }
            }
    
            void OnDestroy()
            {
                panel.OnDestroy();
                //if(panel.OnPanelClick != null)
                //{
                //    panel.OnPanelClick = null;
                //}
    
                if (panelPointDown != null)
                    panelPointDown = null;
                luaEnv.Dispose();
            }
        }
    
        [CSharpCallLua]
        interface ILuaPanel
        {
            //LuaUIEventLocal self { get; set; }
            //OnPanelClickDel OnPanelClick { get; set; }
            void Awake(LuaUIEventLocal self);
            void OnDestroy();
        }
    
    }
```


🔚