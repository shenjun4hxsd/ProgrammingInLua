##迭代器与范型for

&emsp;&emsp;本张将介绍如何编写适用于泛型`for`的迭代器（Iterator）。先从简单的迭代器入手，然后将学习如何利用泛型`for`的各种能力来编写更简单、更有效的迭代器。

&emsp;&emsp;
```
    ● 迭代器与closure
    ● 泛型for的语义
    ● 无状态的迭代器
    ● 具有复杂状态的迭代器
    ● 真正的迭代器
```

&emsp;&emsp;



###&emsp;&emsp;• 迭代器与closure

&emsp;&emsp;所谓“迭代器”就是一种可以遍历一种集合中所有元素的机制。在Lua中，通常将迭代器表示为函数。每调用一次函数，即返回集合中的“下一个”元素。

&emsp;&emsp;每个迭代器都需要在每次成功调用之间保持一些状态，这样才能知道它所在的位置及如何步进到下一个位置。`closure`对于这类任务提供了极佳的支持，一个`closure`就是一种可以访问其外部嵌套环境中的局部变量的函数。对于`closure`而言，这些变量就可用于在成功调用之间保持状态值，从而使`closure`可以记住它在一次遍历中所在的位置。当然，为了创建一个新的`closure`，还必须创建它的这些“非局部的变量”。因此一个`closure`结构通常涉及到两个函数：`closure`本身和一个用于创建该`closure`的工厂函数。


```lua
	function values( t )
		local i = 0;
		return function( )
			i = i + 1;
			return t[i];
		end
	end
	
	t = { 10, 20, 30 }
	iter = values(t)
	while true do
		local element = iter();
		if element == nil then break end
		print( element )
	end
	
	for element in values(t) do
		print( element )
	end
```

&emsp;&emsp;作为示例，来为列表编写一个简单的迭代器。与`ipairs`不同的是该迭代器并不是返回每个元素的索引，而是返回元素的值：

```lua
    function values(t)
        local i = 0
        return function() i = i + 1; return t[i] end
    end
```

&emsp;&emsp;在本例中，`values`就是一个工厂。每当调用这个工厂时，它就创建一个新的`closure`（即迭代器本身）。这个`closure`将它的状态保存在其外部变量`t`和`i`中。每当调用这个迭代器时，它就从列表`t`中返回下一个值。直到最后一个元素返回后，迭代器就会返回`nil`，以此表示迭代的结束。


&emsp;&emsp;可以在一个`while`循环中使用这个迭代器：

```lua
    t = {10, 20, 30}
    iter = values(t)		-- 创建迭代器
    while true do
        local element = iter()
        if element == nil then break end
        print(element)
    end
```

&emsp;&emsp;然而使用泛型`for`则更为简单。接下来会发现，它正是为这种迭代而设计的：

```lua
    t = {10, 20, 30}
    for element in values(t) do
        print(element)
    end
```

&emsp;&emsp;泛型`for`为一次迭代循环做了所有的簿记工作。它在内部保存了迭代器函数，因此不再需要`iter`变量。它在每次新迭代时调用迭代器，并在迭代器返回`nil`时结束循环。在下一节中将会看到泛型`for`能够做更多的事情。

&emsp;&emsp;

>&emsp;&emsp;下面是一个更高级的示例，展现了一个可以遍历当前输入文件中所有单词的迭代器 -- `allwords`。为了完成这样的遍历，需要保持两个值：当前行的内容（变量`line`）及在该行中所处的位置（变量`pos`）。有了这些信息，就可以不断产生下一个单词了。这个迭代器函数的主要部分就是`string.find`的调用。此调用会在当前行中，以当前位置作为起始来搜索一个单词。使用模式（`pattern`）‘`%w+`’来描述一个“单词”，它用于匹配一个或多个的文字／数字字符。如果`string.find`找到了一个单词，迭代器就会将当前位置更新为该单词之后的第一个字符，并返回该单词。否则，它就读取新的一行并反复这个搜索过程。若没有剩余的行，则返回`nil`，以此表示迭代的结束。

>&emsp;&emsp;尽管迭代器本身具有复杂性，但`allwords`的使用还是很简明易懂的：

>```lua
    for word in allwords() do
        print(word)
    end
```

