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
reduce或者fold函数接收一个包含累加值和集合元素两个参数的lambda表达式，它们对集合中的每个元素执行lambda表达式并且将返回结果作为下一次操作的累加值

fold函数接收一个初始值并且将该初始值作为第一步操作的累加值，reduce函数将集合中的前两个元素作为累加值和集合元素

```kotlin
fun main() {
    val numbers = listOf(5, 2, 10, 4)
    val sum = numbers.reduce { sum, element -> sum + element }
    println(sum)
    val sumDoubled = numbers.fold(0) { sum, element -> sum + element * 2 }
    println(sumDoubled)
    
    //由于reduce函数将第一个集合元素作为第一步操作的累加值，因此结果中不包含第一个元素的双倍
    val sumDoubledReduce = numbers.reduce { sum, element -> sum + element * 2 } //incorrect: the first element isn't doubled in the result
    println(sumDoubledReduce)
}
```

foldRight或者reduceRight函数按照逆序对集合元素执行lambda表达式操作，lambda表达式的参数为集合元素和累加值

```kotlin
fun main() {
    val numbers = listOf(5, 2, 10, 4)
    //注意，lamdba表达式的参数顺序为集合元素和累加值
    val sumDoubledRight = numbers.foldRight(0) { element, sum -> sum + element * 2 }
    println(sumDoubledRight)
}
```

reduceIndexed,foldIndexed,reduceRightIndexed,foldRightIndexed几个函数允许在lambda表达式中使用索引值

```kotlin
fun main() {
    val numbers = listOf(5, 2, 10, 4)
    val sumEven = numbers.foldIndexed(0) { idx, sum, element -> if (idx % 2 == 0) sum + element else sum }
    println(sumEven)

    val sumEvenRight = numbers.foldRightIndexed(0) { idx, element, sum -> if (idx % 2 == 0) sum + element else sum }
    println(sumEvenRight)
}
```


