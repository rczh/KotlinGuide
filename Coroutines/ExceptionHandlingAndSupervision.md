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

CoroutineExceptionHandler只在uncaught异常上调用(没有以任何其他方式处理的异常)。特别是所有子协程将异常处理委托给父协程，父协程也将异常处理委托给父协程，依此类推直到根协程，因此注册在它们上下文中的CoroutineExceptionHandler永远不会被使用。除此之外，async构造器总是捕获所有异常并将它们表示在返回结果的Deferred对象中，所以它的CoroutineExceptionHandler也没有效果

运行在supervision作用域中的协程不会将异常传播给父协程，因此被排除在此规则之外。文档的Supervision部分提供了更多的细节

```kotlin
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
    val job = GlobalScope.launch(handler) { // root coroutine, running in GlobalScope
        throw AssertionError()
    }
    val deferred = GlobalScope.async(handler) { // also root, but async instead of launch
        throw ArithmeticException() // Nothing will be printed, relying on user to call deferred.await()
    }
    joinAll(job, deferred)    
}
```

输出结果：

```
CoroutineExceptionHandler got java.lang.AssertionError
```

## Cancellation and exceptions
取消与异常紧密相关。协程内部使用CancellationException执行取消操作，所有处理程序都会忽略这些异常，因此它们应该仅作为额外调试信息的源使用，这些信息可以通过catch块获取。当使用Job.cancel取消协程时，协程会终止但是不会取消它的父协程

```kotlin
fun main() = runBlocking {
    val job = launch {
        val child = launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                println("Child is cancelled")
            }
        }
        yield()
        println("Cancelling child")
        child.cancel()
        child.join()
        yield()
        println("Parent is not cancelled")
    }
    job.join()    
}
```

输出结果：

```
Cancelling child
Child is cancelled
Parent is not cancelled
```

如果协程遇到除CancellationException之外的异常，它会用该异常取消它的父协程。这个行为不能被覆盖，它用于为结构化并发提供稳定的协程层级关系。CoroutineExceptionHandler的实现不被用于子协程

在这些例子中，CoroutineExceptionHandler总是被注册到GlobalScope创建的协程中。对主runBlocking作用域中启动的协程注册异常处理程序是没有意义的，因为当子协程异常完成时主协程总会被取消，尽管注册了异常处理程序

只有当所有子协程终止时，原始异常才会被父协程处理

```kotlin
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
    val job = GlobalScope.launch(handler) {
        launch { // the first child
            try {
                delay(Long.MAX_VALUE)
            } finally {
                withContext(NonCancellable) {
                    println("Children are cancelled, but exception is not handled until all children terminate")
                    delay(100)
                    println("The first child finished its non cancellable block")
                }
            }
        }
        launch { // the second child
            delay(10)
            println("Second child throws an exception")
            throw ArithmeticException()
        }
    }
    job.join()    
}
```

输出结果：

```
Second child throws an exception
Children are cancelled, but exception is not handled until all children terminate
The first child finished its non cancellable block
CoroutineExceptionHandler got java.lang.ArithmeticException
```

## Exceptions aggregation
当协程的多个子协程失败抛出异常时，通常的规则是"第一个异常获胜"，因此第一个异常会得到处理。出现在第一个异常之后的所有其它异常都作为屏蔽异常附加在第一个异常上

```kotlin
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception with suppressed ${exception.suppressed.contentToString()}")
    }
    val job = GlobalScope.launch(handler) {
        launch {
            try {
                delay(Long.MAX_VALUE) // it gets cancelled when another sibling fails with IOException
            } finally {
                throw ArithmeticException() // the second exception
            }
        }
        launch {
            delay(100)
            throw IOException() // the first exception
        }
        delay(Long.MAX_VALUE)
    }
    job.join()  
}
```

注意，上面的代码只有在支持抑制异常的JDK7+上才能正常工作

输出结果：

```
CoroutineExceptionHandler got java.io.IOException with suppressed [java.lang.ArithmeticException]
```

注意，该机制目前只适用于Java1.7+版本。JS和原生限制是临时的，将来会被取消

