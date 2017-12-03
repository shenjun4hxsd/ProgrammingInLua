## Lua程序设计

#### 联系邮箱：380921128@qq.com

 

 

### 关于本书

#### 主版本： beta

#### 网站地址： [shenjun7792.github.io](https://shenjun7792.github.io)

#### 网站作者： 沈军

 

 

## 一、什么是热更新？

热更新，是对hot update或者 hot fix的翻译，计算机术语，表示在不停机的前提下对系统进行更改（摘抄一下）：

“hot就是热，机器运行会发烫，hot就是不停机的意思。

热更新，是个很形象的词，机器烫的时候更新，开着更新。

比如Windows不重启的前提下安装补丁

比如Http服务器在不重启的前提下换掉一个文件

那么对于Unity3D来说，何谓热更新？

额……这个真相实在是不想讲出来，因为很多时候，这个词都用错了。

Unity3D是一个客户端工具，用户是否重启客户端，根本是我们不关心的问题。

很多时候我们用着热更新这个词汇，却不需要真的热更新。

只有少部分游戏，游戏资源在玩的过程中边玩边下，不重启的前提下变更了资源。

我们不需要用户不重启客户端就能实现资源代码的更新，我们需要的是用户重启客户端能实现资源代码的更新。

让我们暂时放过这个我们的需求连词汇都用错了这个基本事实，来总结一下何谓Unity3D热更新，Unity3D热更新就是指：用户重启客户端就能实现客户端资源代码更新的需求或者功能。”

 

## 二、为什么要热更新？

热更新，能够缩短用户取得新版客户端的流程，改善用户体验。

#### 没有热更新：

pc用户：

下载客户端-&gt;等待下载-&gt;安装客户端-&gt;等待安装-&gt;启动-&gt;等待加载-&gt;玩

手机用户：

商城下载APP-&gt;等待下载-&gt;等待安装-&gt;启动-&gt;等待加载-&gt;玩

#### 有了热更新：

pc用户：

启动-&gt;等待热更新-&gt;等待加载-&gt;玩

有独立loader的pc用户：

启动loader-&gt;等待热更新-&gt;启动游戏-&gt;等待加载-&gt;玩

手机用户：

启动-&gt;等待热更新-&gt;等待加载-&gt;玩

通过对比就可以看出，有没有热更新对于用户体验的影响还是挺大的，主要就是省去用户自行更新客户端的步骤。

 

## 三、如何热更新？Unity3D的热更新的方法比较

#### 3.1、Android 应用的热更新

• 将执行代码预编译为assembly dll。

• 将代码作为TextAsset打包进Assetbundle。

• 运行时，使用Reflection机制实现代码的功能。

• 更新相应的Assetbundle即可实现热更新。

#### 3.2、Android 与iOS 热更新的异同

• 苹果官方禁止iOS下的程序热更新；JIT（Just In Time 即时编译技术）在iOS下无效。

• 热更新方案：Unity+Lua插件。

#### 3.3、 使用Lua 插件进行iOS 热更新的原理

![](/assets/屏幕快照 2017-11-07 下午8.00.17.png)

#### 3.4、Unity 热更新的注意点

• 需要更新的代码、资源，都必须打包成AssetBundle（建议使用未压缩的格式打包）

• 熟悉Unity的几个重要的路径

```
    • Resources（只读）

    • StreamingAssets（只读）

    • Application.dataPath（只读）

    • Application.persistentDataPath（可读写）
```

#### 3.5、重要路径之 之Resources

• Resources文件夹下的资源无论使用与否都会被打包

• 资源会被压缩，转化成二进制

• 打包后文件夹下的资源只读

• 无法动态更改，无法做热更新

• 使用Resources.Load加载

#### 3.6、重要路径之StreamingAssets

• 流数据的缓存目录

• 文件夹下的资源无论使用与否都会被打包

• 资源不会被压缩和加密

• 打包后文件夹下的资源只读，主要存放二进制文件

• 无法做热更新

• WWW类加载（一般用CreateFromFile ，若资源是AssetBundle，依据其打包方式看是否是压缩的来决定）

• 相对路径，具体路径依赖于实际平台

•Application.streamingAssetsPath

• IOS: Application.dataPath + “/Raw” 或Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxx.app/Data/Raw

#### 3.7、重要路径之Application.dataPath

• 游戏的数据文件夹的路径（例如在Editor中的Assets）

• 很少用到

• 无法做热更新

• IOS路径: Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxx.app/Data

#### 3.8、重要路径之Application.persistentDataPath

• 持久化数据存储目录的路径（ 沙盒目录，打包之前不存在 ）

• 文件夹下的资源无论使用与否都会被打包

• 运行时有效，可读写

• 无内容限制，从StreamingAsset中读取二进制文件或从AssetBundle读取文件来写入PersistentDataPath中

• 适合热更新

• IOS路径: Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Documents

#### 3.9、使用Lua 插件进行iOS 热更新的总体流程

![](/assets/屏幕快照 2017-11-07 下午8.07.40.png)

 

## 四、支持Unity iOS 热更新的各种Lua 插件的对比

#### 4.1、uLua\(asset store\)

• uLua插件原生版本，开山鼻祖

• 不会产生静态代码

• 反射机制，效率低下，速度慢，gcalloc频繁

• 已停止更新维护，不支持Unity5.x，淡出主流

#### 4.2、uLua & cstoLua

• 开发平台成熟稳定，Bug修复迅速

• 开发者众多，资源丰富

• 静态方法，性能优

• 有成功商业产品案例（啪啪三国、超神战队、酷鱼吧捕鱼、绝地战警、这不是刀塔等）

• 都是基于原生版本的改进；未来，两者会合并成一个插件

开源项目地址：

[https://github.com/topameng/CsToLua](https://github.com/topameng/CsToLua)

#### 4.3、sLua

• 静态方法，性能优

• 核心代码简洁

• 资源较少，开发平台不够成熟稳定

• 无成功商业产品案例成功商业产品案例

• 基于原生版本的改进

开源项目地址：

[https://github.com/pangweiwei/slua](https://github.com/pangweiwei/slua)

#### 4.4、C\#Light\(L\#\)

• 淡出主流，想要了解的小伙伴点击这里：

[https://github.com/lightszero/LSharp](https://github.com/lightszero/LSharp)

#### 4.5、 uniLua

• c\#实现的Lua虚拟机，非完整方案

• 淡出主流

#### 4.6、xLua

xLua是Unity3D下Lua编程解决方案，自2016年初推广以来，已经应用于十多款腾讯自研游戏，凭借其出色的性能，易用性，扩展性而广受好评。

xLua在功能、性能、易用性都有不少突破，这几方面分别最具代表性的是：

Unity3D全平台热补丁技术，可以运行时把C#实现（方法，操作符，属性，事件，构造函数，析构函数，支持泛化）替换成lua实现；
自定义struct，枚举在Lua和C#间传递无C# gc alloc；
编辑器下无需生成代码，开发更轻量；

**综合来看 肯定是 xLua 会更好一些，uLua次之。**

    第二章介绍uLua在Unity中的框架LuaFramework。
    第三章介绍xLua在Unity中的框架。
 

## 五、实践

熟悉NGUI的小伙伴可以参考这里：

[https://github.com/jarjin/SimpleFramework\_NGUI](https://github.com/jarjin/SimpleFramework_NGUI)

熟悉UGUI的小伙伴可以参考这里：

[https://github.com/jarjin/LuaFramework\_UGUI](https://github.com/jarjin/LuaFramework_UGUI)

🔚

