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
### let
使用参数it引用上下文对象，返回值为lambda表达式结果

let可以在调用链的结果上执行一个或多个函数

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two", "three", "four", "five")
    numbers.map { it.length }.filter { it > 3 }.let { 
        println(it)
        // and more function calls if needed
    } 
}
```

如果代码块只包含一个将it作为参数的函数，可以使用函数引用代替lambda表达式

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two", "three", "four", "five")
    numbers.map { it.length }.filter { it > 3 }.let(::println)
}
```

let通常用于执行具有非空上下文对象的代码块

```kotlin
fun processNonNullString(str: String) {}

fun main() {
    val str: String? = "Hello"   
    //processNonNullString(str) // compilation error: str can be null
    val length = str?.let { 
        println("let() called on $it")
        //str可空，但是?.let { }块中it非空
        processNonNullString(it)    // OK: 'it' is not null inside '?.let { }'
        it.length
    }
}
```

可以将let结果赋值给局部变量来提高代码的可读性，可以为上下文对象自定义参数名称

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val modifiedFirstItem = numbers.first().let { firstItem ->
        println("The first item of the list is '$firstItem'")
        if (firstItem.length >= 5) firstItem else "!" + firstItem + "!"
    }.toUpperCase()
    println("First item after modifications: '$modifiedFirstItem'")
}
```

### with
上下文对象作为参数传递给with，在lambda表达式内部使用接收类型this引用上下文对象，返回值为lambda表达式结果

建议在不提供lambda结果的情况下，调用上下文对象上的函数时使用with

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two", "three")
    with(numbers) {
        println("'with' is called with argument $this")
        println("It contains $size elements")
    }
}
```

另一个用例是将接收类型对象作为帮助对象，它的属性或函数将用于计算结果

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two", "three")
    val firstAndLast = with(numbers) {
        "The first element is ${first()}," +
        " the last element is ${last()}"
    }
    println(firstAndLast)
}
```

### run
使用接收类型this引用上下文对象，返回值为lambda表达式结果

当lambda表达式中包含对象初始化和返回值计算时可以使用run

```kotlin
class MultiportService(var url: String, var port: Int) {
    fun prepareRequest(): String = "Default request"
    fun query(request: String): String = "Result for query '$request'"
}

fun main() {
    val service = MultiportService("https://example.kotlinlang.org", 80)
    val result = service.run {
        port = 8080
        query(prepareRequest() + " to port $port")
    }
    println(result)
}
```

还可以使用非扩展的run，它允许在需要表达式的地方执行多个语句块

```kotlin
fun main() {
    //这里的run为非扩展函数
    val hexNumberRegex = run {
        val digits = "0-9"
        val hexDigits = "A-Fa-f"
        val sign = "+-"

        Regex("[$sign]?[$digits$hexDigits]+")
    }

    for (match in hexNumberRegex.findAll("+1234 -FFFF not-a-number")) {
        println(match.value)
    }
}
```

### apply
使用接收类型this引用上下文对象，返回值为上下文对象

对于不返回值并且主要对接收类型对象的成员进行操作时可以使用apply。它通常的使用情况是对象配置

```kotlin
data class Person(var name: String, var age: Int = 0, var city: String = "")

fun main() {
    val adam = Person("Adam").apply {
        age = 32
        city = "London"        
    }
    println(adam)
}
```

由于apply返回上下文对象，可以将apply包含到调用链中从而进行复杂的操作

### also
使用参数it引用上下文对象，返回值为上下文对象

当执行一些以上下文对象作为参数的操作时可以使用also。对于需要引用上下文对象而不是它的属性或函数的操作，或者不想屏蔽来自外部作用域的this引用时，可以使用also

```kotlin
fun main() {
    val numbers = mutableListOf("one", "two", "three")
    numbers
        .also { println("The list elements before adding new one: $it") }
        .add("four")
}
```

## Function selection
下面的表格描述了作用域函数之间的区别

| 函数 | 对象引用 | 返回值 | 扩展函数 |
| ------------ | ------------- | ------------- | ------------- |
let | it | lambda表达式结果 | 是
run | this | lambda表达式结果 | 是
run | - | lambda表达式结果 | 否，调用时没有上下文对象
with | this | lambda表达式结果 | 否，将上下文对象作为参数
apply | this | 上下文对象 | 是
also | it | 上下文对象 | 是

以下是根据预期选择作用域函数的简短指南

* 在非空对象上执行lambda表达式：let

* 将表达式结果赋值给局部变量：let

* 对象配置：apply

* 对象配置和计算结果：run

* 在需要表达式的地方执行语句：非扩展run

* 附加效果：also

* 分组对象上的函数调用：with

不同作用域函数的用例相互重叠，可以根据项目或团队中使用的特定约定来选择作用域函数

避免过度使用作用域函数，它会降低代码的可读性并导致错误。避免嵌套作用域函数，在链接它们时要当心，因为很容易将上下文对象的this或it值混淆

## takeIf and takeUnless
除了作用域函数之外，标准库还包含takeIf和takeUnless函数，它们允许你在调用链中嵌入对象状态的检查

当在提供匹配条件的对象上调用时，如果匹配则takeIf返回该对象，不匹配则返回空。因此，takeIf是针对对象的过滤函数。相反，如果不匹配则takeUnless返回该对象，匹配则返回空

通过lambda表达式参数it引用对象

```kotlin
fun main() {
    val number = Random.nextInt(100)
        
    val evenOrNull = number.takeIf { it % 2 == 0 }
    val oddOrNull = number.takeUnless { it % 2 == 0 }
    println("even: $evenOrNull, odd: $oddOrNull")
}
```

当在takeIf和takeUnless后面链接其他函数时，不要忘记执行空检查或使用安全调用(?.)，因为它们的返回值可能为空

```kotlin
fun main() {
    val str = "Hello"
    val caps = str.takeIf { it.isNotEmpty() }?.toUpperCase()
    //val caps = str.takeIf { it.isNotEmpty() }.toUpperCase() //compilation error
    println(caps)
}
```

takeIf和takeUnless与作用域函数一起特别有用，一个很好的例子是将它们与let链接起来，以便在匹配指定条件的对象上执行代码块。对于不匹配条件的对象，takeIf返回空，let不会执行

```kotlin
fun main() {
    fun displaySubstringPosition(input: String, sub: String) {
        input.indexOf(sub).takeIf { it >= 0 }?.let {
            println("The substring $sub is found in $input.")
            println("Its start position is $it.")
        }
    }

    displaySubstringPosition("010000011", "11")
    displaySubstringPosition("010000011", "12")
}
```
