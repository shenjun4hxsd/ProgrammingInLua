##创建对象

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using XLua;
    
    namespace shenjun
    {
    	public class NewObj : MonoBehaviour {
    
            LuaEnv luaEnv = new LuaEnv();
    
    		void Start () {
    
                luaEnv.DoString("require 'NewObjFunc'");
               
                GameObject obj1 = CreateObj();
                GameObject obj2 = CreateObj("NewObj");
                GameObject obj3 = CreateObj(obj2.transform);
    		}
    		
    		void Update () {
                if(luaEnv != null)
                {
                    luaEnv.Tick();
                }
    		}
    
            private void OnDestroy()
            {
                luaEnv.Dispose();
            }
    
            [XLua.CSharpCallLua]
            public delegate GameObject CreateObjDel();
            GameObject CreateObj()
            {
                CreateObjDel del = luaEnv.Global.Get<CreateObjDel>("New");
                return del();
            }
    
            [XLua.CSharpCallLua]
            public delegate GameObject CreateObjByNameDel(string name);
            GameObject CreateObj(string name)
            {
                CreateObjByNameDel del = luaEnv.Global.Get<CreateObjByNameDel>("New");
                return del(name);
            }
    
            [CSharpCallLua]
            public delegate GameObject CreateObjToParentDel(Transform parent);
            GameObject CreateObj(Transform parent)
            {
                CreateObjToParentDel del = luaEnv.Global.Get<CreateObjToParentDel>("NewToParent");
                return del(parent);
            }
        }
    }
```