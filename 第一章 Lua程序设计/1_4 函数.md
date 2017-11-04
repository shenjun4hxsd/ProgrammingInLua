##函数

```
    • 多重返回值
    • 变长参数
    • 具名实参
    • 深入函数
        • closure（闭合函数）
        • 非全局的函数
        • 正确的尾调用
```

&emsp;&emsp;

&emsp;&emsp;一个函数若只有一个参数，并且此参数是一个字面字符串或`table`构造式，那么圆括号便是可有可无的。

```lua
    -- 例如：
    
    print "Hello World"    <-->    print("Hello World")
    dofile 'a.lua'         <-->    dofile('a.lua')
    print [[a multi-line   <-->    print([[a multi-line
      message]]                      message]]
    f{x=10, y=20}          <-->    f({x=10, y=20})
    type{}                 <-->    type({})
```

&emsp;&emsp;Lua为面向对象式的调用也提供了一种特殊的语法---冒号操作符。

```lua
    o.foo(o, x)      -- 另一种写法是 o::foo(x)
```

&emsp;&emsp;冒号操作符使调用`o.foo`时将`o`隐含地作为函数第一个参数。
&emsp;&emsp;

>&emsp;&emsp;一个Lua程序既可以使用以Lua编写的函数，又可以调用以C语言编写的函数，调用时没有任何区别。

&emsp;&emsp;
&emsp;&emsp;调用函数时提供的实参数量可以与形參数量不同。Lua会自动调整实参的数量，以匹配参数表的要求。

```lua
    -- 假设一个函数如下：
    function f(a, b) return a or b end
    
    -- 在以下几种调用中，实参与形參的对应关系为：
    -- 调用        形參
    f(3)          a=3, b=nil
    f(3, 4)       a=3, b=4
    f(3, 4, 5)    a=3, b=4 (5被丢弃了) 
```

```lua
    -- 默认实参的应用
    
    function incCount(n)
        n = n or 1
        count = count + n
    end
```

&emsp;&emsp;

###&emsp;&emsp;• 多重返回值

&emsp;&emsp;Lua允许函数返回多个结果。例如，用于字符串中定位一个模式的函数`string.find`。该函数若在字符串中找到了指定的模式，将返回匹配的起始字符和结尾字符的索引。

```lua
    startIndex, endIndex = string.find("hello Lua users", "Lua")
    print(startIndex, endIndex)        --> 7   9
```
&emsp;&emsp;
&emsp;&emsp;以Lua编写的函数同样可以返回多个结果，只需在`return`关键字后列出所有的返回值即可。

```lua
    -- 查找数组中的最大元素，并返回该元素的位置：
    
    function maximum(a)
        local index = 1         -- 最大值的索引
        local max = a[index]    -- 最大值
        for i,val in ipairs(a) do
            if val > max then
                max = val; index = i
            end
        end
        return max, index
    end
    
    print(maximum{8, 10, 23, 12, 5})    --> 23   3
```

&emsp;&emsp;Lua会调整一个函数的返回值数量以适应不同的调用情况。若将函数调用作为一条单独语句时，Lua会丢弃函数的所有返回值。若将函数作为表达式的一部分来调用时，Lua只保留函数的第一个返回值。**只有当一个函数调用是一系列表达式中的最后一个元素（或仅有一个元素）时，才能获得它的所有返回值。**
&emsp;&emsp;
&emsp;&emsp;这里所谓的“一系列表达式”在Lua中表现为4种情况：多重赋值、函数调用时传入的实参列表、`table`的构造式和`return`语句。

```lua
    function foo0() end                    -- 无返回值
    function foo1() return "a" end         -- 返回1个结果
    function foo2() return "a", "b" end    -- 返回2个结果
```

