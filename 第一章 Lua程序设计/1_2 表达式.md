##表达式

```
    • 算术操作符
    • 关系操作符
    • 逻辑操作符
    • 字符串连接
    • 优先级
    • table构造式
```

&emsp;&emsp;

&emsp;&emsp;表达式用于表示值。Lua的表达式中可以包含数字常量、字面字符串、变量、一元和二元操作符及函数调用。另外有别于传统的是，表达式中还可以包括函数定义和`table`构造式。

&emsp;&emsp;

###&emsp;&emsp;• 算术操作符

```
    +（加法）、-（减法）、*（乘法）、／（除法）、^（指数）、%（取模）、-（负号）
```
    
```lua
    例如：
        x^0.5 将计算x的平方根
        x^(-1/3) 将计算x立方根的倒数
```

```lua
    取模操作符是根据以下规则定义的：
        a%b == a - floor(a/b)*b
        
    例如：
        x%1 的结果就是x的小数部分
        x - x%1 的结果就是其整数部分
        x - x%0.01 则是x精确到小数点后两位的结果
        
        x = math.pi
        print(x - x%0.01)   --> 3.14
```

&emsp;&emsp;

###&emsp;&emsp;• 关系操作符

```lua
    <、>、<=、>=、==、~=
    所有这些操作符的运算结果都是true或false。

    操作符==用于相等性测试，操作符～=用于不等性测试。
    这两个操作符可以应用于任意两个值。
    
    如果两个值具有不同的类型，Lua就认为它们是不相等的。
    否则，Lua会根据它们的类型来比较。
    
    nil只与其自身相等。
    
    对于table、userdata和函数，Lua是作引用比较的。
    也就是说，只有当它们引用同一个对象时，才认为它们相等。
```   

```lua
    a = {}; a.x = 1; a.y = 0
    b = {}; b.x = 1; b.y = 0
    c = a
    其结果是a==c，但a~=b。
```

>Lua会在遇到字符串和数字的大小比较时引发一个错误，例如`2<"15"`就会导致这种错误。

&emsp;&emsp;

###&emsp;&emsp;• 逻辑操作符

```lua
    and、or、not
    与条件控制语句一样，所有的逻辑操作符将false和nil视为假，而将其他的任何东西视为真。
    
    对于操作符and来说，如果它的第一个操作数为假，就返回第一个操作数；不然返回第二个操作数。
    
    对于操作符or来说，如果它的第一个操作数为真，就返回第一个操作数；不然返回第二个操作数。
```

```lua
    print(4 and 5)        --> 5
    print(nil and 13)     --> nil
    print(false and 13)   --> false
    print(4 or 5)         --> 4
    print(false or 5)     --> 5
```

```lua
    and和or都是用“短路求值”，也就是说，它们只会在需要时才去评估第二个操作数。
    短路求值可以确保像（type(v)=="table" and v.tag == "h1"）这样的表达式不会导致运行时错误。

    有一种常用的Lua习惯写法“x = x or v”，它等价于：
    if not x then x = v end
    
    另外，还有一种习惯写法是“（a and b）or c”，这类似于C语言中的表达式a?b:c，但前提是b不为假。
    例如，为了选出数字x和y中的较大者，可以使用以下语句：
    max = (x > y) and x or y
```

```lua
    操作符not永远只返回true或false：
        print(not nil)        --> true
        print(not false)      --> true
        print(not 0)          --> false
        print(not not nil)    --> false
```

&emsp;&emsp;

###&emsp;&emsp;• 字符串连接

&emsp;&emsp;要在Lua中连接两个字符串，可以使用操作符“`..`”（两个点）。如果其任意一个操作数是数字的话，Lua会将这个数字转换成一个字符串：

```lua
    print("Hello" .. "World")    --> Hello World
    print(0 .. 1)                --> 01
```

&emsp;&emsp;请记住，Lua中的字符串是不可变的值。连接操作符只会创建一个新字符串，而不会对其原操作数进行任何修改：

```lua
    a = "Hello"
    print(a .. "World")    --> Hello World
    print(a)               --> Hello
```

&emsp;&emsp;

###&emsp;&emsp;• 优先级

    |优先级|
    |:--|
    |`^`|
    |`not`、`#`、`-`(一元)|
    |`*`、`／`、`%`|
    |`+`、`-`|
    |`..`|
    | `<`、`>`、`<=`、`>=`、 `~=`、 `==`|
    |`and`|
    |`or`|

&emsp;&emsp;
&emsp;&emsp;在二元操作符中，除了指数操作符“`^`”和连接操作符“`..`”是“右结合”的，所有其他操作符都是“左结合”的。因此，下例中左边的表达式等价于右边的表达式：

