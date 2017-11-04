##I/O库

&emsp;&emsp;I/O库为文件操作提供了两种不同的模型，简单模型（`simple model`）和完整模型（`complete model`）。简单模型假设有一个当前输入文件和一个当前输出文件，它的I/O操作均作用于这些文件。完整模型则使用显式的文件句柄。它采用了面向对象的风格，并将所有的操作定义为文件句柄上的方法。

&emsp;&emsp;在本书前面的章节示例中涉及到的简单操作都使用简单模型并且十分方便。但是对于更多的高级文件操作，例如同时读取多个文件，它就无法做到了。对于这些高级操作，需要使用完整模型。

&emsp;&emsp;

#####&emsp;&emsp;简单I/O模型

&emsp;&emsp;简单模型的所有操作都作用于两个当前文件。I/O库将当前输入文件初始化为进程标准输入（`stdin`），将当前输出文件初始化为进程标准输出（`stdout`）。在执行`io.read()`操作时，就会从标准输入中读取一行。

&emsp;&emsp;用函数`io.input`和`io.output`可以改变这两个当前文件。`io.input(filename)`调用会以只读模式打开指定的文件，并将其设为当前输入文件。之后除非再次调用`io.input`，否则所有的输入都将来源于这个文件。在输出方面，`io.output`也可以完成类似的工作。如果出现错误，这两个函数都会引发（raise）错误。如果想直接处理这些错误，则必须使用完整模型中的`io.open`。

&emsp;&emsp;通常`write`比`read`简单些，首先看一下`write`。函数`io.write`接受任意数量的字符串参数，并将它们写入当前输出文件。它也可以接受数字参数，数字参数会根据常规的转换规则转换为字符串。如果想要完全地控制这种转换，则应该使用函数`string.format`：

```lua
    >io.write("sin (3) = ", math.sin(3), "\n")
    --> sin (3) = 0.14112000805987
    >io.write(string.format("sin (3) = %.4f\n", math.sin(3)))
    --> sin (3) = 0.1411
```

&emsp;&emsp;在实际操作中应当避免写出`io.write(a..b..c)`这样的代码，而是应该调用`io.write(a,b,c)`，它能达到与`io.write(a..b..c)`相同的效果，并且可以避免连接操作，因此效率更高。

&emsp;&emsp;无论使用`print`还是`io.write`都有一个原则。即在随意编写（quick-and-dirty）的程序中，或者为调试目的而编写的代码中，提倡使用`print`；而在其他需要完全控制输出的地方使用`write`。

```lua
    >print("hello", "Lua");print("Hi")
    --> hello Lua
    --> Hi
    
    >io.write("hello", "Lua");io.write("Hi", "\n")
    --> helloLuaHi
```

&emsp;&emsp;`write`与`print`有几点不同。首先，`write`在输出时不会添加像制表符或回车这样的额外字符。其次，`write`使用当前输出文件，而`print`总是使用标准输出。最后，`print`会自动调用其参数的`tostring()`方法，因此它还能显示`table`、函数和`nil`。

&emsp;&emsp;函数`io.read`从当前输入文件中读取字符串，它的参数决定了要读取的数据：

|||
|:--|:--|
|`"*all"`|读取整个文件|
|`"*line"`|读取下一行|
|`"*number"`|读取一个整数|
|`<num>`|读取一个不超过`<num>`个字符的字符串|

&emsp;&emsp;调用`io.read("*all")`会读取当前输入文件的所有内容，以当前位置作为开始。如果当前位置处于文件的末尾，或者文件为空，那么该调用会返回一个空字符串。

&emsp;&emsp;由于Lua可以高效地处理长字符串，因此在Lua中一种简单的、编写过滤器的技术就可以将整个文件读到一个字符串中，然后处理这个字符串（通常使用`gsub`），最后把这个字符串写到输出：

```lua
    t = io.read("*all")        -- 读取整个文件
    t = string.gsub(t, ...)    -- 做相关的处理
    io.write(t)                -- 写输出
```

&emsp;&emsp;下面是一个完整的示例，这段代码使用MIME quoted-printable编码方式对文件内容进行编码。在这种编码方式中，非ASCII字符被编码为`=<xx>`的形式，其中`<xx>`是这个字符的十六进制数字代码。此外，为了保持编码的一致性，字符“`=`”也需要被编码：

```lua
    t = io.read("*all")
    t = string.gsub(t, "([\128-\255=])"), function(c)
            return string.format("=%02X", string.byte(c))
        end)
    io.write(t)
```

