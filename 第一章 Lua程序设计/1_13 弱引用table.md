##弱引用table

&emsp;&emsp;Lua采用了自动内存管理。一个程序只需创建对象，而无须删除对象。通过使用垃圾收集机制，Lua会自动地删除那些已成为垃圾的对象。这减轻了程序员在内存管理方面的负担，更重要的是将程序员从许多内存相关的bug（例如无效指针、内存泄漏）中解放出来。

&emsp;&emsp;Lua的垃圾收集器与一些其他的收集器有所不同，它没有环形引用的问题。当用到环形数据结构时，无须作出任何特殊的处理，它们也可以像其他数据一样被正常回收。不过，有时即使是再聪明的收集器也需要帮助。垃圾收集器无法解决所有内存管理的问题。

&emsp;&emsp;垃圾收集器只能回收那些它认为是垃圾的东西，它不会回收那些用户认为是垃圾的东西。一个典型的例子就是栈，栈通常由一个数组和一个表示顶部的索引来实现。这个数组的有效部分总是向顶部扩展的，但Lua却不知道。如果弹出一个元素时只是简单地递减顶部索引，那么这个仍留在数组中的对象对于Lua来说就不是垃圾。同理，对于那些存储在全局变量中的对象，即使程序不会再用到它们，但对于Lua来说就不是垃圾。在这两种情况中，都需要由用户来将这些对象变量赋值为`nil`。这样才能使它们得以释放。

&emsp;&emsp;不过，简单地清除引用可能还不够。有些情况需要程序和收集器之间进行更多的协作。例如，如果要将一些对象放在一个数组中，这看似简单，好像只需把每个对象插入数组即可。但是，当一个对象处于数组中时，它就无法被回收。这是因为即使当前没有其他地方在使用它，但数组仍引用着它。除非用户告诉Lua这项引用不应该阻碍此对象的回收，否则，Lua是无从得知这个事实的。

&emsp;&emsp;弱引用`table`（`weak table`）就是这样一种机制，用户能用它来告诉Lua一个引用不应该阻碍一个对象的回收。所谓“弱引用（`weak reference`）”就是一种会被垃圾收集器忽视的对象引用。如果一个对象的所有引用都是弱引用，那么Lua就可以回收这个对象了，并且还可以以某种形式来删除这些弱引用本身。Lua用“`弱引用table`”来实现“弱引用”，一个弱引用`table`就是一个具有弱引用条目的`table`。如果一个对象只被一个弱引用`table`所持有，那么最终Lua是会回收这个对象的。

&emsp;&emsp;`table`中有`key`和`value`，这两者都可以包含任意类型的对象。通常，垃圾收集器不会回收一个可访问`table`中作为`key`或`value`的对象。也就是说，这些`key`和`value`都是强引用（`strong reference`），它们会阻止对其所引用对象的回收。在一个弱引用`table`中，`key`和`value`是可以回收的。有3种弱引用`table`：具有弱引用`key`的`table`、具有弱引用`value`的`table`、同时具有两种弱引用`table`。不论是哪种类型的弱引用`table`，只要有一个`key`或`value`被回收了，那么它们所在的整个条目都会从`table`中删除。


&emsp;&emsp;一个table的弱引用类型是通过其元表中的`__mode`字段来决定的。这个字段的值应为一个字符串，如果这个字符串中包含字母‘`k`’，那么这个table的key是弱引用的；如果这个字符串中包含字母‘`v`’，那么这个`table`的`value`是弱引用的。下面这个示例虽然是人为制造的，但演示了弱引用`table`的一些基本行为：

```lua
    a = {}
    b = {__mode = 'k'}
    setmetatable(a, b)    -- 现在'a'的key就是弱引用
    key = {}              -- 创建第一个key
    a[key] = 1
    key = {}              -- 创建第二个key
    a[key] = 2
    collectgarbage()      -- 强制进行一次垃圾收集
    for k, v in pairs(a) do print(v) end
    --> 2
```

&emsp;&emsp;在本例中，第二句赋值`key = {}`会覆盖第一个`key`。当收集器运行时，由于没有其他地方在引用第一个`key`，因此第一个`key`就被回收了，并且`table`中的相应条目也被删除了。至于第二个`key`，变量`key`仍引用着它，因此它没有被回收。

