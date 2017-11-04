##数据文件与持久性

&emsp;&emsp;当涉及到数据文件的处理时，人们往往会认为写数据比读数据简单得多。当写一个文件时，对写的内容拥有完全的控制权。但是当读一个文件时，却无从得知会读到什么内容。一个强健的程序除了需要处理一个合法文件中所包含的所有类型的数据，还应能很好地处理坏损的文件。因此，编写一个强健的输入程序总是比较困难的。

&emsp;&emsp;在本章中，我们将看到如何使用Lua来避免程序中所有有关数据读取的代码，只需将数据按一种适当的格式书写就可以了。

&emsp;&emsp;

####数据文件

&emsp;&emsp;可以借由table构造式来定义一种文件格式。只需在写数据时做一点额外的工作，读取数据就会变得相当容易。这项技术也就是将数据作为Lua代码来输出，当运行这些代码时，程序也就读取了数据。而table的构造式可以使这些输出代码看上去更像是一个普通的数据文件。

&emsp;&emsp;下面通过一个示例来更清楚地理解这种做法。如果数据文件是一种预定义的格式，例如CSV（Comma-Separated Values，逗号分隔值）或XML，那么可以选择的做法很少。不过，如果是为了应用而创建数据文件的话，那么就可以使用Lua的构造式作为格式。在这种格式中，每条数据记录表示为一个Lua构造式。这样，原来以这种形式书写的数据文件：

```lua
    Donald E. Knuth, Literate Programming, CSLI, 1992
    Jon Bentley, More Programming Pearls, Addison-Wesley, 1990
```

&emsp;&emsp;现在可以改为：

```lua
    Entry{ "Donald E. Knuth",
           "Literate Programming",
           "CSLI",
           1992}
           
    Entry{ "Jon Bentley",
           "More Programming Pearls",
           "Addison-Wesley",
           1990}
```

&emsp;&emsp;记住，`Entry{<code>}`与`Entry({<code>})`是完全等价的，都是以一个table作为参数来调用函数Entry。因此，上面这段数据也是一个Lua程序。为了读取该文件，我们只需定义一个合适的Entry，然后运行此程序就可以了。例如，以下程序计算了数据文件中条目的数量：

```lua
    local count = 0
    function Entry(_) count = count + 1 end
    dofile("data")
    print("number of entries: " .. count)
```

&emsp;&emsp;下一个程序则可用于收集数据文件中所有作者的姓名，然后打印出这些姓名（不需要与文件中的次序相同）：

```lua
    local authors = {}            -- 作者姓名的集合
    function Entry(b) authors[b[1]] = true end
    dofile("data")
    for name in pairs(authors) do print(name) end
```

&emsp;&emsp;可以看到这些代码片段都采用了事件驱动的做法。Entry函数作为一个回调函数，在dofile时为数据文件中的每个条目所调用。

&emsp;&emsp;若文件不是非常大，可以使用名值对来表示每个字段：

```lua
    Entry{
        author = "Donald E. Knuth",
        title = "Literate Programming",
        publicsher = "CSLI",
        year = 1992
    }
    
    Entry{
        author = "Jon Bentley",
        title = "More Programming Pearls",
        publicsher = "Addison-Wesley",
        year = 1990
    }
```

&emsp;&emsp;这种格式就是“自描述的数据（self-describing data）”格式，其中每项数据都伴随一个表示其含义的简短描述。自描述的数据比CSV或其他紧缩格式更具可读性。当需要修改时，也易于手工编辑，可以在基本格式中作出一个细小的改动，而不需要同时改变数据文件。例如，如果要新增一个字段，只需修改读取程序中的一小块就可以了，内容就是当该字段不存在时提供一个默认值。

&emsp;&emsp;使用名值对格式后，那个收集作者姓名的程序改为：

```lua
    local authors = {}         -- 作者姓名的集合
    function Entry(b) authors[b.author] = true end
    dofile("data")
    for name in pairs(authors) do print(name) end
```

&emsp;&emsp;现在字段的次序就不重要了，即使有些条目没有作者字段，也只需要修改Entry函数：

```lua
    function Entry(b)
        if b.author then authers[b.author] = true end
    end
```

&emsp;&emsp;Lua不仅运行速度快，而且编译速度也快。例如，上面这个用于列出作者的程序在处理2MB数据时，只需不到1秒钟的时间。这不是偶然的结果，自从Lua创建之初就把数据描述作为Lua的主要应用之一来考虑的，开发人员为能较快地编译大型程序投入了很多的努力。

&emsp;&emsp;

####串行化（Serialization）

&emsp;&emsp;通常需要串行化一些数据，也就是将数据转换为一个字节流或字符流。然后就可以将其存储到一个文件中，或者通过网络连接发送出去了。串行化后的数据可以用Lua代码来表示，这样当运行这些代码时，存储的数据就可以在读取程序中得到重构了。

