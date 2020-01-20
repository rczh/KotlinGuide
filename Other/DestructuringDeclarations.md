# Destructuring Declarations
析构声明用来将一个对象分解成多个变量

```kotlin
data class Person(var name: String, var age: Int)
fun main(){
    //声明两个新的变量
    val (name, age) = Person("name", 20)
    println(name)
    println(age)
}    
```

析构声明被编译为以下形式，实际上任何对象只要提供了相应数量的component函数都可以使用析构声明

```kotlin
val name = person.component1()
val age = person.component2()
```

注意，component函数需要使用operator关键字声明

可以在for循环中使用析构声明

```kotlin
fun main(){
    val col = listOf(Person("name", 20))
    //变量a和b调用Person的component1和component2函数获取值
    for ((a, b) in col) {
        println(a)
        println(b)
    }
}
```

## Example: Returning Two Values from a Function
对于需要从函数中返回两个变量的情况，可以声明一个包含两个变量的数据类，并且从函数中返回该数据类的实例。由于数据类中自动声明了component函数，可以在函数调用时使用析构声明

```kotlin
data class Result(val result: Int, val status: Status)
fun function(...): Result {
    //这里也可以直接使用Pair
    return Result(result, status)
}

fun main(){
    // Now, to use this function:
    val (result, status) = function(...)
}
```

## Example: Destructuring Declarations and Maps
由于kotlin标准库为map提供了iterator和component扩展函数，因此可以在遍历map的过程中使用析构声明

```kotlin
fun main(){
    var map = mapOf("key" to "value")
    for ((key, value) in map) {
        println(key)
        println(value)
    }
}
```

## Underscore for unused variables (since 1.1)
可以使用下划线命名析构声明中不使用的变量，kotlin不会调用该变量对应的component函数

```kotlin
val (_, status) = print(status)
```

## Destructuring in Lambdas (since 1.1)
如果lambda表达式参数为具有相应component函数的对象，可以为该参数使用析构声明

```kotlin
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" }
```

需要注意，lambda表达式中声明两个参数和使用析构声明之间的区别

```kotlin
{ a -> ... } // one parameter
{ a, b -> ... } // two parameters
{ (a, b) -> ... } // a destructured pair
{ (a, b), c -> ... } // a destructured pair and another parameter
```

可以在lambda表达式参数中使用下划线命名析构声明不使用的变量

```kotlin
map.mapValues { (_, value) -> "$value!" }
```

可以为整个析构声明或者析构声明中的变量指定类型

```kotlin
map.mapValues { (_, value): Map.Entry<Int, String> -> "$value!" }
map.mapValues { (_, value: String) -> "$value!" }
```
