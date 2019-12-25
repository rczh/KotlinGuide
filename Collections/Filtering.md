# Filtering
kotlin支持对集合执行筛选操作，筛选操作使用lambda表达式定义筛选条件，lambda表达式使用集合元素作为参数并且返回一个布尔值，如果返回结果为true表示集合元素符合筛选条件，否则不符合

由于筛选操作不会改变原始集合，只读或者可写集合都可以执行筛选操作

## 常用筛选函数
kotlin中最基本的筛选函数是filter，它返回符合筛选条件的集合元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")  
    //对于set或者list集合，filter函数的筛选结果为list
    val longerThan3 = numbers.filter { it.length > 3 }
    println(longerThan3)

    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    //对于map集合，filter函数的筛选结果为map
    val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
    println(filteredMap)
}
```

filterIndexed函数可以在筛选lamdba表达式中同时使用下标和元素值

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val filteredIdx = numbers.filterIndexed { index, s -> (index != 0) && (s.length < 5)  }
    println(filteredIdx)
}
```

filterNot函数可以返回集合中所有不满足筛选条件的元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val filteredNot = numbers.filterNot { it.length <= 3 }
    println(filteredNot)
}
```

filterIsInstance函数可以返回集合中指定类型的元素

```kotlin
fun main() {
    val numbers = listOf(null, 1, "two", 3.0, "four")
    println("All String elements in upper case:")
    //由于filterIsInstance函数返回一个List<String>集合，这里可以直接调String的toUpperCase方法
    numbers.filterIsInstance<String>().forEach {
        println(it.toUpperCase())
    }
}
```

filterNotNull函数可以返回集合中所有非空元素

```kotlin
fun main() {
    val numbers = listOf(null, "one", "two", null)
    numbers.filterNotNull().forEach {
        //it直接作为非空元素
        println(it.length)   // length is unavailable for nullable Strings
    }
}
```

## Partitioning
partition函数可以将不满足筛选条件的元素单独保存在一个列表中，它返回一个包含两个列表的Pair对象，第一个为满足筛选条件的列表，第二个为不满足筛选条件的列表

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val (match, rest) = numbers.partition { it.length > 3 }

    println(match)
    println(rest)
}
```

## 测试筛选条件
kotlin提供一些函数用来为集合元素测试筛选条件

* any函数：如果集合中至少有一个元素满足筛选条件则返回true

* none函数：如果集合中没有元素满足筛选条件则返回true

* all函数：如果集合中所有元素都满足筛选条件则返回true。注意，在空集合上调用all函数也将返回true

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")

    println(numbers.any { it.endsWith("e") })
    println(numbers.none { it.endsWith("a") })
    println(numbers.all { it.endsWith("e") })

    println(emptyList<Int>().all { it > 5 })   // vacuous truth
}
```

通常使用不带筛选条件的any和none函数来检查集合是否为空

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val empty = emptyList<String>()
    //集合非空时any返回true
    println(numbers.any())
    //集合为空时any返回false
    println(empty.any())

    //集合非空时none返回false
    println(numbers.none())
    //集合为空时none返回true
    println(empty.none())
}
```

