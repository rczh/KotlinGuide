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

扩展并不会实际修改接收类，仅使扩展函数能够在接收类上使用点符号调用

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
