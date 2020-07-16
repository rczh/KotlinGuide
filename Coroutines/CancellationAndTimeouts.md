# Cancellation and Timeouts
本节讨论协程的取消和超时

## Cancelling coroutine execution
在长时间运行的程序中，你可能需要对后台协程进行细粒度的控制。例如，用户可能关闭了启动协程的页面，现在不再需要协程的结果并且可以取消它的操作。launch函数返回一个job，可用于取消正在运行的协程

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(1000) { i ->
            println("job: I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancel() // cancels the job
    job.join() // waits for job's completion 
    println("main: Now I can quit.")    
}
```

输出结果如下：

```kotlin
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
main: Now I can quit.
```

如果主协程调用job.cancel，我们就不会看到另一个协程的任何输出，因为它被取消了。还有一个Job扩展函数cancelAndJoin，它组合了cancel和join调用

## Cancellation is cooperative
协程的取消是协作性的。协程代码必须相互协作才能取消。kotlinx.coroutines包中的所有挂起函数都是可以取消的。它们检查协程的取消，并在取消时抛出CancellationException。然而如果协程正在进行运算工作并且没有检查取消，那么它不能被取消

```kotlin
fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) { //模拟运算操作
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")    
}
```

运行这段代码可以看到，即使在取消之后协程仍然继续打印"I'm sleeping"，直到job在五次迭代之后自己完成

## Making computation code cancellable
有两种方法可以使运算代码取消。第一种方法是定期调用一个挂起函数来检查取消，可以选择调用yield函数。另一种方法是明确的检查取消状态。让我们试一下后一种方法

将前面例子中的while(i < 5)替换为while(isActive)然后重新运行

```kotlin
fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (isActive) { //CoroutineScope的扩展属性
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")    
}
```

现在这个循环被取消了。isActive是CoroutineScope对象的扩展属性，可以在协程中使用

## Closing resources with finally
可取消的挂起函数会在取消时抛出CancellationException，可以用常规的方式处理它。例如，在协程被取消时使用try表达式或者use函数执行它们的最终操作

```kotlin
fun main() = runBlocking {
    val job = launch {
        try {
            repeat(1000) { i ->
                println("job: I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            println("job: I'm running finally")
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")    
}
```

join和cancelAndJoin函数都会等待所有最终操作完成，上面例子的输出结果为

```kotlin
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
job: I'm running finally
main: Now I can quit.
```

## Run non-cancellable block
在前面例子的finally块中使用挂起函数会导致CancellationException，因为运行此代码的协程被取消了。通常这不是问题，因为所有行为良好的关闭操作通常都是非阻塞的，不涉及任何挂起函数。然而在极少数情况下，当你需要挂起一个已经取消的协程时，你可以使用withContext函数和NonCancellable上下文来封装相应的代码

```kotlin
fun main() = runBlocking {
    val job = launch {
        try {
            repeat(1000) { i ->
                println("job: I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            withContext(NonCancellable) {
                println("job: I'm running finally")
                delay(1000L)
                println("job: And I've just delayed for 1 sec because I'm non-cancellable")
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")    
}
```

## Timeout
取消协程执行的最实际原因是它的执行时间超过了某个时长。你可以手动跟踪相应Job的引用并且在延迟之后启动单独的协程来取消跟踪的Job。可以使用withTimeout函数完成这个操作

```kotlin
fun main() = runBlocking {
    withTimeout(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
}
```

输出结果：

```kotlin
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 1300 ms
```

withTimeout函数抛出的TimeoutCancellationException异常是CancellationException的子类。我们以前没有看到它的堆栈trace打印在控制台上。这是因为在已经取消的协程中，CancellationException被认为是协程完成的正常原因。但是在本例中，我们在主函数中使用了withTimeout

由于取消只是一个异常，所有资源都按照通常的方式关闭。如果你需要对任何类型的超时执行一些额外操作，你可以将超时代码封装在try块中，或者使用与withTimeout类似的withTimeoutOrNull函数，它在超时时返回null而不是抛出异常

```kotlin
fun main() = runBlocking {
    val result = withTimeoutOrNull(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
        "Done" // will get cancelled before it produces this result
    }
    println("Result is $result")
}
```

运行此代码时没有异常

```kotlin
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Result is null
```
