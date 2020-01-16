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

