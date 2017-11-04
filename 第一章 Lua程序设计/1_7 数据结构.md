##数据结构

```
    ● 数组
    ● 矩阵与多维数组
    ● 链表
    ● 队列与双向队列
    ● 集合与无序组（bag）
    ● 字符串缓冲
    ● 图
```

&emsp;&emsp;Lua中的`table`不是一种简单的数据结构，它可以作为其他数据结构的基础。其他语言提供的数据结构，如数组、记录、线性表、队列、集合等，在Lua中都可以通过`table`来表示。此外，用Lua的`table`来实现这些结构的效率高。

&emsp;&emsp;在C和Pascal这样的传统语言中，尽管可以使用Lua的`table`来实现数组和列表，但通常以数组和列表（记录+指针）来实现大多数的数据结构。因为`table`本身就比数组和列表的功能强大得多。由此许多算法都可以忽略一些细节问题，从而简化它们的实现。例如，在Lua中很少编写搜索算法，这是因为`table`本身就提供了直接访问任意类型的功能。

&emsp;&emsp;高效地使用`table`是问题的关键。接下来将演示如何通过`table`来实现一些传统的数据结构，并且还将给出一些使用这些结构的例子。首先从数组和列表开始，不是因为需要它们来作为其他结构的基础，而是因为大多数程序员都比较熟悉它们了。在前面关于语言的章节中也曾提到过这方面的内容，不过为了完整性起见，本章将更详细地进行讨论。

&emsp;&emsp;

###&emsp;&emsp;● 数组

&emsp;&emsp;使用整数来索引`table`即可在Lua中实现数组。因此，数组没有一个固定的大小，可以根据需要增长。通常，当初始化一个数组时，也就间接地定义了它的大小。

&emsp;&emsp;例如，在执行了以下代码后，任何对字段范围1～1000之外的访问都会返回一个`nil`，而不是0：

```lua
    a = {}                -- 新建一个数组
    for i = 1, 1000 do
        a[i] = 0
    end
```

&emsp;&emsp;长度操作符（`#`）依赖于这个事实来计算数组的大小：

```lua
    print(#a)            --> 1000
```

&emsp;&emsp;可以使用0、1或其他任意值来作为数组的起始索引：

```lua
    -- 使用索引值-5～5来创建一个数组
    a = {}
    for i = -5, 5 do
        a[i] = 0
    end
```

>&emsp;&emsp;然而，在Lua中的习惯一般是以1作为数组的起始索引。Lua库和长度操作符都遵循这个约定。如果你的数组不是从1开始的，那就无法使用这些功能了。

&emsp;&emsp;通过`table`的构造式，可以在一句表达式中创建并初始化数组：

```lua
    squares = {1, 4, 9, 16, 25, 36, 49, 64, 81}
```

&emsp;&emsp;这种构造式可以根据要求变得更长。


&emsp;&emsp;

###&emsp;&emsp;● 矩阵与多维数组

&emsp;&emsp;在Lua中，有两种方式来表示矩阵。第一种是使用一个“数组的数组”，也就是说，一个`table`中的每个元素是另一个`table`。例如，使用以下代码来创建N×M的零矩阵：

```lua
    mt = {}                -- 创建矩阵
    for i = 1, N do
        mt[i] = {}             -- 创建一个新行
        for j = 1, M do
            mt[i][j] = 0
        end
    end
```

&emsp;&emsp;由于在Lua中`table`是一种对象，因此在创建矩阵时，必须显式地创建每一行。从一方面看，这的确比在C和Pascal中直接声明一个多维数组烦琐；但从另一方面看，它也给予了更多的灵活性。例如，创建一个三角形矩阵，只需将前例中的循环`for j = 1, M do ... end`改为`for j = 1, i do ...end`。这种修改同时可以使三角形矩阵只使用原先一半的内存。

