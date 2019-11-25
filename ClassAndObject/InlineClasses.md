# Inline classes
内联类用来封装基本数据类型，编译器为每个内联类定义一个封装类。编译时内联类可以表示为基本数据类型或者封装类型，类似于Int可以表示为基本数据类型int或者封装类型Integer。通常情况下为了提升性能，编译器使用基本数据类型来表示内联类

注意，内联类只在Kotlin 1.3之后可用，目前处于试验阶段

## 定义
kotlin使用关键字inline定义内联类，内联类必须在主构造函数中初始化一个属性

```kotlin
interface Printable {
    fun prettyPrint(): String
}

inline class Name(val s: String) : Printable{
    val length: Int
        get() = s.length

    fun greet() {
        println("Hello, $s")
    }

    override fun prettyPrint(): String = "Let's $s!"
}
```
