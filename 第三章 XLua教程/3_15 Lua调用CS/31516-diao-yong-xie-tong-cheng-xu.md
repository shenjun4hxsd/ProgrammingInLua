##调用协同程序

**LuaCoroutine.lua.txt**

```lua
    local util = require "xlua.util" -- 加载 Resources/xlua/util.lua.txt
    
    local coroutineInstance = CS.UnityEngine.GameObject.FindObjectOfType(typeof(CS.shenjun.Coroutine))
    
    local function async_yield_return(to_yield, cb)
        coroutineInstance:InvokeSpawn(to_yield, cb)
    end
    
    -- 将调用C#协同的方法 转换为可以在lua协同中调用的方法 yield_return需要在lua协同中执行
    local yield_return = util.async_to_sync(async_yield_return)
    
    local function coroutine_callback()
        print("lua coroutine over! callback done!")
    end
    
    local function co_function()
        print("lua coroutine start")
        local wt = CS.UnityEngine.WaitForSeconds(3)
        yield_return(wt, coroutine_callback)
    end
    
    -- local co = coroutine.create(co_function)
    
    function Start()
        -- coroutine.resume(co)
        util.coroutine_call(co_function)()
    end
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    --[===[  lua coroutine 语法
    
    --[[ 协同案例1 （基本语法）
    
    fun1 = function()
        print("hello")
    end
    
    -- 创建一个协同程序 co
    co = coroutine.create(fun1)
    
    -- 协同程序的4种状态 suspended、running、dead、normal
    -- 创建完协同程序以后，处于suspended状态
    print("status : ", coroutine.status(co))    
    
    -- 启动协同程序（第一次唤醒）
    coroutine.resume(co)
    -- 协同程序运行完以后，处于dead状态，再次执行没效果
    print("status : ", coroutine.status(co))
    
    -- 再次唤醒已经dead的协同程序，无反应，说明不会再次执行了
    coroutine.resume(co)
    print("status : ", coroutine.status(co))
    
    -- 协同程序处理dead状态，再次唤醒，返回false及一条错误信息
    print(coroutine.resume(co))
    
    ]]
    
    
    --[[ 协同案例2 （yield方法）
    
    -- 协同的强大之处 coroutine.yield()
    -- 方法中有yield语句
    -- 作用：让一个运行中的程序挂起，而之后可以恢复它的运行
    
    co = coroutine.create(function() 
            print("start coroutine function")
            for i=1, 3 do
                print("co :", i)
                coroutine.yield()
            end
         end)
    
    -- 处于suspended状态
    print(coroutine.status(co))
    
    -- 第一次唤醒，运行到yield方法时返回，挂起发生在yield调用中
    coroutine.resume(co)  -- 打印1
    print("1", coroutine.status(co))
    
    -- 第二次唤醒，从上一次的yield的调用中返回，继续执行，碰到yield再挂起
    coroutine.resume(co)  -- 打印2
    print("2", coroutine.status(co))
    
    -- 第三次唤醒
    coroutine.resume(co)  -- 打印3
    print("3", coroutine.status(co))
    
    print("3-4", coroutine.status(co))  -- suspended状态
    
    -- 第四次唤醒
    coroutine.resume(co)  -- 什么都不打印
    print("4", coroutine.status(co))
    
    --]]
    
    
    
    
    -- 当一个协同程序A唤醒一个协同程序B时，协同程序A就处于一个特殊状态，
    -- 既不是挂起状态（无法继续A的执行），也不是运行状态（是B在运行）。
    -- 所以将这时的状态称为“正常”状态。
    
    -- Lua的协同程序还具有一项有用的机制，就是可以通过一对resume-yield来交换数据。
    
    
    
    
    --[[ 协同案例3 （带参数协同）
    
    -- 创建一个带参数的协同程序
    co = coroutine.create(function(a, b)
            for i=1,3 do
                print("co", a, b, i)
                coroutine.yield()
            end
    
         end)
    
    -- 第一次唤醒 参数1，2传给function
    coroutine.resume(co, 10, 20)   -- 缺少参数使用nil，多余的参数舍弃
    
    -- 第二次唤醒
    coroutine.resume(co, 100, 200) -- 传入100，200无效，有其他用途
    
    --]]
    
    
    --[[ 协同案例4 （resume的返回值）
    
    co = coroutine.create(function(a, b) 
            coroutine.yield(a + b, a - b)
            return 6, 7
         end)
    
    -- 第一次唤醒
    -- 在resume的返回值中，第一个返回值为true表示没有错误
    -- 第2个返回值以及之后的，都对应yield传入的参数
    print(coroutine.resume(co, 10, 3))  -- true  13  7
    -- 当一个协同程序结束时，它的所有返回值都作为resume的返回值。
    print(coroutine.resume(co))         -- true  6   7
    --]]
    
    
    --[[ 协同案例5 yield的返回值
    
    co = coroutine.create(function()
            print("co", coroutine.yield("yield"))
         end)
    
    -- 第一次唤醒，传入的参数“a”是传给方法的，由于方法没有参数，所以舍弃“a”
    print("1", coroutine.resume(co, "a"))   -- "1"  true  yield
    
    -- 第二次唤醒，传入的参数1，2，作为yield的额外返回值使用（yield返回的额外值就是对应resume传入的参数）
    print("2", coroutine.resume(co, 1, 2))  -- "co"  1  2  -- "2"  true
    
    print("3", coroutine.resume(co, 100, 200)) -- "3"  false  cannot resume dead coroutine
    --]]
    
    
    --[[ *协同案例6* 协同程序的嵌套使用（管道与过滤器）
    
    -- 唤醒传入的迭代器
    function receive(prod)
        local status, value = coroutine.resume(prod);
        return value;
    end
    
    funcition send(x)
        coroutine.yield(x)
    end
    
    -- 返回一个挂起的协同程序 协同程序中需要有yield
    function producer()
        return coroutine.create(function()
            while true do
                local x = io.read
                send(x)                 -- 产生新值
            end
        end)
    end
    
    function filter(prod)
        return coroutine.create(function() 
            for line = 1, math.huge do
                local x = receive(prod) -- 接收producer产生的值
                x = string.format("%5d %s", line, x)
                send(x)                 -- 把新值发送给消费者
            end
        end)
    end
    
    function consumer(prod)
        while true do
            local x = receive(prod)     -- 获取新值
            io.write(x, "\n")           -- 消费新值
        end
    end
    
    -- 运行代码
    consumer(filter(producer))
    
    --]]
    
    ]===]
```