&emsp;&emsp;注意，Lua只会回收弱引用`table`中的对象。而像数字和布尔这样的“值”是不可回收的。例如，对于一个插入`table`的数字`key`，收集器是永远不会删除它的。当然，如果一个数字`key`所对应的`value`被回收了，那么整个条目都会从这个弱引用`table`中删除。

&emsp;&emsp;字符串在此则显得有些特殊。虽然从实现的角度看，字符串是可回收的。但在有些方面，字符串却与其他可回收的对象不同。其他对象，例如`table`和函数都是显式创建的。又如，当Lua对表达式`{}`求值时，它就会创建一个新的`table`。同样地，求值`function()...end`时就会创建一个新函数。然而，当Lua对`"a".."b"`求值时，它会创建一个新字符串吗？如果当前系统中已有了一个字符串"ab"，它会复用吗？还是创建一个新的字符串？编译器会在运行程序前先创建这个字符串吗？这些都无关紧要，它们都是实现的细节。从程序员的角度看，字符串就是值，而非对象。因此，字符串就像数字和布尔一样，不会从弱引用`table`中删除。

&emsp;&emsp;

####备忘录（memoize）函数

&emsp;&emsp;一项通用的编程技术是“用空间换时间”。例如有一种做法就可以提高一些函数的运行速度，记录下函数计算的结果，然后当使用同样的参数再次调用该函数时，便可以复用之前的结果了。

&emsp;&emsp;假设有一个普通的服务器，在它收到的请求中包含Lua代码。每当服务器收到一个请求，它就要对代码字符串调用`loadstring`，然后再调用编译好的函数。不过，`loadstring`是一个昂贵的函数，而有些发给服务器的命令具有很高的频率，例如“`closeconnection()`”。与其每次收到一条常见命令就调用`loadstring`，还不如让服务器用一个辅助的`table`记录下所有调用`loadstring`的结果。因此，在每次调用`loadstring`前，服务器先检查`table`中是否已记录了代码字符串编译后的结果。如果没有，才调用`loadstring`，并将结果存储到`table`中。可以将这个行为写成一个新函数：

```lua
    local results = {}
    funcition mem_loadstring(s)
        local res = results[s]
        if res == nil then        -- 是否已记录过？
            res = assert(loadstring(s))        -- 计算新结果
            results[s] = res        -- 存下以备之后复用
        end
        return res
    end
```

&emsp;&emsp;这项优化节省的时间非常可观。但，它也可能导致不易察觉的开销。虽然有些命令会重复出现，但还有许多命令只发生一次。例如，`table results`会逐渐地积累服务器收到的所有命令及其编译结果。经过一定的时间后，这种累积会耗费服务器的内存。弱引用的`table`正好可以解决这个问题，如果`results table`具有弱引用的`value`，那么每次垃圾收集都会删除所有在执行时未使用的编译结果。

```lua
    local results = {}
    setmetatable(results, {__mode = 'v'})        -- 使用value称为弱引用
    funcition mem_loadstring(s)
        <如前>
```

&emsp;&emsp;实际上，由于`key`总是字符串，则可以使这个`table`编程完全弱引用。若这么做：

```lua
    setmetatable(results, {__mode = 'kv'})
```

&emsp;&emsp;则最终效果完全一样。

&emsp;&emsp;“备忘录”技术还可以用于确保某类对象的唯一性。假设一个系统用`table`来表示颜色，其中3个字段`red`、`green`和`blue`都具有相同的取值范围。最简单的颜色生成函数是：

```lua
    function createRGB(r, g, b)
        return {red=r, green=g, blue=b}
    end
```

&emsp;&emsp;通过备忘录技术，可以复用具有相同颜色的`table`。备忘录`table`的`key`可以根据颜色分量来生成，本例中是将颜色分量以分隔符连接起来：

```lua
    local results = {}
    setmetatable(result, {__mode='v'})        -- 使用value成为弱引用
    function createRGB(r, g, b)
        local key = r .. "-" .. g .. "-" .. b
        local color = results[key]
        if color == nil then
            color = {red=r, green=g, blue=b}
            results[key] = color
        end
        return color
    end
    
```

