##协同程序

&emsp;&emsp;协同程序与线程（`thread`）差不多，也就是一条执行序列，拥有自己独立的栈、局部变量和指令指针，同时又与其他协同程序共享全局变量和其他大部分东西。从概念上讲线程与协同程序的主要区别在于，一个具有多个线程的程序可以同时运行几个线程，而协同程序却需要彼此协作地运行。就是说，**一个具有多个协同程序的程序在任意时刻只能运行一个协同程序，并且正在运行的协同程序只会在其显式地要求挂起（suspend）时，它的执行才会暂停。**

&emsp;&emsp;

```
    ● 协同程序基础
    ● 管道与过滤器
    ● 以协同程序实现迭代器
    ● 非抢先式的多线程
```

&emsp;&emsp;

###&emsp;&emsp;● 协同程序基础

&emsp;&emsp;Lua将所有关于协同程序的函数放置在一个名为“`coroutine`”的`table`中。

&emsp;&emsp;函数`create`用于创建新的协同程序，它只有一个参数，就是一个函数。该函数的代码就是协同程序所需执行的内容。`create`会返回一个`thread`类型的值，用以表示新的协同程序。通常`create`的参数是一个匿名函数。

```lua
    co = coroutine.create( function () print( "hi" ) end )
    print( co )    -- thread: 0x7fe2f1506218
```

&emsp;&emsp;一个协同程序可以处于4种不同的状态：挂起（`suspended`）、运行（`running`）、死亡（`dead`）和正常（`normal`）。当创建一个协同程序时，它处于挂起状态。也就是说，协同程序不会在创建它时自动执行其内容。可以通过函数`status`来检查协同程序的状态：

```lua
    print( coroutine.status( co ) )    -- suspended
```

&emsp;&emsp;函数`coroutine.resume`用于启动或再次启动一个协同程序的执行，并将其状态由挂起改为运行：

```lua
    coroutine.resume( co )             -- hi
```

&emsp;&emsp;在本例中，协同程序的内容只是简单地打印了“`hi`”后便终止了，然后它就处于死亡状态，也就再也无法返回了：

```lua
    print( coroutine.status( co ) )    -- dead
```

&emsp;&emsp;到目前为止，协同程序看上去还只是像一种复杂的函数调用方法。其实协同程序的真正强大之处在于函数`yield`的使用上，该函数可以让一个运行中的协同程序挂起，而之后可以再恢复它的运行。

```lua
    co = coroutine.create( function ()
        for i = 1, 10 do
            print( "co", i )
            coroutine.yield()
        end
    end )
```

&emsp;&emsp;现在当唤醒这个协同程序时，它就会开始执行，直到第一个`yield`：

```lua
    coroutine.resume( co )    -- co 1
```

&emsp;&emsp;如果此时检查其状态，会发现协同程序处于挂起状态，因此可以再次恢复其运行：

```lua
    print( coroutine.status( co ) )    -- suspended
```

&emsp;&emsp;从协同程序的角度看，所有在它挂起时发生的活动都发生在`yield`调用中。当恢复协同程序的执行时，对于`yield`的调用才最终返回。然后协同程序继续它的执行，直到下一个`yield`调用或执行结束：

```lua
    coroutine.resume( co )    -- co 2
    coroutine.resume( co )    -- co 3
    ...
    coroutine.resume( co )    -- co 10
    coroutine.resume( co )    -- 什么都不打印
```

&emsp;&emsp;在最后一次调用`resume`时，协同程序的内容已经执行完毕，并已经返回。因此，这时协同程序处于死亡状态。如果试图再次恢复它的执行，`resume`将返回`false`及一条错误消息：

```lua
    print(coroutine.resume( co ))    -- false cannot resume dead coroutine
```

>&emsp;&emsp;请注意，`resume`是在保护模式中运行的。因此，如果在一个协同程序的执行中发生任何错误，Lua是不会显示错误消息的，而是将执行权返回给`resume`调用。

