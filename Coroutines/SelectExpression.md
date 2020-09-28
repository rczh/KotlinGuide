# Select Expression (experimental)
Select表达式可以同时等待多个挂起函数，并且选择第一个可用的挂起函数

Select表达式是kotlinx.coroutines库中的一个实验特性。他们的API预计会在将来的kotlinx.coroutines库更新中进行修改，并具有很大的变化

## Selecting from channels
我们有两个字符串生产者：fizz和buzz。fizz每300ms产生一个"fizz"字符串

```kotlin
fun CoroutineScope.fizz() = produce<String> {
    while (true) { // sends "Fizz" every 300 ms
        delay(300)
        send("Fizz")
    }
}
```

buzz每500ms产生一个"Buzz"字符串

```kotlin
fun CoroutineScope.buzz() = produce<String> {
    while (true) { // sends "Buzz!" every 500 ms
        delay(500)
        send("Buzz!")
    }
}
```

使用receive挂起函数我们可以从一个通道或另一个通道接收消息。但是select表达式允许我们使用它的onReceive语句同时从两者接收消息

```kotlin
suspend fun selectFizzBuzz(fizz: ReceiveChannel<String>, buzz: ReceiveChannel<String>) {
    select<Unit> { // <Unit> means that this select expression does not produce any result 
        //操作符重载
        fizz.onReceive { value ->  // this is the first select clause
            println("fizz -> '$value'")
        }
        buzz.onReceive { value ->  // this is the second select clause
            println("buzz -> '$value'")
        }
    }
}
```

让我们运行7次selectFizzBuzz函数

```kotlin
fun main() = runBlocking<Unit> {
    val fizz = fizz()
    val buzz = buzz()
    repeat(7) {
        selectFizzBuzz(fizz, buzz)
    }
    coroutineContext.cancelChildren() // cancel fizz & buzz coroutines        
}
```

输出结果：

```
fizz -> 'Fizz'
buzz -> 'Buzz!'
fizz -> 'Fizz'
fizz -> 'Fizz'
buzz -> 'Buzz!'
fizz -> 'Fizz'
buzz -> 'Buzz!'
```

## Selecting on close
当通道关闭时select中的onReceive语句会失败，并且导致相应的select抛出异常。我们可以使用onReceiveOrNull语句在通道关闭时执行一个特定的操作。下面的例子还演示了select作为返回它的选择语句结果的表达式

```kotlin
suspend fun selectAorB(a: ReceiveChannel<String>, b: ReceiveChannel<String>): String =
    select<String> {
        a.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'a' is closed" 
            else 
                "a -> '$value'"
        }
        b.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'b' is closed"
            else    
                "b -> '$value'"
        }
    }
```

注意，onReceiveOrNull是仅为具有非空元素的通道定义的扩展函数，这样就不会在关闭通道和空值之间产生混淆

让我们将它与生成4次"Hello"字符串的通道a和生成4次"World"字符串的通道b一起使用

```kotlin
fun main() = runBlocking<Unit> {
    val a = produce<String> {
        repeat(4) { send("Hello $it") }
    }
    val b = produce<String> {
        repeat(4) { send("World $it") }
    }
    repeat(8) { // print first eight results
        println(selectAorB(a, b))
    }
    coroutineContext.cancelChildren()        
}
```

这段代码的结果非常有趣，因此我们将详细分析它

```
a -> 'Hello 0'
a -> 'Hello 1'
b -> 'World 0'
a -> 'Hello 2'
a -> 'Hello 3'
b -> 'World 1'
Channel 'a' is closed
Channel 'a' is closed
```

可以从中观察到两个现象

首先，select偏向于第一个语句。当多个语句同时可以选择时，将选择其中的第一个语句。这里，两个通道不断的产生字符串，因此select中的第一个语句通道a被选中。然而，由于我们使用的是无缓冲通道，所以通道a在调用send时会时常被挂起，从而给通道b提供了调用send的机会

第二个现象是当通道被关闭时，onReceiveOrNull会立即得到选择结果

## Selecting to send
Select表达式拥有onSend语句，它可以很好的与选择的偏向性结合使用

让我们写一个生成整数的例子，当主通道上的消费者跟不上发送速度时，生成器将值发送到side通道

