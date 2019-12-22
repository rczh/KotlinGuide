# 序列
kotlin支持序列，序列提供与集合类似的功能，并且实现了另一种多步集合处理的方式

当一个集合对象的处理包含多个步骤时，这些步骤会被尽快执行。每一个步骤执行后会返回一个中间集合，下面的步骤在这个中间集合上继续执行。然而，序列的多步处理会尽可能延迟，只有当请求整个处理结果时才真正执行

集合和序列对于多步处理的执行顺序也不同，集合为所有元素依次执行每一个处理步骤，而序列为每一个元素逐个执行所有的处理步骤

因为序列不会生成中间结果，它提升了整个多步处理的性能。由于序列的延迟特性增加了一些开销，这些开销在处理较小集合或执行简单计算时可能会很大，需要根据实际情况选择使用集合或者序列

## 构造序列
### 使用元素列表
可以使用sequenceOf函数并且将元素列表作为参数来创建序列

```kotlin
val numbersSequence = sequenceOf("four", "three", "two", "one")
```

### 使用集合
可以使用asSequence函数在原有集合上创建序列

```kotlin
val numbers = listOf("one", "two", "three", "four")
val numbersSequence = numbers.asSequence()
```

### 使用生成函数
可以使用generateSequence函数并且传一个lambda表达式来创建序列，lambda表达式用来计算每一个序列元素。当lambda表达式返回null时停止生成序列元素

```kotlin
fun main() {
    //由于lambda表达式没有返回null，oddNumbers序列是无限的
    val oddNumbers = generateSequence(1) { it + 2 } // `it` is the previous element
    println(oddNumbers.take(5).toList())
    //println(oddNumbers.count())     // error: the sequence is infinite
}
```

使用generateSequence函数创建有限序列

```kotlin
fun main() {
    val oddNumbersLessThan10 = generateSequence(1) { if (it < 10) it + 2 else null }
    println(oddNumbersLessThan10.count())
}
```

### 使用块函数
使用sequence函数可以逐个或按任意大小的块生成序列元素，sequence函数需要传一个调用yield或者yieldAll函数的lambda表达式，它们为序列消费者返回一个元素然后挂起sequence函数的执行，当序列消费者再次请求序列元素时sequence函数将会继续执行

使用无限序列作为参数的yieldAll函数必须位于lambda表达式的最后位置

```kotlin
fun main() {
    val oddNumbers = sequence {
        yield(1)
        yieldAll(listOf(3, 5))
        yieldAll(generateSequence(7) { it + 2 })
    }
    println(oddNumbers.take(5).toList())
}
```

## 序列操作
序列操作可以分为无状态操作和有状态操作

* 无状态操作不需要任何状态并且独立的处理每个元素，比如map或filter函数

* 有状态操作需要保存状态。sort为有状态操作，相同的元素在排序后会保持原有顺序

当一个序列操作返回另一个序列时该序列操作称为中间操作，否则称为终点操作。序列元素只能通过终点操作获取

序列可以遍历多次，某些序列实现会限制只能遍历一次














