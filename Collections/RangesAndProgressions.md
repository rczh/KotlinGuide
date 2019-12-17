# 区间和数列
kotlin允许使用rangeTo函数或者它的操作符形式..来创建区间，通常情况下区间会和in,!in函数一起使用

```kotlin
//if条件中in操作符执行contains方法
if (i in 1..4) {  // equivalent of 1 <= i && i <= 4
    print(i)
}
```

整数类型区间包括IntRange, LongRange, CharRange，他们能够进行迭代

```kotlin
fun main() {
    //for循环中in操作符使用迭代器
    for (i in 1..4) print(i)
}
```

注意，由于区间继承于数列，这些区间也是相应整数类型的数列

可以使用downTo函数以逆序方式遍历区间

```kotlin
fun main() {
    for (i in 4 downTo 1) print(i)
}
```

可以使用任意步长遍历区间

```kotlin
fun main() {
    for (i in 1..8 step 2) print(i)
    println()
    for (i in 8 downTo 1 step 2) print(i)
}
```

可以使用until函数遍历不包含结尾元素的区间

```kotlin
fun main() {
    for (i in 1 until 10) {       // i in [1, 10), 10 is excluded
        print(i)
    }
}
```

## 区间
区间用来表示数学上的闭区间，它定义了两个端点值。只能为可比较的类型定义区间，由于区间是有顺序的，可以定义是否一个实例处于两个区间实例之间

可以使用rangeTo函数或者..操作符为自定义类创建区间，但是自定义类需要实现Comparable接口

```kotlin
class Version(val major: Int, val minor: Int): Comparable<Version> {
    override fun compareTo(other: Version): Int {
        if (this.major != other.major) {
            return this.major - other.major
        }
        return this.minor - other.minor
    }
}

fun main() {
    val versionRange = Version(1, 11)..Version(1, 30)
    println(Version(0, 9) in versionRange)
    println(Version(1, 20) in versionRange)
}
```

## 数列




