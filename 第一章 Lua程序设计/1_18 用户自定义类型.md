##用户自定义类型

&emsp;&emsp;本章将介绍如何用C语言编写新的类型来扩展Lua。下面将从一个小示例入手，使用元表和其他机制来扩展它。

&emsp;&emsp;这个示例实现了一种很简单的类型--布尔数组。选用这个示例是因为它不涉及到复杂的算法，从而可以使读者专注于API的问题。不过，这个示例本身还是具有实用价值的。当然，可以在Lua中用`table`来实现布尔数组。但C语言实现可以将每个布尔值存储在一个`bit`中，从而将内存用量减少到不足`table`方法的`3%`。

&emsp;&emsp;这个实现需要以下定义：

```lua
    #include <limits.h>
    
    #define BITS_PER_WORD (CHAR_BIT* sizeof(unsigned int))
    #define I_WORD(i)    ((unsigned int)(i)/BITS_PER_WORD)
    #define I_BIT(i)    (1 << ((unsigned int)(i) % BITS_PER_WORD))
```

&emsp;&emsp;`BITS_PER_WORD`是一个无符号整型的`bit`数量。宏`I_WORD`根据给定的索引来计算对应的`bit`位所存放的`word`（字），`I_BIT`计算出一个掩码，用于访问这个`word`中的正确`bit`。

&emsp;&emsp;可以使用以下结构来表示数组：

```lua
    typedef struct NumArray {
        int size;
        unsigned int values[1];    /* 可变部分 */
    } NumArray
```

&emsp;&emsp;这里。由于C89标准不允许分配0长度的数组，所以声明了数组`values`需要有一个元素来作为占位符。接下来会在分配数组时定义实际的大小。下面这个表达式可以计算出具有`n`个元素的数组大小：

```lua
    sizeof(NumArray) + I_WORD(n - 1)*sizeof(unsigned int)
```

&emsp;&emsp;注意，这里无须对`I_WORD`加1，因为原来的结构中已经包含了一个元素的空间。

&emsp;&emsp;

####&emsp;&emsp;userdata

&emsp;&emsp;首先要面临的问题是如何在Lua中表示这个NumArray结构。Lua为此提供了一种基本类型`userdata`。`userdata`提供了一块原始的内存区域，可以用来存储任何东西。并且，在Lua中`userdata`没有任何预定义的操作。

&emsp;&emsp;函数`lua_newuserdata`会根据指定的大小分配一块内存，并将对应的userdata压入栈中，最后返回这个内存块的地址：

```lua
    void *lua_newuserdata(lua_State *L, size_t size);
```

&emsp;&emsp;如果由于某些原因，需要通过其他机制来分配内存。那么可以创建只有一个指针大小的`userdata`，然后将指向真正内存块的指针存入其中。在下一章中就有这样的例子。

&emsp;&emsp;以下函数就用`lua_newuserdata`创建了一个新的布尔数组：

```lua
    static int newarray(lua_State *L) {
        int i, n;
        size_t nbytes;
        NumArray *a;
        
        n = luaL_checkint(L, 1);
        luaL_argcheck(L, n >= 1, 1, "invalid size")
        nbytes = sizeof(NumArray) + I_WORD(n - 1)*sizeof(unsigned int);
        a = (NumArray *)lua_newuserdata(L, nbytes);
        
        a->size = n;
        for(i = 0; i <= I_WORD(n-1); i++)
            a->values[i] = 0;        /* 初始化数组 */
        return 1;        /* 新的userdata已在栈上 */
    }
```

&emsp;&emsp;其中，宏`luaL_checkint`只是在调用`luaL_checkinteger`后进行了一个类型转换。只要在Lua中注册好`newarray`，就可以通过语句`a=array.new(1000)`来创建一个新数组。

&emsp;&emsp;可以通过这样的调用`array.set(array, index, value)`，在数组中存储元素。后面的内容会介绍如何使用元表来实现更传统的语法`array[index]=value`。无论哪种写法，底层函数都是相同的。

&emsp;&emsp;下面将遵循Lua惯例，假设索引从1开始。

