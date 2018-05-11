##调用Function

**FunctionLua.lua.txt**

```lua
    function NoParam()
        print('LuaFunction NoParam')
    end
    
    function HaveParam(a, b)
        print('LuaFunction HaveParam : ', 'a: ', a, 'b: ', b)
    end
    
    function HaveMultiParam(...)
        print(...)
    end
            
    function ReturnString()
        print('LuaFunction ReturnString')
        return "i am the return string!"
    end
    
    function ReturnFunction()
        print('LuaFunction ReturnFunction')
        return NoParam
    end
    
    function ReturnMultiValue()
    	print("LuaFunction ReturnMultiValue")
    	return true, "ReturnMultiValue", 1, "num"
    end
```

####一、映射到Delegate

```csharp
    /*
     *  created by shenjun
     */
    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using XLua;
    using System;
    
    namespace shenjun
    {
    	public class FunctionToDelegate : MonoBehaviour {
    
            LuaEnv luaEnv = new LuaEnv();
    
    		void Start () {
    
                luaEnv.DoString("require 'FunctionLua'");
    
                NoParam();
                HaveParam();
                HaveMultiParam();
                HaveReturnString();
                HaveReturnFunction();
                HaveMultiReturn();
    		}
    		
    		void Update () {
    			
    		}
    
            private void OnDestroy()
            {
                luaEnv.Dispose();
            }
    
            #region 无参数
            void NoParam()
            {
                // 无返回值
                Action del1 = luaEnv.Global.Get<Action>("NoParam");
                del1();
            }
            #endregion
    
            #region 有两个参数
            [CSharpCallLua]
            public delegate void HaveParamDel(int a, int b);
            void HaveParam()
            {
                HaveParamDel del = luaEnv.Global.Get<HaveParamDel>("HaveParam");
                del(10, 20);
            }
            #endregion
    
            #region 有任意个参数
            [CSharpCallLua]
            public delegate void HaveMultiParamDel(params int[] nums);
            void HaveMultiParam()
            {
                HaveMultiParamDel del = luaEnv.Global.Get<HaveMultiParamDel>("HaveMultiParam");
                del(10, 20, 30, 40);
            }
            #endregion
    
            #region 有字符串返回值
            [CSharpCallLua]
            public delegate string HaveReturnStringDel();
            void HaveReturnString()
            {
                HaveReturnStringDel del = luaEnv.Global.Get<HaveReturnStringDel>("ReturnString");
                string r = del();
                Debug.Log("GetResturn :" + r);
            }
            #endregion
    
            #region 有方法返回值
            [CSharpCallLua]
            public delegate Action HaveReturnFunctionDel();
            void HaveReturnFunction()
            {
                HaveReturnFunctionDel del = luaEnv.Global.Get<HaveReturnFunctionDel>("ReturnFunction");
                Action result = del();
                result();
            }
            #endregion
    
            #region 多返回值
            // 多返回值的，返回值从左到右的依次映射，返回值，out参数或ref参数 。。。
            [CSharpCallLua]
            public delegate bool HaveMultiReturnDel(ref string a, out int b);
    
            void HaveMultiReturn()
            {
                HaveMultiReturnDel del = luaEnv.Global.Get<HaveMultiReturnDel>("ReturnMultiValue");
                string a = "";
                int b;
                bool result = del(ref a, out b);
                Debug.Log("result :" + result + ", a :" + a + ", b :" + b);
            }
            #endregion
        }
    }
```