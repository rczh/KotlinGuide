# 高阶函数和lambda表达式
kotlin中函数是一类的，他们能够存储在变量和数据结构中，可以作为参数传递给其它高阶函数或从高阶函数返回。kotlin中能够像非函数值一样以任何方式来操作函数

## 高阶函数
高阶函数可以将函数作为参数，或者返回函数

```kotlin
fun <T, R> Collection<T>.fold(
    initial: R, 
    //combine参数的函数类型为(R, T) -> R，函数类型声明可以不写参数名
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}

fun main() {
    val items = listOf(1, 2, 3, 4, 5)

    //lambda表达式(函数文本)作为函数类型的实例
    items.fold(0, { 
        // When a lambda has parameters, they go first, followed by '->'
        acc: Int, i: Int -> 
        print("acc = $acc, i = $i, ") 
        val result = acc + i
        println("result = $result")
        // The last expression in a lambda is considered the return value:
        result
    })

    //如果编译器能够推断类型，lambda表达式可以不写类型
    val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })

    // Function references can also be used for higher-order function calls:
    val product = items.fold(1, Int::times)
}    
```

## 函数类型
kotlin使用函数类型来处理函数声明

函数类型满足以下条件
* 所有函数类型都有一个圆括号括起来的参数类型列表和一个返回类型：(A, B) -> C，参数类型列表可以为空：( ) -> A，Unit返回类型不能省略

* 函数类型可以有一个额外的接收类型：A.(B) -> C，函数文本内部可以调用接收对象中的方法  TODO:

* 挂起函数属于特殊的函数类型：suspend ( ) -> Unit

函数类型声明能够添加函数参数名称：(x: Int, y: Int) -> Int

函数类型需要注意的问题
* 函数类型可空: ((Int, Int) -> Int)?

* 函数类型可以使用括号组合: (Int) -> ((Int) -> Unit)

* 函数类型箭头符号为右结合性，(Int) -> (Int) -> Unit等价于(Int) -> ((Int) -> Unit)

可以为函数类型定义别名

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

## 实例化函数类型
可以使用以下方式实例化函数类型

* 函数文本形式包括lambda表达式和匿名函数
    * lambda表达式: { a, b -> a + b }
    * 匿名函数: fun(s: String): Int { return s.toIntOrNull() ?: 0 }

    TODO:
    
* 使用函数引用或者属性引用
    * 顶级函数，本地函数，成员函数或扩展函数引用: ::isOdd, String::toInt
    * 顶级属性，成员属性或扩展属性引用: List<Int>::size
    * 构造函数引用: ::Regex
    
* 使用将函数类型作为接口实现的类的实例

```kotlin
class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
}

val intFunction: (Int) -> Int = IntTransformer()
```

如果有足够的信息，编译器能够推断出函数类型

```kotlin
val a = { i: Int -> i + 1 } // The inferred type is (Int) -> Int
```

带有接收类型和不带接收类型的函数类型值可以相互替换，函数类型值(A, B) -> C和A.(B) -> C能够相互赋值

```kotlin
fun main() {
    val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
    val twoParameters: (String, Int) -> String = repeatFun // OK

    fun runTransformation(f: (String, Int) -> String): String {
        return f("hello", 3)
    }
    val result = runTransformation(repeatFun) // OK

    println("result = $result")
}
```

编译器默认推断的函数类型不带接收类型，如果需要声明带接收类型的函数类型时需要明确指定

## 调用函数类型实例
能够使用invoke操作符f.invoke(x)或者f(x)来调用函数类型实例

调用带有接收类型的函数类型实例时，接收类型对象应该作为第一个参数，或者将接收类型对象作为函数调用的前缀

```kotlin
fun main() {
    val stringPlus: (String, String) -> String = String::plus
    val intPlus: Int.(Int) -> Int = Int::plus

    println(stringPlus.invoke("<-", "->"))
    println(stringPlus("Hello, ", "world!")) 

    println(intPlus.invoke(1, 1))
    println(intPlus(1, 2))
    println(2.intPlus(3)) // extension-like call

}
```

## lambda表达式和匿名函数
lambda表达式和匿名函数称为函数文本，函数文本指没有声明的函数，这些函数可以赋值给函数类型

```kotlin
fun main() {
    fun max(str: String, f: (String, String) -> Boolean){
        
    }
    max("strings", { a, b -> a.length < b.length })
}
```

### lambda表达式定义
lambda表达式的完整语法定义形式如下。如果返回类型不是Unit，最后一条语句作为lambda表达式的返回结果

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
```

### 尾lambda表达式
如果函数的最后一个参数是函数，作为参数传送的lambda表达式可以放到圆括号外面，这种形式称为尾lambda

```kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

如果函数中只有一个函数类型参数，可以省略圆括号

```kotlin
run { println("...") }
```

### lambda表达式单个参数的隐含名称it
如果lambda表达式只有一个参数，可以省略参数声明，默认情况下编译器使用it表示该参数

```kotlin
ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```

### 从lambda表达式返回结果
默认情况下最后一个语句的值作为整个lambda表达式的返回结果，也可以使用带有限定符的return语句从lambda表达式中明确返回结果

```kotlin
ints.filter {
    val shouldFilter = it > 0 
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter
}
```

使用尾lambda表达式可以很方便的实现链式编程

```kotlin
listOf<String>("strings").filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }
```

### 使用下划线命名未使用的lambda表达式参数
如果lambda表达式中某个参数没有使用，可以使用下划线命名该参数

```kotlin
map.forEach { _, value -> println("$value!") }
```

### 匿名函数
由于lambda表达式无法明确指定函数返回类型，如果需要明确指定函数返回类型可以使用匿名函数。定义匿名函数时不需要指定函数名

```kotlin
fun(x: Int, y: Int): Int = x + y

fun(x: Int, y: Int): Int {
    return x + y
}
```

如果匿名函数的参数类型和返回值类型能够被推断出来，定义匿名函数时可以省略它们

```kotlin
ints.filter(fun(item) = item > 0)
```

当使用表达式作为匿名函数实现时匿名函数的返回值类型能够被自动推断，当使用代码块作为匿名函数实现时必须明确指定函数返回值类型

注意，匿名函数的参数只能在圆括号内传递，匿名函数不能使用尾lambda表达式传递函数参数

lambda表达式只能使用带有限定符的return语句从lambda表达式返回，匿名函数中可以直接使用return语句从匿名函数返回

匿名函数和lambda表达式可以访问外部作用域中的成员

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

### 带有接收类型的字面函数



