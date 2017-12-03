##xLua的配置

`xLua`所有的配置都支持三种方式：打标签；静态列表；动态列表。

####打标签

`xLua`用白名单来指明生成哪些代码，而白名单通过`attribute`来配置，比如你想从`lua`调用`c#`的某个类，希望生成适配代码，你可以为这个类型打一个`LuaCallCSharp`标签：

```csharp
    [LuaCallCSharp]
    public class A
    {
    }
```

该方式方便，但在`il2cpp`下会增加不少的代码量，不建议使用。

####静态列表

有时我们无法直接给一个类型打标签，比如系统`api`，没源码的库，或者实例化的泛化类型，这时你可以在一个静态类里声明一个静态字段，该字段的类型除`BlackList`和`AdditionalProperties`之外只要实现了`IEnumerable<Type>`就可以了（这两个例外后面具体会说），然后为这字段加上标签：

```csharp
    [LuaCallCSharp]
    public static List<Type> mymodule_lua_call_cs_list = new List<Type>()
    {
        typeof(GameObject),
        typeof(Dictionary<string, int>),
    };
```

这个字段需要放到一个静态类里头，建议放到`Editor`目录。

####动态列表

声明一个静态属性，打上相应的标签即可。

```csharp
    [Hotfix]
    public static List<Type> by_property
    {
        get
        {
            return (from type in Assembly.GetExecutingAssembly().GetTypes()
                    where type.Namespace == "XXXX"
                    select type).ToList();
        }
    }
```

`Getter`是代码，你可以实现很多效果，比如按名字空间配置，按程序集配置等等。

这个属性需要放到一个静态类里头，建议放到`Editor`目录。

###XLua.LuaCallCSharp

一个`C#`类型加了这个配置，`xLua`会生成这个类型的适配代码（包括构造该类型实例，访问其成员属性、方法，静态属性、方法），否则将会尝试用性能较低的反射方式来访问。

一个类型的扩展方法（`Extension Methods`）加了这配置，也会生成适配代码并追加到被扩展类型的成员方法上。

`xLua`只会生成加了该配置的类型，不会自动生成其父类的适配代码，当访问子类对象的父类方法，如果该父类加了`LuaCallCSharp`配置，则执行父类的适配代码，否则会尝试用反射来访问。

反射访问除了性能不佳之外，在`il2cpp`下还有可能因为代码剪裁而导致无法访问，后者可以通过下面介绍的`ReflectionUse`标签来避免。

###XLua.ReflectionUse

一个`C#`类型类型加了这个配置，`xLua`会生成`link.xml`阻止`il2cpp`的代码剪裁。

对于扩展方法，必须加上`LuaCallCSharp`或者`ReflectionUse`才可以被访问到。

建议所有要在`Lua`访问的类型，要么加`LuaCallCSharp`，要么加上`ReflectionUse`，这才能够保证在各平台都能正常运行。

###XLua.CSharpCallLua

如果希望把一个`lua`函数适配到一个`C# delegate`（一类是`C#`侧各种回调：`UI`事件，`delegate`参数，比如`List<T>:ForEach`；另外一类场景是通过`LuaTable`的`Get`函数指明一个`lua`函数绑定到一个`delegate`）。或者把一个`lua table`适配到一个`C# interface`，该`delegate`或者`interface`需要加上该配置。

###XLua.GCOptimize

一个`C#`纯值类型（注：指的是一个只包含值类型的`struct`，可以嵌套其它只包含值类型的`struct`）或者`C#`枚举值加上了这个配置。`xLua`会为该类型生成`gc`优化代码，效果是该值类型在`lua`和`c#`间传递不产生（`C#`）`gc alloc`，该类型的数组访问也不产生`gc`。各种无`GC`的场景，可以参考`05_NoGc`例子。

除枚举之外，包含无参构造函数的复杂类型，都会生成`lua table`到该类型，以及改类型的一维数组的转换代码，这将会优化这个转换的性能，包括更少的`gc alloc`。

###XLua.AdditionalProperties

这个是`GCOptimize`的扩展配置，有的时候，一些`struct`喜欢把`field`做成是私有的，通过`property`来访问`field`，这时就需要用到该配置（默认情况下`GCOptimize`只对`public`的`field`打解包）。

标签方式比较简单，配置方式复杂一点，要求是`Dictionary<Type, List<string>>`类型，`Dictionary`的`Key`是要生效的类型，`Value`是属性名列表。可以参考`XLua`对几个`UnityEngine`下值类型的配置，`SysGCOptimize`类。

###XLua.BlackList

如果你不要生成一个类型的一些成员的适配代码，你可以通过这个配置来实现。

标签方式比较简单，对应的成员上加就可以了。

