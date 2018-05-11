##CS调用Lua

####一、调用Lua基本类型

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
    	public class CSCallLuaBasicType : MonoBehaviour {
    
            LuaEnv luaEnv = new LuaEnv();
    
    		void Start () {
    
                luaEnv.DoString("require 'BasicLua'");
    
                int a = luaEnv.Global.Get<int>("a");
                //int a;
                //luaEnv.Global.Get("a", out a);
    
                float b = luaEnv.Global.Get<float>("b");
    
                string c = luaEnv.Global.Get<string>("c");
    
                bool d = luaEnv.Global.Get<bool>("d");
    
                string n = luaEnv.Global.GetInPath<string>("e.f.name");
    
                Debug.Log(string.Format("a :{0}, b :{1}, c :{2}, d :{3}, name :{4}", a, b, c, d, n));
            }
    		
    		void Update () {
                if(luaEnv != null)
                {
                    luaEnv.Tick();
                }
    		}
    
            void OnDestroy()
            {
                luaEnv.Dispose();
            }
        }
    }
```

**BasicLua.lua.txt**

```lua
    a = 1
    b = 1.5
    c = 'hello world'
    d = true
    
    e = {
        ["f"] = { ["name"] = "shenjun" },
        "unity"
    }
    
    --f = { 2, 3 }
```

####二、调用Lua Table类型

#####1)、映射到class和struct

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
    	public class TableToClass : MonoBehaviour {
    
            LuaEnv luaEnv = new LuaEnv();
    
    		void Start () {
    
                luaEnv.DoString("require 'TableLua'");
    
                // 对应public字段，table的属性可以多于或少于class的字段，没有对应的使用该类型的默认值
                Student xm = luaEnv.Global.Get<Student>("student");
                Debug.Log(xm);
    			
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
    
            class Student
            {
                public string name;
                public int age;
                public string Sex { get; set; } // 无法对应table中的键
    
                public override string ToString()
                {
                    return string.Format("name : {0}, age : {1}, sex : {2}", name, age, Sex);
                }
    
                //public string get_Sex()
                //{
                //    return "";
                //}
            }
        }
    }
```

---

```csharp
    using UnityEngine;
    using System.Collections;
    using System.Collections.Generic;
    using XLua;
    using System;
    
    public class CSCallLua : MonoBehaviour {
        LuaEnv luaenv = null;
        string script = @"
            a = 1
            b = 'hello world'
            c = true
    
            d = {
               f1 = 12, f2 = 34, 
               1, 2, 3,
               add = function(self, a, b) 
                  print('d.add called')
                  return a + b 
               end
            }
    
            function e()
                print('i am e')
            end
    
            function f(a, b)
                print('a', a, 'b', b)
                return 1, {f1 = 1024}
            end
            
            function ret_e()
                print('ret_e called')
                return e
            end
        ";
    
        public class DClass
        {
            public int f1;
            public int f2;
        }
        
        [CSharpCallLua]
        public interface ItfD
        {
            int f1 { get; set; }
            int f2 { get; set; }
            int add(int a, int b);
        }
    
        [CSharpCallLua]
        public delegate int FDelegate(int a, string b, out DClass c);
    
        [CSharpCallLua]
        public delegate Action GetE();
    
        // Use this for initialization
        void Start()
        {
            luaenv = new LuaEnv();
            luaenv.DoString(script);
    
            Debug.Log("_G.a = " + luaenv.Global.Get<int>("a"));
            Debug.Log("_G.b = " + luaenv.Global.Get<string>("b"));
            Debug.Log("_G.c = " + luaenv.Global.Get<bool>("c"));
    
    
            DClass d = luaenv.Global.Get<DClass>("d");//映射到有对应字段的class，by value
            Debug.Log("_G.d = {f1=" + d.f1 + ", f2=" + d.f2 + "}");
    
            Dictionary<string, double> d1 = luaenv.Global.Get<Dictionary<string, double>>("d");//映射到Dictionary<string, double>，by value
            Debug.Log("_G.d = {f1=" + d1["f1"] + ", f2=" + d1["f2"] + "}, d.Count=" + d1.Count);
    
            List<double> d2 = luaenv.Global.Get<List<double>>("d"); //映射到List<double>，by value
            Debug.Log("_G.d.len = " + d2.Count);
    
            ItfD d3 = luaenv.Global.Get<ItfD>("d"); //映射到interface实例，by ref，这个要求interface加到生成列表，否则会返回null，建议用法
            d3.f2 = 1000;
            Debug.Log("_G.d = {f1=" + d3.f1 + ", f2=" + d3.f2 + "}");
            Debug.Log("_G.d:add(1, 2)=" + d3.add(1, 2));
    
            LuaTable d4 = luaenv.Global.Get<LuaTable>("d");//映射到LuaTable，by ref
            Debug.Log("_G.d = {f1=" + d4.Get<int>("f1") + ", f2=" + d4.Get<int>("f2") + "}");
    
    
            Action e = luaenv.Global.Get<Action>("e");//映射到一个delgate，要求delegate加到生成列表，否则返回null，建议用法
            e();
    
            FDelegate f = luaenv.Global.Get<FDelegate>("f");
            DClass d_ret;
            int f_ret = f(100, "John", out d_ret);//lua的多返回值映射：从左往右映射到c#的输出参数，输出参数包括返回值，out参数，ref参数
            Debug.Log("ret.d = {f1=" + d_ret.f1 + ", f2=" + d_ret.f2 + "}, ret=" + f_ret);
    
            GetE ret_e = luaenv.Global.Get<GetE>("ret_e");//delegate可以返回更复杂的类型，甚至是另外一个delegate
            e = ret_e();
            e();
    
            LuaFunction d_e = luaenv.Global.Get<LuaFunction>("e");
            d_e.Call();
    
        }
    
        // Update is called once per frame
        void Update()
        {
            if (luaenv != null)
            {
                luaenv.Tick();
            }
        }
    
        void OnDestroy()
        {
            luaenv.Dispose();
        }
    }
```

🔚