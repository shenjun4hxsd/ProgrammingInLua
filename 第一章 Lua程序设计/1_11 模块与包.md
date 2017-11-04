##模块与包


&emsp;&emsp;通常，Lua不会设置规则（policy）。相反，Lua会提供许多强有力的机制来使开发者有能力实现出最适合的规则。然而，这种方法对于模块就不可行了。模块系统的一个主要目标是允许以不同的形式来共享代码。但若没有一项公共的规则就无法实现这样的共享。


&emsp;&emsp;Lua从5.1开始，为模块和包（`package`）定义了一系列的规则。这些规则不需要语言引入额外的技能，程序员可以使用他们早已熟知的`table`、函数、元表和环境来实现这些规则。然而，有两个重要的函数可以很容易通过这些规则，它们是`require`（用于使用模块）和`module`（用于创建模块）。程序员完全可以使用不同的规则来重新实现这两个函数。但是，新的实现可能会使程序无法使用外部模块，或者编写的模块无法被外部程序所使用。


&emsp;&emsp;从用户观点来看，一个模块就是一个程序库，可以通过`require`来加载。然后便得到了一个全局变量，表示一个`table`。这个`table`就像是一个名称空间，其内容就是模块中导出的所有东西，例如函数和常量。一个规范的模块还应使`require`返回这个`table`。


&emsp;&emsp;使用table来实现模块的优点在于，可以像操作普通table那样来操作模块，并且能利用Lua现有的功能来实现各种额外的功能。在大多数语言中，模块不是“`第一类值（first-class value）`”，所以那些语言需要为模块实现一套专门的机制。在Lua中，可以轻易地实现所有这些功能。


&emsp;&emsp;例如，一个用户要调用一个模块中的函数。其中最简单的方法是：

```lua
    require "mod"
    mod.foo()
```


&emsp;&emsp;如果希望使用较短的模块名称，则可以为模块设置一个局部名称：

```lua
    local m = require "mod"
    m.foo()
```


&emsp;&emsp;还可以为个别函数提供不同的名称：

```lua
    require "mod"
    local f = mod.foo
    f()
```


&emsp;&emsp;上述这些方法，都不需要来自于语言的显式支持，只需使用语言现有的内容。


&emsp;&emsp;

####require函数


&emsp;&emsp;Lua提供了一个名为`require`的高层函数用来加载模块，但这个函数只假设了关于模块的基本概念。对于`require`而言，一个模块就是一段定义了一些值的代码。


&emsp;&emsp;要加载一个模块，只需简单地调用`require "<模块名>"`。该调用会返回一个由模块函数组成的`table`，并且还会定义一个包含该`table`的全局变量。然而，这些行为都是由模块完成的，而非`require`。所以，有些模块会选择返回其他值，或者具有其他的效果。


&emsp;&emsp;即使知道某些用到的模块可能已加载了，但只要用到`require`就是一个良好的编程习惯。可以将标准库排除在此规则之外，因为Lua总是会预先加载它们。不过，有些用户还是喜欢为标准库中的模块使用显式的`require`：

```lua
    local m = require "io"
    m.write("hello world\n")
```


&emsp;&emsp;以下代码详细说明了`require`的行为：

```lua
    function require(name)
        if not packag.loaded[name] then        -- 模块是否已加载？
            local loader = findloader(name)
            if loader == nil then
                error("unable to load module " .. name)
            end
            package.loaded[name] = true        -- 将模块标记为已加载
            local res = loader(name)           -- 初始化模块
            if res ~= nil then
                package.loaded[name] = res
            end
        end
        return package.loaded[name]
    end
```


&emsp;&emsp;首先，它在`table package.loaded`中检查模块是否已加载。如果是的话，`require`就返回相应的值。因此，只要一个模块已加载过，后续的`require`调用都将返回同一个值，不会再次加载它。


&emsp;&emsp;如果模块尚未加载，`require`就试着为该模块找一个加载器（`loader`），会先在`table package.preload`中查询传入的模块名。如果在其中找到了一个函数，就以该函数作为模块的加载器。通过这个`preload table`，就有了一种通用的方法来处理各种不同的情况。通常这个`table`中不会找到有关指定模块的条目，那么`require`就会尝试从Lua文件或C程序库中加载模块。


