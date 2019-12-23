# Collection Operations Overview
kotlin标准库为集合操作提供了一系列函数，它们以集合的成员函数或者扩展函数的形式定义

* 使用成员函数的形式为集合定义最基本的操作，比如add,get

* 使用扩展函数的形式为集合定义更加复杂的操作，比如filter,map

## 通用操作
通用操作可用于只读和可变集合，通用操作分为以下组：

* Transformations

* Filtering

* plus and minus operators

* Grouping

* Retrieving collection parts

* Retrieving single elements

* Ordering

* Aggregate operations

这些通用操作的执行不会影响原始集合，它们会返回一个新的结果集合

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")  
    numbers.filter { it.length > 3 }  // nothing happens with `numbers`, result is lost
    println("numbers are still $numbers")
    //通常将通用操作结果保存在变量中
    val longerThan3 = numbers.filter { it.length > 3 } // result is stored in `longerThan3`
    println("numbers longer than 3 chars are $longerThan3")
}
```

标准库提供以To为后缀名的通用操作，这些操作可以指定一个可变的目标集合，操作结果将被加入到这个目标集合中

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val filterResults = mutableListOf<String>()  //destination object
    numbers.filterTo(filterResults) { it.length > 3 }
    numbers.filterIndexedTo(filterResults) { index, _ -> index == 0 }
    println(filterResults) // contains results of both operations
}
```

这些以To为后缀名的通用操作能够返回目标集合

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    // filter numbers right into a new hash set, 
    // thus eliminating duplicates in the result
    val result = numbers.mapTo(HashSet()) { it.length }
    println("distinct item lengths are $result")
}
```

## 写操作
对于可变集合，可以使用改变集合状态的写操作。比如add,remove

标准库为某些操作提供一对函数，比如排序操作，sort函数对原始集合进行排序，而sorted函数创建一个新的集合保存排序结果

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two", "three", "four")
    val sortedNumbers = numbers.sorted()
    println(numbers == sortedNumbers)  // false
    numbers.sort()
    println(numbers == sortedNumbers)  // true
}
```


