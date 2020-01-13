# Map Specific Operations
kotlin标准库提供了一系列map处理函数

## Retrieving keys and values
可以使用get函数或者简写形式[key]从map中获取值，如果指定的key不存在它的返回结果为null

使用getValue函数从map中获取值时，如果指定的key不存在它将抛出异常。可以使用getOrElse或者getOrDefault函数来避免抛出异常

* getOrElse函数可以传递一个lambda表达式，如果指定的key不存在，getOrElse函数将返回lambda表达式的结果

* 如果指定的key不存在，getOrDefault函数将返回一个默认值

```kotlin
fun main() {
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap.get("one"))
    println(numbersMap["one"])
    println(numbersMap.getOrDefault("four", 10))
    println(numbersMap["five"])               // null
    //numbersMap.getValue("six")      // exception!
}
```

可以使用属性keys或者values获取map中的所有key和value集合

```kotlin
fun main() {
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    //keys为一个set集合
    println(numbersMap.keys)
    //values为一个Collection
    println(numbersMap.values)
}
```

## Filtering
使用filter函数可以过滤map，该函数需要传递一个参数为Pair的lambda表达式，通过Pair可以在lambda表达式中同时使用key和value

```kotlin
fun main() {
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    //返回一个新map
    val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
    println(filteredMap)
}
```

可以使用filterKeys或者filterValues函数通过keys或者values过滤map

```kotlin
fun main() {
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    //返回一个新map
    val filteredKeysMap = numbersMap.filterKeys { it.endsWith("1") }
    val filteredValuesMap = numbersMap.filterValues { it < 10 }

    println(filteredKeysMap)
    println(filteredValuesMap)
}
```

## plus and minus operators
plus(+)操作符返回一个包含两个操作数元素的map，右侧操作数可以是Pair或者另一个map，如果右侧操作数元素中包含左侧操作数元素的key，返回结果中使用右侧操作数元素

```kotlin
fun main() {
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap + Pair("four", 4))
    //返回结果中使用{"one",10}代替{"one", 1}
    println(numbersMap + Pair("one", 10))
    println(numbersMap + mapOf("five" to 5, "one" to 11))
}
```

minus(-)操作符返回左侧操作数map中删除右侧操作数key后的map结果，右侧操作数可以是一个key或者key的集合

```kotlin
fun main() {
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    //返回一个新map
    println(numbersMap - "one")
    println(numbersMap - listOf("two", "four"))
}
```

## Map write operations