```lua
    x,y = foo2()                           -- x="a", y="b"
    x = foo2()                             -- x="a", "b"被丢弃
    x,y,z = 10, foo2()                     -- x=10, y="a", z="b"
    
    x,y = foo0()                           -- x=nil, y=nil
    x,y = foo1()                           -- x="a", y=nil
    x,y,z = foo2()                         -- x="a", y="b", z=nil
    
    -- 函数调用不是表达式最后一个元素，将只产生一个值：
    x,y = foo2(), 20                       -- x="a", y=20
    x,y = foo0(), 20, 30                   -- x=nil, y=20, 30被丢弃
    
    -- 函数调用作为另一个函数调用的最后一个（或仅有的）实参时，第一个函数的所有返回值都将作为实参传入第二个函数。如print：
    
    print(foo0())            -->
    print(foo1())            --> a
    print(foo2())            --> a  b
    print(foo2(), 1)         --> a  1
    print(foo2() .. "x")     --> ax
```

&emsp;&emsp;`table`构造式可以完整地接收一个函数调用的所有结果：

```lua
    t = {foo0()}            -- t = {}  (一个空的table)
    t = {foo1()}            -- t = {"a"}
    t = {foo2()}            -- t = {"a", "b"}
```

&emsp;&emsp;不过，这种行为只有当一个函数调用作为最后一个元素时才会发生，而在其他位置上的函数调用总是产生一个结果值：

```lua
    t = {foo0(), foo2(), 4}        -- t[1] = nil, t[2] = "a", t[3] = 4
```

&emsp;&emsp;最后一种情况是`return`语句，诸如`return f()`这样的语句将返回`f`的所有返回值：

```lua
    function foo(i)
        if i==0 then return foo0();
        elseif i==1 then return foo1();
        elseif i==2 then return foo2()
        end
    end
    
    print(foo(1))        -- a
    print(foo(2))        -- a  b
    print(foo(0))        -- (无返回值)
    print(foo(3))        -- (无返回值)
```

&emsp;&emsp;也可以将一个函数调用放入一对圆括号中，从而迫使它只返回一个结果：

```lua
    print((foo0()))      -- nil
    print((foo1()))      -- a
    print((foo2()))      -- a
```

&emsp;&emsp;请注意`return`语句后面的内容是不需要圆括号的。例如`return (f(x))`，将只返回一个值，而无关乎`f`返回了几个值。

&emsp;&emsp;关于多重返回值还要介绍一个特殊函数 --`unpack`。它接受一个数组作为参数，并从下标1开始返回该数组的所有元素：

```lua
    print(unpack{10, 20, 30})            --> 10  20  30
    a,b = unpack{10, 20, 30}             --> a=10, b=20, 30被丢弃
```
>Lua5.1及之前的版本中，`unpack`作为全局函数使用，可以直接
>```lua
    unpack(arg)
```
>而5.2之后，`unpack`被移到了`table`下面，于是直接`unpack`就会导致报错，新的调用应该为：
>```lua
    table.unpack(arg)
```

>同时修改的还有另外一个：arg
以前 `...` 可以在函数内直接使用`arg`来处理，但是5.1之后，就需要自己手动变换成`arg`了

>```lua
    local arg = {...}
    print(arg[1])
```

>提供一种兼容的方法

>```lua
    function test(...)
        if arg ~= nil then
            arg = {...}
        end
        if unpack != nil then		-- 5.1及之前的版本
            print(unpack(arg))
        else						-- 之后的版本
            local arg = {...}
            print(table.unpack(arg))
        end
    end
```

>```lua
    f = string.find
    a = {"hello", "ll"}
```

>&emsp;&emsp;`f(unpack(a))`将返回3和4，这与直接调用`string.find("hello", "ll")` 所返回的结果一摸一样。

>&emsp;&emsp;虽然这个预定义函数`unpack`是用C语言直接编写的，但是仍可以在Lua中通过递归实现一样效果：

>```lua
    function unpack(t, i)
        i = i or 1
        if t[i] then
            return t[i], unpack(t, i+1)
        end
    end
```

&emsp;&emsp;

###&emsp;&emsp;• 变长参数

&emsp;&emsp;Lua中的函数可以接受不同数量的实参。

```lua
    -- 这个函数返回了所有参数的总和：
    
    function add( ... )
        local s = 0
        for i, v in ipairs( ... ) do  -- 表达式{...}表示一个由所有变长参数构成的数组。
            s = s + v
        end
        return s
    end
    
    print(add(3, 4, 10, 25, 12))        --> 54
```

