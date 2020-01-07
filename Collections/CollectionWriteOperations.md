# Collection Write Operations
可变集合支持一些修改集合内容的操作，这里将介绍适用于所有MutableCollection子类的写操作

## 添加元素
add函数可以将一个元素添加到list或者set集合末尾

```kotlin
fun main() {
    val numbers = mutableListOf(1, 2, 3, 4)
    numbers.add(5)
    println(numbers)
}
```

addAll函数可以将集合、序列或者数组中的每个元素按照顺序添加到list或者set集合中

注意，addAll函数的接收对象类型和参数类型可以不同

```kotlin
fun main() {
    val numbers = mutableListOf(1, 2, 5, 6)
    //参数类型为数组，接收对象类型为list
    numbers.addAll(arrayOf(7, 8))
    println(numbers)
    //可以为addAll函数指定插入位置
    numbers.addAll(2, setOf(3, 4))
    println(numbers)
}
```

可以使用+=操作符添加元素

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two")
    numbers += "three"
    println(numbers)
    numbers += listOf("four", "five")    
    println(numbers)
}
```

## 删除元素
可以使用remove函数删除一个元素

```kotlin
fun main() {
    val numbers = mutableListOf(1, 2, 3, 4, 3)
    numbers.remove(3)                    // removes the first `3`
    println(numbers)
    numbers.remove(5)                    // removes nothing
    println(numbers)
}
```

注意，remove函数只能删除集合中第一次出现的元素值，对于集合中包含多个重复元素的情况可以使用以下函数：

* removeAll函数可以将参数中的所有元素一次性的从集合中删除，



```kotlin
fun main() {
    val numbers = mutableListOf(1, 2, 3, 4)
    println(numbers)
    numbers.retainAll { it >= 3 }
    println(numbers)
    numbers.clear()
    println(numbers)

    val numbersSet = mutableSetOf("one", "two", "three", "four")
    numbersSet.removeAll(setOf("one", "two"))
    println(numbersSet)
}
```







