# Collection Aggregate Operations
kotlin支持常用的聚合函数，聚合函数通常基于集合内容返回一个结果值，比如：

* min或者max函数用来返回集合中最小或者最大的元素

* average函数用来返回数字集合中元素的平均值

* sum函数用来返回数字集合中元素的总和

* count函数用来返回集合中元素的个数

```kotlin
fun main() {
    val numbers = listOf(6, 42, 10, 4)
    println("Count: ${numbers.count()}")
    println("Max: ${numbers.max()}")
    println("Min: ${numbers.min()}")
    println("Average: ${numbers.average()}")
    println("Sum: ${numbers.sum()}")
}
```

maxBy或者minBy函数接收一个参数为集合元素的lambda表达式，返回结果为lambda表达式返回的最大或者最小值的集合元素

```kotlin
fun main() {
    val numbers = listOf(5, 42, 10, 4)
    val min3Remainder = numbers.minBy { it % 3 }
    println(min3Remainder)
}
```

maxWith或者minWith函数接收一个Comparator比较器，返回结果为该比较器定义的集合中的最大或者最小元素

```kotlin
fun main() {
    val strings = listOf("one", "two", "three", "four")
    val longestString = strings.maxWith(compareBy { it.length })
    println(longestString)
}
```

sumBy或者sumByDouble函数接收一个参数为集合元素的lambda表达式，返回结果为lambda表达式在所有集合元素上结果值的总和，其中sumBy返回一个Int值，sumByDouble返回一个Double值

```kotlin
fun main() {
    val numbers = listOf(5, 42, 10, 4)
    println(numbers.sumBy { it * 2 })
    println(numbers.sumByDouble { it.toDouble() / 2 })
}
```

## Fold and reduce







