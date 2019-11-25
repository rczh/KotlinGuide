# Type aliases
kotin使用类型别名来为现有的类型提供替代名称

```kotlin
class A {
    inner class Inner
}

typealias NodeSet = Set<Network.Node>
typealias MyHandler = (Int, String, Any) -> Unit
typealias AInner = A.Inner
```

类型别名并不会引入新的类型，它等价于相应的原始类型，编译时kotlin编译器将类型别名替换为相应的原始类型

```kotlin
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)
fun foo2(p: (Int) -> Boolean) = p(42)

fun main() {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // prints "true"

    var x: Predicate<Int> = { it > 0 }
    println(foo2(x))
}
```
