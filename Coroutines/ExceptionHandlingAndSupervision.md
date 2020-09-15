# Exception Handling
本节介绍异常处理和异常取消。我们知道取消协程会在挂起点抛出CancellationException并且它会被协程的处理机制忽略。下面我们来看看如果在取消过程中抛出一个异常或者同一协程的多个孩子抛出一个异常会发生什么

## Exception propagation
协程构造器有两种类型：自动传递异常(launch和actor)，暴露异常给用户(async和produce)。当使用这些构造器创建根协程时(它不是另一个协程的孩子)，前面的构造器将异常作为uncaught异常，类似于Java的Thread.uncaughtExceptionHandler，后者依赖于用户对最终异常的消费，例如通过await或receive

可以通过使用GlobalScope创建根协程的简单例子来演示

```kotlin
fun main() = runBlocking {
    val job = GlobalScope.launch { // root coroutine with launch
        println("Throwing exception from launch")
        throw IndexOutOfBoundsException() // Will be printed to the console by Thread.defaultUncaughtExceptionHandler
    }
    job.join()
    println("Joined failed job")
    val deferred = GlobalScope.async { // root coroutine with async
        println("Throwing exception from async")
        throw ArithmeticException() // Nothing is printed, relying on user to call await
    }
    try {
        deferred.await()
        println("Unreached")
    } catch (e: ArithmeticException) {
        println("Caught ArithmeticException")
    }
}
```

输出结果：

```
Throwing exception from launch
Exception in thread "DefaultDispatcher-worker-2 @coroutine#2" java.lang.IndexOutOfBoundsException
Joined failed job
Throwing exception from async
Caught ArithmeticException
```

## CoroutineExceptionHandler
可以自定义打印uncaught异常到控制台的默认行为。根协程上的CoroutineExceptionHandler上下文元素可以作为能够自定义异常处理的根协程和它所有孩子的catch块。它类似于Thread.uncaughtExceptionHandler。你不能从CoroutineExceptionHandler的异常中恢复。协程在调用处理程序时已经完成了相应的异常。通常情况下，处理程序用来记录异常，显示某种错误消息，终止或者重新启动程序



























































