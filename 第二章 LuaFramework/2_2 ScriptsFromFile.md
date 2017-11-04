####ScriptsFromFile.cs

```csharp
    using UnityEngine;
    using System.Collections;
    using LuaInterface;
    using System;
    using System.IO;
    
    /*
    一、require
    
      1.功能：载入文件并执行代码块，对于相同的文件只执行一次
    
      2.调用：require(“filename”)
    
      注：寻找文件的路径在package.path中，print(package.path)即可得到。
    
    二、dofile
    
      1.功能：载入文件并执行代码块，对于相同的文件每次都会执行
    
      2.调用：dofile("filename")
    
      3.错误处理：如果代码块中有错误则会引发错误
    
      4.优点：对简单任务而言，非常便捷
    
      5.缺点：每次载入文件时都会执行程序块
    
      6.定位：内置操作，辅助函数
    */
    
    //展示searchpath 使用，require 与 dofile 区别
    public class ScriptsFromFile : MonoBehaviour 
    {
        LuaState lua = null;
        private string strLog = "";    
    
    	void Start () 
        {
    #if UNITY_5		
            Application.logMessageReceived += Log;  // 收到日志消息时触发的事件
    #else
            Application.RegisterLogCallback(Log);
    #endif         
            lua = new LuaState();                
            lua.Start();        
            //如果移动了ToLua目录，自己手动修复吧，只是例子就不做配置了
            string fullPath = Application.dataPath + "\\LuaFramework/ToLua/Examples/02_ScriptsFromFile";
            lua.AddSearchPath(fullPath);        
        }
    
        void Log(string msg, string stackTrace, LogType type)
        {
            strLog += msg;
            strLog += "\r\n";
        }
    
        void OnGUI()
        {
            GUI.Label(new Rect(100, Screen.height / 2 - 100, 600, 400), strLog);
    
            if (GUI.Button(new Rect(50, 50, 120, 45), "DoFile"))
            {
                strLog = "";
                lua.DoFile("ScriptsFromFile.lua");                        
            }
            else if (GUI.Button(new Rect(50, 150, 120, 45), "Require"))
            {
                strLog = "";            
                lua.Require("ScriptsFromFile");            
            }
    
            lua.Collect();
            lua.CheckTop();
        }
    
        void OnApplicationQuit()
        {
            lua.Dispose();
            lua = null;
    
    #if UNITY_5		
            Application.logMessageReceived -= Log;
    #else
            Application.RegisterLogCallback(null);
    #endif 
        }
    }
```

&emsp;

####ScriptsFromFile.lua
```lua
    print("This is a script from a utf8 file")
    print("tolua: 你好! こんにちは! 안녕하세요!")
```


🔚