>&emsp;&emsp;对于迭代器而言，一种常见的情况就是：编写迭代器本身或许不太容易，但使用它们却是很容易的。这也不会成为一个大问题，因为通常使用Lua编程的最终用户不会去定义迭代器，而只是使用那些程序提供的迭代器。

>```lua
    function allwords()
        local line = io.read()    -- 当前行
        local pos = 1             -- 一行中的当前位置
        return function()         -- 迭代器函数
            while line do         -- 若为有效的行内容就进入循环
                local s, e = string.find(line, "%w+", pos)
                if s then         -- 是否找到一个单词
                    pos = e + 1   -- 该单词的下一个位置
                    return string.sub(line, s, e)    -- 返回该单词
                else
                    line = io.read()    -- 没有找到单词，尝试下一行
                    pos = 1             -- 在第一个位置上重新开始
                end
            end
            return nil                  -- 没有其余行了，遍历结束
        end
    end
```

&emsp;&emsp;

###&emsp;&emsp;• 泛型for的语义

&emsp;&emsp;上述的那些迭代器都有一个缺点，就是需要为每个新的循环都创建一个新的`closure`。对于大多数情况而言，这或许不会有什么问题。例如在之前的`allwords`迭代器中，创建一个`closure`的代价相对于读取整个文件的代价而言，几乎可以忽略不计。但是，在另外一些情况下，这样的开销就不太容易接受了。因此，在这类情况中，希望能通过泛型`for`的自身来保存迭代器状态。在本节中会详细说明泛型`for`的这种保存状态的机制。

&emsp;&emsp;**泛型`for`在循环过程内部保存了迭代器函数。实际上它保存着3个值：一个迭代器函数、一个恒定状态和一个控制变量。**接下来将对此进行详细说明。

&emsp;&emsp;**泛型`for`的语法如下**：

```lua
    for <var-list> in <exp-list> do
        <body>
    end
```

&emsp;&emsp;其中，`<var-list>` 是一个或多个变量名的列表，以逗号分隔；`<exp-list>`是一个或多个表达式的列表，同样以逗号分隔。通常表达式列表只有一个元素，即一句对迭代器工厂的调用。

&emsp;&emsp;例如，如下代码：

```lua
    for k, v in pairs(t) do print(k, v) end
```

&emsp;&emsp;其中变量列表是“`k, v`”，表达式列表只有一个元素`pairs(t)`。一般来说变量列表中也只有一个变量，例如下面这个循环：

```lua
    for line in io.lines() do
        io.write(line, "\n")
    end
```

>&emsp;&emsp;变量列表的第一个元素称为“控制变量”。在循环过程中该值决不会为`nil`，因为当它为`nil`时循环就结束了。

&emsp;&emsp;**`for`做的第一件事情是对`in`后面的表达式求值。这些表达式应该返回3个值供`for`保存：迭代器函数、恒定状态和控制变量的初值。**这里有点类似于多重赋值，即只有最后一个表达式才会产生多个结果，并且只会保留前3个值，多余的值会被丢弃；而不足的话，将以`nil`补足。

&emsp;&emsp;在初始化步骤之后，`for`会以恒定状态和控制变量来调用迭代器函数。然后`for`将迭代器函数的返回值赋予变量列表中的变量。如果第一个返回值为`nil`，那么循环终止。否则，`for`执行它的循环体，随后再次调用迭代器函数，并重复这个过程。

>&emsp;&emsp;更明确地说，以下语句：

>```lua
    for var_1, ..., var_n in <explist> do <block> end
```

>&emsp;&emsp;等价于以下代码：

>```lua
    do
        local _f, _s, _var = <explist>
        while true do
            local var_1, ..., var_n = _f(_s, _var)
            _var = var_1
            if _var == nil then break end
            <block>
        end
    end
```

&emsp;&emsp;因此，假设迭代器函数为`f`，恒定状态为`s`，控制变量的初值为$$a_0$$。那么在循环过程中控制变量的值依次为$$a_1$$ = f(s, $$a_0$$)、$$a_2$$ = f(s, $$a_1$$)，以此类推，直至$$a_i$$为`nil`结束循环。如果`for`还有其他变量，那么它们也会在每次调用`f`后获得额外的值。

&emsp;&emsp;

###&emsp;&emsp;• 无状态的迭代器

&emsp;&emsp;所谓“无状态的迭代器”，正如其名所暗示的那样，就是一种自身不保存任何状态的迭代器。因此，我们可以在多个循环中使用同一个无状态的迭代器，避免创建新的`closure`开销。

