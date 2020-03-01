# Reflection
反射是一组语言和库提供的功能，用来在运行时获取程序的结构

kotlin为使用反射功能提供了单独的运行组件kotlin-reflect.jar，目的是为了减小不使用反射功能的应用程序所需运行时库的大小。如果使用反射功能需要将kotlin-reflect.jar文件添加到项目的classpath中

## Class References
最基本的反射功能是获取kotlin类的运行时引用

```kotlin
//引用类型为KClass
val c = MyClass::class
```

注意，kotlin类引用与java类引用不同，可以使用KClass实例的java属性来获取java类引用

## Bound Class References (since 1.1)
可以通过对象实例获取指定类的引用

```kotlin
val widget: Widget = ...
//尽管widget的类型为Widget，widget::class的返回结果为GoodWidget类的引用
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

## Callable references
对函数、属性或者构造函数的引用能够做为函数类型的实例被调用或者使用

所有可调用引用的公共父类为KCallable<out R>，其中R为返回类型值，它可以是属性的属性类型，或者是构造函数的构造类型

### Function References
可以使用::操作符将函数声明做为函数类型值传递给其他函数

```kotlin
fun isOdd(x: Int) = x % 2 != 0

fun main() {
    val numbers = listOf(1, 2, 3)
    //这里::isOdd是函数类型(Int) -> Boolean的值
    println(numbers.filter(::isOdd))
}
```

函数引用的类型为KFunction<out R>，具体取决于参数个数，比如KFunction3<T1, T2, T3, R>

如果能从上下文中判断函数类型，可以对重载函数使用::操作符

```kotlin
fun main() {
    fun isOdd(x: Int) = x % 2 != 0
    fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"

    val numbers = listOf(1, 2, 3)
    println(numbers.filter(::isOdd)) // refers to isOdd(x: Int)
}
```

或者，可以将函数引用保存在具有明确函数类型的变量中

```kotlin
fun main() {
    fun isOdd(x: Int) = x % 2 != 0
    fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"

    val predicate: (String) -> Boolean = ::isOdd   // refers to isOdd(x: String)
}
```

引用类的成员函数或者扩展函数时需要指定类型，比如String::toCharArray

注意，如果没有为引用扩展函数的变量指定函数类型，推断的函数类型不带接收类型，默认情况下函数类型将包含一个额外参数来传递接收类型。如果函数类型需要包含接收类型，必须明确指定

```kotlin
    //默认情况下，推断的函数类型为 (List<String>) -> Boolean
    val isEmptyStringList1: (List<String>) -> Boolean = List<String>::isEmpty
    
    //可以明确指定接收类型
    val isEmptyStringList2: List<String>.() -> Boolean = List<String>::isEmpty
```

### Example: Function Composition




