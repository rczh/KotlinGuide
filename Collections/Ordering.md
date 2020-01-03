# Collection Ordering
kotlin中可以使用Comparable或者Comparator来定义对象的顺序

## Comparable
通常情况下使用Comparable接口定义对象的顺序，许多内置类型已经实现了Comparable接口，比如Int,Float,Char,String

可以为自定义类实现Comparable接口，compareTo函数使用另一个相同类型对象作为参数并且返回一个Int结果:

* 正值表示接收对象大

* 负值表示接收对象小

* 零表示两个对象相等

```kotlin
class Version(val major: Int, val minor: Int): Comparable<Version> {
    override fun compareTo(other: Version): Int {
        if (this.major != other.major) {
            return this.major - other.major
        } else if (this.minor != other.minor) {
            return this.minor - other.minor
        } else return 0
    }
}

fun main() {    
    println(Version(1, 2) > Version(1, 3))
    println(Version(2, 0) > Version(1, 5))
}
```

## Comparator
对于不提供Comparable实现的对象，或者需要使用自定义比较器来替换原有Comparable实现的对象，可以使用Comparator接口，compare函数使用两个相同类型对象作为参数并且返回一个Int结果

```kotlin
fun main() {
    val lengthComparator = Comparator { str1: String, str2: String -> str1.length - str2.length }
    //这里使用字符串长度对列表进行排序，不是使用字母顺序
    println(listOf("aaa", "bb", "c").sortedWith(lengthComparator))
}
```

可以使用compareBy函数来创建Comparator实例，lambda表达式的参数为集合元素对象，返回结果为Comparable

```kotlin
fun main() {
    println(listOf("aaa", "bb", "c").sortedWith(compareBy { it.length }))
}
```

## 排序函数
kotlin提供了一些函数用来对集合进行排序，这里将介绍一些用于只读集合的排序函数，这些函数的返回结果是一个按照指定顺序排列原始集合元素的新集合

### Natural order
sorted或者sortedDescending函数返回按照Comparable接口定义的升序或者降序排序的集合元素

注意，sorted或者sortedDescending函数只能对实现了Comparable接口的集合对象排序

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")

    println("Sorted ascending: ${numbers.sorted()}")
    println("Sorted descending: ${numbers.sortedDescending()}")
}
```

### Custom orders
对于不提供Comparable实现的对象，或者需要使用自定义比较器来替换原有Comparable实现的对象，可以使用Comparator接口

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    println("Sorted by length ascending: ${numbers.sortedWith(compareBy { it.length })}")
}
```

可以使用别名函数sortedBy或者sortedByDescending来代替sortedWith(compareBy)或者sortedWith(compareByDescending)函数

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")

    val sortedNumbers = numbers.sortedBy { it.length }
    println("Sorted by length ascending: $sortedNumbers")
    val sortedByLast = numbers.sortedByDescending { it.last() }
    println("Sorted by the last letter descending: $sortedByLast")
}
```


