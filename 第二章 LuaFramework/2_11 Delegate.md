##Lua委托

####TestDelegate.cs

```csharp
    using UnityEngine;
    using System;
    using System.Collections.Generic;
    using LuaInterface;
    
    
    public class TestDelegate: MonoBehaviour
    {
        private string script =
        @"                              
                function DoClick1(go)                
                    print('click1 gameObject is '..go.name)                    
                end
    
                function DoClick2(go)                
                    print('click2 gameObject is '..go.name)                              
                end                       
    
                function AddClick1(listener)       
                    if listener.onClick then
                        listener.onClick = listener.onClick + DoClick1                                                    
                    else
                        listener.onClick = DoClick1                      
                    end                
                end
    
                function AddClick2(listener)
                    if listener.onClick then
                        listener.onClick = listener.onClick + DoClick2                      
                    else
                        listener.onClick = DoClick2
                    end                
                end
    
                function SetClick1(listener)
                    if listener.onClick then
                        listener.onClick:Destroy()
                    end
    
                    listener.onClick = DoClick1         
                end
    
                function RemoveClick1(listener)
                    if listener.onClick then
                        listener.onClick = listener.onClick - DoClick1      
                    else
                        print('empty delegate')
                    end
                end
    
                function RemoveClick2(listener)
                    if listener.onClick then
                        listener.onClick = listener.onClick - DoClick2    
                    else
                        print('empty delegate')                                
                    end
                end
    
                --测试重载问题
                function TestOverride(listener)
                    listener:SetOnFinished(TestEventListener.OnClick(DoClick1))
                    listener:SetOnFinished(TestEventListener.VoidDelegate(DoClick2))
                end
    
                function TestEvent()
                    print('this is a event')
                end
    
                function AddEvent(listener)
                    listener.onClickEvent = listener.onClickEvent + TestEvent
                end
    
                function RemoveEvent(listener)
                    listener.onClickEvent = listener.onClickEvent - TestEvent
                end
    
                local t = {name = 'byself'}
    
                function t:TestSelffunc()
                    print('callback with self: '..self.name)
                end       
    
                function AddSelfClick(listener)
                    if listener.onClick then
                        listener.onClick = listener.onClick + TestEventListener.OnClick(t.TestSelffunc, t)
                    else
                        listener.onClick = TestEventListener.OnClick(t.TestSelffunc, t)
                    end   
                end     
    
                function RemoveSelfClick(listener)
                    if listener.onClick then
                        listener.onClick = listener.onClick - TestEventListener.OnClick(t.TestSelffunc, t)
                    else
                        print('empty delegate')
                    end   
                end
        ";
    
        LuaState state = null;
        TestEventListener listener = null;
    
        LuaFunction SetClick1 = null;
        LuaFunction AddClick1 = null;
        LuaFunction AddClick2 = null;
        LuaFunction RemoveClick1 = null;
        LuaFunction RemoveClick2 = null;
        LuaFunction TestOverride = null;
        LuaFunction RemoveEvent = null;
        LuaFunction AddEvent = null;
        LuaFunction AddSelfClick = null;
        LuaFunction RemoveSelfClick = null;
       
        //需要删除的转LuaFunction为委托，不需要删除的直接加或者等于即可
        void Awake()
        {               
            state = new LuaState();
            state.Start();
            LuaBinder.Bind(state);
            Bind(state);
    
            state.LogGC = true;
            state.DoString(script);
            GameObject go = new GameObject("TestGo");
            listener = (TestEventListener)go.AddComponent(typeof(TestEventListener));
    
            SetClick1 = state.GetFunction("SetClick1");
            AddClick1 = state.GetFunction("AddClick1");
            AddClick2 = state.GetFunction("AddClick2");
            RemoveClick1 = state.GetFunction("RemoveClick1");
            RemoveClick2 = state.GetFunction("RemoveClick2");
            TestOverride = state.GetFunction("TestOverride");
            AddEvent = state.GetFunction("AddEvent");
            RemoveEvent = state.GetFunction("RemoveEvent");
    
            AddSelfClick = state.GetFunction("AddSelfClick");
            RemoveSelfClick = state.GetFunction("RemoveSelfClick");
        }
    
        void Bind(LuaState L)
        {
            L.BeginModule(null);
            TestEventListenerWrap.Register(state);
            L.EndModule();
    
            DelegateFactory.dict.Add(typeof(TestEventListener.OnClick), TestEventListener_OnClick);
            DelegateFactory.dict.Add(typeof(TestEventListener.VoidDelegate), TestEventListener_VoidDelegate);
        }
    
        void CallLuaFunction(LuaFunction func)
        {        
            func.BeginPCall();
            func.Push(listener);
            func.PCall();
            func.EndPCall();                
        }
    
        //自动生成代码后拷贝过来
        class TestEventListener_OnClick_Event : LuaDelegate
        {
            public TestEventListener_OnClick_Event(LuaFunction func) : base(func) { }
    
            public void Call(UnityEngine.GameObject param0)
            {
                func.BeginPCall();
                func.Push(param0);
                func.PCall();
                func.EndPCall();
            }
        }
    
        public static Delegate TestEventListener_OnClick(LuaFunction func, LuaTable self, bool flag)
        {
            if (func == null)
            {
                TestEventListener.OnClick fn = delegate { };
                return fn;
            }
    
            TestEventListener_OnClick_Event target = new TestEventListener_OnClick_Event(func);
            TestEventListener.OnClick d = target.Call;
            target.method = d.Method;
            return d;
        }
    
        class TestEventListener_VoidDelegate_Event : LuaDelegate
        {
            public TestEventListener_VoidDelegate_Event(LuaFunction func) : base(func) { }
    
            public void Call(UnityEngine.GameObject param0)
            {
                func.BeginPCall();
                func.Push(param0);
                func.PCall();
                func.EndPCall();
            }
        }
    
        public static Delegate TestEventListener_VoidDelegate(LuaFunction func, LuaTable self, bool flag)
        {
            if (func == null)
            {
                TestEventListener.VoidDelegate fn = delegate { };
                return fn;
            }
    
            TestEventListener_VoidDelegate_Event target = new TestEventListener_VoidDelegate_Event(func);
            TestEventListener.VoidDelegate d = target.Call;
            target.method = d.Method;
            return d;
        }
    
        void OnGUI()
        {
            if (GUI.Button(new Rect(10, 10, 120, 40), " = OnClick1"))
            {
                CallLuaFunction(SetClick1);
            }
            else if (GUI.Button(new Rect(10, 60, 120, 40), " + Click1"))
            {
                CallLuaFunction(AddClick1);
            }
            else if (GUI.Button(new Rect(10, 110, 120, 40), " + Click2"))
            {
                CallLuaFunction(AddClick2);
            }
            else if (GUI.Button(new Rect(10, 160, 120, 40), " - Click1"))
            {
                CallLuaFunction(RemoveClick1);
            }
            else if (GUI.Button(new Rect(10, 210, 120, 40), " - Click2"))
            {
                CallLuaFunction(RemoveClick2);
            }
            else if (GUI.Button(new Rect(10, 260, 120, 40), "+ Click1 in C#"))
            {
                LuaFunction func = state.GetFunction("DoClick1");
                TestEventListener.OnClick onClick = (TestEventListener.OnClick)DelegateFactory.CreateDelegate(typeof(TestEventListener.OnClick), func);
                listener.onClick += onClick;
            }        
            else if (GUI.Button(new Rect(10, 310, 120, 40), " - Click1 in C#"))
            {
                LuaFunction func = state.GetFunction("DoClick1");
                listener.onClick = (TestEventListener.OnClick)DelegateFactory.RemoveDelegate(listener.onClick, func);
                func.Dispose();
                func = null;
            }
            else if (GUI.Button(new Rect(10, 360, 120, 40), "OnClick"))
            {
                if (listener.onClick != null)
                {
                    listener.onClick(gameObject);
                }
                else
                {
                    Debug.Log("empty delegate!!");
                }
            }
            else if (GUI.Button(new Rect(10, 410, 120, 40), "Override"))
            {
                CallLuaFunction(TestOverride);
            }
            else if (GUI.Button(new Rect(10, 460, 120, 40), "Force GC"))
            {
                //自动gc log: collect lua reference name , id xxx in thread 
                state.LuaGC(LuaGCOptions.LUA_GCCOLLECT, 0);
                GC.Collect();
            }
            else if (GUI.Button(new Rect(10, 510, 120, 40), "event +"))
            {
                CallLuaFunction(AddEvent);
            }
            else if (GUI.Button(new Rect(10, 560, 120, 40), "event -"))
            {
                CallLuaFunction(RemoveEvent);
            }
            else if (GUI.Button(new Rect(10, 610, 120, 40), "event call"))
            {
                listener.OnClickEvent(gameObject);
            }
            else if (GUI.Button(new Rect(200, 10, 120, 40), "+self call"))
            {
                CallLuaFunction(AddSelfClick);
            }
            else if (GUI.Button(new Rect(200, 60, 120, 40), "-self call"))
            {
                CallLuaFunction(RemoveSelfClick);
            }
        }
    
        void Update()
        {
            state.Collect();
            state.CheckTop();        
        }
    
        void SafeRelease(ref LuaFunction luaRef)
        {
            if (luaRef != null)
            {
                luaRef.Dispose();
                luaRef = null;
            }
        }
    
        void OnDestroy()
        {
            SafeRelease(ref AddClick1);
            SafeRelease(ref AddClick2);
            SafeRelease(ref RemoveClick1);
            SafeRelease(ref RemoveClick2);
            SafeRelease(ref SetClick1);
            SafeRelease(ref TestOverride);
            state.Dispose();
            state = null;
        }
    }
```

