##面向对象编程

&emsp;&emsp;Lua中的`table`就是一种对象，这句话可以从3个方面来证实。首先，`table`与对象一样可以拥有状态。其次，`table`也与对象一样拥有一个独立于其值的标识（一个`self`）。例如，两个具有相同值的对象（`table`）是两个不同的对象。最后，`table`与对象一样具有独立于创建者和创建地的生命周期。

&emsp;&emsp;对象有其自己的操作。同样`table`也有这样的操作：

```lua
    Account = {balance = 0}
    function Account.withdraw(v)
        Account.balance = Account.balance - v
    end
```

&emsp;&emsp;上面的代码创建了一个新函数，并将该函数存入`Account`对象的`withdraw`字段中。则可进行如下调用：

```lua
    Account.withdraw(100.00)
```

&emsp;&emsp;这种函数就是所谓的“方法（Method）”。不过，在函数中使用全局名称`Account`是一个不好的编程习惯。因为这个函数只能针对特定对对象工作，并且，这个特定对象还必须存储在特定的全局变量中。如果改变了对象的名称，`withdraw`就再也不能工作了：

```lua
    a = Account;Account = nil
    a.withdraw(100.00)        -- 错误！❌
```

&emsp;&emsp;这种行为违反了前面提到的对象特性，即对象拥有独立的生命周期。

&emsp;&emsp;有一种灵活的方法，即指定一项操作所作用的“接受者”。因此需要一个额外的参数来表示该接受者。这个参数通常称为`self`或`this`：

```lua
    function Account.withdraw(self, v)
        self.balance = self.balance - v
    end
```

&emsp;&emsp;此时当调用该方法时，必须指定其作用的对象：

```lua
    a1 = Account; Account = nil
    ...
    a1.withdraw(a1, 100.00)        -- OK
```

&emsp;&emsp;通过对`self`参数的使用，还可以针对多个对象使用同样的方法：

```lua
    a2 = {balance=0, withdraw=Account.withdraw}
    ...
    a2.withdraw(a2, 260.00)
```

&emsp;&emsp;使用`self`参数是所有面向对象语言的一个核心。大多数面向对象语言都能对程序员隐藏部分`self`参数，从而使得程序员不必显式地声明这个参数。Lua只需使用冒号，则能隐藏该参数。即可将上例重写为：

```lua
    function Account:withdraw(v)
        self.balance = self.balance - v
    end
```

&emsp;&emsp;调用时可写为：

```lua
    a:withdraw(100.00)
```

&emsp;&emsp;冒号的作用是在一个方法定义中添加一个额外的隐藏参数，以及在一个方法调用中添加一个额外的实参。冒号只是一种语法便利，并没有引入任何新的东西。例如，用点语法来定义一个函数，并用冒号语法调用它。反之，只要能正确地处理那个额外参数即可：

```lua
    Account = {balance=0, 
               withdraw=function(self, v)
                            self.balance = self.balance - v
                        end
              }
              
    function Account:deposit(v)
        self.balance = self.balance + v
    end
    
    Account.deposit(Account, 200.00)
    Account:withdraw(100.00)
```

&emsp;&emsp;现在的对象已有一个标识、一个状态和状态之上的操作。不过还缺乏一个类（class）系统、继承和私密性（privacy）。首先解决第一个问题，如何创建多个具有类似行为的对象？更准确地说，如何创建多个account账户对象？

&emsp;&emsp;

####类

&emsp;&emsp;一个类就像是一个创建对象的模具。有些面向对象语言提供了类的概念，在这些语言中每个对象都是某个特定类的实例。Lua则没有类的概念，每个对象只能自定义行为和形态。不过，要在Lua中模拟类也并不困难，可以参照一些基于原型的语言，例如`Self`和`NewtonScript`。在这些语言中，对象是没有“类型”的（objects have no classes）。而是每个对象都有一个原型（prototype）。原型也是一种常规的对象，当其他对象（类的实例）遇到一个未知操作时
，原型会先查找它。在这种语言中要表示一个类，只需创建一个专用作其他对象（类的实例）的原型。类和原型都是一种组织对象间共享行为的方式。