&emsp;&emsp;如果想要恢复一个全局变量的值，那么串行化的结果或许可以是“`varname = <exp>`”，其中`<exp>`是一段用于创建该值的代码。例如，对于一个数字值，方法如下：

```lua
    function serialize(o)
        if type(o) == "number" then
            io.write(o)
        else 
            <其他情况>
        end
    end
```

&emsp;&emsp;对于一个字符串值，方法如下：

```lua
    if type(o) == "string" then
        io.write("'", o, "'")
```

&emsp;&emsp;然而，如果字符串中包含特殊字符（例如引号、换行），那么最终代码就不是一段有效的Lua程序了。

&emsp;&emsp;也可以使用另一种字符串字面表示方法，如下所示：

```lua
    if type(o) == "string" then
        io.write("[[", o "]]")
```

&emsp;&emsp;注意，如果有用户故意使其字符串为 “]]..os.execute('rm *')..[[”，那么最终保存下来的结果将变成：

```lua
    varname = [[]]..os.execute('rm *')..[[]]
```

&emsp;&emsp;若加载这个“数据”将会出现不可估量的后果。

&emsp;&emsp;可以使用一种简单且安全的方法来扩住一个字符串，那就是以“%q”来使用string.format函数。这样它就会用双引号来括住字符串，并且正确地转移其中的双引号和换行符等其他特殊字符。

```lua
    a = 'a "problematic"\\string'
    print(string.format("%q", a))        --> "a \"problematic\" \\string"
```

&emsp;&emsp;通过使用这个特性，serialize函数可以改为：

```lua
    function serialize(o)
        if type(o) == "number" then
            io.write(o)
        elseif type(o) == "string" then
            io.write(string.format("%q", o))
        else
            <其他情况>
        end
    end
```

&emsp;&emsp;Lua5.1还提供了另一种可以以一种安全的方法来括住任意字符串的方法。这是一种新的标记方式`[=[...]=]`，用于长字符串。然而，这种新方式主要是为手写的代码提供方便的，通过它就不需要改变任何字符串的内容了。在自动生成的代码中，要转移那些问题字符，还是使用string.format与"%q"选项更为方便。

&emsp;&emsp;如果仍然在自动生成的代码中使用长字符串标记的话，那么就需要注意两个细节问题。
&emsp;&emsp;首先，必须使用正确数量的等号。这个正确的数量应比字符串中出现的最长的等号序列还大1。由于，在字符串中出现长序列的等号是很有可能的，并且其他序列也不会产生一个错误的字符串结尾的标记，所以要注意等号序列。
&emsp;&emsp;第二个细节是，Lua总是会忽略所有长字符串开头的换行符。一种避免这个问题的简单方法就是，在字符串起始处添加一个换行符。

&emsp;&emsp;以下这个quote函数就是根据上面提到的两个注意点编写的处理函数。

```lua
    function quote(s)
        -- 查找最长的等号序列
        local n = -1
        for w in string.gmatch(s, "]=*") do
            n = math.max(n, #w - 1)
        end

        -- 产生'n' + 1个等号
        local eq = string.rep("=", n + 1)

        -- 生成长字符串的字面表示
        return string.format(" [%s[\n%s]%s] ", eq, s, eq)
    end
```

&emsp;&emsp;它可以接收任意字符串，并放回其格式化为长字符串的结果。对`string.gmatch`的调用会创建一个迭代器，通过该迭代器就可以遍历字符串s中所有出现模式`']=*'`的地方。在每处出现等号的地方，循环就会更新n，使其保持为当前所遇到的最大等号数量。在循环结束后使用`string.rep`将等号重复n+1遍，也就是生成一个等号序列字符串，其长度出现有字符串中的最长等号序列还多1。最后，`string.format`将s嵌入一对具有正确数量等号的方括号对中，并在方括号外添加一些额外的空格，以及在s开头插入一个换行符。


&emsp;&emsp;

####保存无环的table

&emsp;&emsp;下一个任务是保存table。保存table有几种方法，选用哪种方法取决于对table的结构作出了哪些限制性的假设。没有一种算法适用于所有的情况。简单的table不仅需要更简单的算法，而且需要更完美地输出结果。

第一个算法如下：

```lua
    function serialize(o)
        if type(o) == "number" then
            io.write(o)
        elseif type(o) == "string" then
            io.write(string.format("%q", o)) then
        elseif type(o) == "table" then
            io.write("{\n")
            for k, v in pairs(o) do
                io.write(" ", k, " = ")
                serialize(v)
                io.write(",\n")
            end
            io.write("}\n")
        else
            error("cannot serialize a " .. type(o))
        end
    end
```


&emsp;&emsp;尽管这个函数很简单，但却可以完成基本的保存工作。只要table的结构是一个树结构，它甚至还能处理嵌套的table（table中的table）。可以作为一个练习，尝试在输出格式中缩进那些嵌套的table。

&emsp;&emsp;上例函数假设了一个table中的所有key都是合法的标识符。但如果一个table的key为数字或者非法的Lua标识符，那么就会出现问题。一个简单的解决方法是将这行：

```lua
    io.write(" ", k, " = ")
```

&emsp;&emsp;改为：

```lua
    io.write(" ["); serialize(k); io.write("] = ")
```

&emsp;&emsp;这样，便增强了这个函数的强健性，但却损失了结果文件的美观性。对于调用：

```lua
    serialize{a=12, b='Lua', key='another "one"'}
```

&emsp;&emsp;第一个版本的serialize会输出：

```lua
    {
        a = 12,
        b = "Lua",
        key = "another \"one\""
    }
```

&emsp;&emsp;而第二个版本则输出：

```lua
    {
        ["a"] = 12,
        ["b"] = "Lua",
        ["key"] = "another \"one\"",
    }
```

&emsp;&emsp;可以测试每种需要方括号的情况，从而改善结果的美观性。

&emsp;&emsp;

####保存有环的table

&emsp;&emsp;若要处理具有任意拓扑结构（带环的table或共享子table）的table，就需要采用另外一种方法了，table构造式是无法表示这类table的。所以为了表示“环”，则需要引入名称，接下来这个保存函数要求将待保存的值及其名称一起作为参数传入。此外，还必须持有一份所有已保持过的table的名称记录，以此来检测环并复用其中的table。使用一个额外的table用作此项纪录，这个table以其他table作为key，并以其他table的名称作为value。代码如下：

```lua
    function basicSerialize(o)
        if type(o) == "number" then
            return tostring(o)
        else 			-- assume it is a string
            return string.format("%q", o)
        end
    end

    function save(name, value, saved)
        saved = saved or {} 		-- 初始值
        io.write(name, " = ")
        if type(value) == "number" or type(value) == "string" then
            io.write(basicSerialize(value), "\n")
        elseif type(value) == "table" then
            if saved[value] then 		-- 该value是否已保存过？
                io.write(saved[value], "\n") 		-- 使用先前的名字
            else
                saved[value] = name 		-- 为下次使用保持名字
                io.write("{}\n") 			-- 创建一个新的table
                for k, v in pairs(value) do
                    k = basicSerialize(k)
                    local fname = string.format("%s[%s]", name, k)
                    save(fname, v, saved)
                end
            end
        else
            error("cannot save a " .. type(value))
        end
    end
```


&emsp;&emsp;假设准备保存的table的key只为字符串或数字。函数basicSerialize用于串行化这些基本类型，返回串行化的结果。而另一个函数save则完成真正的工作。saved参数是一个table，用于记录已保存过的table。假设有一个table如下所示：

```lua
    a = {x=1, y=2; {3, 4, 5}}
    a[2] = a 		-- 环
    a.z = a[1] 		-- 共享子table
```

&emsp;&emsp;然后，调用save("a", a)将它保存为：

```lua
    a = {}
    a[1] = {}
    a[1][1] = 3
    a[1][2] = 4
    a[1][3] = 5
    a[2] = a
    a["x"] = 2
    a["y"] = 1
    a["z"] = a[1]
```

&emsp;&emsp;这些赋值语句的实际顺序可能会有所不同，这取决于一个table的遍历顺序。不过，该算法可以保证在一句新的定义中所用到的变量都已经定义过了。

&emsp;&emsp;如果想以共享的方式来保存几个table中的共同部分，只需在调用saved时使用相同的saved参数。例如，假设有两个table：

```lua
    a = {{"one", "two"}, 3}
    b = {k = a[1]}
```

&emsp;&emsp;如果以独立的方式保存它们，那么结果中不会有共同部分：

```lua
    save("a", a)
    save("b", b)

    --> a = {}
    --> a[1] = {}
    --> a[1][1] = "one"
    --> a[1][2] = "two"
    --> a[2] = 3
    --> b = {}
    --> b["k"] = {}
    --> b["k"][1] = "one"
    --> b["k"][2] = "two"
```

&emsp;&emsp;然而，当使用同一个saved table来调用save时，串行化结果就会共享共同部分：

```lua
    local t = {}
    save("a", a, t)
    save("b", b, t)

    --> a = {}
    --> a[1] = {}
    --> a[1][1] = "one"
    --> a[1][2] = "two"
    --> a[2] = 3
    --> b = {}
    --> b["k"] = a[1]
```

&emsp;&emsp;在Lua中，还有一些其他比较常见的方法。有的在保存一个值时无须给出一个全局名称（而是通过一段代码来构造一个局部值，并返回这个值），有的则可以处理函数（通过构造一个辅助table，来将函数与它的名称关联起来）等。Lua赋予了构建这些机制的能力。


🔚