# Construction Collections
## 使用元素列表创建集合
通常情况下可以使用标准库函数listOf,setOf,mapOf,mutableListOf,mutableSetOf,mutableMapOf来创建集合。使用逗号分隔的集合元素列表作为参数时，编译器能够自动检测元素类型。创建空集合时需要明确指定元素类型

```kotlin
val numbersSet = setOf("one", "two", "three", "four")
val emptySet = mutableSetOf<String>()
```

map集合的key和value使用Pair对象传递，通常使用中缀函数in来创建Pair对象

```kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
```

由于使用Pair创建map集合时需要创建临时对象。为了避免创建临时对象，可以先创建一个可变的map集合，然后调用apply方法执行元素写操作

```kotlin
val numbersMap = mutableMapOf<String, String>().apply { this["one"] = "1"; this["two"] = "2" }
```

## 创建空集合
可以使用函数emptyList,emptySet,emptyMap来创建空集合，创建空集合时需要指定元素类型

```kotlin
val empty = emptyList<String>()
```

## 使用构造函数创建List集合
使用List构造函数创建集合时需要传两个参数，集合大小和初始化函数

```kotlin
fun main() {
    //初始化函数根据索引定义集合元素
    val doubled = List(3, { it * 2 })  // or MutableList if you want to change its content later
    println(doubled)
}
```

## 使用具体类型构造函数创建集合
可以使用具体类型构造函数来创建集合，比如构造函数ArrayList或者LinkedList

```kotlin
val linkedList = LinkedList<String>(listOf("one", "two", "three"))
val presizedSet = HashSet<Int>(32)
```

## 复制集合
kotlin标准库提供toList,toMutableList,toSet等集合复制函数用来创建浅复制集合，对集合中元素的更改会影响所有副本

集合复制函数可以用来将原始集合转换成其它类型集合

```kotlin
fun main() {
    val sourceList = mutableListOf(1, 2, 3)    
    val copySet = sourceList.toMutableSet()
    copySet.add(3)
    copySet.add(4)    
    println(copySet)
}
```

可以为集合创建一个新的引用，当集合元素变化时所有引用都会更新

```kotlin
fun main() {
    val sourceList = mutableListOf(1, 2, 3)
    val referenceList = sourceList
    referenceList.add(4)
    println("Source size: ${sourceList.size}")
}
```

将可写集合引用赋值给只读集合时，不能在只读集合引用上执行写操作

```kotlin
fun main() {
    val sourceList = mutableListOf(1, 2, 3)
    val referenceList: List<Int> = sourceList
    //referenceList.add(4)            //compilation error
    sourceList.add(4)
    println(referenceList) // shows the current state of sourceList
}
```

## 在集合上执行各种操作来创建新集合
在原有列表上基础上创建一个与筛选器元素匹配的新列表

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")  
    val longerThan3 = numbers.filter { it.length > 3 }
    println(longerThan3)
}
```

在原有set集合基础上创建一个map转换后的列表

```kotlin
fun main() {

    val numbers = setOf(1, 2, 3)
    println(numbers.map { it * 3 })
    println(numbers.mapIndexed { idx, value -> value * idx })
}
```