&emsp;&emsp;参数中的3个点（`...`）表示该函数可接受不同数量的实参。
&emsp;&emsp;
&emsp;&emsp;Lua提供了专门用于格式化文本（`string.format`）和输出文本（`io.write`）的函数。

```lua
    function fwrite(fmt, ...)
    return io.write(string.format(fmt, ...))
```
&emsp;&emsp;注意在3个点前有一个固定参数`fmt`。具有变长参数的函数同样也可以拥有任意数量的固定参数，但固定参数必须放在变长参数之前。
&emsp;&emsp;
&emsp;&emsp;变长参数中可能会包含一些故意传入的`nil`，那么此时就需要用函数`select`来访问变长参数了。调用`select`时，必须传入一个固定实参`selector`(选择开关)和一系列变长参数。如果`selector`为数字`n`，那么`select`返回它的第`n`个可变实参；否则，`selector`只能为字符串“`#`”，这样`select`会返回变长参数的总数。

```lua
    for i=1, select('#', ...) do
        local arg = select(i, ...)		-- 得到第i个参数
        <循环体>
    end
```

&emsp;&emsp;特别需要指出的是，`select('#', ...)`会返回所有变长参数的总数，其中包括`nil`。

&emsp;&emsp;

###&emsp;&emsp;• 具名实参

```lua
    -- 无效的演示代码
    rename(old="temp.lua", new="temp1.lua")   
```

&emsp;&emsp;Lua并不直接支持这种语法，但可以通过一种细微的改变来获得相同的效果。主要是将所有实参组织到一个`table`中，并将这个`table`作为唯一的实参传给函数。另外，还需要用到一种Lua中特殊的函数调用语法，就是当实参只有一个`table`构造式时，函数调用中的圆括号是可有可无的：

```lua
    rename{old="temp.lua", new="temp1.lua"}
```

&emsp;&emsp;另一方面，将`rename`改为只接受一个参数，并从这个参数中获取实际的参数：

```lua
    function rename(arg)
        return os.rename(arg.old, arg.new)
    end
```

&emsp;&emsp;若一个函数拥有大量的参数，而其中大部分参数是可选的话，这种参数传递风格会特别有用。例如在一个GUI库中，一个用于创建新窗口的函数可能会具有许多的参数，而其中大部分都是可选的，那么最好使用具名实参：


```lua
    w = Window{ x=0, y=0, width=300, height=200, 
                title="Lua", background="blue", border=true
              }
```

&emsp;&emsp;`Window`函数可以根据要求检查一些必填参数，或者为某些参数添加默认值。假设“`_Window`”才是真正用于创建新窗口的函数，它要求所有参数以正确的次序传入，那么`Window`函数可以这么写：

```lua
    function Window(options)
        -- 检查必要的参数
        if type(options.title) ~= "string" then
            error("no tile")
        elseif type(options.width) ~= "number" then
            error("no width")
        end

        -- 其他参数都是可选的
        _Window(options.title,
                options.x or 0,            -- 默认值
                options.y or 0,            -- 默认值
                options.width, options.height,
                options.backgournd or "white",        -- 默认值
                options.border            -- 默认值为false(nil)
        )
    end
```

&emsp;&emsp;

###&emsp;&emsp;• 深入函数

&emsp;&emsp;函数可以存储到变量中（无论全局变量还是局部变量）或`table`中，可以作为实参传递给其他函数，还可以作为其他函数的返回值。

&emsp;&emsp;一个函数可以嵌套在另一个函数中，内部的函数可以访问外部函数中的变量。

&emsp;&emsp;函数与其他值一样都是匿名的。当讨论一个函数名时（例如`print`），实际上是在讨论一个持有某函数的变量。

```lua
    a = {p = print}
    a.p("Hello World")        --> Hello World
    print = math.sin          --> 'print'现在引用了正弦函数
    a.p(print(1))             --> 0.841470
    sin = a.p                 --> 'sin'现在引用了print函数
    sin(10, 20)               --> 10  20
```

