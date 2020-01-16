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
使用filter函数可以过滤map，该函数需要传递一个参数为Pair的lambda表达式，通过Pair对象可以在lambda表达式中同时使用key和value

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
plus(+)操作符返回一个包含两个操作数元素的map，右侧操作数可以是Pair或者另一个map，如果右侧操作数元素中包含左侧操作数元素的key，返回结果中使用右侧操作数元素值代替左侧操作数元素值

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
kotlin为可变map提供了特殊的写操作，它们满足以下规则：

* map元素的值可以改变，但是key不能改变

* 对于每个key都有一个与之相关联的值，可以添加或者删除map元素

### Adding and updating entries
可以使用put函数添加一个新的元素，它被加入到LinkedHashMap的最后位置

```kotlin
fun main() {
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    numbersMap.put("three", 3)
    println(numbersMap)
}
```

可以使用putAll函数一次添加多个元素，它的参数可以是map或者Pair集合

```kotlin
fun main() {
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap.putAll(setOf("four" to 4, "five" to 5))
    println(numbersMap)
}
```

如果指定元素的key在map中已经存在，put或者putAll函数会使用指定元素的值覆盖原始map元素的值

```kotlin
fun main() {
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    val previousValue = numbersMap.put("one", 11)
    println("value associated with 'one', before: $previousValue, after: ${numbersMap["one"]}")
    println(numbersMap)
}
```

可以使用简写操作符+=或者[]添加map元素，如果指定元素的key在map中已经存在，简写操作符会使用指定元素的值覆盖原始map元素的值

```kotlin
fun main() {
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    numbersMap["three"] = 3     // calls numbersMap.put("three", 3)
    numbersMap += mapOf("four" to 4, "five" to 5)
    println(numbersMap)
}
```

### Removing entries
可以使用remove函数删除一个map元素

```kotlin
fun main() {
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap.remove("one")
    println(numbersMap)
    //如果同时指定参数key和value，remove函数只删除map中同时匹配key和value的元素
    numbersMap.remove("three", 4)            //doesn't remove anything
    println(numbersMap)
}
```

可以通过在map的keys或者values属性上调用remove函数来删除元素

```kotlin
fun main() {
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3, "threeAgain" to 3)
    numbersMap.keys.remove("one")
    println(numbersMap)
    //在values上调用remove函数时，只删除第一个具有指定值的元素
    numbersMap.values.remove(3)
    println(numbersMap)
}
```

可以使用minusAssign(-=)操作符删除map元素

```kotlin
fun main() {
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap -= "two"
    println(numbersMap)
    numbersMap -= "five"             //doesn't remove anything
    println(numbersMap)
}
```