&emsp;&emsp;当一个协同程序A唤醒一个协同程序B时，协同程序A就处于一个特殊状态，既不是挂起状态（无法继续A的执行），也不是运行状态（是B在运行）。所以将这时的状态称为“正常”状态。

&emsp;&emsp;**Lua的协同程序还具有一项有用的机制，就是可以通过一对`resume-yield`来交换数据。**在第一次调用`resume`时，并没有对应的`yield`在等待它，因此所有传递给`resume`的额外参数都将视为协同程序主函数的参数：

```lua
    co = coroutine.create( function ( a, b, c )
        print( "co", a, b, c )
    end )
    coroutine.resume( co, 1, 2, 3 )    -- co 1 2 3
```

&emsp;&emsp;在`resume`调用返回的内容中，第一个值为`true`则表示没有错误，而后面所有的值都是对应`yield`传入的参数：

```lua
    co = coroutine.create( function ( a, b )
        coroutine.yield( a + b, a - b )
    end )

    print( coroutine.resume( co, 20, 10 ) )    -- true 30 10
```

&emsp;&emsp;与此对应的是，`yield`返回的额外值就是对应`resume`传入的参数：

```lua
    co = coroutine.create( function ()
        print( "co", coroutine.yield() )
    end )
    
    print(coroutine.resume( co, "a" ))      -- true
    print(coroutine.resume( co, 4, 5, 6 ))  -- co 4 5 6  -- true
    print(coroutine.resume( co, 4, 5))      -- false cannot resume dead coroutine
```

&emsp;&emsp;最后，当一个协同程序结束时，它的主函数所返回的值都将作为对应`resume`的返回值：

```lua
    co = coroutine.create( function ()
        return 6, 7
    end )
    print( coroutine.resume( co ) )		-- true 6 7
```

&emsp;&emsp;

###&emsp;&emsp;● 管道与过滤器

```lua
----------------------------------------- 管道与过滤器

    function receive( prod )
    	local status, value = coroutine.resume( prod )
    	return value
    end
    
    function  send( x )
    	coroutine.yield( x )
    end
    
    function producer( )
    	return coroutine.create( function ( )
    		while true do
    			local x = io.read()		-- 产生新值
    			send(x)
    		end
    	end )
    end
    
    function filter( prod )
    	return coroutine.create( function ( )
    		for line = 1, math.huge do
    			local x = recevie(prod)		-- 获取新值
    			x = string.format( "%5d %s", line, x )
    			send(x)						-- 将新值发送给消费者
    		end
    	end )
    end
    
    function consumer( prod )
    	while true do
    		local x = receive(prod)		-- 获取新值
    		io.write( x, "\n" )			-- 消费新值
    	end
    end
    
    -- 运行代码
    consumer(filter(producer()))
```




&emsp;&emsp;

###&emsp;&emsp;● 以协同程序实现迭代器

&emsp;&emsp;将循环迭代器视为“生产者-消费者”模式的一种特例，一个迭代器会产出一些内容，而循环体会消费这些内容。因此，这样看来协同程序似乎也适用于实现迭代器。的确，协同程序为实现这类任务提供了一项有用的工具。那就是先前提到的，协同程序可以一改传统的调用者与被调用者之间的关系。有了这个特性，在编写迭代器时，就无须顾及如何在每次成功的迭代调用之间保存状态信息了。


&emsp;&emsp;为了说明这类应用，下面来写一个迭代器，使其可以遍历某个数组的所有排列组合形式。若直接编写这种迭代器可能不太容易，但若编写一个递归函数来产生所有的排列组合则不会很困难。想法很简单，只要将每个数组元素都依次放在最后一个位置，然后递归地生成其余元素的排列。代码如下：

```lua
    function permgen(a, n)
        n = n or #a 		-- 默认n为a的大小
        if n <= 1 then
            printResult(a)
        else
            for i = 1, n do
                -- 将第i个元素放到数组末尾
                a[n], a[i] = a[i], a[n]
                -- 生成其余元素的排列
                permgen(a, n - 1)
                -- 恢复第i个元素
                a[n], a[i] = a[i], a[n]
            end
        end
    end
```