```lua
    static int setarray(lua_State *L)
        NumArray *a = (NumArray *)lua_touserdata(L, 1);
        int index = luaL_checkint(L, 2) - 1;
        luaL_checkany(L, 3);
        
        luaL_argcheck(L, a != NULL, 1, "'array' expected");
        
        luaL_argcheck(L, 0 <= index && index < a->size, 2, "index out of range");
        
        if(lua_toboolean(L, 3))
            a->values[I_WORD(index)] |= I_BIT(index);    /* 设置bit */
        else
            a->values[I_WORD(index)] &= ~I_BIT(index);    /* 重置bit */
        return 0;
```

&emsp;&emsp;由于Lua中任何值都可以转换为布尔，所以这里对第3个参数使用`luaL_checkany`，它只确保了在这个参数位置上有一个值。如果用错误的参数调用了`setarray`，就会得到这样的错误消息：

```lua
    array.set(0, 11, 0)
    --> stdin:1: bad argument #1 to 'set' ('array' expected)
    array.set(a, 0)
    --> stdin:1: bad argument #3 to 'set' (value expected)
```

&emsp;&emsp;下一个函数用于检索元素：

```lua
    static int getarray(lua_State *L) {
        NumArray *a = (NumArray *)lua_touserdata(L, 1);
        int index = luaL_checkint(L, 2) - 1;
        
        luaL_argcheck(L, a != NULL, 1, "'array' expected");
        
        lua_pushboolean(L, a->values[I_WORD(index)] & I_BIT(index));
        return 1;
    }
```

&emsp;&emsp;下面还定义了一个函数用于检索一个数组的大小：

```lua
    static int getsize(lua_State *L) {
        NumArray *a = (NumArray *)lua_touserdata(L, 1);
        luaL_argcheck(L, a != NULL, 1, "'array' expected");
        lua_pushinteger(L, a->size);
        return 1;
    }
```

&emsp;&emsp;最后，需要一些代码来初始化这个库：

```lua
    static const struct luaL_Reg arraylib [] = {
        {"new", newarray},
        {"set", setarray},
        {"get", getarray},
        {"size", getsize},
        {NULL, NULL}
    };
    
    int luaopen_array(lua_State *L) {
        luaL_register(L, "array", arraylib);
        return 1;
    }
```

&emsp;&emsp;同样其中用到了辅助库的`luaL_register`，它会根据给定的名称（本例中为“`array`”）创建一个`table`，并用数组`arraylib`中指定的名称/函数对来填充它。

&emsp;&emsp;在打开库后，就可以在Lua中使用这个新类型了：

```lua
    a = array.new(1000)
    print(a)    			--> userdata: 0x8064d48
    print(array.size(a))	--> 1000
    for i = 1, 1000 do
        array.set(a, i, i%5 == 0)
    end
    print(array.get(a, 10))		--> true
```

&emsp;&emsp;

####&emsp;&emsp;元表

&emsp;&emsp;当前的实现有一个重大的安全漏洞，假定用户写了这样的语句`array.set(io.stdin, 1, false)`。`io.stdin`的值是一个`userdata`，是一个文件流指针（FILE *）。由于这是一个`userdata`，`array.set`会认为它是一个合法的参数，结果就使内存遭到破坏（如果幸运的话，可能会得到一个索引超出范围的错误）。但对于有些Lua库来说，这种行为是不可接受的。问题的原因不在于如何使用一个C程序库，而在于程序库不应破坏C数据或在Lua中导致核心转储（Core Dump）。

&emsp;&emsp;一种辨别不同类型的`userdata`的方法是，为每种类型创建一个唯一的元表。每当创建了一个`userdata`后，就用相应的元表来标记它。而每当得到一个`userdata`后，就检查它是否拥有正确的元表。由于Lua代码不能改变`userdata`的元表，因此也就无法欺骗代码了。