&emsp;&emsp;这种实现可以使用户通过原始的相等性操作符比较两种颜色。若两种同时存在的颜色相等，那么它们必定是由同一个`table`表示的。不过，相同的颜色也可能在不同时间由不同的`table`表示，这是因为期间执行过垃圾收集，清除了`results table`。只要一种颜色正在使用，就不会被清除出`results`。因此，只要一个颜色未被清除，它就可与新颜色进行比较，它的表示也可作为后续调用来复用。

&emsp;&emsp;


####对象属性

&emsp;&emsp;关于弱引用`table`，还有一项重要的应用是将属性与对象关联起来。有很多情况需要把有些属性绑定到某个对象，例如函数与其名称、`table`的默认值及数组的大小等。

&emsp;&emsp;当对象是一个`table`时，可以通过适当的`key`将属性存储在这个`table`中。正如先前所看到的，创建唯一性`key`的最简单办法是创建一个新对象（通常是一个`table`）。不过，若对象不是一个`table`，它就无法保存属性了。另外，即使是`table`，有时也不想将属性存储在原`table`中。例如，想保持属性的私有性，或者不想让属性扰乱`table`的遍历就需要用其他办法来关联属性与对象了。显然，使用外部`table`来关联它们是一种理想的做法。可以将对象作为`key`，对象的属性作为`value`。这个外部`table`可以保存任意对象的属性，Lua也允许将任何对象作为`table`的`key`。另外，存储在外部对象中的属性不会干扰其他对象，只要`table`本身是私有的，这些属性也会是私有的。

&emsp;&emsp;然而，这个看似完美的做法却有一个重大缺陷。当用户将一个对象作为外部`table`的`key`时，就是引用了它。Lua是无法回收一个作为`table key`的对象。如果用这个外部`table`来关联函数和函数名，那么这些函数就永远无法回收。用户可以使用弱引用`table`来解决这个问题。而本例需要的是弱引用`key`。当一个弱引用`key`没有其他引用时，Lua就可以回收它。注意，这个`table`不能使用弱引用`value`，否则“存留的”对象的属性就有可能被回收。

&emsp;&emsp;

####回顾table的默认值

&emsp;&emsp;前面章节讨论了如何实现具有非`nil`默认值的`table`。库定义的元方法一节中注明了还有两种技术需要弱引用`table`的支持。这里将介绍两种用于默认值的技术，它们其实是上述备忘录和对象属性的特殊应用。

&emsp;&emsp;第一种做法是使用一个弱引用`table`，通过它将每个`table`与其默认值关联起来：

```lua
    local defaults = {}
    setmetatable(defaults, {__mode='k'})
    local mt = {__index=function(t) return defaults[t] end}
    function setDefault(t, d)
        defaults[t] = d
        setmetatable(t, mt)
    end
```

&emsp;&emsp;如果`defaults`没有弱引用`key`，它就会使所有具有默认值的`table`持久存在下去。

&emsp;&emsp;第二种做法是对每种不同的默认值使用不同的元表。不过，只要有重复的默认值，就复用同样的元表。这是备忘录的典型应用：

```lua
    local metas = {}
    setmetatable(metas, {__mode = 'v'})
    function setDefault(t, d)
        local mt = metas[d]
        if mt == nil then
            mt = {__index = function() return d end}
            metas[d] = mt 			-- 备忘录
        end
        setmetatable(t, mt)
    end
```

&emsp;&emsp;这里用到了弱引用`value`，这样当`metas`中的元表在不使用时就可以被回收了。

&emsp;&emsp;这两种默认值的实现，哪种更好呢？一般而言，它们具有类似的复杂度和性能表现。第一种做法需要为每个`table`的默认值（`defaults`中的一个条目）使用内存。而第二种做法则需要为每种不同的默认值使用一组内存（一个新`table`，一个新`closure`和`metas`中的一个条目）。因此，如果程序中有上千个`table`和一些默认值，第二种做法则是首选。但如果只有很少的`table`共享几个公用的默认值，那么就应该选择第一种做法。

🔚
