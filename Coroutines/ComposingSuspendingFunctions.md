# Composing Suspending Functions
本节介绍组合挂起函数的各种方法

## Sequential by default
假设我们在某处定义了两个挂起函数，它们执行一些有用的操作，比如某种类型的远程服务调用或计算。我们假设它们是有用的，对于本例来说实际上它们只是延迟了一秒钟

```kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

如果需要按顺序调用它们并计算它们结果的总和，我们应该怎么办？实际上我们是这样做的，比如我们使用第一个函数的结果来决定是否需要调用第二个函数或决定如何调用它

我们使用正常的顺序调用，因为协程中的代码与常规代码一样默认是顺序的。下面的例子通过计算执行两个挂起函数花费的总时间来演示它

```kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = doSomethingUsefulOne()
        val two = doSomethingUsefulTwo()
        println("The answer is ${one + two}")
    }
    println("Completed in $time ms")    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

输出结果：

```kotlin
The answer is 42
Completed in 2017 ms
```

## Concurrent using async
如果两个函数调用之间没有依赖关系，并且我们希望同时调用这两个函数来更快的得到结果应该怎么办？这时可以使用async

从概念上，async很像launch。它启动一个单独的协程(一个轻量级线程)，可以与所有其他协程并行工作。不同之处在于launch返回一个Job并且不会携带任何结果值，而async返回一个Deferred，它是一个轻量级无阻塞的future，用来表示以后提供结果的promise。你可以调用Deferred的await函数来获取它的最终结果，由于Deferred也是一个Job，你可以在需要时取消它

```kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

输出结果：

```kotlin
The answer is 42
Completed in 1017 ms
```

由于两个协程是并发执行，速度是原来的两倍

## Lazily started async
通过设置start参数为CoroutineStart.LAZY，可以将async变成懒加载。在这种模式下，它只在调用await函数请求结果或者调用Job对象的start函数时启动协程

```kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
        val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
        // some computation
        one.start() // start the first one
        two.start() // start the second one
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

输出结果：

```kotlin
The answer is 42
Completed in 1017 ms
```

这里定义了两个协程但不像前面的例子中被执行，程序员可以通过调用start函数来控制何时开始执行。我们先启动一个再启动第二个，然后等待每个协程完成

注意，如果我们只在println中调用await函数而没有先调用每个协程的start函数，这将导致顺序执行因为await函数会启动协程并等待它完成，这不是懒加载的预期。当值运算涉及到挂起函数时，可以使用async(start = CoroutineStart.LAZY)来替代标准的lazy函数

## Async-style functions
我们可以使用全局作用域中的async协程构建器来定义异步形式的函数，它们异步调用doSomethingUsefulOne和doSomethingUsefulTwo。我们将这些函数以Async后缀命名，用来强调它们仅启动异步运算并且需要使用Deferred对象来获得结果

```kotlin
//somethingUsefulOneAsync的结果类型为Deferred<Int>
fun somethingUsefulOneAsync() = GlobalScope.async {
    doSomethingUsefulOne()
}

//somethingUsefulTwoAsync的结果类型为Deferred<Int>
fun somethingUsefulTwoAsync() = GlobalScope.async {
    doSomethingUsefulTwo()
}
```

注意，这些Async函数不是挂起函数，它们可以在任何地方使用。然而它们的使用意味着异步执行它们的函数调用操作

```kotlin
//注意，我们没有在main函数中使用runBlocking
fun main() {
    val time = measureTimeMillis {
        //在协程外初始化异步操作
        val one = somethingUsefulOneAsync()
        val two = somethingUsefulTwoAsync()
        //但是await函数必须在协程中执行
        //这里使用runBlocking在等待结果时阻塞主线程
        runBlocking {
            println("The answer is ${one.await() + two.await()}")
        }
    }
    println("Completed in $time ms")
}

fun somethingUsefulOneAsync() = GlobalScope.async {
    doSomethingUsefulOne()
}

fun somethingUsefulTwoAsync() = GlobalScope.async {
    doSomethingUsefulTwo()
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

这里提供的这种带有异步函数的编程形式只是为了演示，因为在其他编程语言中它是一种流行的形式。在Kotlin协程中使用这种形式是非常不建议的，原因如下：

考虑一下，如果在val one = somethingUsefulOneAsync()和one.await()之间的代码中出现一些逻辑错误，程序抛出异常并且正在执行的操作被中止会发生什么情况。通常情况下，一个全局错误处理可以捕获这个异常，为开发人员记录并报告错误，程序可以继续执行其他操作。但是这里somethingUsefulOneAsync函数仍然在后台运行，即使初始化它的操作已被中止。这个问题不会在结构化并发中出现

## Structured concurrency with async
我们以第二个程序为例，我们提取一个同时执行doSomethingUsefulOne和doSomethingUsefulTwo的函数，并返回它们结果的总和。由于async协程构建器被定义为协程作用域上的扩展，我们需要在协程作用域中使用它，这正是coroutineScope函数所做的

```kotlin
suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}
```

如果concurrentSum函数内部的代码出现错误并抛出异常，那么在其作用域中启动的所有协程都将被取消

```kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        println("The answer is ${concurrentSum()}")
    }
    println("Completed in $time ms")    
}

suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

两个操作仍然同时执行

```kotlin
The answer is 42
Completed in 1017 ms
```

取消总是通过协程层次结构进行传播

```kotlin
fun main() = runBlocking<Unit> {
    try {
        failedConcurrentSum()
    } catch(e: ArithmeticException) {
        println("Computation failed with ArithmeticException")
    }
}

suspend fun failedConcurrentSum(): Int = coroutineScope {
    val one = async<Int> { 
        try {
            delay(Long.MAX_VALUE) // Emulates very long computation
            42
        } finally {
            println("First child was cancelled")
        }
    }
    val two = async<Int> { 
        println("Second child throws an exception")
        throw ArithmeticException()
    }
    one.await() + two.await()
}
```

注意第一个async和等待的父协程是如何在第二个async失败时被取消的

```kotlin
Second child throws an exception
First child was cancelled
Computation failed with ArithmeticException
```