&emsp;&emsp;如果`require`为指定模块找到了一个Lua文件，它就通过`loadfile`来加载该文件。而如果找到的是一个C程序库，就通过`loadlib`来加载。注意，`loadfile`和`loadlib`都只是加载了代码，并没有运行它们。为了运行代码，`require`会以模块名作为参数来调用这些代码。如果加载器有返回值，`require`就将这个返回值存储到`table package.loaded`中，以此作为将来对同一模块调用的返回值。如果加载器没有返回值，`require`就会返回`table package.loaded`中的值。在本章后面会看到，一个模块还可以将返回给`require`的值直接放入`package.loaded`中。


&emsp;&emsp;上述代码中还有一个重要的细节，就是在调用加载器前，`require`先将`true`赋予了`package.loaded`中的对应字段，以此将模块标记为已加载。这是因为如果一个模块要求加载另一个模块，而后者又要递归地加载前者。那么后者的`require`调用就会马上返回，从而避免了无限循环。


&emsp;&emsp;若要强制使`require`对用一个库加载两次的话，可以简单地删除`package.loaded`中的模块条目。例如，在成功地`require "foo"`后，`package.loaded["foo"]`就不为`nil`了。下面代码就可以再次加载该模块：

```lua
    package.loaded["foo"] = nil
    require "foo"
```


&emsp;&emsp;在搜索一个文件时，`require`所使用的路径与传统的路径有所不同。大部分程序所使用的路径就是一连串目录，指定了某个文件的具体位置。然而，ANSIC却没有任何关于目录的概念。所以，`require`采用的路径是一连串的模式（`pattern`），其中每项都是一种将模块名转换为文件名的方式。进一步说，这种路径中的每项都是一个文件名，每项中还包含一个可选的问号。`require`会用模块名来替换每个“?”，然后根据替换的结果来检查是否存在这样一个文件。如果不存在，就会尝试下一项。路径中的每项以分号隔开。例如，假设路径为：

```lua
    ?;?.lua;c:\windows\?;/usr/local/lua/?/?.lua
```


&emsp;&emsp;那么，调用`require "sql"`就会试着打开以下文件：

```lua
    sql
    sql.lua
    c:\windows\sql
    /usr/local/lua/sql/sql.lua
```


&emsp;&emsp;`require`函数只处理了分号（作为各项之间的分隔符）和问号。其他例如目录分隔符或文件扩展名，都由路径自己定义。


&emsp;&emsp;`require`用于搜索Lua文件的路径存放在变量`package.path`中。当Lua启动后，便以环境变量`LUA_PATH`的值来初始化这个变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。在使用`LUA_PATH`时，Lua会将其中所有的子串“;;”替换成默认路径。例如，假设`LUA_PATH`为“`mydir/?.lua;;`”，那么最终路径就是“`mydir/?.lua`”，并紧随默认路径。

&emsp;&emsp;如果`require`无法找到与模块名相符的Lua文件，它就会找C程序库。这类搜索会从变量`package.cpath`（相对于`package.path`）获取路径。而这个变量则是通过环境变量`LUA_CPATH`（相对于`LUA_PATH`）来初始化。在UNIX中，它的值一般是这样的：

```lua
    ./?.so;/usr/local/lib/lua/5.1/?.so
```


&emsp;&emsp;注意，文件的扩展名是由路径定义的（例如，上例中使用的.so）。而在Windows中，此路径通常可以是这样的：

```lua
    .\?.dll;C:\Program Files\Lua501\dll\?.dll
```

&emsp;&emsp;当找到一个C程序库后，`require`就会通过`package.loadlib`来加载它，`loadlib`在前面章节中已讨论过。C程序库与Lua程序块是不同的，它没有定义一个单一的主函数，而是导出了几个C函数。具有良好行为的C程序库应该导出一个名为“`luaopen_<模块名>`”的函数。`require`会在链接完程序库后，尝试调用这个函数。将在后续章节中讨论如何编写C程序库。

