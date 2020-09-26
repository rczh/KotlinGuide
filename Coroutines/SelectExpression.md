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




