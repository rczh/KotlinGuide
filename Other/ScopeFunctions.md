# Scope Functions
kotlin标准库中包含一些函数，用来在对象的上下文中执行代码块。当你在对象上调用这些函数并且提供一个lambda表达式时，它会形成一个临时作用域。在这个作用域中你可以直接访问对象成员。这些函数称为作用域函数，包含let,run,with,apply,also

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
作用域函数本质上非常相似，每个作用域函数之间主要有两个区别

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

**this**

run, with, apply函数使用this关键字引用上下文对象。在lambda表达式中，上下文对象和在普通类函数中一样可用。通常情况下，当访问上下文对象的成员时可以省略this关键字。另一方面，如果省略this则很难区分上下文对象成员和扩展成员。对于主要操作上下文对象成员的lambda表达式，推荐使用接收类型this引用上下文对象

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

**it**

let, also函数使用lambda参数引用上下文对象。如果没有指定参数名，可以通过默认名称it来访问上下文对象。当访问上下文对象的成员时，并没有像this关键字那样隐含的对象可用。对于使用上下文对象作为函数调用参数，或者在代码块中多次使用对象时，推荐使用参数it引用上下文对象

```kotlin
fun writeToLog(message: String) {
    println("INFO: $message")
}

fun main() {
    fun getRandomInt(): Int {
        return Random.nextInt(100).also {
            writeToLog("getRandomInt() generated value $it")
        }
    }

    val i = getRandomInt()
}
```

当使用上下文对象作为参数时，可以为对象提供自定义名称

```kotlin
fun writeToLog(message: String) {
    println("INFO: $message")
}

fun main() {
    fun getRandomInt(): Int {
        return Random.nextInt(100).also { value ->
            writeToLog("getRandomInt() generated value $value")
        }
    }

    val i = getRandomInt()
}
```

### Return value
作用域函数的返回结果不同

* apply, also返回上下文对象
* let, run, with返回lambda表达式结果

**Context object**

apply, also返回上下文对象，因此他们能够被包含在调用链中。在他们之后你能继续函数链调用

```kotlin
fun main() {
    val numberList = mutableListOf<Double>()
    numberList.also { println("Populating the list") }
        .apply {
            add(2.71)
            add(3.14)
            add(1.0)
        }
        .also { println("Sorting the list") }
        .sort()
    println(numberList)
}
```

可以在返回上下文对象的函数返回声明中使用

```kotlin
fun writeToLog(message: String) {
    println("INFO: $message")
}

fun main() {
    fun getRandomInt(): Int {
        return Random.nextInt(100).also {
            writeToLog("getRandomInt() generated value $it")
        }
    }

    val i = getRandomInt()
}
```

**Lambda result**

let, run, with返回lambda表达式结果。你可以将结果赋值到变量中，或者在结果上执行链操作

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two", "three")
    val countEndsWithE = numbers.run { 
        add("four")
        add("five")
        count { it.endsWith("e") }
    }
    println("There are $countEndsWithE elements that end with e.")
}
```

你可以忽略返回值，仅为对象创建一个临时的作用域

```kotlin
fun main(){
    val numbers = mutableListOf("one", "two", "three")
    //忽略返回结果，为numbers创建临时作用域
    with(numbers) {
        val firstItem = first()
        val lastItem = last()
        println("First item: $firstItem, last item: $lastItem")
    }
}
```

## Functions



