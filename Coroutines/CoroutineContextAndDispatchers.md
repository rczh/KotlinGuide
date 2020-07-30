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

在常规代码中不应该使用无限顶调度器

## Debugging coroutines and threads











































