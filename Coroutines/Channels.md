# Channels
Deferred对象提供了一种方便的方式在协程之间传输单个值。通道提供了一种方式来传输值流

## Channel basics
通道在概念上非常类似于BlockingQueue。一个主要的区别是它使用挂起操作符send而不是阻塞操作符put，使用挂起操作符receive而不是阻塞操作符take

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>()
    launch {
        // this might be heavy CPU-consuming computation or async logic, we'll just send five squares
        for (x in 1..5) channel.send(x * x)
    }
    // here we print five received integers:
    repeat(5) { println(channel.receive()) }
    println("Done!")
}
```

输出结果：

```
1
4
9
16
25
Done!
```

## Closing and iteration over channels
与队列不同，通道可以被关闭以表示没有更多的元素输入。在接收端使用常规的for循环从通道接收元素是很方便的

从概念上来说，close类似于向通道发送一个特殊的关闭标记。一旦收到这个关闭标记迭代就会停止，因此可以保证在关闭标记之前发送的所有元素都会被收到

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>()
    launch {
        for (x in 1..5) channel.send(x * x)
        channel.close() // we're done sending
    }
    // here we print received values using `for` loop (until the channel is closed)
    for (y in channel) println(y)
    println("Done!")
}
```

## Building channel producers
协程产生一个元素序列的模式是非常常见。这是经常出现在并发代码中的生产-消费模式的一部分。你可以将生产者抽象为以通道作为参数的函数，但这违反了必须从函数返回结果的常识

可以使用协程构造器produce从生产者中返回通道，并且使用扩展函数consumeEach代替消费者中的for循环

```kotlin
fun CoroutineScope.produceSquares(): ReceiveChannel<Int> = produce {
    for (x in 1..5) send(x * x)
}

fun main() = runBlocking {
    val squares = produceSquares()
    squares.consumeEach { println(it) }
    println("Done!")
}
```

## Pipelines
管道是一种模式，其中的一个协程产生值流(可能是无限的)

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++) // infinite stream of integers starting from 1
}
```

另一个协程消费这个流，做一些处理并且产生一些其他结果。在下面的例子中，数字只是被平方

```kotlin
fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
    for (x in numbers) send(x * x)
}
```

main函数启动并且连接整个管道

```kotlin
fun main() = runBlocking {
    val numbers = produceNumbers() // produces integers from 1 and on
    val squares = square(numbers) // squares integers
    repeat(5) {
        println(squares.receive()) // print first five
    }
    println("Done!") // we are done
    coroutineContext.cancelChildren() // cancel children coroutines
}

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++) // infinite stream of integers starting from 1
}

fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
    for (x in numbers) send(x * x)
}
```

所有创建协程的函数都被定义为协程作用域上的扩展函数，因此我们可以依赖结构化并发来确保程序中没有持久的全局协程

## Prime numbers with pipeline
让我们来看一个极端的管道例子，它使用协程的管道生成素数。我们从一个无穷序列开始

```kotlin
fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start
    while (true) send(x++) // infinite stream of integers from start
}
```

下面的管道部分过滤传入的数字流，删除所有能被给定素数整除的数字

```kotlin
fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int) = produce<Int> {
    for (x in numbers) if (x % prime != 0) send(x)
}
```

现在我们通过从2开始的数字流来构建管道，从当前通道中获取一个素数，并且为找到的每个素数启动新的管道

```
numbersFrom(2) -> filter(2) -> filter(3) -> filter(5) -> filter(7) ...
```

下面的例子打印前10个素数，并且在主线程上下文中运行整个管道。因为所有的协程都是在主runBlocking协程作用域中启动的，我们不需要为已经启动的所有协程保留一个明确的列表。我们使用cancelChildren扩展函数在打印前10个素数后取消所有的子协程

```kotlin
fun main() = runBlocking {
    var cur = numbersFrom(2)
    repeat(10) {
        val prime = cur.receive()
        println(prime)
        cur = filter(cur, prime)
    }
    coroutineContext.cancelChildren() // cancel all children to let main finish    
}

fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start
    while (true) send(x++) // infinite stream of integers from start
}

fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int) = produce<Int> {
    for (x in numbers) if (x % prime != 0) send(x)
}
```

输出结果：

```
2
3
5
7
11
13
17
19
23
29
```

注意，你可以使用标准库中的iterator协程构造器来构造相同的管道。使用iterator替换produce，用yield替换send，用next替换receive，用Iterator替换ReceiveChannel，并且去掉协程作用域。你也不需要runBlocking。然而，如上所示使用通道实现管道的好处是，如果在Dispatchers.Default上下文中运行它实际上可以使用多个CPU核心

无论如何，这是一种非常不切实际的求素数方法。实际上，管道涉及一些其他的挂起调用(比如对远程服务的异步调用)，而且这些管道不能使用sequence/iterator构造因为它们不允许任意挂起，不像produce是完全异步的

## Fan-out
多个协程可以从同一通道接收数据，协程之间独立工作。让我们从一个定期生成整数的生产者协程开始(每秒10个数字)

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1 // start from 1
    while (true) {
        send(x++) // produce next
        delay(100) // wait 0.1s
    }
}
```

然后我们可以有几个处理协程。在本例中，他们只打印id和收到的数值

```kotlin
fun CoroutineScope.launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    for (msg in channel) {
        println("Processor #$id received $msg")
    }    
}
```

现在让我们启动5个处理器，让它们工作近一秒。看看会发生什么

```kotlin
fun main() = runBlocking<Unit> {
    val producer = produceNumbers()
    repeat(5) { launchProcessor(it, producer) }
    delay(950)
    producer.cancel() // cancel producer coroutine and thus kill them all
}

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1 // start from 1
    while (true) {
        send(x++) // produce next
        delay(100) // wait 0.1s
    }
}

fun CoroutineScope.launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    for (msg in channel) {
        println("Processor #$id received $msg")
    }    
}
```

输出类似于下面，尽管接收每个整数的处理器id可能不同

```
Processor #2 received 1
Processor #4 received 2
Processor #0 received 3
Processor #1 received 4
Processor #3 received 5
Processor #2 received 6
Processor #4 received 7
Processor #0 received 8
Processor #1 received 9
Processor #3 received 10
```

注意，取消一个生产者协程将关闭它的通道，从而最终终止处理协程执行的通道上的迭代

另外，请注意我们是如何明确的使用for循环遍历通道在launchProcessor代码中执行扇出的。与consumeEach不同，这个for循环模式在多协程中使用是非常安全的。如果其中一个处理协程失败，那么其他处理协程仍然可以处理该通道，而使用consumeEach实现的处理器会在其正常或异常完成时消费或取消通道

## Fan-in
多个协程可以发送到同一个通道。例如，我们有一个字符串通道和一个挂起函数，它以特定的延迟重复向该通道发送特定的字符串

```kotlin
suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
    while (true) {
        delay(time)
        channel.send(s)
    }
}
```

现在让我们看看如果启动两个协程发送字符串会发生什么(在本例中，我们在主线程的上下文中作为主协程的孩子来启动他们)

```kotlin
fun main() = runBlocking {
    val channel = Channel<String>()
    launch { sendString(channel, "foo", 200L) }
    launch { sendString(channel, "BAR!", 500L) }
    repeat(6) { // receive first six
        println(channel.receive())
    }
    coroutineContext.cancelChildren() // cancel all children to let main finish
}

suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
    while (true) {
        delay(time)
        channel.send(s)
    }
}
```

输出结果：

```
foo
foo
BAR!
foo
foo
BAR!
```

## Buffered channels
目前为止展示的通道都没有缓存。当发送方和接收方相遇时(会和)，无缓存通道开始传输元素。如果先调用send，发送方会被挂起直到调用receive，如果先调用receive，接收方会被挂起直到调用send

Channel工厂函数和produce构造器都使用了可选的capacity参数来指定缓存大小。缓存允许发送方在挂起之前发送多个元素，类似于具有指定容量的BlockingQueue，它会在缓存满时阻塞