&emsp;&emsp;在Lua中实现原型很简单，使用后续章节所述的继承即可。更准确地说，如果有两个对象a和b，要让b作为a的一个原型，只需输入如下语句：

```lua
    setmetatable(a, {__index = b})
```

&emsp;&emsp;在此之后，a就会在b中查找所有它没有的操作。若将b称为是对象a的类，只不过是术语上的一个变化。

&emsp;&emsp;回到先前银行账号的示例。为了创建更多与`Account`行为类似的账号，可以让这些新对象从`Account`行为中继承这些操作。具体做法就是使用`__index`元方法。可以应用一项小优化，则无须创建一个额外的`table`作为账户对象的元表。而是使用`Account table`自身作为元表：

```lua
    function Account:new(o)
        o = o or {}        -- 如果用户没有提供table，则创建一个
        setmetatable(o, self)
        self.__index = self
        return o
    end
```

&emsp;&emsp;当调用`Account:new`时，`self`就等于`Account`。因此可以直接使用`Account`来代替`self`。不过，当引入类继承时，使用`self`则会更为准确。在这段代码之后，创建一个新账户或调用一个方法时会发生什么呢？

```lua
    a = Account:new{balance = 0}
    a:deposit(100.00)
```

&emsp;&emsp;当创建新账户时，a会将`Account`(`Account:new`调用中的`self`)作为其元表。而当调用`a:deposit(100.00)`时，就是调用了`a.deposit(a, 100.00)`。因此冒号只不过是一个“语法糖”。当Lua无法在`table a`中找到条目“`deposit`”时，它会进一步搜索元表的`__index`条目。最终的调用情况为：

```lua
    getmetatable(a).__index.deposit(a, 100.00)
```

&emsp;&emsp;`a`的元表是`Account`，`Account.__index`也是`Account`。因此，上面这个表达式可以简化为：

```lua
    Account.deposit(a, 100.00)
```

&emsp;&emsp;结果为Lua调用了原来的`deposit`函数，但传入`a`作为`self`参数。因此新账户`a`从`Account`继承了`deposit`函数。同样，它还能从`Account`继承所有的字段。

&emsp;&emsp;继承不仅可以作用于方法，还可以作用于所有其他在新账户中没有的字段。因此，一个类不仅可以提供方法，还可以为实例中的字段提供默认值。回忆一下，在第一个`Account`定义中，有一个`balance`字段为0。如果在创建新账户时没有提供`balance`的初值，那么它就会继承这个默认值：

```lua
    b = Account:new()
    print(b.balance)            --> 0
```

&emsp;&emsp;在`b`上调用`deposit`方法时，`self`就是`b`，就相当于执行了：

```lua
    b.balance = b.balance + v
```

&emsp;&emsp;在第一次调用`deposit`时，对表达式`b.balance`的求值结果为0，然后一个初值被赋予了`b.balance`。后续对`b.balance`的访问就不会再涉及到`__index`元方法了，因为此时`b`已有自己的`balance`字段。

&emsp;&emsp;

####继承

&emsp;&emsp;由于类也是对象，它们也可以从其他类获得方法。这种行为就是一种继承，可以很容易地在Lua中。

&emsp;&emsp;假设有一个基类`Account`：

```lua
    Account = {}

    function Account:new(o)
        o = o or {}
        setmetatable(o, self)
        self.__index = self
        return o
    end

    function Account:deposit(v)
        self.balance = self.balance + v
    end

    function Account:withdraw(v)
        if v > self.balance then error "insufficient funds" end
        self.balance = self.balance - v
    end
```

&emsp;&emsp;若想从这个类派生出一个子类`SpecialAccount`，以使客户能够透支。则先需要创建一个空的类，从基类继承所有的操作：

