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
有时我们需要为协程上下文定义多个元素，我们可以使用+运算符。例如，我们可以同时使用一个明确指定的调度器和一个明确指定的名称来启动协程

```kotlin
fun main() = runBlocking<Unit> {
    launch(Dispatchers.Default + CoroutineName("test")) {
        println("I'm working in thread ${Thread.currentThread().name}")
    }    
}
```

使用-Dkotlinx.coroutines.debug虚拟机选项的输出结果：

```kotlin
I'm working in thread DefaultDispatcher-worker-1 @test#2
```

## Coroutine scope
让我们把上下文、孩子和Job的知识放在一起。假设我们的应用程序有一个具有生命周期的对象，但该对象不是协程。例如，我们正在写一个Android应用程序，并且在一个Activity的上下文中启动各种协程来执行异步操作去获取和更新数据，执行动画。当Activity被销毁时必须取消所有协程以避免内存泄漏。当然，我们可以手动操作上下文和Job以绑定Activity和协程的生命周期，但是kotlinx.coroutines包提供了对其进行封装的抽象：CoroutineScope。你应该已经熟悉协程作用域，因为所有协程构建器都被声明为它的扩展

我们通过创建一个与Activity生命周期相关联的CoroutineScope实例来管理协程的生命周期。CoroutineScope实例可以通过CoroutineScope()或MainScope()工厂函数创建。前者创建一个通用作用域，后者为UI应用程序创建一个作用域并且使用Dispatchers.Main作为默认的调度器

```kotlin
class Activity {
    private val mainScope = MainScope()
    
    fun destroy() {
        mainScope.cancel()
    }
    // to be continued ...
```

现在，我们可以使用定义的mainScope在Activity的作用域内启动协程

```kotlin
// class Activity continues
    fun doSomething() {
        // launch ten coroutines for a demo, each working for a different time
        repeat(10) { i ->
            mainScope.launch {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, ... etc
                println("Coroutine $i is done")
            }
        }
    }
} // class Activity ends
```

我们在主函数中创建Activity，调用测试doSomething函数，并在500ms后销毁Activity。这取消了所有从doSomething中启动的协程。我们可以看到在Activity被销毁之后，即使我们再等一会儿也没有更多的消息被打印

```kotlin
fun main() = runBlocking<Unit> {
    val activity = Activity()
    activity.doSomething() // run test function
    println("Launched coroutines")
    delay(500L) // delay for half a second
    println("Destroying activity!")
    activity.destroy() // cancels all coroutines
    delay(1000) // visually confirm that they don't work    
}
```

输出结果：

```kotlin
Launched coroutines
Coroutine 0 is done
Coroutine 1 is done
Destroying activity!
```

可以看到，只有前两个协程打印消息，其他协程通过调用Activity.destroy中的job.cancel被取消

注意，Android在带有生命周期的所有实体中有对协程作用域的官方支持，请看相应的文档

## Thread-local data
有时能够在协程之间传递线程本地数据是很方便的。由于它们没有绑定到任何特定的线程，如果手动执行可能会导致问题

对于ThreadLocal，可以使用asContextElement扩展函数进行补救。它创建了一个额外的上下文元素，该元素保存给定的ThreadLocal值，并且在协程每次切换上下文时恢复它

```kotlin
val threadLocal = ThreadLocal<String>() // declare thread-local variable

**
 * 基于协程的线程本地变量
 */
fun main() = runBlocking<Unit> {
    threadLocal.set("main")
    println("Pre-main, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
    //由于不能确定协程在哪个线程中运行，这里将threadLocal和value作为一个元素传给上下文，由上下文决定何时设置value
    //updateThreadContext
    val job = launch(Dispatchers.Default + threadLocal.asContextElement(value = "launch")) {
        println("Launch start, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
        //restoreThreadContext
        yield()
        //如果不使用asContextElement，当线程发生变化时，threadLocal.get返回值可能不确定
        //updateThreadContext
        println("After yield, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
        //restoreThreadContext
    }
    job.join()
    println("Post-main, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")    
}
```

在本例中，我们使用Dispatchers.Default在后台线程池中启动一个新的协程，它在线程池的另一个线程上运行，无论协程在哪个线程上运行，它仍然有我们使用threadLocal.asContextElement(value = "launch")指定的线程本地变量值

输出结果：

```kotlin
Pre-main, current thread: Thread[main @coroutine#1,5,main], thread local value: 'main'
Launch start, current thread: Thread[DefaultDispatcher-worker-1 @coroutine#2,5,main], thread local value: 'launch'
After yield, current thread: Thread[DefaultDispatcher-worker-2 @coroutine#2,5,main], thread local value: 'launch'
Post-main, current thread: Thread[main @coroutine#1,5,main], thread local value: 'main'
```

由于很容易忘记设置相应的上下文元素。如果运行协程的线程不同，从协程访问线程本地变量可能会有一个不确定值。为了避免这种情况，建议使用ensurepresentation方法在不正确的用法时快速失败

ThreadLocal具有良好的支持，可以与任何kotlinx.coroutines包提供的原生协程一起使用。不过，它有一个关键的限制。当线程本地变量值发生变化时，新值不会传播到协程调用者(因为一个上下文元素不能跟踪所有ThreadLocal值的设置)，它在下一次挂起时会丢失。使用withContext在协程中更新线程本地变量的值，请看asContextElement了解更多细节

或者可以将变量保存在一个类中，比如class Counter(var i: Int)，然后将它存储在线程本地变量中。但是在这种情况下，你要完全负责去同步类中变量可能的并发修改

对于线程本地变量的高级用法，请看ThreadContextElement接口的文档

