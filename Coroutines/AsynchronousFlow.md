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



