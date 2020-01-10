# Set Specific Operations
kotlin包含一些用于set集合操作的扩展函数

可以使用union函数或者中缀形式a union b将两个集合合并为一个集合，合并结果中第一个操作数元素在第二个操作数元素之前

可以使用intersect函数或者中缀形式a intersect b返回两个集合的交集

可以使用subtract函数或者中缀形式a subtract b返回另一个集合中不存在的集合元素

```kotlin
fun main() {
    val numbers = setOf("one", "two", "three")

    println(numbers union setOf("four", "five"))
    println(setOf("four", "five") union numbers)

    println(numbers intersect setOf("two", "one"))
    println(numbers subtract setOf("three", "four"))
    println(numbers subtract setOf("four", "three")) // same output
}
```

注意，可以将set操作用于list集合，但是set操作的结果仍然为set集合，操作结果中不包含重复元素