&emsp;

####TestEventListener.cs

```csharp
    using UnityEngine;
    using System;
    using System.Collections;
    using LuaInterface;
    
    public class TestEventListener : MonoBehaviour
    {
        public delegate void VoidDelegate(GameObject go);
        public delegate void OnClick(GameObject go);    
        public OnClick onClick = delegate { };
    
        public event OnClick onClickEvent = delegate { };
    
        public Func<bool> TestFunc = null;
    
        public void SetOnFinished(OnClick click)
        {
            Debugger.Log("SetOnFinished OnClick");
        }
        
        public void SetOnFinished(VoidDelegate click)
        {
            Debugger.Log("SetOnFinished VoidDelegate");
        }
    
        [NoToLuaAttribute]
        public void OnClickEvent(GameObject go)
        {
            onClickEvent(go);
        }
    }
```

&emsp;

####TestEventListenerWrap.cs

```csharp
    //this source code was auto-generated by tolua#, do not modify it
    using System;
    using LuaInterface;
    
    public class TestEventListenerWrap
    {
    	public static void Register(LuaState L)
    	{
    		L.BeginClass(typeof(TestEventListener), typeof(UnityEngine.MonoBehaviour));
    		L.RegFunction("SetOnFinished", SetOnFinished);
    		L.RegFunction("__eq", op_Equality);
    		L.RegFunction("__tostring", Lua_ToString);
    		L.RegVar("onClick", get_onClick, set_onClick);
    		L.RegVar("onClickEvent", get_onClickEvent, set_onClickEvent);
    		L.RegFunction("OnClick", TestEventListener_OnClick);
    		L.RegFunction("VoidDelegate", TestEventListener_VoidDelegate);
    		L.EndClass();
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int SetOnFinished(IntPtr L)
    	{
    		try
    		{
    			int count = LuaDLL.lua_gettop(L);
    
    			if (count == 2 && TypeChecker.CheckTypes(L, 1, typeof(TestEventListener), typeof(TestEventListener.VoidDelegate)))
    			{
    				TestEventListener obj = (TestEventListener)ToLua.ToObject(L, 1);
    				TestEventListener.VoidDelegate arg0 = null;
    				LuaTypes funcType2 = LuaDLL.lua_type(L, 2);
    
    				if (funcType2 != LuaTypes.LUA_TFUNCTION)
    				{
    					 arg0 = (TestEventListener.VoidDelegate)ToLua.ToObject(L, 2);
    				}
    				else
    				{
    					LuaFunction func = ToLua.ToLuaFunction(L, 2);
    					arg0 = DelegateFactory.CreateDelegate(typeof(TestEventListener.VoidDelegate), func) as TestEventListener.VoidDelegate;
    				}
    
    				obj.SetOnFinished(arg0);
    				return 0;
    			}
    			else if (count == 2 && TypeChecker.CheckTypes(L, 1, typeof(TestEventListener), typeof(TestEventListener.OnClick)))
    			{
    				TestEventListener obj = (TestEventListener)ToLua.ToObject(L, 1);
    				TestEventListener.OnClick arg0 = null;
    				LuaTypes funcType2 = LuaDLL.lua_type(L, 2);
    
    				if (funcType2 != LuaTypes.LUA_TFUNCTION)
    				{
    					 arg0 = (TestEventListener.OnClick)ToLua.ToObject(L, 2);
    				}
    				else
    				{
    					LuaFunction func = ToLua.ToLuaFunction(L, 2);
    					arg0 = DelegateFactory.CreateDelegate(typeof(TestEventListener.OnClick), func) as TestEventListener.OnClick;
    				}
    
    				obj.SetOnFinished(arg0);
    				return 0;
    			}
    			else
    			{
    				return LuaDLL.luaL_throw(L, "invalid arguments to method: TestEventListener.SetOnFinished");
    			}
    		}
    		catch(Exception e)
    		{
    			return LuaDLL.toluaL_exception(L, e);
    		}
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int op_Equality(IntPtr L)
    	{
    		try
    		{
    			ToLua.CheckArgsCount(L, 2);
    			UnityEngine.Object arg0 = (UnityEngine.Object)ToLua.ToObject(L, 1);
    			UnityEngine.Object arg1 = (UnityEngine.Object)ToLua.ToObject(L, 2);
    			bool o = arg0 == arg1;
    			LuaDLL.lua_pushboolean(L, o);
    			return 1;
    		}
    		catch(Exception e)
    		{
    			return LuaDLL.toluaL_exception(L, e);
    		}
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int Lua_ToString(IntPtr L)
    	{
    		object obj = ToLua.ToObject(L, 1);
    
    		if (obj != null)
    		{
    			LuaDLL.lua_pushstring(L, obj.ToString());
    		}
    		else
    		{
    			LuaDLL.lua_pushnil(L);
    		}
    
    		return 1;
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int get_onClick(IntPtr L)
    	{
    		object o = null;
    
    		try
    		{
    			o = ToLua.ToObject(L, 1);
    			TestEventListener obj = (TestEventListener)o;
    			TestEventListener.OnClick ret = obj.onClick;
    			ToLua.Push(L, ret);
    			return 1;
    		}
    		catch(Exception e)
    		{
    			return LuaDLL.toluaL_exception(L, e, o == null ? "attempt to index onClick on a nil value" : e.Message);
    		}
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int get_onClickEvent(IntPtr L)
    	{
    		ToLua.Push(L, new EventObject("TestEventListener.onClickEvent"));
    		return 1;
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int set_onClick(IntPtr L)
    	{
    		object o = null;
    
    		try
    		{
    			o = ToLua.ToObject(L, 1);
    			TestEventListener obj = (TestEventListener)o;
    			TestEventListener.OnClick arg0 = null;
    			LuaTypes funcType2 = LuaDLL.lua_type(L, 2);
    
    			if (funcType2 != LuaTypes.LUA_TFUNCTION)
    			{
    				 arg0 = (TestEventListener.OnClick)ToLua.CheckObject(L, 2, typeof(TestEventListener.OnClick));
    			}
    			else
    			{
    				LuaFunction func = ToLua.ToLuaFunction(L, 2);
    				arg0 = DelegateFactory.CreateDelegate(typeof(TestEventListener.OnClick), func) as TestEventListener.OnClick;
    			}
    
    			obj.onClick = arg0;
    			return 0;
    		}
    		catch(Exception e)
    		{
    			return LuaDLL.toluaL_exception(L, e, o == null ? "attempt to index onClick on a nil value" : e.Message);
    		}
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int set_onClickEvent(IntPtr L)
    	{
    		try
    		{
    			TestEventListener obj = (TestEventListener)ToLua.CheckObject(L, 1, typeof(TestEventListener));
    			EventObject arg0 = null;
    
    			if (LuaDLL.lua_isuserdata(L, 2) != 0)
    			{
    				arg0 = (EventObject)ToLua.ToObject(L, 2);
    			}
    			else
    			{
    				return LuaDLL.luaL_throw(L, "The event 'TestEventListener.onClickEvent' can only appear on the left hand side of += or -= when used outside of the type 'TestEventListener'");
    			}
    
    			if (arg0.op == EventOp.Add)
    			{
    				TestEventListener.OnClick ev = (TestEventListener.OnClick)DelegateFactory.CreateDelegate(typeof(TestEventListener.OnClick), arg0.func);
    				obj.onClickEvent += ev;
    			}
    			else if (arg0.op == EventOp.Sub)
    			{
    				TestEventListener.OnClick ev = (TestEventListener.OnClick)LuaMisc.GetEventHandler(obj, typeof(TestEventListener), "onClickEvent");
    				Delegate[] ds = ev.GetInvocationList();
    				LuaState state = LuaState.Get(L);
    
    				for (int i = 0; i < ds.Length; i++)
    				{
    					ev = (TestEventListener.OnClick)ds[i];
    					LuaDelegate ld = ev.Target as LuaDelegate;
    
    					if (ld != null && ld.func == arg0.func)
    					{
    						obj.onClickEvent -= ev;
    						state.DelayDispose(ld.func);
    						break;
    					}
    				}
    
    				arg0.func.Dispose();
    			}
    
    			return 0;
    		}
    		catch(Exception e)
    		{
    			return LuaDLL.toluaL_exception(L, e);
    		}
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int TestEventListener_OnClick(IntPtr L)
    	{
    		try
    		{
    			LuaFunction func = ToLua.CheckLuaFunction(L, 1);
    			Delegate arg1 = DelegateFactory.CreateDelegate(typeof(TestEventListener.OnClick), func);
    			ToLua.Push(L, arg1);
    			return 1;
    		}
    		catch(Exception e)
    		{
    			return LuaDLL.toluaL_exception(L, e);
    		}
    	}
    
    	[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    	static int TestEventListener_VoidDelegate(IntPtr L)
    	{
    		try
    		{
    			LuaFunction func = ToLua.CheckLuaFunction(L, 1);
    			Delegate arg1 = DelegateFactory.CreateDelegate(typeof(TestEventListener.VoidDelegate), func);
    			ToLua.Push(L, arg1);
    			return 1;
    		}
    		catch(Exception e)
    		{
    			return LuaDLL.toluaL_exception(L, e);
    		}
    	}
    }
```

🔚