```lua
    -- Lua中最常见的是函数编写方式，诸如：
    function foo(x) return 2*x end
```

&emsp;&emsp;只是一种所谓的“语法糖”而已。也就是说，这只是以下代码的一种简化书写形式：

```lua
    foo = function(x) return 2*x end
```

&emsp;&emsp;因此，一个函数定义实际就是一条语句（更准确地说是一条赋值语句），这条语句创建了一种类型为“函数”的值，并将这个值赋予一个变量。
&emsp;&emsp;可以将表达式`“function(x)<body>end”`视为一种函数的构造式，就像table的构造式`{}`一样。
&emsp;&emsp;将这种函数构造式的结果称为一个“匿名函数”。虽然一般情况下，会将函数赋予全局变量，即给予其一个名称。但在某些特殊情况下，仍会需要用到匿名函数。

&emsp;&emsp;
&emsp;&emsp;`table`库提供了一个函数`table.sort`，它接受一个`table`并对其中的元素排序。像这种函数就必须支持各种各样可能的排序准则，例如升序还是降序、按数字顺序还是按字符顺序或者按`table`中`key`的顺序等。`sort`函数并没有提供所有这些排序准则，而是提供了一个可选的参数，所谓“次序函数”。这个函数接受两个元素，并返回在有序情况下第一个元素是否应排在第二个元素之前。举例来说，假设有一个`table`内容如下：

&emsp;&emsp;假设有一个table内容如下：

```lua
    network = {
        {name = "grauna", IP = "210.26.30.34"},
        {name = "arraial", IP = "210.26.30.23"},
        {name = "lua", IP = "210.26.23.12"},
        {name = "derain", IP = "210.26.23.20"}
    }
```

&emsp;&emsp;如果想以`name`字段、按反向的字符顺序来对这个`table`排序的话，只需这么写：

```lua
    table.sort(network, function(a,b) return (a.name > b.name) end)
```

```lua
    for i, v in ipairs( network ) do
        print( v["name"] )
    end
```

&emsp;&emsp;可见匿名函数在这条语句中就显示出了极好的便捷性。

&emsp;&emsp;像`sort`这样的函数，接受另一个函数作为实参的，称其是一个“**高阶函数**”。高阶函数是一种强大的编程机制，应用匿名函数来创建高阶函数所需的实参则可以带来更大的灵活性。

&emsp;&emsp;

###&emsp;&emsp;• closure（闭合函数）

&emsp;&emsp;若将一个函数写在另一个函数之内，那么这个位于内部的函数便可以访问外部函数中的局部变量，这项特征称之为“词法域”。
&emsp;&emsp;
&emsp;&emsp;先来看一个简单的例子。假设有一个学生姓名的列表和一个对应于每个姓名的年级列表，需要根据每个学生的年级来对他们的姓名进行排序（由高到低）。可以这么做：

```lua
    name = {"Peter", "Paul", "Mary"} 
    grades = {Mary = 10, Paul = 7, Peter = 8}
    table.sort(names, function(n1, n2)
        return grades[n1] > grades[n2] 		-- 比较年级
    end)
```

&emsp;&emsp;现在假设要单独创建一个函数来做这项工作：

```lua
    function sortbygrade(names, grades)
        table.sort(names, function(n1, n2) return grades[n1] > grades[n2] end)
    end
```

&emsp;&emsp;在上例中有一点很有趣，传递给`sort`的匿名函数可以访问参数`grades`，而`grades`是外部函数`sortbygrade`的局部变量。在这个匿名函数内部，`grades`既不是全局变量也不是局部变量，将其称为一个“非局部的变量”。

&emsp;&emsp;
&emsp;&emsp;为什么在Lua中允许这种访问呢？原因在于函数是“第一类值”。考虑以下代码：

```lua
    function newCounter()
        local i = 0
        return function() i = i + 1 return i; end
    end

    c1 = newCounter()
    print(c1())		-->  1
    print(c2())		-->  2
```

