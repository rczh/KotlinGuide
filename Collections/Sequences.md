# 序列
kotlin支持序列，序列提供与集合类似的功能，并且实现了另一种多步集合处理的方式

当一个集合对象的执行包含多个操作步骤时，这些操作会被尽快执行。每一个操作执行后会返回一个中间结果，下面的操作在这个中间结果上继续执行。然而，序列的多个操作步骤处理会尽可能延迟，只有当请求整个处理结果时才真正执行

集合和序列对于多个操作步骤处理的执行顺序也不同，集合为所有元素依次执行每一个操作步骤，而序列为每一个元素逐个执行所有的操作步骤

因为序列不会生成中间结果，它提升了整个多步处理的性能。另外由于序列的延迟特性增加了一些开销，这些开销在处理较小集合或执行简单计算时可能会很大，所以需要根据实际情况选择使用集合或者序列

## 构造序列
### 使用元素列表
可以使用sequenceOf函数并且将元素列表作为参数来创建序列

```kotlin
val numbersSequence = sequenceOf("four", "three", "two", "one")
```

### 使用集合
可以使用asSequence函数在原有集合基础上创建序列

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
使用sequence函数可以逐个或按任意大小生成序列元素，sequence函数需要传一个调用yield或者yieldAll函数的lambda表达式，它们为序列消费者返回一个元素然后挂起sequence函数的执行，当序列消费者再次请求序列元素时sequence函数将会继续执行

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

序列可以遍历多次，某些序列的实现会限制只能遍历一次

## 例子
### 使用集合执行多步操作

```kotlin
fun main() {    
    val words = "The quick brown fox jumps over the lazy dog".split(" ")
    val lengthsList = words.filter { println("filter: $it"); it.length > 3 }
        .map { println("length: ${it.length}"); it.length }
        .take(4)

    println("Lengths of first 4 words longer than 3 chars:")
    println(lengthsList)
}
```

集合操作会被尽快执行，集合为所有元素依次执行每一个操作步骤

![list-processing.png](https://github.com/rczh/KotlinGuide/blob/master/Collections/list-processing.png)

### 使用序列执行多步操作

```kotlin
fun main() {
    val words = "The quick brown fox jumps over the lazy dog".split(" ")
    //convert the List to a Sequence
    val wordsSequence = words.asSequence()

    val lengthsSequence = wordsSequence.filter { println("filter: $it"); it.length > 3 }
        .map { println("length: ${it.length}"); it.length }
        .take(4)

    println("Lengths of first 4 words longer than 3 chars")
    // terminal operation: obtaining the result as a List
    println(lengthsSequence.toList())
}
```

只有当请求结果时才真正执行序列操作，序列为每一个元素逐个执行所有操作步骤

![sequence-processing.png](https://github.com/rczh/KotlinGuide/blob/master/Collections/sequence-processing.png)

在本例中，序列操作需要执行18步，而集合操作执行23步