```lua
    SpecialAccount = Account:new()
```

&emsp;&emsp;直到现在，`SpecialAccount`还只是`Account`的一个实例。如下所示：

```lua
    s = SpecialAccount:new{limit=1000.00}
```

&emsp;&emsp;`SpecialAccount`从`Account`继承了`new`，就像继承其他方法一样。不过这次`new`在执行时，它的`self`参数表示为`SpecialAccount`。因此，`s`的元表为`SpecialAccount`，`SpecialAccount`中字段`__index`的值也是`SpecialAccount`。`s`继承自`SpecialAccount`，而`SpecialAccount`又继承自`Account`。当执行：

```lua
    s:deposit(100.00)
```

&emsp;&emsp;Lua在`s`中找不到`deposit`字段时，就会查找`SpecialAccount`。如果仍找不到`deposit`字段，就查找`Account`。最终会在那里找到`deposit`的原始实现。

&emsp;&emsp;`SpecialAccount`之所以特殊是因为可以重定义那些从基类继承的方法。编写一个方法的新实现只需：

```lua
    function SpecialAccount:withdraw(v)
        if v - self.balance >= self:getLimit() then
            error "insufficient funds"
        end
        self.balane = self.balance - v
    end

    function SpecialAccount:getLimit()
        return self.limit or 0
    end
```

&emsp;&emsp;现在，当调用`s:withdraw(200.00)`时，Lua就不会在`Account`中查找了。因为Lua会在`SpecialAccount`中先找到`withdraw`方法。由于`s.limit`为1000.00，程序会执行取款，并使`s`变成一个负的余额。

&emsp;&emsp;Lua中的对象有一个特殊现象，就是无须为指定一种新行为而创建一个新类。如果只有一个对象需要某种特殊的行为，那么可以直接在该对象中实现这个行为。例如，账户`s`表示一个特殊的客户，这个客户的透支额度总是其余额的10%。那么可以只修改这个对象：

```lua
    function s:getLimit()
        return self.balance * 0.10
    end
```

&emsp;&emsp;在这段代码后，调用`s:withdraw(200.00)`还是会执行`SpecialAccount`的`withdraw`。但`withdraw`所调用的`self:getLimit`则是上面这个定义。

&emsp;&emsp;

####多重继承

&emsp;&emsp;由于Lua中的对象不是原生的（Primitive），因此在Lua中进行面向对象编程时有几种方法。上面介绍了一种使用`__index`元方法的做法，这是集简易、性能和灵活性于一体的做法。另外还有一些其他的做法，可能更适用于某些特殊的情况。在此将介绍另一种做法，可以在Lua中实现多重继承。

&emsp;&emsp;这种做法的关键在于同一个函数作为`__index`元字段。例如，若在一个`table`的元表中，`__index`字段为一个函数。那么只要Lua在原来的`table`中找不到一个`key`，就会调用这个函数。基于这点，就可以让`__index`函数在其他地方查找缺失的`key`。

&emsp;&emsp;多重继承意味着一个类可以具有多个基类。因此无法使用一个类中的方法来创建子类，而是需要定义一个特殊的函数来创建。下面的`createClass`就是这样的函数，它会创建一个`table`表示新类，其中一个参数表示新类的所有基类。创建时它会设置元表中的`__index`元方法，而多重继承正是在这个`__index`元方法中完成的。虽然是多重继承，但每个对象实例仍属于单个类，并且都在这个类中查找所有的方法。因此，类和基类之间的关系不同于类和实例之间的关系。尤其是一个类不能同时作为其实例和子类的元表。在以下代码中，将类作为其实例的元表，并创建了另一个`table`作为类的元表。

