# Asynchronous Flow
挂起函数异步返回一个值，如果需要返回多个异步计算的值，可以使用Flow

## Representing multiple values
在Kotlin中可以使用集合表示多个值。例如，我们有一个simple函数返回三个数字的列表，然后使用forEach打印它们

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)
 
fun main() {
    simple().forEach { value -> println(value) } 
}
```

输出结果：

```kotlin
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

这段代码在打印每个数字前等待100ms，并且不会阻塞主线程。可以通过运行在主线程中的独立协程每隔100ms打印"I'm not blocked"来验证这一点

```kotlin
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
```

