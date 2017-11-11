##资源热更新

&emsp;

热更新涉及资源热更新和代码热更新（其实lua代码也是资源），那接下来看看如何动态加载一个模型，然后热更成其他素材。这一部分涉及资源打包、动态创建资源等内容。

####一、创建物体

为了调试的方便，先将框架配置为本地模式，待测试热更新时再改成更新模式。

```csharp
        public const bool UpdateMode = false;  ✅                     //更新模式-默认关闭 
        public const bool LuaByteMode = false;                       //Lua字节码模式-默认关闭 
        public const bool LuaBundleMode = false;  ✅                  //Lua代码AssetBundle模式
```