&emsp;&emsp;另外还需要有个地方来存储这个新的元表，然后才能用它来创建新的`userdata`，并检查给定的`userdata`是否具有正确的类型。在前面已提到过，有三个候选地可用于存储元表：注册表、环境或程序库中函数的`upvalue`。在Lua中，通常习惯是将所有新的C类型注册到注册表中，以一个类型名作为`key`，元表作为`value`。由于注册表中还有其他的内容，所以必须小心地选择类型名，以避免与`key`冲突。在示例中，将使用“`LuaBook.array`”作为其新类型的名称。

&emsp;&emsp;通常，辅助库中提供了一些函数来帮助实现这些内容。可以使用的辅助库函数有：

```lua
    int luaL_newmetatable(lua_State *L, const char *tname);
    void luaL_getmetatable(lua_State *L, const char *tname);
    void *luaL_checkudata(lua_State *L, int index, const char *tname);
```

&emsp;&emsp;`luaL_newmetatable`函数会创建一个新的table用作元表，并将其压入栈顶，然后将这个`table`与注册表中的指定名称关联起来。`luaL_getmetatable`函数可以在注册表中检索与`tname`关联的元表。`luaL_checkudata`可以检查栈中指定位置上是否为一个`userdata`，并且是否具有与给定名称相匹配的元表。如果该对象不是一个`userdata`，或者它不具有正确的元表，就会引发一个错误；否则，它就返回这个`userdata`的地址。

&emsp;&emsp;现在可以修改前面的实现了。第一步是修改打开库的函数。新版本必须为数组创建一个元表：

```lua
    int luaopen_array(lua_State *L) {
        luaL_newmetatable(L, "LuaBook.array");
        luaL_register(L, "array", arraylib);
        return 1;
    }
```

&emsp;&emsp;下一步是修改`newarray`，使其能为所有新建的数组设置这个元表：

```lua
    static int newarray(lua_State *L) {
        <如前>

        luaL_getmetatable(L, "LuaBook.array");
        lua_setmetatable(L, -2);

        return 1; 		/* 新的userdata已在栈中 */
    }
```

&emsp;&emsp;`lua_setmetatable`函数会从栈中弹出一个`table`，并将其设为指定索引上对象的元表。在本例中，这个对象就是一个新建的`userdata`。

&emsp;&emsp;最后，`setarray`、`getarray`和`getsize`必须检查其第一个参数是否为一个合法的数组。为了简化这样的任务，定义了以下宏：

```lua
    #define checkarray(L) \
        (NumArray *)luaL_checkudata(L, 1, "LuaBook.array")
```

&emsp;&emsp;使用这个宏`getsize`同样简单：

```lua
    static int getsize(lua_State *L)
        NumArray *a = checkarray(L);
        lua_pushinteger(L, a->size);
        return 1;
```

&emsp;&emsp;由于`setarray`和`getarray`使用了一段相同的代码来检查第二个索引参数，所以将这段公共部分单独组成以下函数：

```lua
    static unsigned int *getindex(lua_State *L, unsigned int *mask)
    {
        NumArray *a = checkarray(L);
        int index = luaL_checkint(L, 2) - 1;

        luaL_argcheck(L, 0 <= index && index < a->size, 2, 
                 "index out of range");

        /* 返回元素地址 */
        *mask = I_BIT(index);
        return &a->values[I_WORD(index)];
    }
```

&emsp;&emsp;使用了`getindex`的`setarray`和`getarray`也同样简单明了：

```lua
    static int getarray(lua_State *L) {
        unsigned int mask;
        unsigned int *entry = getindex(L, &mask);
        luaL_checkany(L, 3);
        if(lua_toboolean(L, 3))
            *entry |= mask;
        else
            *entry &= ~mask;
        return 0;
    }

    static int getarray(lua_State *L) {
        unsigned int mask;
        unsigned int *entry = getindex(L, &mask);
        lua_pushboolean(L, *entry & mask);
        return 1;
    }
```

&emsp;&emsp;现在，如果试图这样调用`array.get(io.stdin, 10)`，就会得到一个正确的错误消息：

```lua
    error: bad argument #1 to 'get' ('array' expected)
```

&emsp;&emsp;

####&emsp;&emsp; 面向对象的访问

&emsp;&emsp;下一步是将这种类型变换成一个对象，然后就可以用普通的面向对象语法来操作它的实例了。例如：