&emsp;&emsp;在这段代码中，匿名函数访问了一个“非局部的变量”`i`，该变量用于保持一个计数器。
&emsp;&emsp;初看上去，由于创建变量i的函数（`newCounter`）已经返回，所以之后每次调用匿名函数时，`i`都应是已超出了作用范围的。但其实不然，Lua会以`closure`的概念来正确地处理这种情况。简单地讲，一个`closure`就是一个函数加上该函数所需访问的所有“非局部的变量”。如果再次调用`newCounter`，那么它会创建一个新的局部变量`i`，从而也将得到一个新的`closure`：

```lua
    c2 = newCounter()
    print(c2())		-->  1
    print(c1())		-->  3
    print(c2())		-->  2
```

&emsp;&emsp;因此`c1`和`c2`是同一个函数所创建的两个不同的`closure`，它们各自拥有局部变量`i`的独立实例。

&emsp;&emsp;从技术上讲，Lua中只有`closure`，而不存在“函数”。因为，函数本身就是一种特殊的`closure`。不过只要不会引起混淆，仍将采用术语“函数”来指代`closure`。

&emsp;&emsp;
&emsp;&emsp;`closure`在另一种情况中也非常有用。例如在Lua中函数是存储在普通变量中的，因此可以轻易地重新定义某些函数，甚至是重新定义那些预定义的函数。这也正是Lua相当灵活的原因之一。通常当重新定义一个函数的时候，需要在新的实现中调用原来的那个函数。举例来说，假设要重新定义函数`sin`，使其参数能使用角度来代替原来的弧度。那么这个新函数就必须得转换它的实参，并调用原来的`sin`函数完成真正的计算。这段代码可能是这样的：

```lua
    oldSin = math.sin
    math.sin = function(x)
        return oldSin(x*math.pi/180)
    end
```

还有一种更彻底的做法是这样的：

```lua
    do
        local oldSin = math.sin
        local k = math.pi/180
        math.sin=function(x)
            return oldSin(x*k)
        end
    end
```

&emsp;&emsp;将老版本的`sin`保存到了一个私有变量中，现在只有通过新版本的`sin`才能访问到它了。

&emsp;&emsp;
&emsp;&emsp;可以使用同样的技术来创建一个安全的运行环境，即所谓的“沙盒”。当执行一些未受信任的代码时就需要一个安全的运行环境，例如在服务器中执行那些从Internet上接收到的代码。举例来说，如果要限制一个程序访问文件的话，只需使用`closure`来重新定义函数`io.open`就可以了。

```lua
    do
        local oldOpen = io.open
        local access_OK = function(filename, mode)
            <检查访问权限>
        end
        io.open = function(filename, mode)
            if access_OK(filename, mode) then
                return oldOpen(filename, mode)
            else
                return nil, "access denied"
            end
        end
    end
```

&emsp;&emsp;这个示例的精彩之处在于，经过重新定义后，一个程序就只能通过新的受限版本来调用原来那个未受限的`open`函数了。示例将原来不安全的版本保存到了`closure`的一个私有变量中，从而使得外部再也无法直接访问到原来的版本了。通过这种技术，可以在Lua的语言层面上就构建出一个安全的运行环境，且不失简易性和灵活性。相对于提供一套大而全的解决方案，Lua提供的则是一套“元机制”，因此可以根据特定的安全需要来创建一个安全的运行环境。

&emsp;&emsp;

###&emsp;&emsp;• 非全局的函数

&emsp;&emsp;由于函数是一种“第一类值”，因此一个显而易见的推论就是，函数不仅可以存储在全局变量中，还可以存储在`table`的字段中和局部变量中。
前面讲到了几个将函数存储在`table`字段中的示例，大部分Lua库也采用了这种机制（例如`io.read`、`math.sin`）。若要在Lua中创建这种函数，只需将常规的函数语法与`table`语法结合起来使用即可：

```lua
    Lib = {}
    Lib.foo = function(x, y) return x + y end
    Lib.goo = function(x, y) return x - y end
```

当然，还可以使用构造式：

```lua
    Lib = {
        foo = function(x, y) return x + y end,
        goo = function(x, y) return x - y end
    }
```