&emsp;&emsp;在Lua中表示矩阵的第二种方式是将两个索引合并为一个索引。如果两个索引是整数，可以将第一个索引乘以一个适当的常量，并加上第二个索引。以下代码就使用这种方法来创建N×M：

```lua
    mt = {}        -- 创建矩阵
    for i = 1, N do
        for j = 1, M do
            mt[(i-1)*M + j] = 0
        end
    end
```

&emsp;&emsp;如果索引是字符串，那么可以把索引拼接起来，中间使用一个字符来分隔。例如，使用字符串`s`和`t`来索引一个矩阵，可以通过代码`m[s..":"..t]`。其中，`s`和`t`都不能包含冒号，否则像`（"a:", "b"）`或`（"a", ":b"）`这样的索引会使最终索引变成“`a::b`”。如果无法保证这点的话，可以使用例如'`\0`'这样的控制字符来分隔两个索引。

&emsp;&emsp;

&emsp;&emsp;通常应用程序会用到一种特殊的矩阵，称为“稀疏矩阵”，这种矩阵中的大多数元素为0或`nil`。例如，可以通过稀疏矩阵来表示一个图（graph）。当矩阵的`m`，`n`位置上有一个值`x`，即表示图中的结点`m`和`n`是相连的，其权重（cost）为`x`；若图中这些节点不相连的话，则矩阵`m`，`n`位置上的值为`nil`。若要表示一个具有1万个节点的图，其中每个结点大约会与其他5个结点相连，那么就需要一个能包含1亿个元素的矩阵，但是其中大约只有5万个元素不为`nil`。许多数据结构的书籍都会讨论到这个大小问题，如何才能实现这种稀疏矩阵而不浪费400MB内存。当在Lua中编程时，则无须用到这些技术。因为，数组是以`table`来表示的，它们本身就是稀疏的。在第一种表示（`table`的`table`）中，需要1万个`table`，每个`table`包含5个元素，总共5万个条目。在第二种表示中，需要一个`table`，其中包含5万个条目。无论哪种表示方式，都只需要为非`nil`的元素付出空间。

&emsp;&emsp;虽然对稀疏矩阵使用长度操作符不是一种语法错误。但也不能进行该操作，因为在有效条目之间存在“空洞（`nil`值）”。在大多数对稀疏矩阵的操作中，由于存在许多空条目，遍历矩阵是非常低效的。所以，一般使用`pairs`且只遍历那些非`nil`的元素。

&emsp;&emsp;例如，要将矩阵的一行与一个常量相乘，可以使用以下代码：

```lua
    function mult(a, rowindex, k)
        local row = a[rowindex]
        for i, v in pairs(row) do
            row[i] = v * k
        end
    end
```

&emsp;&emsp;注意，`table`中的`key`是无序的，所以使用`pairs`的迭代并不保证会按递增次序来访问元素。对于一些任务而言（像上面这个例子），这没有问题；但对于另一些任务而言，或许就需要采用另外的方法了，比如链表。


&emsp;&emsp;

###&emsp;&emsp;● 链表

&emsp;&emsp;由于`table`是动态的实体，所以在Lua中实现链表是很方便的。每个结点以一个`table`来表示，一个“链接”只是结点`table`中的一个字段，该字段包含了对其他`table`的引用。

&emsp;&emsp;例如，要实现一个基础的列表，其中每个结点具有两个字段：`next`和`value`，先创建一个用作列表头结点的变量：

```lua
    list = nil
```

&emsp;&emsp;在表头插入一个元素，元素值为`v`：

```lua
    list = {next = list, value = v}
```

&emsp;&emsp;遍历此列表：

```lua
    local l = list
    while l do
        <访问l.value>
        l = l.next
    end
```

&emsp;&emsp;至于其他类型的列表，例如双向链表或环形表，都可以使用相同的方法实现。然而，在Lua中很少需要这类结构，因为通常存在着一些更简单的方式来表示数据。例如，可以通过一个（几乎无限大的）数组来表示一个栈。

