##Lua 程序设计


&emsp;&emsp;

&emsp;&emsp;1、C语言高超的性能、对底层操作的能力及第三方软件间的接口。

&emsp;&emsp;2、Lua擅长相对于硬件的高层抽象、动态结构、无冗余（No Redundancy）、简易的测试和调试。

&emsp;&emsp;3、Lua实现了一个安全的运行环境、一套自动内存管理机制、优秀的字符串处理能力和动态大数据的处理能力。

&emsp;&emsp;4、Lua的主要特性就是可扩展性。

&emsp;&emsp;5、动态类型为多态提供了支持。

&emsp;&emsp;6、自动内存管理简化了接口，从而无须决定谁分配内存、谁释放内存及如何处理溢出等。

&emsp;&emsp;7、高阶（Higher-order）函数和匿名函数允许实现更高层的参数化，能使函数变革更加通用。

&emsp;&emsp;8、Lua除了是一门可扩展语言，还是一门“胶水语言”。

&emsp;&emsp;9、支持基于组件的软件开发方法，这种方法可以通过粘合现有的高层组件来创建新的应用程序。

####&emsp;&emsp;特性：

```
        可扩展性
        简易性
        高效
```      



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