由于考虑到有可能需要把重载函数的其中一个重载列入黑名单，配置方式比较复杂，类型是`List<List<string>>`，对于每个成员，在第一层`List`有一个条目，第二层`List`是个`string`的列表，第一个`string`是类型的全路径名，第二个`string`是成员名，如果成员是一个方法，还需要从第三个`string`开始，把其参数的类型全路径全列出来。
例如下面是对`GameObject`的一个属性以及`FileInfo`的一个方法列入黑名单：

```csharp
    [BlackList]
    public static List<List<string>> BlackList = new List<List<string>>()  {
        new List<string>(){"UnityEngine.GameObject", "networkView"},
        new List<string>(){"System.IO.FileInfo", "GetAccessControl", "System.Security.AccessControl.AccessControlSections"},
    };
```

###下面是生成期配置，必须放到Editor目录下

####CSObjectWrapEditor.GenPath

配置生成代码的放置路径，类型是`string`。默认放在“`Assets/XLua/Gen/`”下。

####CSObjectWrapEditor.GenCodeMenu

该配置用于生成引擎的二次开发，一个无参数函数加了这个标签，在执行“`XLua/Generate Code`”菜单时会触发这个函数的调用。


####ExampleGenConfig.cs

```csharp
    using System.Collections.Generic;
    using System;
    using UnityEngine;
    using XLua;
    
    //配置的详细介绍请看Doc下《XLua的配置.doc》
    public static class ExampleGenConfig
    {
        //lua中要使用到C#库的配置，比如C#标准库，或者Unity API，第三方库等。
        [LuaCallCSharp]
        public static List<Type> LuaCallCSharp = new List<Type>() {
                    typeof(System.Object),
                    typeof(UnityEngine.Object),
                    typeof(Vector2),
                    typeof(Vector3),
                    typeof(Vector4),
                    typeof(Quaternion),
                    typeof(Color),
                    typeof(Ray),
                    typeof(Bounds),
                    typeof(Ray2D),
                    typeof(Time),
                    typeof(GameObject),
                    typeof(Component),
                    typeof(Behaviour),
                    typeof(Transform),
                    typeof(Resources),
                    typeof(TextAsset),
                    typeof(Keyframe),
                    typeof(AnimationCurve),
                    typeof(AnimationClip),
                    typeof(MonoBehaviour),
                    typeof(ParticleSystem),
                    typeof(SkinnedMeshRenderer),
                    typeof(Renderer),
                    typeof(WWW),
                    typeof(System.Collections.Generic.List<int>),
                    typeof(Action<string>),
                    typeof(UnityEngine.Debug)
                };
    
        //C#静态调用Lua的配置（包括事件的原型），仅可以配delegate，interface
        [CSharpCallLua]
        public static List<Type> CSharpCallLua = new List<Type>() {
                    typeof(Action),
                    typeof(Func<double, double, double>),
                    typeof(Action<string>),
                    typeof(Action<double>),
                    typeof(UnityEngine.Events.UnityAction),
                    typeof(System.Collections.IEnumerator)
                };
    
        //黑名单
        [BlackList]
        public static List<List<string>> BlackList = new List<List<string>>()  {
                    new List<string>(){"UnityEngine.WWW", "movie"},
        #if UNITY_WEBGL
                    new List<string>(){"UnityEngine.WWW", "threadPriority"},
        #endif
                    new List<string>(){"UnityEngine.Texture2D", "alphaIsTransparency"},
                    new List<string>(){"UnityEngine.Security", "GetChainOfTrustValue"},
                    new List<string>(){"UnityEngine.CanvasRenderer", "onRequestRebuild"},
                    new List<string>(){"UnityEngine.Light", "areaSize"},
                    new List<string>(){"UnityEngine.AnimatorOverrideController", "PerformOverrideClipListCleanup"},
        #if !UNITY_WEBPLAYER
                    new List<string>(){"UnityEngine.Application", "ExternalEval"},
        #endif
                    new List<string>(){"UnityEngine.GameObject", "networkView"}, //4.6.2 not support
                    new List<string>(){"UnityEngine.Component", "networkView"},  //4.6.2 not support
                    new List<string>(){"System.IO.FileInfo", "GetAccessControl", "System.Security.AccessControl.AccessControlSections"},
                    new List<string>(){"System.IO.FileInfo", "SetAccessControl", "System.Security.AccessControl.FileSecurity"},
                    new List<string>(){"System.IO.DirectoryInfo", "GetAccessControl", "System.Security.AccessControl.AccessControlSections"},
                    new List<string>(){"System.IO.DirectoryInfo", "SetAccessControl", "System.Security.AccessControl.DirectorySecurity"},
                    new List<string>(){"System.IO.DirectoryInfo", "CreateSubdirectory", "System.String", "System.Security.AccessControl.DirectorySecurity"},
                    new List<string>(){"System.IO.DirectoryInfo", "Create", "System.Security.AccessControl.DirectorySecurity"},
                    new List<string>(){"UnityEngine.MonoBehaviour", "runInEditMode"},
                };
    }
```


🔚