&emsp;&emsp;`gsub`中使用的模式可以捕获所有编码为`128～255`的字符及等号字符。

&emsp;&emsp;调用`io.read("*line")`会返回当前文件的下一行，但不包括换行字符。当到达文件末尾时，该调用会返回`nil`，以表示无后续行可返回。它也是`read`的默认模式。通常，我只在需要逐行处理的算法中使用这种模式。另外，建议使用`*all`一次性读取整个文件，或者像后面介绍的按块来读取。

&emsp;&emsp;作为使用该模式的一个简单示例，下面这个程序将当前输入中的内容复制到当前输出中，并对每行进行编号：

```lua
    for count = 1, math.huge do
        local line = io.read()
        if line == nil then break end
        io.write(string.format("%6d ", count), line, "\n")
    end
```

&emsp;&emsp;如果只为了迭代文件中的所有行，那么`io.lines`迭代器更为合适。例如，下面这个程序可以对文件中的所有行进行排序：

```lua
    local lines = {}
    -- 读取table 'lines'中所有行
    for line in io.lines() do lines[#lines + 1] = line end
    -- 排序
    table.sort(lines)
    -- 输出所有行
    for _, l in ipairs(lines) do io.write(l "\n") end
```

&emsp;&emsp;调用`io.read("*number")`会从当前输入文件中读取一个数字。此时，`read`会返回一个数字，而不是字符串。当一个程序需要从文件中读取大量数字时，应当避免生成中间的字符串过渡形式，这样可以提高程序的性能。`*number`选项会忽略数字前面所有的空格，并且能处理像`-3`、`+5.2`、`1000`及`-3.4e-23`这样的数字格式。如果无法在当前文件位置读到一个数字，`read`会返回`nil`。

&emsp;&emsp;在调用`read`时可以指定多个选项，函数会根据每个选项参数返回相应地内容。假设，有一个文件，其中每行有3个数字：

```lua
    6.0    -3.23    15e12
    4.3    234      1000001
    ...
```

&emsp;&emsp;现在要打印出每一行中最大的数字。可以用一次`read`函数调用来读取每行的3个数字：

```lua
    while true do
        local n1, n2, n3 = io.read("*number", "*number", "*number")
        if not nil then break end
        print(math.max(n1, n2, n3))
    end
```

&emsp;&emsp;对于这类问题，还可以采用“`*all`”读取整个文件，然后再用`gmatch`来提取其中内容：

```lua
    local pat = "(%S+)%s+(%S+)%s+(%S+)%s+"
    for n1, n2, n3 in string.gmatch(io.read("*all"), pat) do
        print(math.max(tonumber(n1), tonumber(n2), tonumber(n3)))
    end
```

&emsp;&emsp;除了以上这些基本的读取模式，还可以用一个数字`n`作为`read`的参数。此时，`read`会试着从输入文件中读取`n`个字符。如果读不到任何字符，它会返回`nil`。否则会返回一个最多`n`个字符的字符串。下面这个示例演示了这种读取模式，它是一种将数据从`stdin`复制到`stdout`的高效方法：

```lua
    while true do
        local block = io.read(2^13)         -- 缓冲大小为8K
        if not block then break end
        io.write(block)
    end
```

&emsp;&emsp;`io.read(0)`是一种特殊情况，它用于检查是否到达了文件末尾。如果还有数据可以读取，它会返回一个空字符串，否则返回`nil`。

&emsp;&emsp;

####&emsp;&emsp;完整I/O模型

&emsp;&emsp;若要作更多的I/O控制，可以使用完整模型。这个模型是基于文件句柄的，它等价于C语言中的流（FILE *），表示一个具有当前位置的打开文件。

&emsp;&emsp;要打开一个文件，可以使用`io.open`函数。它仿照了C语言中的`fopen`函数，并且具有两个参数一个是要打开的文件名，另一个是模式（Mode）字符串。模式字符串可以是：“`r`”表示读取、“`w`”表示写入（同时会删除文件原来的内容）及“`a`”表示追加，另外还有一个可选的“`b`”表示打开二进制文件。`open`函数会返回表示文件的新句柄。若发生错误，则返回`nil`，及一条错误消息和一个错误代码。

```lua
    print(io.open("non-existent-file", "r"))
    --> nil  non-existent-file: No such file or directory  2

    print(io.open("/etc/passwd", "w"))
    --> nil  /etc/passwd: Permission denied  13
```

