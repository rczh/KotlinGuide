# Inline classes
内联类用来封装基本数据类型，编译器为每个内联类定义一个封装类。编译时内联类可以表示为基本数据类型或者封装类型，类似于Int可以表示为基本数据类型int或者封装类型Integer。通常情况下为了提升性能，编译器使用基本数据类型来表示内联类

注意，内联类只在Kotlin 1.3之后可用，目前处于试验阶段

## 定义
kotlin使用关键字inline定义内联类，内联类必须在主构造函数中定义一个属性

```kotlin
inline class Password(val value: String)

//在运行时不会创建Password对象，仅仅将字符串"Don't try this in production"赋值给securePassword，避免了创建对象的开销
val securePassword = Password("Don't try this in production")
```

## 成员
内联类中可以声明属性和方法。内联类中不能有init块，内联类的属性不能有幕后字段

```kotlin
inline class Name(val s: String) {
    val length: Int
        get() = s.length

    fun greet() {
        println("Hello, $s")
    }
}    

fun main() {
    val name = Name("Kotlin")
    name.greet() // method `greet` is called as a static method
    println(name.length) // property getter is called as a static method
}
```

## 继承
内联类只能继承接口。内联类不能继承其他类并且必须是final的

```kotlin
interface Printable {
    fun prettyPrint(): String
}

inline class Name(val s: String) : Printable {
    override fun prettyPrint(): String = "Let's $s!"
}    

fun main() {
    val name = Name("Kotlin")
    println(name.prettyPrint()) // Still called as a static method
}
```