&emsp;&emsp;然后，还需要定义其中调用到的打印函数`printResult`，并以适当的参数来调用`permgen`：

```lua
    function printResult(a)
        for i = 1, #a do
            io.write(a[i], " ")
        end
        io.write("\n")
    end

    permgen({1, 2, 3, 4})

    --> 2 3 4 1
    --> 3 2 4 1
    --> 3 4 2 1
    ...
    --> 2 1 3 4
    --> 1 2 3 4
```

&emsp;&emsp;当生成函数完成后，将其转换为一个迭代器就非常容易了。首先，将`printResult`改为`yield`：

```lua
    function permgen(a, n)
        n = n or #a
        if n <= 1 then
            coroutine.yield(a)
        else
            for i = 1, n do
                -- 将第i个元素放到数组末尾
                a[n], a[i] = a[i], a[n]
                -- 生成其余元素的排列
                permgen(a, n - 1)
                -- 恢复第i个元素
                a[n], a[i] = a[i], a[n]
            end
        end
    end
```


&emsp;&emsp;然后，定义一个工厂函数，用于将生成函数放到一个协同程序中运行，并创建迭代器函数。迭代器只是简单地唤醒协同程序，让其产生下一种排列：

```lua
    function permutations(a)
        local co = coroutine.create(function() permgen(a) end)
        return function()  -- 迭代器
            local code, res = coroutine.resume(co)
            return res
        end
    end
```

&emsp;&emsp;有了上面的函数，在`for`语句中遍历一个数组的所有排列就非常简单了：

```lua
    for p in permutations{"a", "b", "c"} do
        printResult(p)
    end

    --> b c a
    --> c a b
    --> a c b
    --> b a c
    --> a b c
```

&emsp;&emsp;`permutations`函数使用了一种在Lua中比较常见的模式，就是将一条唤醒协同程序的调用包装在一个函数中。由于这种模式比较常见，所以Lua专门提供了一个函数`coroutine.wrap`来完成这个功能。类似于`create`，`wrap`创建了一个新的协同程序。但不同的是，`wrap`并不是返回协同程序本身，而是返回一个函数。每当调用这个函数，即可唤醒一次协同程序。但这个函数与`resume`的不同之处在于，它不会返回错误代码。当遇到错误时，它会引发错误。若使用`wrap`，可以这么写`permutations`：

```lua
    function permutations(a)
        return coroutine.wrap(function() permgen(a) end)
    end
```

&emsp;&emsp;通常，`coroutine.wrap`比`coroutine.create`更易于使用。它提供了一个对于协同程序编程实际所需的功能，即一个可以唤醒协同程序的函数。但也缺乏灵活性。无法检查`wrap`所创建的协同程序的状态，此外，也无法检测出运行时的错误。


&emsp;&emsp;

###&emsp;&emsp;● 非抢先式的多线程

&emsp;&emsp;协同程序提供了一种协作式的多线程。每个程序都等于是一个线程。一对`yield-resume`可以将执行权在不同线程之间切换。然后，协同程序与常规的多线程的不同之处在于，协同程序是非抢先式的。就是说，当一个协同程序运行时，是无法从外部停止它的。只有当协同程序显式地要求挂起时（调用`yield`），它才会停止。对于有些应用而言，这没有问题，而对于另外一些应用则可能无法接受这种情况。当不存在抢先时，编程会简单许多。无须为同步的`bug`而抓狂，在程序中所有线程间的同步都是显式的，只需确保一个协同程序在它的临界区域之外调用`yield`即可。

&emsp;&emsp;对于非抢先式的多线程来说，只要有一个线程调用了一个阻塞的（`blocking`）操作，整个程序在该操作完成前，都会停止下来。对于大多数应用程序来说，这种行为是无法接受的。这也导致了许多程序员放弃协同程序，转而使用传统的多线程。接下来会用一个有趣的方法来解决这个问题。


