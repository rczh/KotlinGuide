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





