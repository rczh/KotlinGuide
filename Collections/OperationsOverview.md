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

某些操作可以指定一个可变的目标集合，







