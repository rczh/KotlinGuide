# Inline classes
内联类用来封装基本数据类型，编译器为每个内联类定义一个封装类。编译时内联类可以表示为基本数据类型或者封装类型，类似于Int可以表示为基本数据类型int或者封装类型Integer。通常情况下为了提升性能(避免创建封装对象)，编译器使用基本数据类型来表示内联类

注意，内联类只在Kotlin 1.3之后可用，目前处于试验阶段

## 定义
kotlin使用关键字inline定义内联类，内联类必须在主构造函数中定义一个属性

```kotlin
inline class Password(val value: String)

//在运行时不会创建Password对象，仅仅将字符串"Don't try this in production"赋值给securePassword，避免了创建对象的开销
val securePassword = Password("Don't try this in production")
```

## 成员
内联类中可以声明属性和方法。内联类中不能有init块，内联类的属性不能有幕后字段

```kotlin
inline class Name(val s: String) {
    val length: Int
        get() = s.length

    fun greet() {
        println("Hello, $s")
    }
}    

fun main() {
    val name = Name("Kotlin")
    name.greet() // method `greet` is called as a static method
    println(name.length) // property getter is called as a static method
}
```

## 继承
内联类只能继承接口。内联类不能继承其他类并且必须是final的

```kotlin
interface Printable {
    fun prettyPrint(): String
}

inline class Name(val s: String) : Printable {
    override fun prettyPrint(): String = "Let's $s!"
}    

fun main() {
    val name = Name("Kotlin")
    println(name.prettyPrint()) // Still called as a static method
}
```

## 自动装箱
在某些情况下，内联类必须使用封装类型

```kotlin
interface Amount { val value: Int }
inline class Points(override val value: Int) : Amount

private var totalScore = 0L
fun addToScore(amount: Amount) {
    totalScore += amount.value
}

fun main() {
    repeat(1000) {
        val points = Points(it)//Points类在这是内联的，并被当做Int替换
        repeat(1000) {
            addToScore(points)//因为这里不能被传入Int，所以这里必须传入Points实例
        }
    }
}
```

addToScore方法接收Amount类型的参数，由于Int类型并不是Amount的子类型，这里不能使用Int类型做为addToScore方法的实参，编译器只能进行装箱操作将Points做为实参传给addToScore方法

使用内联类时，由于二级循环中每次都需要执行装箱操作，程序总共执行1000000次装箱操作，而使用普通类时只在一级循环中执行装箱操作，程序执行1000次装箱操作，所以在本例中使用内联类会比使用普通类执行更慢

在以下例子中，当内联类被用作另一种类型时，它们将被自动装箱

```kotlin
interface I

inline class Foo(val i: Int) : I

fun asInline(f: Foo) {}
fun <T> asGeneric(x: T) {}
fun asInterface(i: I) {}
fun asNullable(i: Foo?) {}

fun <T> id(x: T): T = x

fun main() {
    val f = Foo(42) 
    
    asInline(f)    //不需要装箱: used as Foo itself
    asGeneric(f)   //自动装箱: used as generic type T
    asInterface(f) //自动装箱: used as type I
    asNullable(f)  //自动装箱: used as Foo?, which is different from Foo
    
    // below, 'f' first is boxed (while being passed to 'id') and then unboxed (when returned from 'id') 
    // In the end, 'c' contains unboxed representation (just '42'), as 'f' 
    val c = id(f)  
}
```

注意，由于内联类能够同时表示为基本数据类型或者封装类型，不能对内联类对象使用===操作符

## 变形
使用内联类做为函数参数时可能造成函数定义冲突，这时编译器会为这些函数重命名为"functionName-&lt;hashcode>"

```kotlin
inline class UInt(val x: Int)

// Represented as 'public final void compute(int x)' on the JVM
fun compute(x: Int) { }

// Also represented as 'public final void compute(int x)' on the JVM!
fun compute(x: UInt) { }//编译时重命名为public final void compute-<hashcode>(int x)
```

注意，由于-符号在java中是非法的，不能从java中调用这些使用内联类的方法

## 内联类与类型别名的区别
内联类和类型别名都可以用来表示原始类型，内联类引入了新的封装类型，而类型别名只是引入了别名



