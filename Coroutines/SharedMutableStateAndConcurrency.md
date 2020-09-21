# Shared mutable state and concurrency
协程可以使用多线程调度器(比如Dispatchers.Default)并发执行，它也会遇到所有常见的并发问题。主要问题是对共享变量的同步访问。协程中解决这个问题的一些方案与多线程中的解决方案类似，但是其他的解决方案是独特的

## The problem
让我们启动100协程，所有协程执行相同的操作1000。我们还将测量它们的完成时间，以便进一步比较

```kotlin
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}
```

我们从一个非常简单的操作开始，它使用多线程Dispatchers.Default递增一个共享变量

```kotlin
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
```

它最后会打印什么。打印"Counter = 100000"是不可能的，因为100个协程在没有任何同步的情况下从多个线程并发的增加counter

## Volatiles are of no help
有一种常见的误区认为将变量声明为volatile可以解决并发问题。让我们试一试

```kotlin
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

@Volatile // in Kotlin `volatile` is an annotation 
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
```

这段代码运行很慢，但我们最后仍然没有得到"Counter = 100000"。因为volatile保证了对相应变量的读写是有序的，但是不能保证操作的原子性(例如递增)

## Thread-safe data structures
对线程和协程都适用的解决方案是使用线程安全的数据结构(比如synchronized，atomic)，它为需要在共享变量上执行的相应操作提供了所有必要的同步。在计数的例子中，我们可以使用具有原子操作incrementAndGet函数的AtomicInteger类

```kotlin
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

val counter = AtomicInteger()

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.incrementAndGet()
        }
    }
    println("Counter = $counter")
}
```

这是这个问题的最快解决方案。该方案适用于普通计数，集合，队列和其他标准数据结构及其它们之上的基本操作。然而，它不容易扩展到没有可用的线程安全实现的复杂状态或复杂操作上

## Thread confinement fine-grained
线程约束是解决共享变量问题的一种方法，在这种情况下所有对共享变量的访问都被限制在单个线程中。它通常用于UI应用程序中，其中所有UI状态都被限制在一个主线程中。通过使用单线程上下文可以很容易的用于协程

```kotlin
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            // confine each increment to a single-threaded context
            withContext(counterContext) {
                counter++
            }
        }
    }
    println("Counter = $counter")
}
```

这段代码运行非常慢，因为它执行细粒度的线程约束。每次递增时都从多线程Dispatchers.Default上下文切换到使用withContext(counterContext)的单线程上下文

## Thread confinement coarse-grained
在实际中，线程约束是在大代码块中执行的。例如状态更新业务逻辑的整块代码被限制在单个线程中。下面的例子就是这样做的，它在单线程上下文中启动每个协程

```kotlin
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main() = runBlocking {
    // confine everything to a single-threaded context
    withContext(counterContext) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
```

该代码执行更快，并产生正确的结果

## Mutual exclusion
这个问题的互斥解决方案是用一个永远不会并发执行的临界区来保护共享变量的所有修改。对于阻塞的解决方案，你通常应该使用synchronized或ReentrantLock。协程的替代方案称为Mutex。它使用lock和unlock函数来分隔临界区。主要区别在于lock是一个挂起函数，它不会阻塞线程

使用withLock扩展函数，可以方便的表示mutex.lock(); try { ... } finally { mutex.unlock() }模式

```kotlin
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

val mutex = Mutex()
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            // protect each increment with lock
            mutex.withLock {
                counter++
            }
        }
    }
    println("Counter = $counter")
}
```

本例中锁是细粒度的，因此它有一些开销。然而在某些情况下它是一个很好的选择，比如你必须定期修改某些共享状态，但是没有任何此状态被限制的自然线程

## Actors
actor是由协程，被限制和封装到协程中的状态以及与其他协程通信的通道组合而成的实体。简单的actor可以编写为函数，但是具有复杂状态的actor更适合用于类

使用actor协程构建器可以方便的地将actor的接收通道组合到其作用域中用来从中接收消息，并且将发送通道组合到生成的job对象中，以便对actor的单个引用可以作为它的处理程序使用

使用actor的第一步是定义actor将要处理的消息类。Kotlin的封闭类非常适合这个目的。我们定义CounterMsg密闭类，使用IncCounter消息来增加计数器，使用GetCounter消息来获得它的值，后者需要发送一个响应。这里使用了一个CompletableDeferred通信原语来发送响应，它表示一个将被通信的单一值

```kotlin
// Message types for counterActor
sealed class CounterMsg
object IncCounter : CounterMsg() // one-way message to increment counter
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg() // a request with reply
```








