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























