&emsp;&emsp;错误代码的解释依赖于系统。

&emsp;&emsp;一个错误检查的典型做法是：

```lua
    local f = assert(io.open(filename, mode))
```

&emsp;&emsp;如果打开失败，错误消息就会成为`assert`的第二个参数，然后`assert`会显示这个消息。

&emsp;&emsp;当打开一个文件后，就可以用`read/write`方法读写文件了。这与`read/write`函数相似，但是需要用冒号语法，将它们作为文件句柄的方法来调用。例如，要打开一个文件，并读取其所有内容，那么这么做：

```lua
    local f = assert(io.open(filename, "r"))
    local t = f:read("*all")
    f:close()
```

&emsp;&emsp;I/O库提供了3个预定义C语言流的句柄：`io.stdin`、`io.stdout`和`io.stderr`。这样，就可以将信息直接写到错误流：

```lua
    io.stderr:write(message)
```

&emsp;&emsp;用户可以混合使用完整模式和简单模式。通过不指定参数调用`io.input()`，可以得到当前输入文件的句柄。而通过`io.input(handle)`，可以设置当前输入文件的句柄。例如，要临时改变当前输入文件，可以这么做：

```lua
    local temp = io.input()     -- 保存当前文件
    io.input("newinput")        -- 打开一个新的当前文件
    <对新的输入文件做一些操作>
    io.input():close()          -- 关闭当前文件
    io.input(temp)              -- 恢复原来的输入文件
```
&emsp;&emsp;

#####&emsp;&emsp;● 性能小诀窍

&emsp;&emsp;通常在Lua中，一次性读取整个文件比逐行地读取要快一些。但必须处理一些大文件（几十或几百兆字节）时，就无法一次性地读取所有的内容。如果希望以最高性能来处理这种大文件，那么最快的方法就是用足够大的块（例如，8KB大小的块）来读取文件。为了避免在行中间断开，只需在读一个块时再加上一行：

```lua
    local lines, rest = f:read(BUFSIZE, "*line")
```

&emsp;&emsp;变量`rest`包含了被块所断开的那行的剩余部分。这样就可以将块与行的剩余部分连接起来，从而得到了一个总是起止于行边界上的块。

&emsp;&emsp;下面这个示例运行此技术实现了`wc`，`wc`是一个用于统计文件中字符数、单词数和行数的程序：

```lua
    local BUFSIZE = 2^13            -- 8K
    local f = io.input(arg[1])      -- 打开输入文件
    local cc, lc, wc = 0, 0, 0      -- 字符、行、单词的计数
    while true do
       local lines, rest = f:read(BUFSIZE, "*line")
       if not lines then break end
       if rest then lines = lines .. rest .. "\n" end
       cc = cc + #lines
       -- 统计块中的单词数
       local _, t = string.gsub(lines, "%S+", "")
       wc = wc + t
       -- 统计块中的换行字符数量
       _, t = string.gsub(lines, "\n", "\n")
       lc = lc + t
    end
    print(lc, wc, cc)
```

&emsp;&emsp;

#####&emsp;&emsp;● 二进制文件

&emsp;&emsp;简单模式中的函数`io.input`和`io.output`总是以文本方式打开文件（默认行为）。在UNIX中，二进制文件和文本文件是没有差别的。但在其他一些系统中，特别是在Windows中，必须用特殊的标识来打开二进制文件。在处理二进制文件时，`io.open`的模式字符串中必须带有字母“`b`”。

&emsp;&emsp;在Lua中，二进制数据的处理与文本处理类似。Lua中的字符串可能包含任意字节，库中几乎所有函数都能处理任意字节。只要模式字符串中不包含值为零的字节，甚至还可以对二进制数据作模式匹配。如果确实需要在模式字符串中包含值为零的字节，可以用转义字符`%z`来表示。

&emsp;&emsp;通常在读取二进制数据时，使用`*all`模式来读取整个文件，或者使用`<num>`模式来读取`n`个字节。下面是一个简单的示例程序，它会把DOS格式的文本文件转换为UNIX格式。它并没有使用标准的I/O文件（`stdin`和`stdout`），因为这些文件都是以文本方式打开的。它假设输入文件和输出文件的名称分别由程序的参数指定：

```lua
    local inp = assert(io.open(arg[1], "rb"))
    local out = assert(io.open(arg[2], "wb"))

    local data = inp:read("*all")
    data = string.gsub(data, "\r\n", "\n")
    out:write(data)

    assert(out:close())
```

