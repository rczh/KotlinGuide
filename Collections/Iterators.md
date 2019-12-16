# Iterators

kotlin支持使用迭代器iterator遍历集合元素，可以调用Iterable接口的iterator方法获取迭代器对象，默认迭代器对象指向集合的第一个元素，调用next方法返回当前元素并且将迭代器指针移动到下一个元素

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

使用forEach函数遍历集合

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    numbers.forEach {
        println(it)
    }
}
```

## List迭代器
kotlin为List集合提供了特殊的迭代器接口ListIterator，它同时支持向前和向后迭代List集合

ListIterator迭代器使用hasPrevious和previous方法实现向后迭代，使用nextIndex和previousIndex方法获取元素下标

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

由于ListIterator支持双向迭代，当遍历到最后一个元素时，仍然能够使用它向后迭代

## Mutable迭代器
kotlin为可变集合提供了迭代器MutableIterator，它提供了remove方法可以在迭代过程中删除元素

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

MutableListIterator继承于MutableIterator迭代器接口并且提供了add和set方法

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

