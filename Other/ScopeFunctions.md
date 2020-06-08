# Scope Functions
kotlin标准库中包含一些函数，用来在对象的上下文中执行代码块。当你在对象上调用这个函数并且提供一个lambda表达式时，它会形成一个临时作用域。在这个作用域中你可以直接访问对象。这些函数称为作用域函数，包含let,run,with,apply,also

本质上，这些函数的功能都是在对象上执行一个代码块。不同的是这个对象如何在代码块中可用，以及整个表达式的结果

```kotlin
data class Person(var name: String, var age: Int, var city: String) {
    fun moveTo(newCity: String) { city = newCity }
    fun incrementAge() { age++ }
}

fun main() {
    Person("Alice", 20, "Amsterdam").let {
        println(it)
        it.moveTo("London")
        it.incrementAge()
        println(it)
    }
}
```

作用域函数没有引入任何新的技术，但是它们可以使代码更加简洁和可读

## Distinctions
作用域函数本质上非常相似，每个作用域函数之间有两个主要区别

* 对于上下文对象的引用方式
* 返回结果

### Context object: this or it
在作用域函数的lambda表达式内部，通过引用来访问上下文对象而不是使用名字。有两种方式访问上下文对象：lambda表达式接收类型this, lambda表达式参数it

```kotlin
fun main() {
    val str = "Hello"
    // this
    str.run {
        println("The receiver string length: $length")
        //println("The receiver string length: ${this.length}") // does the same
    }

    // it
    str.let {
        println("The receiver string's length is ${it.length}")
    }
}
```

#### this
run, with, apply函数使用lambda表达式接收类型this引用上下文对象。在lambda表达式中，上下文对象像在普通类函数中一样可用。通常情况下，当访问接收对象的成员时可以省略this使代码更简短。另一方面，如果省略this则很难区分接收对象的成员和扩展对象或者函数。因此推荐对主要操作上下文对象成员的lambda使用上下文接收对象this

```kotlin
data class Person(var name: String, var age: Int = 0, var city: String = "")

fun main() {
    val adam = Person("Adam").apply { 
        age = 20                       // same as this.age = 20 or adam.age = 20
        city = "London"
    }
    println(adam)
}
```