&emsp;&emsp;一般通过模块的名称来使用它们。但有时必须将一个模块改名，以避免冲突。一种典型的情况是，在测试中需要加载同一模块的不同版本。对于一个Lua模块来说，其内部名称不是固定的，可以轻易地编辑它以改变其名称。但是却无法编辑一个二进制数据模块中`luaopen_*函数的名称`。为了允许这种重命名，`require`用到了一个小技巧：如果一个模块名中包含了连字符，`require`就会用连字符后的内容来创建`luaopen_*函数名`。例如，若一个模块名为`a-b`，`require`就认为它的`open`函数名为`luaopen_b`，而不是`luaopen_a-b`。因此，如果要使用的两个模块名都为`mod`，那么可以将其中一个重命名为`v1-mod`（或者`-mod`，或其他类似形式）。当调用`m1 = require "v1-mod"`时，`require`会找到改名后的文件`v1-mod`，并将其中的函数`luaopen_mod`作为`open`函数。

&emsp;&emsp;

####编写模块的基本方法

&emsp;&emsp;在Lua中创建一个模块最简单的方法是：创建一个`table`，并将所有需要导出的函数放入其中，最后返回这个`table`。以下代码演示这种方法。注意，将`inv`声明为程序块的局部变量，就是将其定义成一个私有的名称。

```lua
    complex = {}

    function complex.new(r, i) return {r=r, i=i} end

    -- 定义一个常量'i'
    complex.i = complex.new(0, 1)

    function complex.add(c1, c2)
        return complex.new(c1.r  c2.r, c1.i + c2.i)
    end

    function complex.sub(c1, c2)
        return complex.new(c1.r - c2.r, c1.i - c2.i)
    end

    function complex.mul(c1, c2)
        return complex.new(c1.r*c2.r - c1.i*c2.i,
                           c1.r*c2.i + c1.i*c2.r)
    end

    local function inv(c)
        local n = c.r^2 + c.i^2
        return complex.new(c.r/n, -c.i/n)
    end

    function complex.div(c1, c2)
        return complex.mul(c1, inv(c2))
    end

    return complex
```

&emsp;&emsp;上例中使用`table`编写模块时，没有提供与真正模块完全一致的功能性，首先，必须显式地将模块名放到每个函数定义中。其次，一个函数在调用同一模块中的另一个函数时，必须限定被调用函数的名称。可以使用一个固定的局部名称（例如`M`）来定义和调用模块内的函数，然后将这个局部名称赋予模块的最终名称。通过这种方法，可以将上例改写为：

```lua
    local M = {}
    complex = M 					-- 模块名

    M.i = {r=0, i=1}
    function M.new(r, i) return {r=r, i=i} end

    function M.add(c1, c2)
        return M.new(c1.r + c2.r, c1.i + c2.i)
    end
    <如前>
```

&emsp;&emsp;只要一个函数调用了同一模块中另一个函数（或者递归地调用自己），就仍需要一个前缀名称。但至少两个函数之间的连接不再需要依赖模块名，并且也只需在整个模块中的一处写出模块名。实际上，可以完全避免写模块名，因为`require`会将模块名作为参数传给模块：

```lua
    local modname = ...
    local M = {}
    _G[modname] = M

    M.i = {r=0, i=1}
    <如前>
```

&emsp;&emsp;经过这样的修改，若需要重命名一个模块，只需重命名并定义它的文件就可以了。

&emsp;&emsp;另一项小改进与结尾的`return`语句有关。若能将所有与模块相关的设置任务集中在模块开头，会更好。消除`return`语句的一种方法是，将模块`table`直接赋予`package.loaded`：

```lua
    local modname = ...
    local M = {}
    _G[modname] = M
    package.loaded[modname] = M
    <如前>
```

&emsp;&emsp;通过这样的赋值，就不需要在模块结尾返回`M`了。注意，如果一个模块无返回值的话，`require`就会返回`package.loaded[modname]`的当前值。

&emsp;&emsp;

####使用环境

&emsp;&emsp;创建模块的基本方法的缺点在于，它要求程序员投入一些额外的关注。当访问同一模块中的其他公共实体时，必须限定其名称。并且，只要一个函数的状态从私有改为公有（或从公有改为私有），就必须修改调用。另外，在私有声明中也很容易忘记关键字`local`。

&emsp;&emsp;“函数环境”是一种有趣的技术，它能够解决所有上述创建模块时遇到的问题。基本想法就是让模块的主程序块有一个独占的环境。这样不仅它的所有函数都可共享这个`table`，而且它的所有全局变量也都记录在这个`table`中。还可以将所有公有函数声明为全局变量，这样它们就都自动地记录在一个独立的`table`中了。模块所要做的就是将这个`table`赋予模块名和`package.loaded`。以下代码片段演示了这种技术：

