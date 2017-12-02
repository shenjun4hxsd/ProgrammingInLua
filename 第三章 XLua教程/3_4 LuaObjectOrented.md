##LuaObjectOrented

####InvokeLua.cs

```csharp
    using UnityEngine;
    using System.Collections;
    using XLua;
    
    public class InvokeLua : MonoBehaviour
    {
        [CSharpCallLua]
        public interface ICalc
        {
            int Add(int a, int b);
            int Mult { get; set; }
        }
    
        [CSharpCallLua]
        public delegate ICalc CalcNew(int mult, params string[] args);
    
        private string script = @"
                    local calc_mt = {
                        __index = {
                            Add = function(self, a, b)
                                return (a + b) * self.Mult
                            end
                        }
                    }
    
                    Calc = {
    	                New = function (mult, ...)
                            print(...)
                            return setmetatable({Mult = mult}, calc_mt)
                        end
                    }
    	        ";
        // Use this for initialization
        void Start()
        {
            LuaEnv luaenv = new LuaEnv();
            Test(luaenv);//调用了带可变参数的delegate，函数结束都不会释放delegate，即使置空并调用GC
            luaenv.Dispose();
        }
    
        void Test(LuaEnv luaenv)
        {
            luaenv.DoString(script);
            CalcNew calc_new = luaenv.Global.GetInPath<CalcNew>("Calc.New");
            ICalc calc = calc_new(10, "hi", "john"); //constructor
            Debug.Log("sum(*10) =" + calc.Add(1, 2));
            calc.Mult = 100;
            Debug.Log("sum(*100)=" + calc.Add(1, 2));
        }
    
        // Update is called once per frame
        void Update()
        {
    
        }
    }
```

🔚