```kotlin
fun main() = runBlocking<Unit> {
    val channel = Channel<Int>(4) // create buffered channel
    val sender = launch { // launch sender coroutine
        repeat(10) {
            println("Sending $it") // print before sending each element
            channel.send(it) // will suspend when buffer is full
        }
    }
    // don't receive anything... just wait....
    delay(1000)
    sender.cancel() // cancel sender coroutine    
}
```

使用容量为4的缓存通道会打印5次"sending"

```
Sending 0
Sending 1
Sending 2
Sending 3
Sending 4
```

前四个元素被添加到缓存中，发送方在尝试发送第五个元素时挂起

## Channels are fair
通道的发送和接收操作是公平的，并且遵循它们从多个协程的调用顺序。他们满足先进先出原则，例如，第一个调用receive的协程会获取到元素。在下面的例子中，两个协程"ping"和"pong"从共享的"table"通道接收"ball"对象

```kotlin
data class Ball(var hits: Int)

fun main() = runBlocking {
    val table = Channel<Ball>() // a shared table
    launch { player("ping", table) }
    launch { player("pong", table) }
    table.send(Ball(0)) // serve the ball
    delay(1000) // delay 1 second
    coroutineContext.cancelChildren() // game over, cancel them
}

suspend fun player(name: String, table: Channel<Ball>) {
    for (ball in table) { // receive the ball in a loop
        ball.hits++
        println("$name $ball")
        delay(300) // wait a bit
        table.send(ball) // send the ball back
    }
}
```

"ping"协程首先开始，所以它第一个收到ball。即使"ping"协程在把ball发送回通道后立即再次接收，ball仍然会被"pong"协程接收，因为它已经在等待了

```
ping Ball(hits=1)
pong Ball(hits=2)
ping Ball(hits=3)
pong Ball(hits=4)
```

请注意，由于所使用执行器的特性，有时通道可能会产生看起来不公平的执行

## Ticker channels
Ticker通道是一种特殊的集合通道，从通道的最后一个元素被消费开始，在每次给定的延迟之后产生一个Unit。单独来说，它看起来是无用的。对于创建复杂的基于时间的管道或者执行窗口操作或者其他依赖时间的处理来说，它是一个有用的构造器。可以在select中使用Ticker通道执行"on tick"操作

使用工厂方法ticker创建Ticker通道。使用通道的ReceiveChannel.cancel方法表示不需要更多的元素

让我们看看它是如何工作的

```kotlin
fun main() = runBlocking<Unit> {
    val tickerChannel = ticker(delayMillis = 100, initialDelayMillis = 0) // create ticker channel
    var nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
    println("Initial element is available immediately: $nextElement") // no initial delay

    nextElement = withTimeoutOrNull(50) { tickerChannel.receive() } // all subsequent elements have 100ms delay
    println("Next element is not ready in 50 ms: $nextElement")

    nextElement = withTimeoutOrNull(60) { tickerChannel.receive() }
    println("Next element is ready in 100 ms: $nextElement")

    // Emulate large consumption delays
    println("Consumer pauses for 150ms")
    delay(150)
    // Next element is available immediately
    nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
    println("Next element is available immediately after large consumer delay: $nextElement")
    // Note that the pause between `receive` calls is taken into account and next element arrives faster
    nextElement = withTimeoutOrNull(60) { tickerChannel.receive() } 
    println("Next element is ready in 50ms after consumer pause in 150ms: $nextElement")

    tickerChannel.cancel() // indicate that no more elements are needed
}
```

输出结果：

```
Initial element is available immediately: kotlin.Unit
Next element is not ready in 50 ms: null
Next element is ready in 100 ms: kotlin.Unit
Consumer pauses for 150ms
Next element is available immediately after large consumer delay: kotlin.Unit
Next element is ready in 50ms after consumer pause in 150ms: kotlin.Unit
```

注意，ticker会意识到可能的消费者暂停，默认情况下，如果发生暂停它会调整下一个生成元素的延迟，以保持一个固定的产生元素的比率

可选的，可以指定mode参数为TickerMode.FIXED_DELAY来保持元素之间的固定延迟