&emsp;&emsp;先假设一个典型的多线程使用情况：希望通过`HTTP`下载几个远程的文件。当然，若要下载几个远程文件，就必须先知道如何下载一个远程文件。在本例中，将使用`Diego Nehab`开发的`LuaSocket`。为了下载一个文件，必须先打开一个到该站点的连接，然后发送下载文件的请求，并接收文件（数据块），最后关闭连接。在Lua中可以按以下步骤来完成这项任务。首先，加载`LuaSocket`库。

```lua
    require "socket"
```


&emsp;&emsp;然后，定义主机和下载的文件。本例，将从`World Wide Consortium`（环球网协会）下载《`HTML 3.2参考规范》`。

```lua
    host = "www.w3.org"
    file = "/TR/REC-html32.html"
```

&emsp;&emsp;接下来，打开一个`TCP`连接，连接到该站点的`80`端口。

```lua
    c = assert(socket.connect(host, 80))
```

&emsp;&emsp;这步操作将返回一个连接对象，可以用它来发送文件请求。

```lua
    c:send("GET" .. file .. "HTTP/1.0\r\n\r\n")
```

&emsp;&emsp;下一步，按1K的字节块来接收文件，并将每块写到标准输出：

```lua
    while true do
        local s, status, partial = c:receive(2^10)
        io.write(s or partial)
        if status == "closed" then break end
    end
```


&emsp;&emsp;在正常情况下`receive`函数会返回一个字符串。若发生错误，则会返回`nil`，并且附加错误代码（`status`）及出错前读取到的内容（`partial`）。当主机关闭连接时，就将其余接收到的内容打印出来，然后退出接收循环。

&emsp;&emsp;下载完文件后，关闭连接。

```lua
    c:close()
```

&emsp;&emsp;现在已经掌握了如何下载一个文件，那么再回到下载几个文件的问题上。最繁琐的做法是逐个地下载文件。因为，这种顺序的做法太慢了，它只能在下载完一个文件后才开始读取该文件。当接收一个远程文件时，程序将大部分的时间花费在等待数据接收上。更明确地说，是将时间用在了对`receive`阻塞调用上。因此，如果一个程序可以同时下载所有文件的话，那么它的运行速度就可以快很多了。当一个连接没有可用数据时，程序便可以从其他连接处读取数据。很明显协同程序提供了一种简便的方式来构建这种并发下载的结构。可以为每个下载任务创建一个新的线程，只要一个线程无可用数据，它就可以将控制权转让给一个简单的调度程序，而这个调度程序则会去调用其他的下载线程。

&emsp;&emsp;在以协同程序来重写程序前，先将前面的下载代码重新写为一个函数。代码如下：

```lua
    function download(host, file)
        local c = assert(socket.connect(host, 80))
        local count = 0 		-- 记录接收到的字节数
        c:send("GET " .. file .. "HTTP/1.0\r\n\r\n")
        while true do
            local s, status, partial = receive(c)
            count = count + #(s or partial)
            if status == "closed" then break end
        end
        c:close()
        print(file, count)
    end
```

&emsp;&emsp;由于对远程文件的内容并不感兴趣，所以不需要将文件内容写到标准输出中，只需计算并打印出文件大小即可。在上述代码中，还使用了一个辅助函数`receive`来从连接接收数据。在顺序下载的方法中，`receive`的代码可以是这样的：

```lua
    function receive(connection)
        return connection:receive(2^10)
    end
```

&emsp;&emsp;而在并发的实现中，这个函数在接收数据时绝对不能阻塞。因此，它需要在没有足够的可用数据时挂起执行。新代码如下：

```lua
    function receive(connection)
        connection.settimeout(0) 	-- 使recevie调用不会阻塞
        local s, status, partial = connection:receive(2^20)
        if status == "timeout" then
            coroutine.yield(connection)
        end
        return s or partial, status
    end
```