&emsp;&emsp;

###&emsp;&emsp;● 队列与双向队列

&emsp;&emsp;在Lua中实现队列的一种简单方法是使用`table`库的函数`insert`和`remove`。这两个函数可以在一个**数组**的任意位置插入或删除元素，并且根据操作要求移动后续元素。不过对于较大的结构，移动的开销是很大的。一种更高效的实现是使用两个索引，分别用于首尾的两个元素：

```lua
    function ListNew()
        return {first = 0, last = -1}
    end
```

&emsp;&emsp;为了避免污染全局名称空间，将在一个`table`内部定义所有的队列操作，这个`table`且称为`List`。这样，将上例重新写为：

```lua
    List = {}
    function List.new()
        return {first = 0, last = -1}
    end
```

&emsp;&emsp;现在就可以在常量时间内完成在两端插入或删除元素了。

```lua
    function List.pushfirst(list, value)
        local first = list.first - 1
        list.first = first
        list[first] = value
    end
    
    function List.pushlast(list, value)
        local last = list.last + 1
        list.last = last
        list[last] = value
    end

    function List.popfirst(list)
        local first = list.first
        if first > list.last then error("list is empty") end
        local value = list[first]
        list[first] = nil 			-- 为了允许垃圾收集
        list.first = first + 1
        return value
    end

    function List.poplast(list)
        local last = list.last
        if list.first > last then error("list is empty") end
        local value = list[last]
        list[last] = nil 			-- 为了允许垃圾收集
        list.last = last - 1
        return value
    end
```

&emsp;&emsp;如果希望该结构能严格地遵循队列的操作规范，那么只调用`pushlist`和`poplist`就可以了，这样`first`和`last`都会不断地增长。然而，在Lua中使用`table`来表示数组，即可以在1～20的范围内索引，也可以在16777216～16777236的范围内索引。因为Lua使用双精度来表示数字，程序可以以每秒1百万次的速度进行插入操作，如此运行200年都不会发生溢出问题。

&emsp;&emsp;

###&emsp;&emsp;● 集合与无序组（bag）

&emsp;&emsp;假设列一份程序代码中的所有标识符，并且过滤掉其中所有的保留字。一些C程序员会倾向于使用字符串的数组来表示保留字集合，然后搜索这个数组来查看一个单词是否属于该集合。为了提高搜索的速度，他们可能还会使用二叉树来表示该集合。

&emsp;&emsp;在Lua中有一种高效且简单的方式来表示这类集合，就是将集合元素作为索引放入一个`table`中。那么对于任意值都无须搜索`table`，只需用该值来索引`table`，并查看结果是否为`nil`。在当前假设的示例中，可如下：

```lua
    reserved = {["while"] = true, ["end"] = true, ["function"] = true, ["local"] = true, }

    for w in allwords() do
        if not reserved[w] then
            <对'w'作任意处理>  		-- 'w'不是保留字
        end
    end
```

&emsp;&emsp;若要使初始化过程变得更清晰，可以借助一个辅助函数来创建集合：

```lua
    function Set(list)
        local set = {}
        for _, l in ipairs(list) do set[l] = true end
        return set
    end

    reserved = Set {"while", "end", "function", "local"}
```

&emsp;&emsp;包，有时也称为“多重集合（Multiset）”，与普通的集合的不同之处在于其每个元素可以出现多次。在Lua中包的表示类似于上面的集合表示，只不过包需要将一个计数器与`table`的`key`关联。若要插入一个元素，则需要递增其计数器：

```lua
    function insert(bag, element)
        bag[element] = (bag[element] or 0) + 1
    end
```

&emsp;&emsp;若要删除一个元素，则需要递减其计数器：

```lua
    function remove(bag, element)
        local count = bag[element]
        bag[element] = (count and count > 1) and count - 1 or nil
    end
```

&emsp;&emsp;只有当计数器已存在或大于0时，才保留它。

