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

在JVM上可以通过使用ServiceLoader注册CoroutineExceptionHandler为所有协程重新定义全局异常处理程序。全局异常处理程序类似于Thread.defaultUncaughtExceptionHandler，它在没有更多特定的处理程序被注册时使用。在Android上，uncaughtExceptionPreHandler是作为全局协程异常处理程序被使用

CoroutineExceptionHandler只在uncaught异常上调用(没有以任何其他方式处理的异常)。特别是所有子协程将异常处理委托给父协程，父协程也将异常处理委托给父协程，依此类推直到根协程，因此注册在它们上下文中的CoroutineExceptionHandler永远不会被使用。除此之外，async构造器总是捕获所有异常并将它们表示在结果的Deferred对象中，所以它的CoroutineExceptionHandler也没有效果


























































