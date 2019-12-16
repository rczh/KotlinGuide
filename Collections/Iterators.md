# Iterators

kotlin支持使用迭代器iterator遍历集合元素，可以调用Iterable接口的iterator方法获取迭代器对象，默认迭代器对象指向集合的第一个元素，调用next方法返回该元素并且将迭代器指针移动到下一个元素

如果迭代器已经遍历到最后一个元素，想要再次遍历集合需要创建一个新的迭代器

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val numbersIterator = numbers.iterator()
    while (numbersIterator.hasNext()) {
        println(numbersIterator.next())
    }
}
```

for循环内部自动使用迭代器遍历集合

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    for (item in numbers) {
        println(item)
    }
}
```

使用forEach函数自动遍历集合，并且在每个集合元素上执行lambda表达式

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    numbers.forEach {
        println(it)
    }
}
```

## List迭代器
kotlin为List集合提供了ListIterator迭代器，它能够同时支持向前和向后遍历集合

ListIterator迭代器使用hasPrevious和previous方法向后遍历集合，它还提供nextIndex和previousIndex方法用来获取元素下标

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val listIterator = numbers.listIterator()
    while (listIterator.hasNext()) listIterator.next()
    println("Iterating backwards:")
    while (listIterator.hasPrevious()) {
        print("Index: ${listIterator.previousIndex()}")
        println(", value: ${listIterator.previous()}")
    }
}
```

由于ListIterator迭代器支持双向遍历，当遍历到最后一个元素时仍然能够使用ListIterator迭代器向后遍历

## Mutable迭代器
kotlin为可变集合提供了MutableIterator迭代器，它提供了remove方法可以在迭代过程中删除集合元素

```kotlin
    val numbers = mutableListOf("one", "two", "three", "four")
    val mutableIterator = numbers.iterator()

    while (mutableIterator.hasNext()) {
        val item = mutableIterator.next()
        if(item == "two"){
            mutableIterator.remove()
            mutableIterator.next()
            continue
        }
    }
    println("After removal: $numbers")
```

MutableListIterator迭代器继承于MutableIterator，并且提供了add和set方法

```kotlin
fun main() {
    val numbers = mutableListOf("one", "four", "four") 
    val mutableListIterator = numbers.listIterator()

    mutableListIterator.next()
    mutableListIterator.add("two")
    mutableListIterator.next()
    mutableListIterator.set("three")   
    println(numbers)
}
```

