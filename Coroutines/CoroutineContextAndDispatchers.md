# Coroutine Context and Dispatchers
协程总是在由kotlin标准库中定义的CoroutineContext类型值表示的某个上下文中执行

协程上下文是一组元素集合。主要元素是协程的Job和Dispatcher


## Dispatchers and threads
协程上下文包含一个协程调度器，它决定相应的协程执行时使用哪个或哪些线程。协程调度器可以将协程的执行限制在特定的线程，将其分派到线程池或让其不受限制的运行

所有协程构建器如launch和async都接受一个可选的CoroutineContext参数，它可用于明确地为新协程指定调度器和其他上下文元素

```kotlin
fun main() = runBlocking<Unit> {
    launch { // context of the parent, main runBlocking coroutine
        println("main runBlocking      : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
        println("Unconfined            : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Default) { // will get dispatched to DefaultDispatcher 
        println("Default               : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(newSingleThreadContext("MyOwnThread")) { // will get its own new thread
        println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
    }    
}
```

输出结果(可能顺序不同)：

```kotlin
Unconfined             : I'm working in thread main
Default               : I'm working in thread DefaultDispatcher-worker-1
newSingleThreadContext: I'm working in thread MyOwnThread
main runBlocking      : I'm working in thread main
```

当使用不带参数的launch函数时，它从启动它的协程作用域中继承上下文以及调度器

Dispatchers.Unconfined是一个特殊的调度器，它看起来也运行在主线程中，但实际上它是一种不同的机制

## Unconfined vs confined dispatcher
无限定协程调度器在调用者线程中启动协程，但是只到第一个挂起点。挂起之后调度器在由被调用的挂起函数决定的线程中恢复协程。无限定调度器适用于既不消耗CPU时间也不更新仅限于特定线程数据(比如UI)的协程

另一方面，默认情况下调度器从外部协程作用域中被继承。特别是runBlocking协程的默认调度器被限定在调用者线程中，因此继承它的效果是使用可预期的FIFO机制将执行限定在这个线程上


```kotlin
fun main() = runBlocking<Unit> {
    launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
        println("Unconfined      : I'm working in thread ${Thread.currentThread().name}")
        delay(500)
        println("Unconfined      : After delay in thread ${Thread.currentThread().name}")
    }
    launch { // context of the parent, main runBlocking coroutine
        println("main runBlocking: I'm working in thread ${Thread.currentThread().name}")
        delay(1000)
        println("main runBlocking: After delay in thread ${Thread.currentThread().name}")
    }    
}
```

输出结果：

```kotlin
Unconfined      : I'm working in thread main
main runBlocking: I'm working in thread main
Unconfined      : After delay in thread kotlinx.coroutines.DefaultExecutor
main runBlocking: After delay in thread main
```

因此带有继承于runBlocking上下文的协程继续在主线程中执行，而无限定协程在delay函数使用的默认执行线程中恢复执行

无限定调度器是一种高级机制，在某些特殊情况下它可能很有帮助。比如为协程稍后执行的调度器是不需要的或者会产生不良副作用，因为协程中的某些操作必须立刻执行

在常规代码中不应该使用无限定调度器

## Debugging coroutines and threads
协程可以在一个线程上挂起，然后在另一个线程上恢复。即便使用单线程调度器，可能也很难弄清协程在何时何处正在做什么。调试使用线程的应用程序的常用方法是在日志文件的每个日志语句中打印线程名。日志框架普遍支持这个特性。在使用协程时，线程名本身并不能提供太多上下文，因此kotlinx.coroutines包中提供了调试工具使打印更方便