&emsp;&emsp;可以用以下命令行来调用这个程序：

```lua
    >lua prog.lua file.dos file.unix
```

&emsp;&emsp;下面是另外一个示例，它打印了在一个二进制文件中找到的所有字符串：

```lua
    local f = assert(io.open(arg[1], "rb"))
    local data = f:read("*all")
    local validchars = "[%w%p%s]"
    local pattern = string.rep(validchars, 6) .. "+%z"
    for w in string.gmatch(data, pattern) do
        print(w)
    end
```

&emsp;&emsp;这个程序假定字符串是一个以0结尾，并包含至少6个有效字符的序列。所谓“有效字符”是指被模式`validchars`所认可的任意字符。在这个示例中，这个模式包含了数字、字母、标点符号和空格字符。然后，通过`string.rep`和连接操作创建了一个新的模式，这个新模式可用于捕获6个或更多的`validchars`。而其结尾的`%z`用于匹配字符串末尾的零字节。

&emsp;&emsp;下面是最后一个示例，它打印了一个二进制文件的内容：

```lua
    local f = assert(io.open(arg[1], "rb"))
    local block = 16
    while true do
        local bytes = f:read(block)
        if not bytes then break end
        for _, b in ipairs{string.byte(bytes, 1, -1)} do
            io.write(string.format("%02X ", b))
        end
        io.write(string.rep("  ", block - string.len(bytes)))
        io.write(" ", string.gsub(bytes, "%c", "."), "\n")
    end
```

&emsp;&emsp;同样，程序的第一个参数是输入文件名，而结果则被输出到标准输出。这个程序以16字节作为一块读取文件。对于每个块，它先输出每个字节的十六进制表示。然后，将整个块作为文本输出，而块中的控制字符都会替换为点符号。

&emsp;&emsp;以下是在UNIX系统上将这个程序应用于自身后的结果。

```lua
    6C 6F 63 61 6C 20 66 20 3D 20 61 73 73 65 72 74 local f = assert
    28 69 6F 2E 6F 70 65 6E 28 61 72 5B 31 5D 2C 2C (io.open(arg[1],
    6C 6F 63 61 6C 20 66 20 3D 20 61 73 73 65 72 74  "rb")).local bl
    6C 6F 63 61 6C 20 66 20 3D 20 61 73 73 65 72 74  ock = 16.while t
    6C 6F 63 61 6C 20 66 20 3D 20 61 73 73 65 72 74  rue do. local b
    6C 6F 63 61 6C 20 66 20 3D 20 61 73 73 65 72 74  ng.gsub(bytes, "
    6C 6F 63 61 6C 20 66 20 3D 20 61 73 73 65 72 74  %c", "."), "\n"
    6C 6F 63 61 6C 20                                .end.
```
&emsp;&emsp;

#####&emsp;&emsp;● 其他文件操作

&emsp;&emsp;函数`tmpfile`返回一个临时文件的句柄，这个句柄是以读/写方式打开。这个文件会在程序结束时自动删除。函数`flush`会将缓冲中的数据写入文件。它与`write`函数一样，将其作为一个函数调用时，`io.flush()`会刷新当前输出文件；而将其作为一个方法调用时，`f:flush()`会刷新某个特定的文件`f`。

&emsp;&emsp;函数`seek`可以获取和设置一个文件的当前位置。它的一般形式是`f:seek(whence, offset)`，其中`whence`参数是一个字符串，指定了如何解释`offset`参数。它的有效值包括：“`set`”，`offset`解释为相对于文件起始的偏移量。函数的返回值与`whence`无关，它总是返回文件的当前位置，即相对于文件起始处的偏移字节数。

&emsp;&emsp;`whence`参数的默认值是“`cur`”，`offset`的默认值是0。因此，调用`file:seek()`不会改变文件的当前位置，并会返回当前的文件位置。调用`file:seek("set")`会将当前位置设置到文件的起始处（并返回0）。调用`file:seek("end")`会将当前位置设置到文件的末尾，并返回文件的大小。下面这个函数可以不改变文件的当前位置而获取文件的大小：

```lua
    function fsize(file)
        local current = file:seek()     -- 获取当前位置
        local size = file:seek("end")   -- 获取文件大小
        file:seek("set", current)       -- 恢复位置
        return size
    end
```

&emsp;&emsp;如果发生错误，所有这些函数都会返回`nil`和一条错误消息。

🔚