```lua
    local modname = ...
    local M = {}
    _G[modname] = M
    package.loaded[modname] = M
    setfenv(1, M)
```

&emsp;&emsp;此时，当声明函数`add`时，它就称为了`complex.add`：

```lua
    function add(c1, c2)
        return new(c1.r + c2.r, c1.i + c2.i)
    end
```

&emsp;&emsp;此外，在调用同一模块的其他函数时，也不再需要前缀。例如，`add`会从其环境中得到`new`，也就是`complex.new`。

&emsp;&emsp;这种方法为模块提供了一种良好的支持，并且只引入了一点额外的工作。此时完全不需要前缀，并且调用一个导出的函数与调用一个私有函数没有什么区别。如果程序员忘记了写local关键字，那么也不会污染全局名称空间。只会将一个私有函数变成了公有而已。

&emsp;&emsp;还缺少什么？是的，那就是访问其他模块。当创建了一个空`table M`作为环境后，就无法访问前一个环境中全局变量了。以下提出几种重获访问的方法，每种方法各有其优缺点。

&emsp;&emsp;最简单的方法是继承，就像之前看到的那样：

```lua
    local modname = ...
    local M = {}
    _G[modname] = M
    package.loaded[modname] = M
    setmetatable(M, {__index = _G})
    setfenv(1, M)
```

&emsp;&emsp;必须先调用`setmetatable`再调用`setfenv`，因为通过这种方法，模块就能直接访问任何全局标识了，每次访问只需付出很小的开销。这种方法导致了一个后果，即从概念上说，此时的模块中包含了所有的全局变量。例如，某人可以通过你的模块来调用标准的正弦函数：`complex.math.sin(x)`。

&emsp;&emsp;还有一种更便捷的方法来访问其他模块，即声明一个局部变量，用以保存对旧环境的访问：

```lua
    local modname = ...
    local M = {}
    _G[modname] = M
    package.loaded[modname] = M
    local _G = _G
    setfenv(1, M)
```

&emsp;&emsp;此时必须在所有全局变量的名称前加“`_G.`”。由于没有涉及到元方法，这种访问会比前面的方法略快。

&emsp;&emsp;一种更正规的方法是将那些需要用到的函数或模块声明为局部变量：

```lua
    -- 模块设置
    local modname = ...
    local M = {}
    _G[modname] = M
    package.loaded[modname] = M

    -- 导入段：
    -- 声明这个模块从外界所需的所有东西
    local sqrt = math.sqrt
    local io = io

    -- 在这句之后就不再需要外部访问了
    setfenv(1, M)
```

&emsp;&emsp;这种技术要求做更多的工作，但是它能清晰地说明模块的依赖性。同时，较之前面的两种方法，它的运行速度也更快。

&emsp;&emsp;

####module函数

&emsp;&emsp;读者或许注意到了，前面几个示例中的代码形式。它们都以相同的模式开始：

```lua
    local modname = ...
    local M = {}
    _G[modname] = M
    package.loaded[modname] = M
    <setup for external access>
    setfenv(1, M)
```

&emsp;&emsp;Lua5.1提供了一个新函数`module`，它囊括了以上这些功能。在开始编写一个模块时，可以直接用以下代码来取代前面的设置代码：

```lua
    module(...)
```

&emsp;&emsp;这句调用会创建一个新的`table`，并将其赋予适当的全局变量和`loaded table`，最后还会将这个`table`设为主程序块的环境。

&emsp;&emsp;默认情况下，`module`不提供外部访问。必须在调用它前，为需要访问的外部函数或模块声明适当的局部变量。也可以通过继承来实现外部访问，只需在调用`module`时加一个选项`package.seeall`。这个选项等价于以下代码：

```lua
    setmetatable(M, {__index = _G})
```

&emsp;&emsp;因而只需这么做：

```lua
    module(..., package.seeall)
```

&emsp;&emsp;在一个模块文件的开头有了这句调用后，后续所有的代码都可以像普通的Lua代码那样编写了。不需要限定模块名和外部名字，同样也不需要返回模块`table`。要做的只是加上这么一句调用。

