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

## 扩展是静态解析的