&emsp;&emsp;除了这些之外，Lua还提供了另一种语法来定义这类函数：

```lua
    Lib = {}
    function Lib.foo(x, y) return x + y end
    function Lib.goo(x, y) return x - y end
```

&emsp;&emsp;只要将一个函数存储到一个局部变量中，即得到了一个“局部函数”，也就是说该函数只能在某个特定的作用域中使用。对于“程序包”而言，这种函数定义是非常有用的。因为Lua是将每个程序块作为一个函数来处理的，所以在一个程序块中声明的函数就是局部函数，这些局部函数只在该程序块中可见。词法域确保了程序包中的其他函数可以使用这些局部函数：

```lua
    local f = function(<参数>)
        <函数体>
    end

    local g = function(<参数>)
        <一些代码>
        f()
        <一些代码>
    end
```

&emsp;&emsp;对于这种局部函数的定义，Lua还支持一种特殊的“语法糖”：

```lua
    local function f(<参数>)
        <函数体>
    end
```
&emsp;&emsp;

&emsp;&emsp;在定义递归的局部函数时，还有一个特别之处需要注意。像下面这种采用了基本函数定义语法的代码多数是错误的：

```lua
    local fact = function(n)
        if n == 0 then return 1
        else return n * fact(n-1)		-- 错误❌
        end
    end
```

&emsp;&emsp;当Lua编译到函数体中调用`fact(n-1)`的地方时，由于局部的`fact`尚未定义完毕，因此这句表达式其实是调用了一个全局的`fact`，而非此函数自身。为了解决这个问题，可以先定义一个局部变量，然后再定义函数本身：

```lua
    local fact
    fact = function(n)
        if n == 0 then return 1
        else return n * fact(n-1)
        end
    end
```

&emsp;&emsp;现在函数中的`fact`调用就表示了局部变量。即使在函数定义时，这个局部变量的值尚未完成定义，但之后在函数执行时，`fact`则肯定已经拥有了正确的值。

&emsp;&emsp;

&emsp;&emsp;当Lua展开局部函数定义的“语法糖”时，并不是使用基本函数定义语法。而是对于局部函数定义：

```lua
    local funciton foo(<参数>) <函数体> end
```

&emsp;&emsp;Lua将其展开为：

```lua
    local foo
    foo = function(<参数>) <函数体> end
```

&emsp;&emsp;因此，使用这种语法来定义递归函数不会产生错误：

```lua
    local function fact(n)
        if n == 0 then return 1
        else return n * fact(n-1)
        end
    end
```

&emsp;&emsp;当然，这个技巧对于间接递归的函数而言是无效的。在间接递归的情况中，必须使用一个明确的前向声明：

```lua
    local f, g        -- 前向声明

    function g()
        <一些代码> f() <一些代码>
    end

    function f()
        <一些代码> g() <一些代码>
    end
```

>&emsp;&emsp;注意⚠️，别把第二个函数定义写为“`local function f`”。如果那样的话，Lua会创建一个全新的局部变量`f`，而将原来声明的`f`（函数`g`中所引用的那个）置于未定义的状态。

&emsp;&emsp;

###&emsp;&emsp;• 正确的尾调用

&emsp;&emsp;Lua中的函数还有一个有趣的特征，那就是Lua支持“尾调用消除”。
&emsp;&emsp;所谓“尾调用”就是一种类似于`goto`的函数调用。**当一个函数调用是另一个函数的最后一个动作时，该调用才算一条“尾调用”。**

&emsp;&emsp;

&emsp;&emsp;举例来说，以下代码中对`g`的调用就是一条“尾调用”：

```lua
    function f(x) return g(x) end
```

&emsp;&emsp;也就是说，当`f`调用完`g`之后就再无其他事情可做了。因此在这种情况中，程序就不需要返回那个“尾调用”所在的函数了。
&emsp;&emsp;
&emsp;&emsp;所以在“尾调用”之后，程序也不需要保存任何关于该函数的栈信息了。当`g`返回时，执行控制权可以直接返回到调用`f`的那个点上。有一些语言实现（例如Lua解释器）可以得益于这个特点，使得在进行“尾调用”时不耗费任何栈空间。将这种实现称为支持“尾调用消除”。


