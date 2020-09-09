# Asynchronous Flow
挂起函数异步返回一个值，如果需要返回多个异步计算的值，可以使用Flow

## Representing multiple values
在Kotlin中可以使用集合来表示多个值。例如，我们有一个simple函数返回三个数字的列表，然后使用forEach打印它们

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)
 
fun main() {
    simple().forEach { value -> println(value) } 
}
```

输出结果：

```
1
2
3
```

## Sequences
如果我们使用一些消耗cpu的阻塞代码(每次计算花费100ms)来计算这些数字，那么我们可以使用序列来表示它们

```kotlin
fun simple(): Sequence<Int> = sequence { // sequence builder
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it
        yield(i) // yield next value
    }
}

fun main() {
    simple().forEach { value -> println(value) } 
}
```

这段代码输出相同的数字，但是在打印每个数字之前等待100ms

## Suspending functions
上面的代码阻塞了主线程。当使用异步代码计算这些值时，我们可以使用suspend关键字标记simple函数，这样可以在不阻塞的情况下进行计算并且以列表的形式返回结果

```kotlin
suspend fun simple(): List<Int> {
    delay(1000) // pretend we are doing something asynchronous here
    return listOf(1, 2, 3)
}

fun main() = runBlocking<Unit> {
    simple().forEach { value -> println(value) } 
}
```

这段代码在等待一秒后打印数字

## Flows
使用List&lt;Int>类型意味着我们只能一次返回所有的值。为了表示正在异步计算的流值，我们可以使用Flow&lt;Int>类型，就像使用Sequence&lt;Int>类型来表示同步计算的值一样

```kotlin
fun simple(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    // Collect the flow
    simple().collect { value -> println(value) } 
}
```

这段代码在打印每个数字前等待100ms，并且不会阻塞主线程。可以通过运行在主线程中的独立协程每隔100ms打印一次"I'm not blocked"来验证这一点

```
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
```

注意使用Flow的代码和前面示例之间的以下区别

* 用于Flow类型的构造器函数flow被调用

* 在flow构造器中的代码可以挂起

* simple函数不再使用suspend关键字标记

* 使用emit函数从流中发送值

* 使用collect函数从流中收集值

我们可以使用Thread.sleep替换flow函数体中的delay，可以看到在这种情况下主线程被阻塞了

## Flows are cold
Flows是类似于序列的冷流。flow构造器中的代码在collect函数调用之前不会运行

```kotlin
fun simple(): Flow<Int> = flow { 
    println("Flow started")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    println("Calling simple function...")
    val flow = simple()
    println("Calling collect...")
    flow.collect { value -> println(value) } 
    println("Calling collect again...")
    flow.collect { value -> println(value) } 
}
```

输出结果：

```
Calling simple function...
Calling collect...
Flow started
1
2
3
Calling collect again...
Flow started
1
2
3
```

这就是simple函数(返回flow)没有标记为suspend的一个关键原因。就其本身而言，simple函数调用快速返回并且不会等待任何东西。flow在每次调用collect时启动，这就是为什么我们在调用collect时看到"Flow started"

## Flow cancellation basics
Flow遵循协程的协作性取消。通常情况下，当Flow在可取消的挂起函数(比如delay)中挂起时，可以取消Flow收集。下面的例子演示了Flow在withTimeoutOrNull中运行时，它是如何在超时时被取消并且停止执行其代码的

```kotlin
fun simple(): Flow<Int> = flow { 
    for (i in 1..3) {
        delay(100)          
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(250) { // Timeout after 250ms 
        simple().collect { value -> println(value) } 
    }
    println("Done")
}
```

注意在simple函数中Flow只发送两个数字：

```
Emitting 1
1
Emitting 2
2
Done
```

## Flow builders
前面例子中的flow构造器是最基本的一个，还有其他更容易声明Flow的构造器：

* flowOf构造器用来定义发送一组固定值的Flow

* 可以使用asFlow扩展函数将各种集合和序列转换为Flow

因此，从Flow中打印从1到3数字的例子可以写成：

```kotlin
fun main() = runBlocking<Unit> {
    // Convert an integer range to a flow
    (1..3).asFlow().collect { value -> println(value) } 
}
```

## Intermediate flow operators
Flow可以使用操作符进行转换，就像使用集合和序列一样。中间操作符应用于一个上游流并且返回一个下游流。像Flow一样，这些操作符是冷的。对此类操作符的调用本身不是挂起函数。它快速执行并且返回一个新的转换流的定义

基本操作符有熟悉的名字，比如map和filter。与序列的重要区别在于，这些操作符中的代码块可以调用挂起函数

例如，输入请求Flow可以使用map操作符映射成为结果，即使执行的请求是由挂起函数实现的长时间运行的操作

```kotlin
suspend fun performRequest(request: Int): String {
    delay(1000) // imitate long-running asynchronous work
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow() // a flow of requests
        .map { request -> performRequest(request) }
        .collect { response -> println(response) }
}
```

每秒钟之后输出一行结果：

```
response 1
response 2
response 3
```

**Transform operator**

在Flow变换操作中最常用的是transform。它可以用来模拟简单的变换，比如map和filter，以及实现更复杂的变换。使用transform操作符，我们可以发送任意次数的任意值

例如，我们可以使用transform在执行一个长时间运行的异步请求之前发送一个字符串，然后在后面增加一个响应

```kotlin
suspend fun performRequest(request: Int): String {
    delay(1000) // imitate long-running asynchronous work
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow() // a flow of requests
        .transform { request ->
            emit("Making request $request") 
            emit(performRequest(request)) 
        }
        .collect { response -> println(response) }
}
```

输出结果：

```
Making request 1
response 1
Making request 2
response 2
Making request 3
response 3
```

**Size-limiting operators**

当达到相应的限制时，限制大小操作符(比如take)将取消Flow的执行。协程中的取消总是通过抛出异常来实现，因此所有资源管理函数(比如try{}finally{})在取消时都能正常工作

```kotlin
fun numbers(): Flow<Int> = flow {
    try {                          
        emit(1)
        emit(2) 
        println("This line will not execute")
        emit(3)    
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking<Unit> {
    numbers() 
        .take(2) // take only the first two
        .collect { value -> println(value) }
}  
```

从输出结果中可以看到，在发送第二个数字之后Flow的执行被停止

```
1
2
Finally in numbers
```

## Terminal flow operators
Flow上的终端操作符是启动Flow收集的挂起函数。collect是最常用的一个，还有其他一些终端操作符

* 转换到各种集合，比如toList和toSet

* first操作符获取第一个值，并且确保Flow发出单个值

* 使用reduce和fold将Flow减少到一个值

```kotlin
fun main() = runBlocking<Unit> {

    val sum = (1..5).asFlow()
        .map { it * it } // squares of numbers from 1 to 5                           
        .reduce { a, b -> a + b } // sum them (terminal operator)
    println(sum)     
}
```

输出结果：

```
55
```

## Flows are sequential
Flow的每个单独集合都是按顺序执行的，除非使用了对多个Flow进行操作的特殊操作符。集合直接在调用终端操作符的协程中运行，默认情况下不会启动新的协程。每个发送的值由从上游到下游的所有中间操作符进行处理，然后交给终端操作符

下面的例子过滤偶数并将其映射为字符串

```kotlin
fun main() = runBlocking<Unit> {

    (1..5).asFlow()
        .filter {
            println("Filter $it")
            it % 2 == 0              
        }              
        .map { 
            println("Map $it")
            "string $it"
        }.collect { 
            println("Collect $it")
        }                      
}
```

输出结果：

```
Filter 1
Filter 2
Map 2
Collect string 2
Filter 3
Filter 4
Map 4
Collect string 4
Filter 5
```

## Flow context
Flow的集合总是在调用协程的上下文中运行。例如，下面的代码中simple流将在该代码作者指定的上下文中运行，而与simple流的实现细节无关

```kotlin
withContext(context) {
    simple().collect { value ->
        println(value) // run in the specified context 
    }
}
```

Flow的这个特性称为上下文保留

默认情况下，flow构造器中的代码在相应Flow收集器提供的上下文中运行。例如，参考simple函数的实现，该函数打印调用它的线程并发送三个数字

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    log("Started simple flow")
    for (i in 1..3) {
        emit(i)
    }
}  

fun main() = runBlocking<Unit> {
    simple().collect { value -> log("Collected $value") } 
}
```

输出结果：

```
[main @coroutine#1] Started simple flow
[main @coroutine#1] Collected 1
[main @coroutine#1] Collected 2
[main @coroutine#1] Collected 3
```

由于collect从主线程中调用，flow的实现也从主线程中调用。对于快速运行或者不关心执行上下文并且不会阻塞调用者的异步代码，这是完美的默认行为

**Wrong emission withContext**

长时间运行的消耗cpu代码可能需要在Dispatchers.Default上下文中运行，更新UI的代码可能需要在Dispatchers.Main上下文中运行。通常情况下，使用withContext在Kotlin协程代码中更改上下文，但是Flow构造器中的代码必须遵守上下文保留特性并且不允许从不同的上下文中发送数据

```kotlin
fun simple(): Flow<Int> = flow {
    // The WRONG way to change context for CPU-consuming code in flow builder
    kotlinx.coroutines.withContext(Dispatchers.Default) {
        for (i in 1..3) {
            Thread.sleep(100) // pretend we are computing it in CPU-consuming way
            emit(i) // emit next value
        }
    }
}

fun main() = runBlocking<Unit> {
    simple().collect { value -> println(value) } 
}
```

输出结果：

```
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
        Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@5511c7f8, BlockingEventLoop@2eac3323],
        but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@2dae0000, Dispatchers.Default].
        Please refer to 'flow' documentation or use 'flowOn' instead
    at ...
```

**flowOn operator**

这个异常指出应该使用flowOn函数更改Flow发送的上下文。下面的例子展示了更改Flow上下文的正确方法，它还打印相应线程的名称以显示其工作原理

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it in CPU-consuming way
        log("Emitting $i")
        emit(i) // emit next value
    }
}.flowOn(Dispatchers.Default) // RIGHT way to change context for CPU-consuming code in flow builder

fun main() = runBlocking<Unit> {
    simple().collect { value ->
        log("Collected $value") 
    } 
} 
```

注意Flow是如何在后台线程中运行的，而收集器是在主线程中运行的

这里要注意的另一件事是flowOn操作符改变了Flow的默认连续特性。现在收集器运行在一个协程中("coroutine#1")，而发送器运行在另一个协程中("coroutine#2")，它与收集器协程并行地运行在另一个线程中。当flowOn操作符必须在其上下文中更改协程调度器时，它会为上游流创建另一个协程

## Buffering
从流收集所花费的总时间角度来看，在不同协程中运行流的不同部分可能很有帮助，尤其是在涉及长时间运行的异步操作时。例如，考虑这种情况，当simple流的发送器很慢需要100ms来产生一个元素，收集器也很慢需要300ms处理一个元素。让我们看看收集这样一个包含三个数字的流需要多长时间

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
    }   
    println("Collected in $time ms")
}
```

输出结果如下，整个收集大约花费1200ms(三个数字，每个400ms)

```
1
2
3
Collected in 1220 ms
```

我们可以在流上使用buffer操作符在收集器运行的同时并行地运行发送器，而不是按顺序运行它们

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple()
            .buffer() // buffer emissions, don't wait
            .collect { value -> 
                delay(300) // pretend we are processing it for 300 ms
                println(value) 
            } 
    }   
    println("Collected in $time ms")
}
```

它更快的产生相同的数字，因为我们有效地创建了一个处理管道，只需等待100ms来处理第一个数字，然后只需花费300ms来处理每个数字。这种方法运行大约需要1000ms

```
1
2
3
Collected in 1071 ms
```

注意，当必须更改协程调度器时，flowOn操作符使用了类似的Buffering机制。但是这里，我们明确的请求Buffering而不更改执行上下文

**Conflation**

当流表示操作的部分结果或操作状态更新时，可能不需要处理每个值，而只需处理最近的值。在这种情况下，当收集器太慢无法处理中间值时，可以使用conflate操作符跳过他们

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple()
            .conflate() // conflate emissions, don't process each one
            .collect { value -> 
                delay(300) // pretend we are processing it for 300 ms
                println(value) 
            } 
    }   
    println("Collected in $time ms")
}
```

