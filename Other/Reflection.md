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

函数引用的类型为KFunction&lt;out R>，具体取决于参数个数，比如KFunction3<T1, T2, T3, R>

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

注意，如果没有为引用扩展函数的变量指定函数类型，推断的函数类型不包含接收类型，默认情况下推断的函数类型将包含一个额外参数来传递接收类型。如果函数类型需要包含接收类型，必须明确指定

```kotlin
    //默认情况下，推断的函数类型为 (List<String>) -> Boolean
    val isEmptyStringList1: (List<String>) -> Boolean = List<String>::isEmpty
    
    //可以明确指定接收类型
    val isEmptyStringList2: List<String>.() -> Boolean = List<String>::isEmpty
```

### Example: Function Composition
compose函数返回两个函数的组合compose(f, g) = f(g(\*))，可以对compose函数的参数使用函数引用

```kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}

fun isOdd(x: Int) = x % 2 != 0

fun main() {
    fun length(s: String) = s.length

    val oddLength = compose(::isOdd, ::length)
    val strings = listOf("a", "ab", "abc")

    println(strings.filter(oddLength))
}
```

### Property References
kotlin中也可以使用::操作符访问全局变量，表达式::x返回KProperty&lt;Int>类型的属性对象，可以使用get()函数获取属性对象的值，或者使用name属性获取属性对象的属性名

```kotlin
val x = 1

fun main() {
    println(::x.get())
    println(::x.name) 
}
```

对于可变变量比如var y = 1， 表达式::y返回KMutableProperty&lt;Int>类型的属性对象

```kotlin
var y = 1

fun main() {
    ::y.set(2)
    println(y)
}
```

由于KProperty0&lt;out R>和KProperty1&lt;T, out R>属性对象分别实现了() -> R和(T) -> R函数类型接口，可以使用属性引用做为函数类型实现

```kotlin
fun main() {
    val strs = listOf("a", "bc", "def")
    //String::length的属性对象类型为KProperty1<String, Int>
    println(strs.map(String::length))
}
```

引用类属性时需要指定接收类型

```kotlin
fun main() {
    class A(val p: Int)
    //接收类型为A
    val prop = A::p
    println(prop.get(A(1)))
}
```

引用扩展属性的方式与引用类属性类似

```kotlin
val String.lastChar: Char
    get() = this[length - 1]

fun main() {
    println(String::lastChar.get("abc"))
}
```

### Interoperability With Java Reflection
kotlin标准库为反射类提供了一系列扩展，扩展提供了到java反射对象或者从java反射对象的映射。比如为kotlin属性查找java幕后字段或者java get()方法

```kotlin
import kotlin.reflect.jvm.*
class A(val p: Int)
 
fun main() {
    //javaGetter为扩展属性，返回Method对象
    println(A::p.javaGetter) // prints "public final int A.getP()"
    //javaField为扩展属性，返回Field对象
    println(A::p.javaField)  // prints "private final int A.p"
}
```

可以通过Class的扩展属性kotlin获得KClass

```kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```

### Constructor References
构造函数可以像方法和属性一样被引用，它们能够在任何需要具有与构造函数相同参数并且返回相应类型对象的函数类型对象处被使用。通过::操作符和类名来引用构造函数

```kotlin
class Foo

fun function(factory: () -> Foo) {
    val x: Foo = factory()
}
```

使用构造函数引用来调用function函数

```kotlin
function(::Foo)
```

根据参数的个数，构造函数引用的类型为KFunction&lt;out R>的子类型

## Bound Function and Property References (since 1.1)
可以引用对象中的方法

```kotlin
fun main() {
    val numberRegex = "\\d+".toRegex()
    println(numberRegex.matches("29"))

    val isNumber = numberRegex::matches
    println(isNumber("29"))
}
```

该引用被绑定到它的接收类型中，它能够被直接调用或者在需要函数类型表达式时使用

```kotlin
fun main() {
    val numberRegex = "\\d+".toRegex()
    val strings = listOf("abc", "124", "a70")
    //numberRegex::matches作为函数类型(T) -> Boolean的实例被使用
    println(strings.filter(numberRegex::matches))
}
```

比较绑定和相对应的非绑定引用类型。绑定引用类型附加了它的接收类型，所以接收类型不再是一个参数

```kotlin
//对于绑定引用类型，函数类型不包含接收类型
val isNumber: (CharSequence) -> Boolean = numberRegex::matches
//对于非绑定引用类型，函数类型包含接收类型
val matches: (Regex, CharSequence) -> Boolean = Regex::matches
```

可以引用对象中的属性

```kotlin
fun main() {
    val prop = "abc"::length
    println(prop.get())
}
```

从kotlin1.2开始，不需要明确指定this作为接收类型，this::foo和::foo相同

### Bound constructor references
可以通过外部类的对象来获取绑定内部类的构造函数引用

```kotlin
class Outer {
    inner class Inner
}

val o = Outer()
val boundInnerCtor = o::Inner
```