&emsp;&emsp;由于“尾调用”不会耗费栈空间，所以一个程序可以拥有无数嵌套的“尾调用”。

&emsp;&emsp;

&emsp;&emsp;举例来说，在调用以下函数时，传入任何数字作为参数都不会造成栈溢出：

```lua
    function foo(n)
        if n > 0 then return foo(n-1) end
    end
```

&emsp;&emsp;有一点需要注意的是，当想要受益于“尾调用消除”时，务必要确定当前的调用是一条“尾调用”。判断的准则就是“一个函数在调用完另一个函数之后，是否就无其他事情需要做了”。有一些看似是“尾调用”的代码，其实都违背了这条准则。

&emsp;&emsp;
&emsp;&emsp;举例来说，在下面的代码中，对`g`的调用就不是一条“尾调用”：

```lua
    function f(x) g(x) end
```

&emsp;&emsp;这个示例的问题在于，当调用完`g`后，`f`并不能立即返回，它还需要丢弃`g`返回的临时结果。类似地，以下所有调用也都不符合上述准则：

```lua
    return g(x) + 1					-- 必须做一次加法
    return x or g(x)				-- 必须调整为一个返回值
    return (g(x))					-- 必须调整为一个返回值
```
&emsp;&emsp;
&emsp;&emsp;**在Lua中，只有“`return <func>(<args>)`”这样的调用形式才算是一条“尾调用”**。Lua会在调用前对`<func>`及其参数求值，所以它们可以是任意复杂的表达式。

&emsp;&emsp;举例来说，下面的调用就是一条“尾调用”：

```lua
    return x[i].foo(x[j] + a * b, i + j)
```

&emsp;&emsp;

&emsp;&emsp;在之前提到了，一条“尾调用”就好比是一条`goto`语句。因此，在Lua中“尾调用”的一大应用就是编写“状态机”。这种程序通常以一个函数来表示一个的状态，改变状态就是`goto`（或调用）到另一个特定的函数。举一个简单的迷宫游戏的例子来说明这个问题。

&emsp;&emsp;例如，一个迷宫有几间房间，每间房间中最多有东南西北4扇门。用户在每一步移动中都需要输入一个移动的方向。如果在某个方向上有门，那么用户可以进入相应的房间；不然，程序就打印一条警告。游戏目标就是让用户从最初的房间走到最终的房间。

&emsp;&emsp;这个游戏就是一种典型的状态机，其中当前房间就是一个状态。可以将迷宫中的每间房间实现为一个函数，并使用“尾调用”来实现从一间房间移动到另一间房间。在以下代码中，实现一个具有4间房间的迷宫：

```lua
    function room1()
        local move = io.read()
        if move == "south" then return room3()
        elseif move == "east" then return room2()
        else 
            print("invalid move")
            return room1()			-- stay in the same room
        end
    end

    function room2()
        local move = io.read()
        if move == "south" then return room4()
        elseif move == "west" then return room1()
        else 
            print("invalid move")
            return room2()			-- stay in the same room
        end
    end

    function room3()
        local move = io.read()
        if move == "north" then return room1()
        elseif move == "east" then return room4()
        else 
            print("invalid move")
            return room3()			-- stay in the same room
        end
    end

    function room4()
        print("congratulations!")
    end
```

&emsp;&emsp;通过调用初始房间来开始这个游戏：

```lua
    room1()
```

&emsp;&emsp;若没有“尾调用消除”的话，每次用户的移动都会创建一个新的栈层，移动若干步之后就有可能会导致栈溢出。而“尾调用消除”则对用户移动的次数没有任何限制。这是因为每次移动实际上都只是完成一条`goto`语句到另一个函数，而非传统的函数调用。

&emsp;&emsp;对于这个简单的游戏而言，或许会觉得将程序设计为数据驱动的会更好一点，其中将房间和移动记录在一些`table`中。不过，如果游戏中的每间房间都有各自特殊的情况的话，采用这种状态机的设计则更为合适。


🔚