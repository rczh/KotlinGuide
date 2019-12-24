# Collection Transformations
kotlin标准库为集合转换提供了一系列扩展函数，这些扩展函数根据转换规则从原始集合生成新的集合

## Mapping
Mapping装换通过在原始集合元素上执行转换函数，将转换函数的结果用来创建新集合

基本的Mapping装换使用map函数，它对每一个集合元素执行转换函数并且返回转换函数执行结果的列表。map函数返回结果的顺序和元素的原始顺序相同

```kotlin
fun main() {
    val numbers = setOf(1, 2, 3)
    println(numbers.map { it * 3 })
    //mapIndexed函数可以在lambda表达式中使用元素的索引
    println(numbers.mapIndexed { idx, value -> value * idx })
}
```

使用mapNotNull或者mapIndexedNotNull函数可以将装换结果中的空值过滤掉

```kotlin
fun main() {
    val numbers = setOf(1, 2, 3)
    println(numbers.mapNotNull { if ( it == 2) null else it * 3 })
    println(numbers.mapIndexedNotNull { idx, value -> if (idx == 0) null else value * idx })
}
```

对map进行转换时，可以使用mapKeys函数只转换key保留value不变，或者使用mapValues函数只转换value保留key不变

```kotlin
fun main() {
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    println(numbersMap.mapKeys { it.key.toUpperCase() })
    println(numbersMap.mapValues { it.value + it.key.length })
}
```

## Zipping
Zipping转换使用两个集合中相同位置的元素来创建Pair

kotlin使用zip函数进行Zipping转换，zip函数返回Pair对象列表，其中接收集合的元素作为Pair对象的第一个元素。如果两个集合的长度不同，返回列表长度为较小集合元素长度，较大集合中超出的元素不会包含在返回结果中

```kotlin
fun main() {
    val colors = listOf("red", "brown", "grey")
    val animals = listOf("fox", "bear", "wolf")
    //zip函数可以使用中缀形式
    println(colors zip animals)

    val twoAnimals = listOf("fox", "bear")
    println(colors.zip(twoAnimals))
}
```

可以使用带有转换函数的zip函数，zip函数的结果为转换函数返回结果的列表

```kotlin
fun main() {
    val colors = listOf("red", "brown", "grey")
    val animals = listOf("fox", "bear", "wolf")

    println(colors.zip(animals) { color, animal -> "The ${animal.capitalize()} is $color"})
}
```

可以使用unzip函数将Pair列表拆分成两个单独的列表

```kotlin
fun main() {
    val numberPairs = listOf("one" to 1, "two" to 2, "three" to 3, "four" to 4)
    println(numberPairs.unzip())
}
```

## Association
Association转换允许使用集合元素和与它们相关联的值来创建map

associateWith函数使用原始集合元素作为key，转换函数的结果作为value来创建map。如果原始集合中有两个相同元素，map中只保留最后一个元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.associateWith { it.length })
}
```

associateBy函数使用原始集合元素作为value，转换函数的结果作为key来创建map。如果原始集合中有两个相同元素，map中只保留最后一个元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")

    println(numbers.associateBy { it.first().toUpperCase() })
    //可以为associateBy函数使用值转换函数
    println(numbers.associateBy(keySelector = { it.first().toUpperCase() }, valueTransform = { it.length }))
}
```

associate函数可以使用原始集合元素同时生成key和value，装换函数返回一个包含key和value的Pair对象

注意，由于associate函数会产生临时的Pair对象，它会影响函数的性能

```kotlin
data class FullName (val firstName: String, val lastName: String)
fun parseFullName(fullName: String): FullName {
    val nameParts = fullName.split(" ")
    if (nameParts.size == 2) {
        return FullName(nameParts[0], nameParts[1])
    } else throw Exception("Wrong name format")
}

fun main(){
    val names = listOf("Alice Adams", "Brian Brown", "Clara Campbell")
    //associate函数会生成临时Pair对象，它会影响性能
    println(names.associate { name -> parseFullName(name).let { it.lastName to it.firstName } })
}
```

## Flattening








