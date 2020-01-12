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

