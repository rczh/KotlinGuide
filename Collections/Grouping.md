# Grouping
kotlin支持对集合元素进行分组

groupBy函数需要传递一个lambda表达式然后返回一个map结果，map中的key为lambda表达式的返回结果，value为返回该lambda结果的元素列表

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five")
    //valueTransform参数表示map中的value值为valueTransform转换函数的结果，而不是原始元素
    println(numbers.groupBy { it.first().toUpperCase() })
    println(numbers.groupBy(keySelector = { it.first() }, valueTransform = { it.toUpperCase() }))
}
```

groupingBy函数返回一个Grouping对象，Grouping对象提供了一些方法可以对所有分组结果执行指定的操作

Grouping对象支持以下操作：

* eachCount函数，统计每个分组中的元素个数

* fold和reduce函数，为每个组作执行fold和reduce操作

* aggregate函数，当fold和reduce函数不能满足需求时可以使用aggregate函数实现自定义操作

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.groupingBy { it.first() }.eachCount())
}
```