&emsp;&emsp;

###&emsp;&emsp;● 字符串缓冲

&emsp;&emsp;假设正在编写一段关于字符串的代码，例如正在逐行地读取一个文件。典型的读取代码是这样的：

```lua
    local buff = ""
    for line in io.lines() do
        buff = buff .. line .. "\n"
    end
```

&emsp;&emsp;这段代码看似可以正常工作，但如果面对较大的文件时，它却会导致极大的性能开销。例如，用这段代码来读取一个350KB大小的文件就需要将近1分钟的时间。

&emsp;&emsp;为什么会这样呢？为了搞清楚执行期间到底发生了什么，来设想一下当前正在处于读取循环的中间。假设每行有20个字节，已读了2500行，那么buff现在就是一个50KB大小的字符串。当Lua作字符串连接`buff .. line .. "\n"`时，就创建了一个长50020字节的新字符串，并从buff中复制了50000字节到这个新字符串。这样，对于后面的每一行，Lua都需要移动50KB甚至更多的内存。在读取了100行（仅2KB）以后，Lua就已经移动了至少5MB的内存。此外，这个算法还具有二次复杂度。最后，当Lua完成了350KB的读取后，它已至少移动了50GB的数据。

&emsp;&emsp;这个问题不是Lua所特有的，在其他语言中，只要字符串是不可变的（immutable）值，也会有类似的问题。Java就是最有名的例证。

&emsp;&emsp;在继续讲解前，需要注明一下这种情况并不是一种常见的问题。对于较小的字符串，上述循环可以很好地工作。当需要读取整个文件时，Lua提供了`io.read("*all")`选项，这样便可以一次性读取整个文件。不过，Java也提供了`StringBuffer`来解决这个问题。在Lua中，我们可以将一个`table`作为字符串缓冲。其关键是使用函数`table.concat`，它会将给定列表中的所有字符串连接起来，并返回连接的结果。使用`concat`来重写上述循环：

```lua
    local t = {}
    for line in io.lines() do
        t[#t + 1] = line .. "\n"
    end
    local s = table.concat(t)
```

&emsp;&emsp;先前代码读取同样的文件需要1分钟，而这个实现只需花小于0.5秒的时间。

&emsp;&emsp;`concat`函数还有第二个可选的参数，可以指定一个插在字符串间的分隔符。有了这个分隔符参数，就不必在每行后插入一个“`\n`”了。

```lua
    local t = {}
    for line in io.lines() do
        t[#t + 1] = line
    end
    s = table.concat(t, "\n") .. "\n"
```

&emsp;&emsp;虽然`concat`函数会在字符串之间插入分隔符，但还需要在结尾处添加一个换行。为此上例在最后一次连接时复制了整个结果字符串，而这时的字符串也已经相当长了。没有直接的选项让`concat`插入这个额外的分隔符，不过可以“欺骗”`concat`，只需在t后面添加一个空字符串：

```lua
    t[#t + 1] = ""
    s = table.concat(t, "\n")
```

&emsp;&emsp;`concat`在空字符串前插入了这个额外的换行符，位于结果字符串的末尾。

&emsp;&emsp;从内部来看，`concat`和`io.read("*all")`都使用了一个相同的算法来连接许多小的字符串。标准库中的其他几个函数也使用这个算法来创建较大的字符串。下面来看一下它是如何工作的。

&emsp;&emsp;一开始循环采用了一种线性的方法来连接字符串，把较小的字符串逐个地连接起来，然后每次都将连接结果存入一个累加器。而新的算法可以避免这么做，它采用了一种二分的方法（binary approach）。它只在某些情况下将几个较小的字符串连接起来，然后再将结果字符串与更大的字符串进行连接。其算法的核心是一个栈，已创建的大字符串位于栈的底部，而较小的字符串则通过栈顶进入。对栈中元素的处理类似于著名的“汉诺塔（Tower of Hanoi）”。栈中的任意字符串都比下面的字符串短。如果压入的新字符串比下面已存在的字符串长，就将两者连接。然后，再将连接后的新字符串与更下面的字符串比较，如果是新建字符串更长的话，则再次连接它们。这样的连接一直向下延续应用，直到遇到一个更大的字符串或者到达了栈底为止。

