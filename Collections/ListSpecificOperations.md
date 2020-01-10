# List Specific Operations
kotlin为列表提供了一组通过索引访问元素的操作

## 获取某个元素
使用get函数或者简写形式[index]可以通过索引访问列表元素，如果索引值超出列表大小get函数将抛出异常

可以使用getOrElse或者getOrNull函数避免抛出异常

* getOrElse函数可以传递一个lambda表达式，如果索引值超出集合范围，getOrElse函数将返回lambda表达式的结果

* 如果索引值超出集合范围，getOrNull函数将返回空

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.get(0))
    println(numbers[0])
    //numbers.get(5)                         // exception!
    println(numbers.getOrNull(5))             // null
    println(numbers.getOrElse(5, {it}))        // 5
}
```

## 获取部分元素
subList函数以列表形式返回一个基于原始列表元素的视图。如果原始列表元素发生变化，subList函数的结果也会发生变化

```kotlin
fun main() {
    val numbers = (0..13).toMutableList()
    var sub = numbers.subList(3, 6)
    println(sub)
    
    numbers.set(3, 1)
    println(numbers)
     //原始列表元素发生变化，导致sub结果也发生变化
    println(sub)
}
```

## 查找元素位置
###线性查找
可以使用indexOf或者lastIndexOf函数查找列表中元素的位置，它们返回列表中指定元素参数的第一个或者最后一个位置，如果没有找到指定元素则返回-1

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4, 2, 5)
    println(numbers.indexOf(2))
    println(numbers.lastIndexOf(2))
}
```

indexOfFirst或者indexOfLast函数可以传递一个lambda表达式，它们返回列表中与lambda表达式相匹配的第一个或者最后一个元素位置

```kotlin
fun main() {
    val numbers = mutableListOf(1, 2, 3, 4)
    println(numbers.indexOfFirst { it > 2})
    println(numbers.indexOfLast { it % 2 == 1})
}
```

### 二分查找
二分查找要求列表元素按照指定顺序升序排序，它的查找速度明显优于其它普通查找

可以使用binarySearch函数进行二分查找，如果指定元素参数存在则返回该元素索引，否则返回-insertionPoint-1，其中insertionPoint值为该元素应该被插入的位置

注意，如果列表中包含多个与指定元素参数相同的元素，返回结果为任意一个元素索引

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two", "three", "four")
    numbers.sort()
    println(numbers)
    println(numbers.binarySearch("two"))  // 3
    println(numbers.binarySearch("z")) // -5
    //可以指定二分查找范围
    println(numbers.binarySearch("two", 0, 2))  // -3
}
```

#### Comparator binary search
如果列表元素没有实现Comparable接口，可以为二分查找自定义Comparator比较器，注意，列表中的元素必须按照该比较器升序排序

```kotlin
//Product类没有实现Comparable接口
data class Product(val name: String, val price: Double)

fun main() {
    //列表元素必须按照price升序排序
    val productList = listOf(
        Product("WebStorm", 49.0),
        Product("AppCode", 99.0),
        Product("DotTrace", 129.0),
        Product("ReSharper", 149.0))

    println(productList.binarySearch(Product("AppCode", 99.0), compareBy<Product> { it.price }.thenBy { it.name }))
}
```

使用自定义比较器代替原始字符串字典顺序

```kotlin
fun main() {
    val colors = listOf("Blue", "green", "ORANGE", "Red", "yellow")
    println(colors.binarySearch("RED", String.CASE_INSENSITIVE_ORDER)) // 3
}
```

#### Comparison binary search
可以为binarySearch函数指定comparison比较函数，comparison比较函数是一个参数为列表元素返回结果为int值的lambda表达式，binarySearch函数的返回结果为与comparison比较函数相匹配的元素索引

注意，列表元素必须按照comparison比较函数升序排序

```kotlin
data class Product(val name: String, val price: Double)

fun priceComparison(product: Product, price: Double) = sign(product.price - price).toInt()

fun main() {
    //列表元素必须按照price升序排序
    val productList = listOf(
        Product("WebStorm", 49.0),
        Product("AppCode", 99.0),
        Product("DotTrace", 129.0),
        Product("ReSharper", 149.0))

    println(productList.binarySearch { priceComparison(it, 99.0) })
}
```

可以对列表区间执行二分查找

### List write operations