&emsp;&emsp;**在每次迭代中，`for`循环都会用恒定状态和控制变量来调用迭代器函数。一个无状态的迭代器可以根据这两个值来为下次迭代生成下一个元素。**这类迭代器的一个典型例子就是`ipairs`，它可以用来迭代一个数组的所有元素：

```lua
    a = {"one", "two", "three"}
    for i, v in ipairs(a) do
        print(i, v)
    end
```

&emsp;&emsp;在这里，迭代的状态就是需要遍历的`table`（一个恒定状态，它不会在循环中改变）及当前的索引值（控制变量）。`ipairs`（工厂）和迭代器都非常简单，在Lua中就可以编写出来：

```lua
    local function iter(a, i)
        i = i + 1
        local v = a[i]
        if v then
            return i, v
        end
    end

    function ipairs(a)
        return iter, a, 0
    end
```

&emsp;&emsp;在Lua调用`for`循环中的`ipairs(a)`时，它会获得3个值：迭代器函数`iter`、恒定状态`a`和控制变量的初值0。然后Lua调用`iter(a, 0)`，得到1，`a[1]`。在第二次迭代中，继续调用`iter(a,1)`，得到2，`a[2]`，以此类推，直至得到第一个nil元素为止。


&emsp;&emsp;**函数`pairs`与`ipairs`类似，也是用于遍历一个`table`中的所有元素。不同的是，它的迭代器函数是Lua中的一个基本函数`next`。**

```lua
    function pairs(t)
        return next, t, nil
    end
```

>&emsp;&emsp;在调用`next(t,k)`时，`k`是`table t`的一个`key`。此调用会以`table`中的任意次序返回一组值：此`table`的下一个`key`，及这个`key`所对应的值。而调用`next(t,nil)`时，返回`table`的第一组值。若没有下一组值时，`next`返回`nil`。

&emsp;&emsp;有些用户喜欢不通过`pairs`调用而直接使用`next`:

```lua
    for k, v in next, t do
        <loop body>
    end
```
&emsp;&emsp;记住，Lua会自动将`for`循环中表达式列表的结果调整为3个值。因此上例中得到了`next`、`t`和`nil`，这也正与调用`pairs(t)`的结果完全一致。

&emsp;&emsp;关于无状态迭代器的另一个有趣例子是一种可以遍历链表的迭代器。

```lua
    local function getnext(list, node)
        if not node then
            return list
        else
            return node.next
        end
    end

    function traverse(list)
        return getnext, list, nil
    end
```

&emsp;&emsp;这里使用了一个技巧就是将链表的头节点作为恒定状态（`traverse`返回的第二个值），而将当前节点作为控制变量。第一次调用迭代器函数`getnext`时，`node`为`nil`，因此函数返回`list`作为第一个结点。在后续调用中`node`不再为`nil`了，所以迭代器如期望的那样返回`node.next`。

&emsp;&emsp;对于此迭代器的使用则非常简单：

```lua
    list = nil
    for line in io.lines() do
        list = {val = line, next = list}
    end

    for node in traverse(list) do
        print(node.val)
    end
```

>```lua
    -- 链表的实现

>     node = {}
    head = node

>    -- 初始化
    function init(v)
        node.val = v
    end

>    -- 在尾部插入
    function push_back(v)
        node.next = {val = v}
        node = node.next
    end

>     init(10)
    push_back(8)
    push_back(6)

>     -- 迭代器函数
>     local function getnext(list, node)
        if node then
            return node.next
        else
            return list
        end
    end

>     function traverse( list )
        return getnext, list, nil
    end

>     for v in traverse(head) do
        print( v.val )
    end
```

&emsp;&emsp;

###&emsp;&emsp;• 具有复杂状态的迭代器

&emsp;&emsp;通常，迭代器需要保存许多状态，可是泛型`for`却只提供的一个恒定状态和一个控制变量用于状态的保存。一个最简单的解决方法就是使用`closure`。或者还可以将迭代器所需的所有状态打包为一个`table`，保存在恒定状态中。一个迭代器通过这个`table`就可以保存任意多的数据了。此外，它还能在循环过程中改变这些数据。虽然，在循环过程中恒定状态总是同一个`table`(故称之“恒定”)，但这个`table`的内容却可以发生改变。由于这种迭代器可以在恒定状态中保存所有数据，所以它们通常可以忽略泛型`for`提供的第二个参数（控制变量）。