&emsp;&emsp;`module`函数还提供了一些额外的功能。虽然大部分模块不需要这些功能，但有些发行模块可能需要一些特殊处理（例如，一个模块中同时包含C函数和Lua函数）。`module`在创建模块`table`之前，会先检查`package.loaded`是否已包含了这个模块，或者是否已存在与模块同名的变量。如果`module`由此找到了这个`table`，它就会复用该`table`作为模块。也就是说，可以用`module`来打开一个已创建的模块。如果没有找到模块`table`，`module`就会创建一个模块`table`。然后在这个`table`中设置一些预定义的变量，包括：`_M`，包含了模块`table`自身（类似于`_G`）；`_NAME`，包含了模块名（传给`module`的第一个参数）；`_PACKAGE`，包含了包（`package`）的名称。

&emsp;&emsp;


####子模块与包

&emsp;&emsp;Lua支持具有层级性的模块名，可以用一个点来分隔名称中的层级。假设，一个模块名为`mod.sub`，那么它就是`mod`的一个子模块。因此，可以认为模块`mod.sub`会将其所有值都定义在`table mod.sub`中，也就是一个存储在`table mod`中且`key`为`sub`的`table`。一个“包（Package）”就是一个完整的模块树，它是Lua中发行的单位。

&emsp;&emsp;当`require`一个模块`mod.sub`时，`require`会用原始的模块名“`mod.sub`”作为`key`来查询`table package.loaded`和`package.preload`，其中，模块名中的点在搜索中没有任何含义。

&emsp;&emsp;然而，当搜索一个定义子模块的文件时，`require`会将点转换为另一个字符，通常就是系统的目录分隔符。转换之后`require`就像搜索其他名称一样来搜索这个名称。例如，假设路径为：

```lua
    ./?.lua;/usr/local/lua/?.lua;/usr/local/lua/?/init.lua
```

&emsp;&emsp;并且目录分隔符为“`/`”，那么调用`require "a.b"`就会尝试打开以下文件：

```lua
    ./a/b.lua
    /usr/local/lua/a/b.lua
    /usr/local/lua/a/b/init.lua
```

&emsp;&emsp;通过这样的加载策略，就可以将一个包中的所有模块组织到一个目录中。例如，一个包中有模块`p`、`p.a`和`p.b`，那么它们对应的文件名就分别为`p/init.lua`，`p/a.lua`和`p/b.lua`，它们都是目录`p`下的文件。

&emsp;&emsp;Lua使用的目录分隔符是编译时配置的，可以是任意的字符串。例如，在没有目录层级的系统中，就可以使用“`_`”作为“目录分隔符”。那么`require "a.b"`就会搜索到文件`a_b.lua`。

&emsp;&emsp;C函数名中不能包含点，因此一个用C编写的子模块`a.b`无法导出函数`luaopen_a.b`。所以，`require`会将点转换为下划线。例如，一个名为`a.b`的C程序库就应将其初始化函数命名为`luaopen_a_b`。在此又可以巧用连字符，来实现一些特殊的效果。例如，有一个C程序库`a`，现在想将它作为`mod`的一个子模块，那么就可以将文件名改为`mod/-a`。当执行`require "mod.-a"`时，`require`就会找到改名后的文件`mod/-a`及其中的函数`luaopen_a`。

&emsp;&emsp;作为一项扩展功能，`require`在加载C子模块时还有一些选项。当`require`加载子模块时，无法找到对应的Lua文件或C程序库。它就会再次搜索C路径，不过这次将以包的名称来查找。例如，一个程序`require`子模块`a.b.c`，当无法找到文件`a/b/c`时，再次搜索就会找到文件`a`。如果找到了C程序库`a`，`require`就查看该程序库中是否有`open`函数`luaopen_a_b_c`。这项功能使得一个发行包可以将几个子模块组织到一个单一C程序库中，并且具有各自的`open`函数。

&emsp;&emsp;`module`函数也为子模块提供了显式的支持。当我们创建一个子模块时，调用`module ("a.b.c")`，`module`就会将环境`table`放入变量`a.b.c`，也就是“`table a中的table b中的table c`”。如果这些中间的`table`不存在，`module`就会创建它们。否则，就复用它们。

&emsp;&emsp;从Lua的观点看，同一个包中的子模块除了它们的环境`table`是嵌套的之外，它们之间并没有显式的关联性。`require`模块`a`并不会自动地加载它的任何子模块。同样，`require`子模块`a.b`也并不会自动地加载`a`。当然只要愿意，包的实现者完全可以实现这种关联。例如，模块`a`的一个子模块在加载时会显式地加载`a`。



🔚