CancellationException是透明的，默认情况下是取消包装的

```kotlin
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception")
    }
    val job = GlobalScope.launch(handler) {
        val inner = launch { // all this stack of coroutines will get cancelled
            launch {
                launch {
                    throw IOException() // the original exception
                }
            }
        }
        try {
            inner.join()
        } catch (e: CancellationException) {
            println("Rethrowing CancellationException with original cause")
            throw e // cancellation exception is rethrown, yet the original IOException gets to the handler  
        }
    }
    job.join()    
}
```

输出结果：

```
Rethrowing CancellationException with original cause
CoroutineExceptionHandler got java.io.IOException
```

## Supervision
正如我们之前所学习的，取消是一种通过协程整个层级结构传播的双向关系。让我们看一下需要单向取消的情况

此类需求的一个很好的例子是具有在其作用域内定义Job的UI组件。如果任何UI的子任务失败，并不总是需要取消(杀掉)整个UI组件，但是如果UI组件被销毁(并且它的Job被取消)，那么有必要让所有的子任务失败，因为不再需要它们的结果

另一个例子是一个服务器进程产生了多个子Job，服务器进程需要监控Job的执行，跟踪失败Job并且重新启动它们

**Supervision job**

SupervisorJob可以用于单向取消。它类似于常规Job，唯一的区别是取消只会向下传播。通过下面的例子可以很容易的展示这一点

```kotlin
fun main() = runBlocking {
    val supervisor = SupervisorJob()
    with(CoroutineScope(coroutineContext + supervisor)) {
        // launch the first child -- its exception is ignored for this example (don't do this in practice!)
        val firstChild = launch(CoroutineExceptionHandler { _, _ ->  }) {
            println("The first child is failing")
            throw AssertionError("The first child is cancelled")
        }
        // launch the second child
        val secondChild = launch {
            firstChild.join()
            // Cancellation of the first child is not propagated to the second child
            println("The first child is cancelled: ${firstChild.isCancelled}, but the second one is still active")
            try {
                delay(Long.MAX_VALUE)
            } finally {
                // But cancellation of the supervisor is propagated
                println("The second child is cancelled because the supervisor was cancelled")
            }
        }
        // wait until the first child fails & completes
        firstChild.join()
        println("Cancelling the supervisor")
        supervisor.cancel()
        secondChild.join()
    }
}
```

输出结果：

```
The first child is failing
The first child is cancelled: true, but the second one is still active
Cancelling the supervisor
The second child is cancelled because the supervisor was cancelled
```

**Supervision scope**

代替coroutineScope，我们可以为作用域并发使用supervisorScope。它只在一个方向上传播取消，并且只在自己失败时取消所有子任务。它也像coroutineScope一样等待所有的子任务完成

```kotlin
fun main() = runBlocking {
    try {
        supervisorScope {
            val child = launch {
                try {
                    println("The child is sleeping")
                    delay(Long.MAX_VALUE)
                } finally {
                    println("The child is cancelled")
                }
            }
            // Give our child a chance to execute and print using yield 
            yield()
            println("Throwing an exception from the scope")
            throw AssertionError()
        }
    } catch(e: AssertionError) {
        println("Caught an assertion error")
    }
}
```

输出结果：

```
The child is sleeping
Throwing an exception from the scope
The child is cancelled
Caught an assertion error
```

**Exceptions in supervised coroutines**

常规Job和SupervisorJob之间的另一个关键区别是异常处理。每个子任务都应该通过异常处理机制自己处理异常。造成这种差异的原因是子任务失败不会传播给父任务。这意味着直接在supervisorScope中启动的协程可以像根协程一样使用在它们作用域内注册的CoroutineExceptionHandler

```kotlin
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
    supervisorScope {
        val child = launch(handler) {
            println("The child throws an exception")
            throw AssertionError()
        }
        println("The scope is completing")
    }
    println("The scope is completed")
}
```

输出结果：

```
The scope is completing
The child throws an exception
CoroutineExceptionHandler got java.lang.AssertionError
The scope is completed
```