```lua
    a + i < b/2 + 1        <-->        (a+i) < ((b/2) + 1)
    5 + x^2 * 8            <-->        5 + ((x^2) * 8)
    a < y and y <= z       <-->        (a < y) and (y <= z)
    -x^2                   <-->        -(x^2)
    x^y^z                  <-->        x^(y^z)
```

&emsp;&emsp;

###&emsp;&emsp;• table构造式

&emsp;&emsp;构造式是用于创建和初始化`table`的表达式。这是Lua特有的一种表达式，并且也是Lua中最有用、最通用的机制之一。

&emsp;&emsp;最简单的构造式就是一个空构造式`{}`，用于创建一个空`table`。构造式还可以用于初始化数组。元素之间可以用逗号也可以用分号，最后一个元素后面写逗号也是合法的。

```lua
    days = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}
```

&emsp;&emsp;会将`days[1]`初始化为字符串“`Sunday`”、`days[2]`初始化为“`Monday`”，以此类推。

&emsp;&emsp;

&emsp;&emsp;Lua还提供了一种特殊的语法用于初始化记录风格的`table`：

```lua
    a = {x=10, y=20}
```

&emsp;&emsp;以上这行代码等价于这些语句：

```lua
    a = {}; a.x = 10; a.y = 20
```

&emsp;&emsp;无论使用哪种方式来创建`table`，都可以在`table`创建之后添加或删除其中的某些字段：

```lua
    w = {x=0, y=0, label="console"}
    x = {math.sin(0), math.sin(1), math.sin(2)}
    w[1] = "another field"        -- 添加key 1到table w
    x.f = w                       -- 添加key "f"到table x
    print(w["x"])                 --> 0
    print(w[1])                   --> another field
    print(x.f[1])                 --> another field
    w.x = nil                     -- 删除字段"x"
```

&emsp;&emsp;

&emsp;&emsp;每当Lua评估一个构造式时，都会先创建一个新`table`，然后初始化它。

```lua
    -- 用table实现链表：
    
    list = nil
    for line in io.lines() do            -- 从标准输入读取每行的内容
        list = {next=list, value=line}   -- 按相反的次序存储到一个链表中
    end
```

```lua
    -- 遍历该链表
    
    local l = list
    while l do
        print(l.value)
        l = l.next
    end
```

&emsp;&emsp;

&emsp;&emsp;将记录风格的初始化与列表风格的初始化混合在一个构造式中使用：

```lua
    polyline = {
        color = "blue", thickness = 2, npoints = 4,
        {x = 0, y = 0},
        {x = -1, y = 0},
        {x = -10, y = 1},
        {x = 0, y = 1}
    }
```

&emsp;&emsp;上例演示了如何通过嵌套的构造式来表示复杂的数据结构。每个`polyline[i]`元素都是一个`table`，表示一条记录：


```lua
    print(polyline[2].x)		--> -1
    print(polyline[4].y)		--> 1
```

&emsp;&emsp;这两种风格的构造式各有其限制。例如，不能使用负数的索引，也不能用运算符作为记录的字段名。

&emsp;&emsp;

&emsp;&emsp;为了满足这些要求，Lua还提供了一种更通用的格式。这种格式允许在方括号之间，显式地用一个表达式来初始化索引值：

```lua
    opnames = { ["+"] = "add", ["-"] = "sub", ["*"] = "mul", ["/"] = "div" }

    i = 20; s = "-"
    a = {[i+0] = s, [i+1]=s..s, [i+2]=s..s..s}

    print(opnames[s])          --> sub
    print(a[22])               --> ---		
```

```lua
    构造式{x=0, y=0}等价于{["x"]=0, ["y"]=0}
    构造式{"r", "g", "b"}等价于{[1]="r", [2]="g", [3]="b"}
```

&emsp;&emsp;

&emsp;&emsp;对于某些情况如果真的需要以0作为一个数组的起始索引的话，通过这种语法也可以轻松做到：

```lua
days = {[0]="Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}
```

&emsp;&emsp;现在第一个值“`Sunday`”的索引就为0了。这个索引0并不影响其他元素，“`Monday`”照常索引为1，因为它是构造式中列表风格中的第一个值，后续其他值的索引依次递增。**但无论是否使用这种初始化语法，都不推荐在Lua中以0作为数组的起始索引。大多数内建函数都假设数组起始于索引1，若遇到以索引0开始的数组，它们就无法进行正确地处理了**。

&emsp;&emsp;

🔚