```lua
    a = array.new(1000)
    print(a:size())  		--> 1000
    a:set(10, true)
    print(a:get(10))		--> true
```

&emsp;&emsp;注意，`a:size()`等价于`a.size(a)`。因此，必须使表达式`a.size`返回前面定义的函数`getsize`。实现这点的关键是使用`__index`元方法。对于`table`而言，Lua会在找不到指定`key`时调用这个元方法。对于`userdata`，则会在每次访问时都调用它，因为`userdata`根本没有`key`。

&emsp;&emsp;假设，运行了以下代码：

```lua
    local metaarray = getmetatable(array.new(1))
    metaarray.__index = metaarray
    metaarray.set = array.set
    metaarray.get = array.get
    metaarray.size = array.size
```

&emsp;&emsp;第一行创建了一个数组，并将它的元表赋予了`metaarray`。然后将`metaarray.__index`设为`metaarray`。当对`a.size`求值时，由于`a`是一个`userdata`，所以Lua无法在对象`a`中找到`key "size"`。因此，Lua会尝试通过`a`的元表的`__index`字段来查找这个值，而这个字段也就是`metaarray`自身。由于`metaarray.size`为`array.size`，因此`a.size(a)`的结果就是`array.size(a)`。

&emsp;&emsp;其实，在C中也可以达到相同的效果，甚至还可以做得更好。现在的数组是一种具有操作的对象，可以无须在`table array`中保存这些操作。程序库只要导出一个用于创建新数组的函数new就可以了，所有其他操作都可作为对象的方法。C代码同样可以直接注册这些方法。

&emsp;&emsp;操作`getsize`、`getarray`和`setarray`无须作任何改变，唯一需要改变的是注册它们的方式。现在，需要修改打开程序库的函数。首先，需要设置两个独立的函数列表，一个用于常规的函数，另一个用于方法：

```lua
    static const struct luaL_Reg arraylib_f[] = {
        {"new", newarray},
        {NULL, NULL}
    };

    static const struct luaL_Reg arraylib_m[] = {
        {"set", setarray},
        {"get", getarray},
        {"size", getsize},
        {NULL, NULL}
    };
```

&emsp;&emsp;新的打开函数`luaopen_array`必须创建元表，并将它赋予`__index`字段，然后在元表中注册所有的方法，最后创建并填充`array table`：

```lua
    int luaopen_array(lua_State *L) {
        luaL_newmetatable(L, "LuaBook.array");

        /* 元表.__index = 元表 */
        lua_pushvalue(L, -1);	/* 复制元表 */

        luaL_register(L, NULL, arraylib_m);

        luaL_register(L, "array", arraylib_f);
        return 1;
    }
```

&emsp;&emsp;其中用到了`luaL_register`的另一个特性。在第一次调用中，以`NULL`作为库名，`luaL_register`不会创建任何用于存储函数的`table`，而是以栈顶的`table`作为存储函数的`table`。在本例中，栈顶`table`就是元表本身，因此`luaL_register`会将所有的方法放入其中。第二次调用`luaL_register`则提供了一个库名，它就根据此名（`array`）创建了一个新`table`，并将指定的函数注册在这个`table`中（也就是本例中唯一的`new`函数）。

&emsp;&emsp;最后，给这个数组类型添加一个`__tostring`方法。使`print(a)`可以打印出“`array`”以及数组的大小，就像“`array(1000)`”。这个函数如下：

```lua
    int array2string(lua_State *L) {
        NumArray *a = checkarray(L);
        lua_pushfstring(L, "array(%d)", a->size);
        return 1;
    }
```

&emsp;&emsp;`lua_pushfstring`可以在栈顶创建并格式化一个字符串。另外，还需要将`array2string`加到列表`arraylib_m`中，从而将这个函数加入数组对象的元表。

```lua
    static const struct luaL_Reg arraylib_m[] = {
        {"__tostring", array2string},
        <其他方法>
    };
```

&emsp;&emsp;

####&emsp;&emsp;数组访问