&emsp;&emsp;作为该技术的一个示例，将重写`allwords`迭代器，这个迭代器可以遍历当前输入文件中的所有单词。这次将它的状态保存到一个`table`中，这个`table`具有两个字段：`line`和`pos`。


&emsp;&emsp;迭代的起始函数比较简单，它只需返回迭代器函数和初始状态：

```lua
    local iterator		-- 在后面定义

    function  allwords()
        local state = {line = io.read(), pos = 1}
        return iterator, state
    end
```

&emsp;&emsp;`iterator`函数才开始真正的工作：


```lua
    function iterator(state)
        while state.line do 		-- 若为有效的行内容就进入循环
            -- 搜索下一个单词
            local s, e = string.find(state.line, "%w+", state.pos)
            if s then 			-- 找到了一个单词？
                -- 更新下一个位置（到这个单词之后）
                state.pos = e + 1
                return string.sub(state.line, s, e)
            else 		-- 没有找到单词
                state.line = io.read()		-- 尝试下一行...
                state.pos = 1		-- 从第一个位置开始
            end
        end
        return nil 			-- 没有更多行了，结束循环
    end
```


&emsp;&emsp;尽可能地尝试编写无状态的迭代器，那些迭代器将所有状态保存在`for`变量中，不需要在开始一个循环时创建任何新的对象。如果迭代器无法套用这个模型，那么就应该尝试使用`closure`。`closure`显得更加优雅一点，通常一个基于`closure`实现的迭代器会比一个使用`table`的迭代器更为高效。这是因为，首先创建一个`closure`就比创建一个`table`更廉价，其次访问“非局部的变量”也比访问`table`字段更快。以后会看到另一种使用协同程序（`coroutine`）编写迭代器的方式，这种方式是功能最强的，但稍微有一点开销。

&emsp;&emsp;

###&emsp;&emsp;• 真正的迭代器


&emsp;&emsp;“迭代器”这个名称多少有点误导的成分。因为迭代器并没有做实际的迭代，真正做迭代的是`for`循环。而迭代器只是为每次迭代提供一些成功后的返回值。或许，更准确地应称其为“生成器”。不过“迭代器”这个名称已在其他语言中被广泛使用，例如Java。

&emsp;&emsp;还有一种创建迭代器的方式就是，在迭代器中做实际的迭代操作。当使用这种迭代器时，就不需要写一个循环了。相反，需要一个描述了在每次迭代时需要做什么的参数，并以此参数来调用迭代器。更确切地说，迭代器接受一个函数作为参数，并在其内部的循环中调用这个函数。

&emsp;&emsp;在此列举一个更具体的例子，就使用这种风格来再次重写`allwords`迭代器：

```lua
    function allwords(f)
        for line in io.lines() do
            for word in string.gmatch(line, "%w+") do
                f(word)		-- call the function
            end
        end
    end
```


&emsp;&emsp;使用这个迭代器时，需要传入一个描述循环体的函数。例如，只想打印每个单词，那么可以使用`print`：

```lua
    allwords(print)
```

&emsp;&emsp;通常，还可以使用一个匿名函数作为循环体。例如，以下代码计算了单词“`hello`”在输入文件中出现的次数：

```lua
    local count = 0
    allwords(function(w)
        if w == "hello" then count = count + 1 end
    end)
    print(count)
```


&emsp;&emsp;同样的任务若采用之前的迭代器风格也不是很困难的：

```lua
    local count = 0
    for w in allwords() do
        if w == "hello" then count = count + 1 end
    end
    print(count)
```

&emsp;&emsp;“真正的迭代器”在老版本的Lua中曾非常流行，那时语言还没有`for`语句。那它们比之生成器风格的迭代器又如何呢？这两种风格都有大致相同的开销，即每次迭代都有一次函数调用。编写真正的迭代器相对比较容易。不过，生成器风格的迭代器则更具灵活性。这种灵活性体现在两方面，首先它允许两个或多个并行的迭代过程，其次它允许在迭代体中使用`break`和`return`语句。对于真正的迭代器来说，`return`语句只能从匿名函数中返回，而并不能从做迭代的函数中返回。综上所述，我一般更喜爱生成器。

🔚