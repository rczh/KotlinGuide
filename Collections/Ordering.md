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














