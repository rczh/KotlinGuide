# Coroutine Basics
本节介绍协程的基本概念

## Your first coroutine

```kotlin
fun main() {
    GlobalScope.launch { //在后台启动一个新的协程
        delay(1000L) //非阻塞延迟1s
        println("World!")
    }
    println("Hello,") //当协程被延迟时，主线程继续执行
    Thread.sleep(2000L) //阻塞主线程2s
}
```

本质上，协程是轻量级线程。它们由协程构建器launch在一些协程作用域的上下文中启动。这里我们在全局作用域中启动一个新的协程，这意味着新协程的生命周期仅受整个应用程序生命周期的限制

你可以使用thread替换GlobalScope.launch，使用Thread.sleep替换delay来获得同样的结果

如果你仅使用thread替换GlobalScope.launch，编译器将产生以下错误：

Error: Kotlin: Suspend functions are only allowed to be called from a coroutine or another suspend function

这是因为delay是一种特殊的挂起函数，它不会阻塞线程，但会挂起协程，并且只能从协程中使用它

## Bridging blocking and non-blocking worlds
上面的例子在同一代码中混合了非阻塞函数delay和阻塞函数Thread.sleep。这样很容易忘记哪个是阻塞的，哪个是非阻塞的。让我们使用协程构建器runBlocking来明确一下阻塞

```kotlin
fun main() { 
    GlobalScope.launch { //在后台启动一个新的协程
        delay(1000L)
        println("World!")
    }
    println("Hello,") //主线程执行
    runBlocking {     //runBlocking函数会阻塞主线程
        delay(2000L)  //延迟2s
    } 
}
```

结果是相同的，但是这段代码只使用了非阻塞函数delay。执行runBlocking的主线程被阻塞，直到runBlocking内的协程完成为止

这个例子可以用更加惯用的方式重写，使用runBlocking封装主函数的执行

```kotlin
fun main() = runBlocking<Unit> { //启动主协程
    GlobalScope.launch { //在后台启动一个新的协程
        delay(1000L)
        println("World!")
    }
    println("Hello,") //主协程中执行
    delay(2000L)      //延迟2s
}
```

这里runBlocking作为一个适配器用来启动顶级主协程。我们明确指定它的Unit返回类型，因为Kotlin中的主函数必须返回Unit

这也是为挂起函数编写单元测试的一种方法

```kotlin
class MyTest {
    @Test
    fun testMySuspendingFunction() = runBlocking<Unit> {
        // here we can use suspending functions using any assertion style that we like
    }
}
```

## Waiting for a job
在另一个协程工作时延迟一段时间不是一个好的方法。让我们以非阻塞的方式明确地等待，直到启动的后台作业完成

```kotlin
fun main() = runBlocking {
    val job = GlobalScope.launch { //启动一个新协程并且引用它的job
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() //挂起协程，等待子协程完成
}
```

结果是相同的，但是主协程的代码没有以任何方式与后台作业的持续时间绑定

## Structured concurrency
对于协程的实际使用还有一些需要改进的地方。当使用GlobalScope.launch时，我们创建了一个顶级协程。即便它是轻量级的，它在运行时仍然会消耗一些内存资源。如果我们忘记保存对新启动协程的引用，它一直运行。如果协程中的代码挂起怎么办(比如我们错误地延迟了太长时间)，如果我们启动了太多的协程并耗尽了内存怎么办。手动保存对所有启动协程的引用并且join它们很容易出错

有一个好的解决办法，我们可以在代码中使用结构化并发。我们可以在正在执行操作的特定作用域内启动协程，而不是像通常使用线程那样在全局作用域内启动协程

在上面的例子中，我们使用runBlocking协程构建器将main函数转换为协程。每个协程构建器(包括runBlocking)都将一个协程作用域的实例添加到其代码块的作用域中。我们可以在此作用域中启动协程而不必明确地join它们，因为外部协程会等到在其作用域中启动的所有子协程完成后才会完成

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { //在runBlocking作用域中启动一个新协程
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

## Scope builder
除了由不同构建器提供的协程作用域外，还可以使用coroutineScope构建器声明自己的作用域。它创建一个协程作用域并且等到所有启动的子协程完成后才会完成

runBlocking和coroutineScope看起来很相似，因为它们都会等待它们的函数体以及所有子协程完成。主要区别在于runBlocking方法会阻塞当前线程，而coroutineScope只挂起并且释放底层线程以用于其他用途。由于这种差异，runBlocking是一个常规函数，而coroutineScope是一个挂起函数

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { 
        delay(200L)
        println("Task from runBlocking")
    }
    
    coroutineScope { // Creates a coroutine scope
        launch {
            delay(500L) 
            println("Task from nested launch")
        }
    
        delay(100L)
        println("Task from coroutine scope") // This line will be printed before the nested launch
    }
    
    println("Coroutine scope is over") // This line is not printed until the nested launch completes
}
```

注意，即使coroutineScope尚未完成，也会执行并打印"Task from runBlocking"消息

## Extract function refactoring
让我们提取launch内部的代码块到一个独立的函数。在此代码上执行重构的提取函数时，会得到一个带有suspend关键字的新函数。这是一个挂起函数。挂起函数可以像常规函数一样在协程中使用，但是它们的附加功能是它们可以反过来使用其他挂起函数(如delay)来挂起协程的执行

```kotlin
fun main() = runBlocking {
    launch { doWorld() }
    println("Hello,")
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

但是如果提取函数包含在当前作用域上被调用的协程构建器会怎么样。这种情况下提取函数上的suspend关键字是不够的。使doWorld成为CoroutineScope的扩展函数是一种解决方案，但它可能不总是适用，因为它不能使API更清晰。通常的解决方案是在包含目标函数的类中明确的使用CoroutineScope成员变量，或者在外部类实现CoroutineScope时使用隐含的成员变量。作为最后一种方案，可以使用CoroutineScope(coroutineContext)，但是这种方法是结构上不安全的，因为你不再控制此方法的执行作用域

## Coroutines ARE light-weight
下面的代码启动十万个协程，一秒后每个协程打印一个点

```kotlin
fun main() = runBlocking {
    repeat(100_000) { // launch a lot of coroutines
        launch {
            delay(1000L)
            print(".")
        }
    }
}
```

如果使用线程代码很可能产生OOM错误

## Global coroutines are like daemon threads
下面的代码在全局作用域中启动一个长时间运行的协程，它每秒打印两次"I'm sleeping"，然后经过一段时间的延迟后从主函数返回

```kotlin
fun main() = runBlocking {
    GlobalScope.launch {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // just quit after delay    
}
```

你可以运行并看到它打印三行然后终止了

在全局作用域中启动的活动协程不会使进程保持活动状态。它们类似于守护线程
