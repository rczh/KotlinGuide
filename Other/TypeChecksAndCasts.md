# Type Checks and Casts
## is Operator
可以使用is操作符检查对象是否符合指定的类型

```kotlin
fun main(){
    val obj = "str"
    if (obj is String) {
        print(obj.length)
    }
    if (obj !is String) { // same as !(obj is String)
        print("Not a String")
    }
    else {
        print(obj.length)
    }
}    
```

## Smart Casts
由于编译器会跟踪is检查并且自动插入类型转换，通常情况下kotlin中不需要使用显式的类型转换操作符

```kotlin
fun demo(x: Any) {
    if (x is String) {
        print(x.length) // x is automatically cast to String
    }
}
```

如果前面的反向is检查导致程序退出，编译器能够自动执行类型转换

```kotlin
if (x !is String) return
print(x.length) // x is automatically cast to String
```

编译器能为&&或者||右侧的操作符自动执行类型转换

```kotlin
// x is automatically cast to string on the right-hand side of `||`
if (x !is String || x.length == 0) return

// x is automatically cast to string on the right-hand side of `&&`
if (x is String && x.length > 0) {
    print(x.length) // x is automatically cast to String
}
```

编译器能为when表达式或者while循环自动执行类型转换

```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

满足以下规则可以使用自动类型转换：

* val本地变量：除了本地委托属性之外的val本地变量都可以使用自动类型转换

* val属性：除了open属性或者具有自定义get方法之外的val属性都可以使用自动类型转换

* var本地变量：除了在is检查和使用之间发生改变或者在修改它的lambda表达式中使用或者本地委托属性之外的var本地变量都可以使用自动类型转换

* var属性：由于var属性能够在任何时间被修改，所有var属性都不能使用自动类型转换

## "Unsafe" cast operator
如果无法进行转换时类型转换操作会抛出异常，通常称这种类型转换为不安全的类型转换。kotlin中使用中缀操作符as执行这种不安全的类型转换

```kotlin
//由于null不能转换成String，如果y为null则y as String会抛出异常
val x: String = y as String
```

对于y为null的情况，可以使用可空的转换类型

```kotlin
val x: String? = y as String?
```

## "Safe" (nullable) cast operator
为了避免出现异常可以使用安全类型转换操作符as?，如果转换失败as?返回null

```kotlin
//对于y为null的情况，虽然as?的转换类型为String，但是y as? String的结果为空
val x: String? = y as? String
```

## Type erasure and generic type checks
由于kotlin仅在编译时对泛型类型执行类型安全检查，在运行时泛型类型实例不包含实际参数类型信息，比如List&lt;Foo>在运行时被擦除为List&lt;*>，通常无法在运行时检查实例是否属于具有特定参数类型的泛型类型

由于泛型类型擦除，编译器禁止在运行时对泛型类型执行is类型检查，但是允许在运行时检查星型投影类型

```kotlin
//检查星型投影类型
if (something is List<*>) {
    something.forEach { println(it) } // The items are typed as `Any?`
}
```

如果在编译时已经检查了实例的泛型类型参数，运行时可以对非泛型部分使用is检查或者类型转换

```kotlin
fun handleStrings(list: List<String>) {
    //只检查非泛型部分ArrayList
    if (list is ArrayList<String>) {
        // `list` is smart-cast to `ArrayList<String>`
    }
    //只转换非泛型部分
    val tmpList = list as ArrayList<String>
}
```

带有具体化类型参数reified的内联函数允许使用is检查，比如arg is B。如果参数本身是一个泛型类型实例，它的类型信息也会被擦除

```kotlin
inline fun <reified A, reified B> Pair<*, *>.asPairOf(): Pair<A, B>? {
    if (first !is A || second !is B) return null
    return first as A to second as B
}

val somePair: Pair<Any?, Any?> = "items" to listOf(1, 2, 3)

val stringToSomething = somePair.asPairOf<String, Any>()
val stringToInt = somePair.asPairOf<String, Int>()
val stringToList = somePair.asPairOf<String, List<*>>()
//second和具体化类型参数List<String>都被擦除为List<*>，所以这里可以正常执行
val stringToStringList = somePair.asPairOf<String, List<String>>() // Breaks type safety!

fun main() {
    println("stringToSomething = " + stringToSomething)
    println("stringToInt = " + stringToInt)
    println("stringToList = " + stringToList)
    println("stringToStringList = " + stringToStringList)
}
```

## Unchecked casts
由于类型擦除使编译器无法在运行时检查泛型类型实例的实际类型参数，对泛型类型实例执行强制类型转换时编译器会产生一个类型转换不能被完全检查的警告信息

```kotlin
fun readDictionary(file: File): Map<String, *> = file.inputStream().use { 
    TODO("Read a mapping of strings to arbitrary elements.")
}

// We saved a map with `Int`s into that file
val intsFile = File("ints.dictionary")

//警告，不能保证map中的值是整型的
// Warning: Unchecked cast: `Map<String, *>` to `Map<String, Int>`
val intsDictionary: Map<String, Int> = readDictionary(intsFile) as Map<String, Int>
```

可以使用以下方式来避免未检查的类型转换警告

* 使用高级程序逻辑来代替类型转换，比如使用抽象类将类型转换放到具体实现中

* 对于泛型函数，可以使用具体化类型参数reified执行类型转换。注意，如果参数本身是一个泛型类型实例，它的类型信息也会被擦除

* 可以使用注解@Suppress("UNCHECKED_CAST")来禁止警告信息

```kotlin
inline fun <reified T> List<*>.asListOfType(): List<T>? =
    if (all { it is T })
        @Suppress("UNCHECKED_CAST")
        this as List<T> else
        null
```

由于数组类型会保留元素类型的信息，对数组执行类型转换时可以进行部分检查

```kotlin
class Foo{}
fun main(){
    //数组类型可以保留元素类型，这里编译后变成new Foo[]，保留了Foo类型
    val test :Array<Foo> = arrayOf(Foo())
}
```

如果数组中包含泛型类型元素，泛型类型元素的实际类型参数仍然被擦除

```kotlin
fun main(){
    //如果数组中包含泛型元素，这里编译后变成new List[]，丢弃了具体泛型类型Int
    val foo: Array<List<Int>> = arrayOf(listOf(1))
    //允许转换，这里不检查具体泛型类型，也不检查可空性
    val cast = foo as Array<List<String>?>
}
```

另外，对数组执行类型转换时也不会检查元素的可空性