```lua
    function addString(stack, s)
        stack[#stack + 1] = s 			-- 将's'压入栈
        for i = #stack - 1, i, -1 do
            if #stack[i] > #stack[i + 1] then
                break
            end
            stack[i] = stack[i] .. stack[i + 1]
            stack[i + 1] = nil
        end
    end
```

&emsp;&emsp;为了获取栈缓冲中的最终内容，只需连接其中所有的字符串就可以了。

&emsp;&emsp;

###&emsp;&emsp;● 图


&emsp;&emsp;就像其他编程语言一样，Lua允许程序员写出多种图的实现，每种实现都有其所适用的算法。接下来将介绍一种简单的面向对象的实现，其中结点表示为对象及边（arc）表示为结点间的引用。

&emsp;&emsp;每个结点表示为一个`table`，这个`table`有两个字段：`name`（结点的名称）和`adj`（与此结点邻接的结点集合）。由于从一个文本文件中读取图数据，所以需要一种通过一个结点名来找到该结点的方法。因此，使用了一个额外的`table`来将名称对应到结点。函数`name2node`可以根据给定的名称返回对应的结点：

```lua
    local function name2node(graph, name)
        if not graph[name] then
            -- 结点不存在，创建一个新的
            graph[name] = {name = name, adj = {}}
        end
        return graph[name]
    end
```


&emsp;&emsp;以下这个函数用于构造一个图。它逐行地读取一个文件，文件中的每行都有两个结点名称，表示了在两个结点之间有一条边，边的方向从第一个结点到第二个结点。函数对于每行都使用`string.match`来切分一行中的两个名称，然后根据名称查找结点（如果需要还会创建结点），最后连接结点。

```lua
    function readgraph()
        local graph = {}
        for line in io.lines() do
            -- 切分行中的两个名称
            local namefrom, nameto = string.match(line, "(%S+)%s+(%S+)")
            -- 查找相应的结点
            local from = name2node(graph, namefrom)
            local to = name2node(graph, nameto)
            -- 将'to'添加到'from'的邻接集合
            from.adj[to] = true
        end
        return graph
    end
```


&emsp;&emsp;以下这个函数展示了一个使用图的算法。函数`findpath`采用深度优先的遍历，在两个结点间搜索一条路径。它的第一个参数是当前结点，第二个参数是其目标结点，第三个参数用于保存从起点到当前结点的路径，最后一个参数是所有已访问过结点的集合（用于避免回路）。注意，该算法直接对结点进行操作，而不是它们的名称。例如，`visited`是一个结点集合，而不是结点名称的集合。`path`也一样是一个结点的列表。

```lua
    function findpath(curr, to, path, visited)
        path = path or {}
        visited = visited or {}
        if visited[curr] then 		-- 结点是否已访问过？
            return nil 					-- 这里没有路径
        end
        visited[curr] = true 		-- 将结点标记为已访问过
        path[#path + 1] = curr 		-- 将其加到路径中
        if curr == to then
            return path
        end
        -- 尝试所有的邻接结点
        for node in pairs(curr.adj) do
            local p = findpath(node, to, path, visited)
            if p then return p end
        end
        path[#path] = nil 			-- 从路径中删除结点
    end
```

&emsp;&emsp;为了测试上述代码，首先编写一个函数来打印一条路径，然后再编写一些代码来使其运行。

```lua
    function printpath(path)
        for i = 1, #path do
            print(path[i].name)
        end
    end

    g = readgraph()
    a = name2node(g, "a")
    b = name2node(g, "b")
    p = findpath(a, b)
    if p then printpath(p) end
```

🔚