```lua
    -- 在table 'plist'中查找'k'
    local function search(k, plist)
        for i=1, #plist do
            local v = plist[i][k]		-- 尝试第i个基类
            if v then return v end
        end
    end

    function createClass(...)
        local c = {}					-- 新类
        local parents = {...}

        -- 类在其父类列表中的搜索方法
        setmetatable(c, {__index = function(t, k)
            return search(k, parents)
        end})

        -- 将'c'作为其实例的元表
        c.__index = c

        -- 为这个新类定义一个新的构造函数（construction）
        function c:new(o)
            o = o or {}
            setmetatable(o, c)
            return o
        end

        return c 				-- 返回新类
    end
```

&emsp;&emsp;接下来是一个使用`createClass`的例子。假设有两个类，一个是前面提到的`Account`类；另一个是`Named`类，它有两个方法`setname`和`getname`：

```lua
    Named = {}
    function Named:getname()
        return self.name
    end

    function Named:setname(n)
        self.name = n
    end
```

&emsp;&emsp;要创建一个新类`NamedAccount`，同时从`Account`和`Named`派生，那么只需调用`createClass`：

```lua
    NamedAccount = createClass(Account, Named)
```

&emsp;&emsp;如下要创建并使用实例：

```lua
    account = NamedAccount:new{name = "Paul"}
    print(account:getname())		--> Paul
```

&emsp;&emsp;现在，来研究最后代码是如何工作的。首先，Lua在`account`中无法找到字段“`getname`”。因此，就查找`account`元表中的`__index`字段，该字段为`NamedAccount`。由于在`NamedAccount`也无法提供字段“`getname`”。因此，Lua查找`NamedAccount`元表中的`__index`字段。由于这个字段也是一个函数，Lua就调用了它。该函数则先在`Account`中查找“`getname`”。未找到后，继而查找`Named`。最终在`Named`中找到了一个非`nil`的值，即为搜索的最终结果。

&emsp;&emsp;由于这项搜索具有一定的复杂性，则多重继承的性能不如单一继承。有一种改进性能的简单做法是将继承的方法复制到子类中。通过这种技术，类的`__index`元方法如下所示：

```lua
    setmetatable(c, {__index = function(t, k)
        local v = search(k, parents)
        t[k] = v 				-- 保存下来，以备下次访问
        return v
    end})
```

&emsp;&emsp;用了这种技术后，访问继承的方法就能像访问局部方法一样快了。但缺点是当系统运行后就较难修改方法的定义，因为这些修改不会沿着继承体系向下传播。

&emsp;&emsp;

####私密性

&emsp;&emsp;许多人认为私密性应成为面向对象语言不可或缺的一部分，每个对象的状态都应该是由它自己掌握。在一些面向对象语言中，例如`C++`和`Java`，能控制对象中的字段或方法是否在对象之外可见。而对于其他语言，例如`Smalltalk`，规定所有的变量都是私有的，但所有的方法却都是公有的。第一个面向对象语言`Simula`则不提供任何形式的私密性保护。

&emsp;&emsp;Lua在设计对象时，没有提供私密性机制，这具体章节已看到了。一方面这是因为使用了普通的结构（`table`）来表示对象，另一方面也反映了Lua某些基本的设计决定。Lua并不打算构建需要许多程序员长期投入的大型程序。相反，Lua定位于开发中小型程序，这些程序通常是一个更大系统的一部分。而参与编程的程序员一般只有一名或几名，甚至还可以是非程序员。因此，Lua尽量避免过多冗余和人为限制。如果不想访问一个对象中的内容，则无须进行操作。

&emsp;&emsp;Lua的另外一项设计目标是灵活性。Lua提供给程序员各种元机制，以使他们能模拟许多不同的机制。虽然在Lua对象的基础设计中没有提供私密性机制。但可以用其他方法来实现对象，从而获得对象的访问限制。这种实现不常用，只做基本的了解，它既探索了Lua中的某些知识又可以成为其他问题的解决方案。

