# Retrieving Single Elements
kotlin提供了一组从集合中获取单个元素的函数

列表是一个有序集合，列表中每个元素都有自己的下标。set本身是一个无序集合，但是set集合的具体实现使用特定的顺序来存储元素，比如LinkedHashSet使用插入顺序来存储元素，所以set集合中仍然可以使用根据位置获取元素的函数

## Retrieving by position
可以使用elementAt函数获取指定位置的集合元素，elementAt函数常用于不提供通过下标访问元素的集合，比如set

```kotlin
fun main() {
    val numbers = linkedSetOf("one", "two", "three", "four", "five")
    println(numbers.elementAt(3))    

    val numbersSortedSet = sortedSetOf("one", "two", "three", "four")
    println(numbersSortedSet.elementAt(0)) // elements are stored in the ascending order
}
```

elementAtOrNull或者elementAtOrElse函数可以在获取不存在位置元素时避免出现异常

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five")
    //当指定位置不存在时elementAtOrNull函数返回null
    println(numbers.elementAtOrNull(5))
    //当指定位置不存在时elementAtOrElse函数返回该位置上lambda表达式的结果
    println(numbers.elementAtOrElse(5) { index -> "The value for index $index is undefined"})
}
```

可以使用first或者last函数用来获取集合中的第一个或者最后一个元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five")
    println(numbers.first())    
    println(numbers.last())    
}
```

## Retrieving by condition
可以为first或者last函数指定过滤条件，返回结果为集合中第一个或者最后一个与过滤条件相匹配的元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.first { it.length > 3 })
    println(numbers.last { it.startsWith("f") })
}
```

注意，当集合中没有任何元素能够匹配过滤条件时，first或者last函数将抛出异常。可以使用firstOrNull或者lastOrNull函数来避免出现异常

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    //集合中没有匹配过滤条件的元素时firstOrNull函数返回null
    println(numbers.firstOrNull { it.length > 6 })
}
```

可以使用别名函数find或者findLast来代替firstOrNull或者lastOrNull

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4)
    //find函数内部调用firstOrNull函数
    println(numbers.find { it % 2 == 0 })
    //findLast函数内部调用lastOrNull函数
    println(numbers.findLast { it % 2 == 0 })
}
```

## Random element
可以使用random函数获取集合中的任意一个元素

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.random())
}
```

## Checking existence
contains函数用来检查集合中是否存在指定的元素，可以使用in关键字来调用contains函数

可以使用containsAll函数检查集合中是否存在多个元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.contains("four"))
    println("zero" in numbers)

    println(numbers.containsAll(listOf("four", "two")))
    println(numbers.containsAll(listOf("one", "zero")))
}
```

可以使用isEmpty或者isNotEmpty函数检查集合是否为空

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.isEmpty())
    println(numbers.isNotEmpty())

    val empty = emptyList<String>()
    println(empty.isEmpty())
    println(empty.isNotEmpty())
}
```