**LuaCoroutine.cs**

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
        [LuaCallCSharp]
        public class Coroutine : MonoBehaviour {
    
    		LuaEnv luaEnv = new LuaEnv();
    
            void Start () {
                luaEnv.DoString("require 'LuaCoroutine'");
                System.Action luaStart = luaEnv.Global.Get<System.Action>("Start");
    
                if (null != luaStart)
                    luaStart();
            }
    
            void Update () {
                
            }
    
            private void OnDestroy()
            {
                luaEnv.Dispose();
            }
    
            public void InvokeSpawn(object yield_return, System.Action callback)
            {
                StartCoroutine(SpawnObj(yield_return, callback));
            }
    
            IEnumerator SpawnObj(object yield_return, System.Action callback)
            {
                for (int i = 0; i < 10; i++)
                {
                    GameObject go = GameObject.CreatePrimitive(PrimitiveType.Sphere);
                    go.transform.position = Random.insideUnitSphere * 5;
                    if(yield_return is IEnumerator)
                    {
                        yield return StartCoroutine(yield_return as IEnumerator);
                    }
                    else
                    {
    					yield return yield_return;
                    }
                }
                callback();
            }
        }
    
        [LuaCallCSharp]
        public static class CoroutineConfig
        {
            public static List<System.Type> LuaCallCSharp
            {
                get{
                    return new List<System.Type>() 
                    { 
                        typeof(WaitForSeconds)
                    };
                }
            }
        }
    }
```

🔚