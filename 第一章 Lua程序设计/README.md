##Lua 程序设计


&emsp;&emsp;
####特点：
	
&emsp;&emsp;设计目标：能与C语言或其他常用语言相互集成。这样做可以带来很多好处。

&emsp;&emsp;1、简单   

&emsp;&emsp;因为它不需要去做其他语言已经做的很好的方面。

&emsp;&emsp;而Lua提供的特性则是C语言不太擅长的：

	    相对于硬件的高层抽象
	    动态结构
	    无冗余
	    简易的测试和调试

&emsp;&emsp;Lua提供了：

	    安全的运行环境
	    一套自动内存管理机制：不用管分配内存、释放内存、内存溢出等。
	    优秀的字符串处理能力
	    动态大小数据的处理能力：动态类型，支持多态
	    高级函数和匿名函数：允许更高层的参数化，能使函数变得更加通用

&emsp;&emsp;2、可扩展性
&emsp;&emsp;语言特性体现了：自动内存管理、高级函数和匿名函数

&emsp;&emsp;3、“胶水语言”
&emsp;&emsp;基于组件的开发：可以通过黏合现在的高层组件来创建新的应用程序
&emsp;&emsp;这些组件可以是已编译好的，也可以是静态类型语言（C、C++）编写的。
&emsp;&emsp;Lua则可以成为组织和连接各种组件的胶水。

&emsp;&emsp;一个项目：窗口、数据结构 很少修改
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;有一部分内容可能会反复修改，则这部分内容可以使用Lua开发。


&emsp;&emsp;● 有很多脚本语言，但是Lua提供了一组特性，使它与众不同，成为解决许多问题的首选语言：

		可扩展性：既可用Lua代码来扩展，又可以用外部的C代码来扩展。
		（Lua的大部分基础功能就是通过外部库实现的）。可以集成到C/C++、Java、C#等语言中。

		简易性：简单、小巧。概念不多，但每个概念都很有用。易于学习。
		
		高效：脚本（解释型）语言中最快的语言之一。
		
		可移植性：可运行在任何平台上。编译只依赖于ANSI（ISO）C标准
    



&emsp;&emsp;
&emsp;&emsp;

###语法规范

&emsp;&emsp;

**● 标识符可以是字母、数字和下划线，但不能以数字开头**

        避免用下划线跟着一个或多个大写字母
        一个下划线作为“哑变量”使用
        区分大小写
        不能是保留字

**● 单行注释**

```lua
        --
```

**● 块注释**

```lua
        --[[ ... ]]
        
        例如：
                不执行
                --[[
                print(10)
                --]]
                
                可执行
                ---[[
                print(10)
                --]]
```

**● 全局变量**

```lua
        不需要声明。只需将一个值赋予一个全局变量就可以创建了。

        例如：
                print(b) --> nil        访问一个未初始化的变量不会引发错误，访问结果是一个特殊的值nil。

                b = 10
                print(b) --> 10
                
                b=nil                   删除一个全局变量
                print(b) --> nil        如果存在一个全局变量，那么它必定具有一个非nil的值。
```




&emsp;&emsp;

&emsp;&emsp;

&emsp;&emsp;

###Lua环境配置
&emsp;&emsp;
&emsp;&emsp;**● 下载Lua：**

                www.lua.org
        
&emsp;&emsp;**● Mac终端安装：**

                make macosx
                sudo make install
                lua -v        （测试是否安装成功）
 
&emsp;&emsp;
&emsp;&emsp;**● Windows下源码编译**
新建VC++解决方案，在该解决方案下建3个项目，分别是lua库项目，lua编译器项目，lua解释器项目。

		1). 建lua库项目：Lua53
			选择Dll和空项目。
			编译模式改成release模式
			把源码中src目录下的所有.h文件拷贝到项目中的头文件节点下
			把源码中src目录下的除了lua.c和luac.c所有.c文件加入工程下的源文件目录
			然后设置项目属性，更改项目编译库类型为 静态库(.lib)
			最后编译运行，在项目文件夹release目录下产生了Lua53.lib文件，即lua库生成成功。

		2). 建lua编译器项目：Luac
			选择控制台应用程序和空项目
			编译模式改成release模式
			把源码中src目录下的所有.h文件拷贝到项目中的头文件节点下
			把源码中src目录下的除了lua.c所有.c文件加入工程下的源文件目录
			然后设置项目属性，更改项目编译库类型为 应用程序（.exe）
			最后编译运行，在项目文件夹release目录下产生了Luac.exe文件，即lua编译器文件生成成功。

		3). 建lua解释器项目：Lua
			步骤同建编译器项目，区别是添加src源文件时，不添加luac.c文件。
			编译完成后release目录下多了一个lua.exe文件。即lua解释器文件生成成功。
		4). 把编译好的release目录下的文件拷贝到 C:/Program Files (x86)/Lua5.3 目录下，并修改环境变量，把此路径添加进去。


       
&emsp;&emsp;**● sublime 设置：**

&emsp;&emsp;方法1:

                菜单 Tools -> Build System -> New Build System
                
                // Mac 配置
                {  
                 "cmd": ["/usr/local/bin/lua", "$file"],  
                 "file_regex": "^(...*?):([0-9]*):?([0-9]*)",  
                 "selector": "source.lua"  
                }  
                
                // Windows 配置
                { 
	                "cmd": ["C:/Program Files (x86)/Lua5.3/luac.exe", "$file"], 
	                 "file_regex": "^(?:lua:)?[\t ](...*?):([0-9]*):?([0-9]*)", 
	                 "selector": "source.lua" 
                }

&emsp;&emsp;方法2:

        View -> Show Console
        在控制台输入以下代码：

```        
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

&emsp;&emsp;

&emsp;&emsp;**● Sublime Text2 可选插件安装：**

```
        Tools -> Command Palette ...
        
        输入：
        install package 
        JsFormat
        
        install package 
        LuaJumpDefinition
        
        install package
        Tariling Spaces
        
        install package
        Terminal
        
        install package
        Fix Mac Path

        install package
        FormatLua

        install package
        LuaExtended
```        
        
&emsp;&emsp;
&emsp;&emsp;**● 运行：**

        command + b
        F7
        
&emsp;&emsp;
&emsp;&emsp;**● Sublime Text2 激活码：**

        —– BEGIN LICENSE —–
        Michael Barnes
        Single User License
        EA7E-821385
        8A353C41 872A0D5C DF9B2950 AFF6F667
        C458EA6D 8EA3C286 98D1D650 131A97AB
        AA919AEC EF20E143 B361B1E7 4C8B7F04
        B085E65E 2F5F5360 8489D422 FB8FC1AA
        93F6323C FD7F7544 3F39C318 D95E6480
        FCCC7561 8A4A1741 68FA4223 ADCEDE07
        200C25BE DBBC4855 C4CFB774 C5EC138C
        0FEC1CEF D9DCECEC D3A5DAD1 01316C36
        —— END LICENSE ——
        




🔚