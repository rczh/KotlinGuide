# Retrieving Collection Parts
kotlin标准库提供了扩展函数用来获取集合中的部分元素

## Slice
slice函数返回指定下标的集合元素列表，下标可以通过区间或者整数集合指定

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")    
    println(numbers.slice(1..3))
    println(numbers.slice(0..4 step 2))
    println(numbers.slice(setOf(3, 5, 0)))    
}
```

## Take and drop
take函数用来从第一个元素开始获取指定数量的集合元素，如果指定的数量大于元素总数则返回全部集合元素。使用takeLast函数从最后一个元素开始获取指定数量的集合元素

drop函数用来获取除去从第一个元素开始指定数量的所有集合元素，使用dropLast函数获取除去从最后一个元素开始指定数量的所有集合元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.take(3))
    println(numbers.takeLast(3))
    println(numbers.drop(1))
    println(numbers.dropLast(5))
}
```

可以为take或者drop函数指定过滤条件:
 
 * takeWhile函数从第一个元素开始获取与过滤条件相匹配的集合元素。如果集合中第一个元素不匹配过滤条件，则返回结果为空
 
 * takeLastWhile函数从最后一个元素开始获取与过滤条件相匹配的集合元素，返回结果中第一个元素为集合中最后一个与过滤条件不匹配元素的下一个元素。如果集合中最后一个元素不匹配过滤条件，则返回结果为空
 
 * dropWhile函数获取从第一个不匹配过滤条件的元素位置开始到最后一个元素的所有集合元素
 
* dropLastWhile函数获取从第一个元素开始到最后一个不匹配过滤条件元素位置的所有集合元素
 
 ```kotlin
 fun main() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.takeWhile { !it.startsWith('f') })
    println(numbers.takeLastWhile { it != "three" })
    println(numbers.dropWhile { it.length == 3 })
    println(numbers.dropLastWhile { it.contains('i') })
}
 ```
 
 ## Chunked
 使用chunked函数可以将集合拆分成指定大小的子集合，它的返回结果是一个子集合列表，其中最后一个子集合的大小可能小于指定大小
 
  ```kotlin
 fun main() {
    val numbers = (0..13).toList()
    println(numbers.chunked(3))
}
 ```
 
 可以使用lambda转换函数对返回结果中的子集合进行转换，转换函数的参数为临时的子集合对象
 
```kotlin
fun main() {
    val numbers = (0..13).toList() 
    println(numbers.chunked(3) { it.sum() })  // `it` is a chunk of the original collection
}
```

## Windowed
使用windowed函数可以从每个集合元素开始返回指定大小的子集合，它类似于通过一个指定大小的滑动窗口来观测集合，返回结果是一个子集合列表

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five")    
    println(numbers.windowed(3))
}
```

可以为windowed函数设置参数

* step参数用来定义两个相邻子集合第一个元素之间的距离

* partialWindows参数用来定义是否包含从集合末尾元素开始长度小于指定大小的子集合，默认情况下partialWindows为false

* 可以使用lambda转换函数对返回结果中的子集合进行转换

```kotlin
fun main() {
    val numbers = (1..10).toList()
    println(numbers.windowed(3, step = 2, partialWindows = true))
    println(numbers.windowed(3) { it.sum() })
}
```

## zipWithNext
zipWithNext函数是windowed的一种特殊形式用来创建包含两个元素的子集合，它为除集合最后一个元素之外的每个元素创建Pair对象

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five")    
    println(numbers.zipWithNext())
    //可以为zipWithNext函数使用转换函数，转换函数的参数为两个集合元素
    println(numbers.zipWithNext() { s1, s2 -> s1.length > s2.length})
}
```

