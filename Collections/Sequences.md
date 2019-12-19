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

