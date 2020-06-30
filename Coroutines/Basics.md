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
上面的例子在同一代码中混合了非阻塞函数delay和阻塞函数Thread.sleep。这样很容易忘记哪个是阻塞的，哪个是非阻塞的。让我们使用协程构建器runBlocking来定义代码块

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