```kotlin
fun CoroutineScope.produceNumbers(side: SendChannel<Int>) = produce<Int> {
    for (num in 1..10) { // produce 10 numbers from 1 to 10
        delay(100) // every 100 ms
        select<Unit> {
            onSend(num) {} // Send to the primary channel
            side.onSend(num) {} // or to the side channel     
        }
    }
}
```

消费者非常慢，需要250ms来处理每个数字

```kotlin
fun main() = runBlocking<Unit> {
    val side = Channel<Int>() // allocate side channel
    launch { // this is a very fast consumer for the side channel
        side.consumeEach { println("Side channel has $it") }
    }
    produceNumbers(side).consumeEach { 
        println("Consuming $it")
        delay(250) // let us digest the consumed number properly, do not hurry
    }
    println("Done consuming")
    coroutineContext.cancelChildren()        
}
```

输出结果：

```
Consuming 1
Side channel has 2
Side channel has 3
Consuming 4
Side channel has 5
Side channel has 6
Consuming 7
Side channel has 8
Side channel has 9
Consuming 10
Done consuming
```

## Selecting deferred values
可以使用onAwait语句选择延迟值。让我们从一个异步函数开始，它在随机delay一段时间之后返回一个延迟的字符串值

```kotlin
fun CoroutineScope.asyncString(time: Int) = async {
    delay(time.toLong())
    "Waited for $time ms"
}
```

让我们使用随机delay来创建12个延迟字符串值

```kotlin
fun CoroutineScope.asyncStringsList(): List<Deferred<String>> {
    val random = Random(3)
    return List(12) { asyncString(random.nextInt(1000)) }
}
```

现在，main函数等待其中的第一个延迟函数完成，并统计仍然处于活动状态的延迟值数量。注意，我们在这里将select表达式作为一个Kotlin领域特定语言，因此我们可以使用任意代码为它提供条件。在本例中，我们遍历一个延迟值列表，并为每个延迟值提供onAwait语句

```kotlin
fun main() = runBlocking<Unit> {
    val list = asyncStringsList()
    val result = select<String> {
        list.withIndex().forEach { (index, deferred) ->
            deferred.onAwait { answer ->
                "Deferred $index produced answer '$answer'"
            }
        }
    }
    println(result)
    val countActive = list.count { it.isActive }
    println("$countActive coroutines are still active")
}
```

输出结果：

```
Deferred 4 produced answer 'Waited for 128 ms'
11 coroutines are still active
```

## Switch over a channel of deferred values
让我们编写一个通道生产函数，它消费一个延迟字符串值的通道，它等待每个接收到的延迟值，但是仅到下一个延迟值到来或者通道关闭。这个例子将onReceiveOrNull和onAwait语句放在同一个select中

```kotlin
fun CoroutineScope.switchMapDeferreds(input: ReceiveChannel<Deferred<String>>) = produce<String> {
    var current = input.receive() // start with first received deferred value
    while (isActive) { // loop while not cancelled/closed
        val next = select<Deferred<String>?> { // return next deferred value from this select or null
            input.onReceiveOrNull { update ->
                update // replaces next value to wait
            }
            current.onAwait { value ->  
                send(value) // send value that current deferred has produced
                input.receiveOrNull() // and use the next deferred from the input channel
            }
        }
        if (next == null) {
            println("Channel was closed")
            break // out of loop
        } else {
            current = next
        }
    }
}
```

为了测试它，我们将使用一个简单的async函数，该函数在指定的时间后生成一个指定的字符串

```kotlin
fun CoroutineScope.asyncString(str: String, time: Long) = async {
    delay(time)
    str
}
```

main函数启动一个协程来打印switchMapDeferreds的结果并且发送一些测试数据

```kotlin
fun main() = runBlocking<Unit> {
    val chan = Channel<Deferred<String>>() // the channel for test
    launch { // launch printing coroutine
        for (s in switchMapDeferreds(chan)) 
        println(s) // print each received string
    }
    chan.send(asyncString("BEGIN", 100))
    delay(200) // enough time for "BEGIN" to be produced
    chan.send(asyncString("Slow", 500))
    delay(100) // not enough time to produce slow
    chan.send(asyncString("Replace", 100))
    delay(500) // give it time before the last one
    chan.send(asyncString("END", 500))
    delay(1000) // give it time to process
    chan.close() // close the channel ... 
    delay(500) // and wait some time to let it finish
}
```

输出结果：

```
BEGIN
Replace
END
Channel was closed
```