&emsp;&emsp;这种做法的基本思想是，通过两个`table`来表示一个对象。一个`table`用来保存对象的状态；另一个用于对象的操作，或称为“接口”。对象本身是通过第二个`table`来访问的，即通过其接口的方法来访问。为了避免未授权的访问，表示状态的`table`不保存在其他`table`中，而只是保存在方法的`closure`中。例如，若使用这种设计来表示一个银行账户，可以调用下面这个工厂函数来创建新的账户对象：

```lua
    function newAccount(initialBalance)
        local self = {balance = initialBalance}

        local withdraw = function(v)
            self.balance = self.balance - v
        end

        local deposit = function(v)
            self.balance = self.balance + v
        end

        local getBalance = function() return self.balance end

        return {
            withdraw = withdraw,
            deposit = deposit,
            getBalance = getBalance
        }
    end
```

&emsp;&emsp;这个函数先创建了一个`table`，用于保存对象的内部状态，并将其存储在局部变量`self`中。然后再创建对象的方法。最后，函数创建并返回一个供外部使用的对象，其中将方法名与真正的方法实现匹配起来。区别关键在于，这些方法不需要额外的`self`参数，因为它们可以直接访问`self`变量。由于没有了额外的参数，也就无须使用冒号语法来操作对象。则可以像普通函数那样来调用这些方法：

```lua
    acc1 = newAccount(100.00)
    acc1.withdraw(40.00)
    print(acc1.getBalance())		--> 60
```

&emsp;&emsp;这种设计给予存储在`self table`中所有东西完全的私密性。当`newAccount`返回后，就无法直接访问这个`table`了。只能通过`newAccount`中创建的函数来访问它。上例只将一个成员变量放到了私有`table`中，其实可以将一个对象中所有的私有部分都存入这个`table`。另外还可以定义私有的方法，它们类似于公有方法，但不放入接口中。例如，该账户可以给那些余额大于某个值的用户额外10%的信用额度，但是又不想让用户访问到这些计算细节。那么可以将这个功能按以下方法实现：

```lua
    function newAccount(initialBalance)
        local self = {
            balance = initialBalance,
            LIM = 10000.00
        }

        local extra = function()
            if self.balance > self.LIM then
                return self.balance * 0.10
            else
                return 0
            end
        end

        local getBalance = function()
            return self.balance + extra()
        end

        <如前>
```

&emsp;&emsp;与前一个示例一样，任何用户都无法直接访问`extra`函数。

&emsp;&emsp;


####单一方法（single-method）做法

&emsp;&emsp;上述面向对象编程的做法有一种特殊情况，就是当一个对象只有一个方法时，可以不用创建接口`table`，但要将这个单独的方法作为对象表示来返回。如果无法理解，请参阅前面章节。前面章节介绍了如何构造一个迭代器函数，那个函数将状态保存为`closure`。一个具有状态的迭代器是一个单一方法对象。

&emsp;&emsp;单一方法对象还有一种情况，若这个方法是一个调度（`dispatch`）方法，它根据某个参数来完成不同的操作。则可以这样来实现一个对象：

```lua
    function newObject(value)
        return function(action, v)
            if action == "get" then return value
            elseif action == "set" then value = v
            else error "invalid action"
            end
        end
    end
```

&emsp;&emsp;如下所示：

```lua
    d = newObject(0)
    print(d("get"))				--> 0
    d("set", 10)
    print(d("get"))				--> 10
```

&emsp;&emsp;这种非传统的对象实现方式是很高效的。语句`d("set", 10)`虽然有些奇特，但只比传统的`d:set(10)`多出两个字符。每个对象都用一个`closure`，这比都用一个`table`更高效。虽然无法实现继承，却拥有了完全的私密性控制。访问一个对象状态只有一个方式，那就是通过它的单一方法。

&emsp;&emsp;Tcl/Tk对它的窗口部件使用了类似的做法。在Tk中，一个窗口部件的名称就是一个函数名，通过这个函数就可以完成所有针对该部件的操作。


🔚