&emsp;&emsp;对`settimeout(0)`的调用可使以后所有对此连接进行的操作不会阻塞。若一个操作返回的`status`为“`timeout`（超时）”，就表示该操作在返回时还未完成。此时，线程就会挂起执行。而以非假的参数来调用`yield`，可以告诉调度程序线程仍在执行任务中。注意，即使在超时的情况下，连接也是会返回已经读取到的内容，即记录在`partial`变量中的值。

&emsp;&emsp;以下这段代码展示了调度程序及一些辅助代码。`table threads`为调度程序保存着所有正在运行中的线程。函数`get`确保每个下载任务都在一个独立的线程中执行。调度程序本身主要就是一个循环，它遍历所有的线程，逐个唤醒它们的执行。并且当线程完成任务时，将该线程从列表中删除。在所有线程都完成运行后，停止循环。

```lua
    threads = {}  	-- 用于记录所有正在运行的线程

    function get(host, file)
        -- 创建协同程序
        local co = coroutine.create(function()
            download(host, file)
        end)
        -- 将其插入记录表中
        table.insert(threads, co)
    end

    function dispatch()
        local i = 1
        while true do
            if threads[i] == nil then                    -- 还有线程吗？
                if threads[1] == nil then break end      -- 列表是否为空？
                i = 1                                    -- 重新开始循环
            end
            local status, res = coroutine.resume(threads[i])
            if not res then                              -- 线程是否已经完成了任务？
                table.remove(threads, i)
            else
                i = i + 1
            end
        end
    end
```


&emsp;&emsp;最后，主程序需要创建所有的线程，并调用调度程序。例如，若要下载W3C站点上的4个文件，主程序如下：

```lua
    host = "www.w3.org"

    get(host, "/TR/html401/html40.txt")
    get(host, "/TR/2002/REC-xhtml1-20020801/xhtml1.pdf")
    get(host, "/TR/REC-html32.html")
    get(host, "/TR/2000/REC-DOM-Level-2-Core-20001113/DOM2-Core.txt")

    dispatch() 		-- 主循环
```

&emsp;&emsp;通过协同程序，计算机只需要6秒便可下载完成这4个文件。但若使用顺序下载的话，则需要多耗费两倍的时间（15秒左右）。

&emsp;&emsp;除了速度有所提高外，上面这个实现还不够完美。只要有一个线程在读取数据，就不会有问题。但若所有线程都没有数据可读，调度程序就会执行一个“忙碌等待（`Busy Wait`）”，不断地从一个线程切换到另一个线程，仅仅是为了检测是否还有数据可读。这样便导致了这个协同程序的实现会比顺序下载多耗费将近30倍的CPU时间。

&emsp;&emsp;为了避免这样的情况，可以使用`LuaSocket`中的`select`函数。这个函数可以用于等待一组`socket`的状态改变，在等待时程序陷入阻塞（`block`）状态。若要在当前实现中应用这个函数，只需要修改调度程序即可，新版本如下：

```lua
    function dispatch()
        local i = 1
        local connections = {}
        while true do
            if threads[i] == nil then                    -- 还有线程吗？
                if threads[1] == nil then break end
                i = 1                                    -- 重新开始循环
                connections = {}
            end
            local status, res = coroutine.resume(threads[i])
            if not res then                              -- 线程是否已经完成了任务？
                table.remove(threads, i)
            else 				-- 超时
                i = i + 1
                connections[#connections + 1] = res
                if #connections == #threads then         -- 所有线程都阻塞了吗？
                    socket.select(connections)
                end
            end
        end
    end
```


&emsp;&emsp;新的调度程序将所有超时的连接收集到一个名为`connections`的`table`中。记住，`receive`会将超时的连接通过`yield`传递，也就是`resume`会返回它们。如果所有的连接都超时了，调度程序就调用`select`来等待这些连接的状态发生变化。这个最终版本的实现与上一个使用协同程序的实现一样快，另外由于它不会有“忙碌等待”，所以只比顺序下载耗费CPU资源略多而已。


🔚