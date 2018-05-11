##调用Lua Table类型

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

####一、映射到class和struct

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

####二、映射到Interface

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

####三、映射到Dictionary和List

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
    	public class TableToDicOrList : MonoBehaviour {
    
            LuaEnv luaEnv = new LuaEnv();
    
    		void Start () {
    
                luaEnv.DoString("require 'TableLua'");
    
                // 值拷贝 只能获取匹配的
    
                Dictionary<string, string> dic = new Dictionary<string, string>();
                dic = luaEnv.Global.Get<Dictionary<string, string>>("student");
                foreach (var item in dic.Keys)
                {
                    Debug.Log("key :" + item + ", value :" + dic[item]);
                }
    
                List<int> list = new List<int>();
                list = luaEnv.Global.Get<List<int>>("student");
                foreach (var item in list)
                {
                    Debug.Log(item);
                }
            }
    		
    		void Update () {
    
    		}
    
            private void OnDestroy()
            {
                luaEnv.Dispose();
            }
        }
    }
```