&emsp;&emsp;另一种面向对象写法是使用常规的数组访问写法。相对于`a:get(i)`，可以简单地写为`a[i]`。对于上面的示例，很容易可以做到这点。由于函数`setarray`和`getarray`所接受的参数次序暗合相关元方法的参数次序，因此在Lua代码中可以快速地将这些元方法定义为：

```lua
    local metaarray = getmetatable(array.new(1))
    metaarray.__index = array.get
    metaarray.__newindex = array.set
    metaarray.__len = array.size
```

必须在第一个数组实现上运行这段代码，而不能应用于那个为面向对象访问而修改的版本。使用这些标准语法很简单：

```lua
    a = array.new(1000)
    a[10] = true			-- setarray
    print(a[10])			-- getarray		--> true
    print(#a)				-- getsize		--> 1000
```

&emsp;&emsp;如果还要更完美，可以在C代码中注册这些方法。为此，需要再次修改初始化函数：

```lua
    static const struct luaL_Reg arraylib_f [] = {
        {"new", newarray},
        {NULL, NULL}
    };

    static const struct luaL_Reg arraylib_m [] = {
        {"__newindex", setarray},
        {"__index", getarray},
        {"__len", getsize},
        {"__tostring", array2string},
        {NULL, NULL}
    };

    int luaopen_array(lua_State *L) {
        luaL_newmetatable(L, "LuaBook.array");
        luaL_register(L NULL, arraylib_m);
        luaL_register(L, "array", arraylib_f);
        return 1;
    }
```

&emsp;&emsp;在这个新版本中，仍只有一个公共函数`new`。所有其他函数都作为特定操作的元方法。

&emsp;&emsp;

####&emsp;&emsp; 轻量级userdata（light userdata）

&emsp;&emsp;到现在为止所使用的`userdata`都称为“完全`userdata(full userdata)`”。Lua还提供另一种“轻量级`userdata(light userdata)`”。

&emsp;&emsp;轻量级`userdata`是一种表示C指针的值（即`void *`）。由于它是一个值，所以不用创建它。要将一个轻量级`userdata`放入栈中，只需调用`lua_pushlightuserdata`即可：

```lua
    void lua_pushlightuserdata(lua_State *L, void *p);
```

&emsp;&emsp;尽管两种`userdata`在名称上差不多，但它们之间还是存在很大不同的。轻量级`userdata`不是缓冲，只是一个指针而已。它也没有元表，就像数字一样，轻量级`userdata`无须受垃圾收集器的管理。

&emsp;&emsp;有时会将轻量级`userdata`当作一种廉价的完全`userdata`来使用。但这种用法并没有太大意义。首先，使用轻量级`userdata`时用户必须自己管理内存，因为轻量级`userdata`不属于垃圾收集的范畴。其次，不要被“完全”二字所迷惑，完全`userdata`的开销并不比轻量级`userdata`大多少。它们只为分配内存增加了一些`malloc`的开销。

&emsp;&emsp;轻量级`userdata`的真正用途是相等性判断。一个完全`userdata`是一个对象，它只与自身相等。而一个轻量级`userdata`则表示了一个C指针的值。因此，它与所有表示同一个指针的轻量级`userdata`相等。可以将轻量级`userdata`用于查找Lua中的C对象。

&emsp;&emsp;以下是一种比较典型的情况，假设正在实现一种Lua与某个窗口系统的绑定。在这种绑定中，用完全`userdata`表示窗口。每个`userdata`可以包含整个窗口的数据结构，也可以只包含一个指向系统所创建窗口的指针。当在一个窗口中发生了一个事件时（例如单击鼠标），系统要调用对应于该窗口的回调函数。而窗口是通过其地址来识别的。为了调用Lua中实现的回调函数，必须先找到表示指定窗口的`userdata`。若要寻找这个`userdata`，可以用一个`table`来保存窗口的信息，它的`key`是表示窗口地址的轻量级`userdata`，而`value`则是表示窗口本身的完全`userdata`。当得到一个窗口地址时，就可以把它作为一个轻量级`userdata`压入栈中，并用这个`userdata`来索引`table`。从而得到那个表示窗口本身的完全`userdata`，并由此调用回调函数。

🔚