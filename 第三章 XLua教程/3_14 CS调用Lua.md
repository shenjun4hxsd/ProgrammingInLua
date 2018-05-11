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

**TableLua.lua.txt**

```lua
    student = {
        name = "xm", age = 18, Sex = "man",
        80, 90, 95,
        getSex = function(self)
            return "人妖"
        end,
    
        totalScore = function(self, a, b)
            return a + b;
        end
    }
```

#####1）、映射到class和struct

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
        public class TableToStruct : MonoBehaviour
        {
    
            LuaEnv luaEnv = new LuaEnv();
    
            void Start()
            {
                luaEnv.DoString("require 'TableLua'");
    
                // 对应public字段，table的属性可以多于或少于class的字段，没有对应的使用该类型的默认值
                Student xm = luaEnv.Global.Get<Student>("student");
                Debug.Log(xm);
            }
    
            void Update()
            {
                if (luaEnv != null)
                {
                    luaEnv.Tick();
                }
            }
    
            private void OnDestroy()
            {
                luaEnv.Dispose();
            }
    
            /*
             * xLua复杂值类型（struct）的默认传递方式是引用传递，这种方式要求先对值类型boxing，传递给lua，
             * lua使用后释放该引用。由于值类型每次boxing将产生一个新对象，当lua侧使用完毕释放该对象的引用时，
             * 则产生一次gc。为此，xLua实现了一套struct的gc优化方案，您只要通过简单的配置，
             * 则可以实现满足条件的struct传递到lua侧无gc。
             * 
             * struct需要满足什么条件？
             * 1、struct允许嵌套其它struct，但它以及它嵌套的struct只能包含这几种基本类型：
             * byte、sbyte、short、ushort、int、uint、long、ulong、float、double；
             * 例如UnityEngine定义的大多数值类型：Vector系列，Quaternion，Color。。。均满足条件，
             * 或者用户自定义的一些struct
             * 2、该struct配置了GCOptimize属性（对于常用的UnityEngine的几个struct，
             * Vector系列，Quaternion，Color。。。均已经配置了该属性），这个属性可以通过配置文件或者C# Attribute实现；
             * 3、使用到该struct的地方，需要添加到生成代码列表；
             */
            [GCOptimize]
            [LuaCallCSharp]
            struct Student
            {
                public string name;
                public int age;
                public string Sex { get; set; }  // 无法对应table中的键
    
                public override string ToString()
                {
                    return string.Format("name : {0}, age : {1}, sex : {2}", name, age, Sex);
                }
            }
    
            //[LuaCallCSharp]
            //public static class StructConfig
            //{
            //    public static List<System.Type> LuaCallCSharp
            //    {
            //        get
            //        {
            //            return new List<System.Type>()
            //            {
            //                typeof(Student)
            //            };
            //        }
            //    }
            //}
        }
    }
```

####2）、映射到Interface

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
    	public class TableToInterface : MonoBehaviour {
    
            LuaEnv luaEnv = new LuaEnv();
    
    		void Start () {
    
                luaEnv.DoString("require 'TableLua'");
    
                // 引用类型映射 代码生成器会生成一个实例
                IStudent iTable = luaEnv.Global.Get<IStudent>("student");
    
                // 修改table中某键的值
                luaEnv.Global.SetInPath<int>("student.age", 20);
    
                string _name = iTable.name;
                int age = iTable.age;
                string sex = iTable.Sex;
    
                Debug.Log(string.Format("name :{0}, age :{1}, sex :{2}", _name, age, sex));
    
                // 引用类型映射
                iTable.name = "xz";
                Debug.Log(luaEnv.Global.GetInPath<string>("student.name"));
    
                int result = iTable.totalScore(100, 200);
                Debug.Log(result);
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
    
            [CSharpCallLua]
            interface IStudent
            {
                string name { get; set; }
                int age { get; set; }
                string Sex { get; set; }
    
                //string get_Sex();
    
                int totalScore(int a, int b);
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