可以看到，当第一个数字还在处理的时候，第二个和第三个数字已经产生了，所以第二个数字被跳过了，只有最近的(第三个数字)被发送给收集器

```
1
3
Collected in 758 ms
```

**Processing the latest value**

当发送器和收集器都很慢时，Conflation是一种提升处理速度的方法。它通过丢弃发送的值来实现。另一种方法是取消缓慢的收集器，并且在每次发送新值时重新启动它。有一类xxxLatest操作符，它们执行与xxx操作符相同的逻辑，但是在收到新值时会取消前面接收器中的代码

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple()
            .collectLatest { value -> // cancel & restart on the latest value
                //每次发送数据时会把之前慢的收集器取消
                println("Collecting $value") 
                delay(300) // pretend we are processing it for 300 ms
                println("Done $value") 
            } 
    }   
    println("Collected in $time ms")
}
```

由于collectLatest的执行需要300ms，但是每100ms就会发送一个新值，我们看到收集器在每个值上都会执行，但是只在最后一个值上完全执行

```
Collecting 1
Collecting 2
Collecting 3
Done 3
Collected in 741 ms
```

## Composing multiple flows
有很多方法可以组合多个流

**Zip**

与Kotlin标准库中的Sequence.zip扩展函数一样，流也有一个zip操作符用来组合两个流的对应值

```kotlin
fun main() = runBlocking<Unit> { 

    val nums = (1..3).asFlow() // numbers 1..3
    val strs = flowOf("one", "two", "three") // strings 
    nums.zip(strs) { a, b -> "$a -> $b" } // compose a single string
        .collect { println(it) } // collect and print
}
```

输出结果：

```
1 -> one
2 -> two
3 -> three
```

**Combine**

当流表示变量或操作的最新值时，可能需要执行依赖于相应流的最新值的计算，并且在任何上游流发送数据时重新进行计算。相对应的运算符称为combine

例如，如果前面例子中的数字每300ms更新一次，但是字符串每400ms更新一次，使用zip操作符组合它们仍然会产生相同的结果，尽管每400ms打印一次结果

在本例中，我们使用onEach中间操作符来延迟每个元素，使发送示例流的代码更具声明性并且更短

```kotlin
fun main() = runBlocking<Unit> { 
    val nums = (1..3).asFlow().onEach { delay(300) } // numbers 1..3 every 300 ms
    val strs = flowOf("one", "two", "three").onEach { delay(400) } // strings every 400 ms
    val startTime = System.currentTimeMillis() // remember the start time 
    nums.zip(strs) { a, b -> "$a -> $b" } // compose a single string with "zip"
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

当在这里使用combine操作符代替zip时

```kotlin
fun main() = runBlocking<Unit> { 
    val nums = (1..3).asFlow().onEach { delay(300) } // numbers 1..3 every 300 ms
    val strs = flowOf("one", "two", "three").onEach { delay(400) } // strings every 400 ms          
    val startTime = System.currentTimeMillis() // remember the start time 
    nums.combine(strs) { a, b -> "$a -> $b" } // compose a single string with "combine"
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

输出结果完全不同，每次nums或strs流发送数据时都会打印一行

```
1 -> one at 452 ms from start
2 -> one at 651 ms from start
2 -> two at 854 ms from start
3 -> two at 952 ms from start
3 -> three at 1256 ms from start
```

## Flattening flows
流表示异步接收的值序列，因此很容易出现这种情况，每个值触发对另一个值序列的请求。例如，我们可以使用以下函数返回两个间隔500ms的字符串流

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}
```

现在，如果我们有三个整数的流，并且像这样为每个整数调用requestFlow

```kotlin
(1..3).asFlow().map { requestFlow(it) }
```

然后我们得到了一个元素为流的流(Flow&lt;Flow&lt;String>>)，为了进一步处理需要将其简化为单个流。集合和序列可以使用flatten和flatMap操作符。然而由于流的异步特性，它们需要不同的简化模式，因此流上有一系列简化操作符

**flatMapConcat**

连接模式由flatMapConcat和flattenConcat操作符实现。它们对应于相应的序列操作符。如下面的例子所示，它们等待内部流完成然后开始收集下一个流

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapConcat { requestFlow(it) }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

输出结果中可以看到flatMapConcat的顺序特性

```
1: First at 121 ms from start
1: Second at 622 ms from start
2: First at 727 ms from start
2: Second at 1227 ms from start
3: First at 1328 ms from start
3: Second at 1829 ms from start
```

**flatMapMerge**

另一种简化模式是并行地收集所有输入流，并且将它们的值合并到单个流中以便尽快的发送值。它由flatMapMerge和flattenMerge操作符实现。它们都接受一个可选的concurrency参数，用来限制同一时刻收集的并发流的数量(默认值为DEFAULT_CONCURRENCY)

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapMerge { requestFlow(it) }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

输出结果中可以看到flatMapMerge的并发性

```
1: First at 136 ms from start
2: First at 231 ms from start
3: First at 333 ms from start
1: Second at 639 ms from start
2: Second at 732 ms from start
3: Second at 833 ms from start
```

注意，flatMapMerge顺序调用它的代码块(本例中为{requestFlow(it)})，但是并行地收集产生的流，它相当于先执行map，然后对结果调用flattenMerge

**flatMapLatest**

类似于collectLatest操作符，有一种相对应的"Latest"简化模式，在该模式中一旦发送新的流就会取消前一个流的收集。它由flatMapLatest操作符实现

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapLatest { requestFlow(it) }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

输出结果展示了flatMapLatest是如何工作的

```
1: First at 142 ms from start
2: First at 322 ms from start
3: First at 425 ms from start
3: Second at 931 ms from start
```

注意，在发送新值时flatMapLatest会取消代码块中的所有代码(本例中为{requestFlow(it)})。在这个例子中看不出任何区别，因为对requestFlow的调用是快速的，不会挂起并且不能取消。然而，如果我们在那里使用像delay这样的挂起函数就会看到区别

## Flow exceptions
当发送器或操作符内的代码抛出异常时，流收集器可以在异常情况下执行完成。有几种方法去处理这些异常

**Collector try and catch**

收集器可以使用try/catch块来处理异常

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value ->         
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}
```

正如我们所看到的，这段代码成功地捕获了collect终端操作符中的异常，在那之后没有发送任何值

```
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2
```

**Everything is caught**

前面的例子实际上捕获了发送器或任何中间操作符或者终端操作符中发生的任何异常。例如，让我们修改代码使发送的值映射为字符串，但是相应的代码会产生一个异常

```kotlin
fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}
```

这个异常仍然被捕获并且收集器被停止

```
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```

## Exception transparency
但是发送器的代码如何封装它的异常处理行为

流必须对异常透明，在流构造器中从try/catch块内部发送值违反了异常透明性。异常透明性保证了抛出异常的收集器可以像前面例子中那样始终使用try/catch块来捕获异常

发送器可以使用catch操作符来保持异常透明性，并且允许对其异常处理进行封装。catch操作符可以分析异常，并且根据捕获的异常以不同的方式对其作出响应：

* 可以使用throw重新抛出异常

* 可以使用emit将异常转换为发送值

* 异常可以被忽略、记录或由其他代码进行处理

例如，让我们在捕获异常时发送文本

```kotlin
fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> emit("Caught $e") } // emit on exception
        .collect { value -> println(value) }
}
```

例子的输出相同，即使我们没有使用try/catch

**Transparent catch**

catch操作符遵循异常透明性，它只捕获上游异常(来自于catch上面的所有异常，不包含catch下面的异常)。如果collect块中抛出一个异常，它会被漏过

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> println("Caught $e") } // does not catch downstream exceptions
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}
```

尽管使用了catch操作符，但是不会打印"Caught..."消息

```
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
    at ...
```

**Catching declaratively**

我们可以将catch操作符的声明性质与处理所有异常的愿望结合起来，将收集器的实现移动到onEach中并将其放在catch操作符之前。流的收集必须由不带参数的collect调用触发

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .onEach { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
        .catch { e -> println("Caught $e") }
        .collect()
}
```

可以看到打印了"Caught..."消息，这样我们可以在不明确地使用try/catch块的情况下捕获所有异常

```
Emitting 1
1
Emitting 2
Caught java.lang.IllegalStateException: Collected 2
```

## Flow completion
当流收集完成时(通常或异常殊情况下)，可能需要执行一个操作。可以通过两种方式来实现：命令式或者声明式

**Imperative finally block**

除了try/catch，收集器还可以使用finally块在collect完成时执行操作

```kotlin
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("Done")
    }
} 
```

这段代码输出由simple流生成的三个数字，后面跟一个"Done"字符串

```
1
2
3
Done
```

**Declarative handling**




























































































