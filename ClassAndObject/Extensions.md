# Extensions
kotlin提供了使用新成员扩展类的能力，而不必继承类或者使用设计模式

扩展包括扩展函数和扩展属性，扩展的函数或者属性可以用来以通常的方式调用，就好像它们是原始类的函数或属性一样

## 扩展函数
声明扩展函数时需要在函数名前加一个被扩展的接收类型

```kotlin
//泛型参数需要在扩展函数名之前定义
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    //扩展函数内部的this代表接收对象
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}

val list = mutableListOf(1, 2, 3)
list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'
```

通常直接在包下定义top-level扩展函数，在定义扩展函数声明包的外部使用扩展函数时需要导入它

注意，扩展并不会实际修改接收类，仅使扩展函数能够在接收类上使用点符号调用

## 静态解析扩展函数
扩展函数是静态调用的，它不支持按照接收对象多态调用

```kotlin
fun main() {
    open class Shape

    class Rectangle: Shape()

    fun Shape.getName() = "Shape"

    fun Rectangle.getName() = "Rectangle"

    fun printClassName(s: Shape) {
        println(s.getName())
    }    

    printClassName(Rectangle())
}
```

如果定义的扩展函数和接收类型中成员函数有相同的函数名和参数，则优先调用成员函数

```kotlin
fun main() {
    class Example {
        fun printFunctionType() { println("Class method") }
    }

    fun Example.printFunctionType() { println("Extension function") }

    Example().printFunctionType()
}
```

可以使用可空的接收类型定义扩展函数，并且在函数体中使用this==null检查接收对象是否为空，从而实现在不检查接收对象是否为空的情况下调用扩展函数

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // after the null check, 'this' is autocast to a non-null type, so the toString() below
    // resolves to the member function of the Any class
    return toString()
}
```

## 扩展属性
kotlin支持扩展属性。由于扩展并不会实际修改接收类，导致扩展属性不能拥有幕后字段。因为没有幕后字段，扩展属性不能使用初始化值，只能使用set,get方法

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```    

## 伴生对象扩展
伴生对象的扩展函数调用能够直接使用接收类名作为前缀

```kotlin
class MyClass {
    companion object { }  // will be called "Companion"
}

fun MyClass.Companion.printCompanion() { println("companion") }

fun main() {
    MyClass.printCompanion()
}
```

## 扩展函数作为类成员
可以在类中为另一个类声明扩展函数，在这样的扩展函数内部有两个隐含的接收类型对象

* 分发接收对象：声明扩展的类的实例

* 扩展接收对象：扩展方法的接收类型的实例

```kotlin
class Host(val hostname: String) {
    fun printHostname() { print(hostname) }
}

class Connection(val host: Host, val port: Int) {
     fun printPort() { print(port) }

     fun Host.printConnectionString(p: Int) {
         printHostname()   //调用扩展接收对象Host.printHostname方法
         print(":")
         printPort()   //调用分发接收对象Connection.printPort方法
     }

     fun connect() {
         /*...*/
         host.printConnectionString(port)   // calls the extension function
     }
}

fun main() {
    Connection(Host("kotl.in"), 443).connect()
    //Host("kotl.in").printConnectionString(443)  // error, the extension function is unavailable outside Connection
}
```

当扩展函数中调用的函数名称在分发接收对象和扩展接收对象中冲突时，优先使用扩展接收对象中的函数。可以通过"this@分发接收类型"明确指定使用分发接收对象中的函数

```kotlin
class Connection {
    fun Host.getConnectionString() {
        toString()         // calls Host.toString()
        this@Connection.toString()  // calls Connection.toString()
    }
}
```

类中声明的扩展函数可以被子类重写，也就是说扩展函数在分发接收对象上的继承是多态的，但是在扩展接收对象上的调用是静态的

```kotlin
open class Base { }

class Derived : Base() { }

open class BaseCaller {
    open fun Base.printFunctionInfo() {
        println("Base extension function in BaseCaller")
    }

    open fun Derived.printFunctionInfo() {
        println("Derived extension function in BaseCaller")
    }

    fun call(b: Base) {
        b.printFunctionInfo()   // call the extension function
    }
}

class DerivedCaller: BaseCaller() {
    override fun Base.printFunctionInfo() {
        println("Base extension function in DerivedCaller")
    }

    override fun Derived.printFunctionInfo() {
        println("Derived extension function in DerivedCaller")
    }
}

fun main() {
    BaseCaller().call(Base())   // "Base extension function in BaseCaller"
    DerivedCaller().call(Base())  // "Base extension function in DerivedCaller" - dispatch receiver is resolved virtually
    DerivedCaller().call(Derived())  // "Base extension function in DerivedCaller" - extension receiver is resolved statically
}
```

## 扩展的可见性
扩展函数的可见性与在相同作用域内声明的常规函数的可见性相同

如果扩展函数在它的接收类外部声明，则扩展函数不能访问接收类的私有成员