使用-Dkotlinx.coroutines.debug虚拟机选项执行以下代码：

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking<Unit> {
    val a = async {
        log("I'm computing a piece of the answer")
        6
    }
    val b = async {
        log("I'm computing another piece of the answer")
        7
    }
    log("The answer is ${a.await() * b.await()}")    
}
```

有三个协程，runBlocking中的主协程#1和两个运算协程#2和#3。它们都在runBlocking的上下文中执行，并且被限定在主线程

输出结果：

```kotlin
[main @coroutine#2] I'm computing a piece of the answer
[main @coroutine#3] I'm computing another piece of the answer
[main @coroutine#1] The answer is 42
```

log函数在方括号中打印线程名，可以看到它是主线程后面附加了当前正在执行协程的标识符。当调试模式打开时，此标识符将连续分配给所有创建的协程

当虚拟机使用-ea选项运行时也会打开调试模式。你可以在DEBUG_PROPERTY_NAME属性的文档中阅读关于调试工具的更多信息

## Jumping between threads
使用-Dkotlinx.coroutines.debug虚拟机选项执行以下代码：

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() {
    //use会调用T.close函数释放线程
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                withContext(ctx2) {
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }    
}
```

它展示了几种新技术。一种是使用带有一个指定上下文的runBlocking函数，另一种是使用withContext来改变协程上下文的同时仍然保持在同一个协程中

输出结果：

```kotlin
[Ctx1 @coroutine#1] Started in ctx1
[Ctx2 @coroutine#1] Working in ctx2
[Ctx1 @coroutine#1] Back to ctx1
```

注意，这个例子还使用了kotlin标准库中的use函数来释放newSingleThreadContext创建的不再使用的线程

## Job in the context
协程的Job是上下文的一部分，可以使用coroutineContext[Job]表达式从上下文中获取Job

```kotlin
fun main() = runBlocking<Unit> {
    println("My job is ${coroutineContext[Job]}")    
}
```

在调试模式下，输出结果为：

```kotlin
My job is "coroutine#1":BlockingCoroutine{Active}@6d311334
```

注意，协程作用域中的isActive只是coroutineContext[Job]?.isActive == true表达式的快捷方式

## Children of a coroutine
当一个协程在另一个协程的作用域中被启动时，它通过CoroutineScope.coroutineContext来继承另一个协程的上下文，并且新协程的Job变成父协程Job的孩子。当父协程被取消时，它的所有子协程也被递归地取消

然而当使用GlobalScope启动协程时，新协程的Job没有父亲。因此新协程不与启动它的作用域绑定并且独立运行

```kotlin
fun main() = runBlocking<Unit> {
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        // it spawns two other jobs, one with GlobalScope
        GlobalScope.launch {
            println("job1: I run in GlobalScope and execute independently!")
            delay(1000)
            println("job1: I am not affected by cancellation of the request")
        }
        // and the other inherits the parent context
        launch {
            delay(100)
            println("job2: I am a child of the request coroutine")
            delay(1000)
            println("job2: I will not execute this line if my parent request is cancelled")
        }
    }
    delay(500)
    request.cancel() // cancel processing of the request
    delay(1000) // delay a second to see what happens
    println("main: Who has survived request cancellation?")
}
```

输出结果：

```kotlin
job1: I run in GlobalScope and execute independently!
job2: I am a child of the request coroutine
job1: I am not affected by cancellation of the request
main: Who has survived request cancellation?
```

## Parental responsibilities
父协程总是等待所有子协程执行完成。父协程不需要明确地跟踪它启动的所有子协程，也不需要使用Job.join在最后等待他们

```kotlin
fun main() = runBlocking<Unit> {
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        repeat(3) { i -> // launch a few children jobs
            launch  {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, 600ms
                println("Coroutine $i is done")
            }
        }
        println("request: I'm done and I don't explicitly join my children that are still active")
    }
    request.join() //这里可以不使用join
    println("Now processing of the request is complete")
}
```

输出结果：

```kotlin
request: I'm done and I don't explicitly join my children that are still active
Coroutine 0 is done
Coroutine 1 is done
Coroutine 2 is done
Now processing of the request is complete
```

## Naming coroutines for debugging
当协程经常记录日志并且你只需要关联来自同一协程的日志记录时，自动分配id是很好的。然而当协程被绑定到一个特定请求的执行或者执行某些特定后台任务时，为了便于调试最好明确地命名它。CoroutineName上下文元素的作用与线程名相同。当打开调试模式时，它被包含在执行此协程的线程名中

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking(CoroutineName("main")) {
    log("Started main coroutine")
    // run two background value computations
    val v1 = async(CoroutineName("v1coroutine")) {
        delay(500)
        log("Computing v1")
        252
    }
    val v2 = async(CoroutineName("v2coroutine")) {
        delay(1000)
        log("Computing v2")
        6
    }
    log("The answer for v1 / v2 = ${v1.await() / v2.await()}")    
}
```

使用-Dkotlinx.coroutines.debug虚拟机选项的输出结果：

```kotlin
[main @main#1] Started main coroutine
[main @v1coroutine#2] Computing v1
[main @v2coroutine#3] Computing v2
[main @main#1] The answer for v1 / v2 = 42
```

## Combining context elements


