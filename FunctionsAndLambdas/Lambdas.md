# 高阶函数和lambda表达式
kotlin中函数是一类的，他们能够存储在变量和数据结构中，可以作为参数传递给其它高阶函数或从高阶函数返回。kotlin中能够像非函数值一样以任何方式来操作函数

## 高阶函数
高阶函数可以将函数作为参数，或者返回函数

```kotlin
fun <T, R> Collection<T>.fold(
    initial: R, 
    //combine参数的函数类型为(R, T) -> R，函数类型声明可以不写参数名
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}

fun main() {
    val items = listOf(1, 2, 3, 4, 5)

    //lambda表达式(函数文本)作为函数类型的实例
    items.fold(0, { 
        // When a lambda has parameters, they go first, followed by '->'
        acc: Int, i: Int -> 
        print("acc = $acc, i = $i, ") 
        val result = acc + i
        println("result = $result")
        // The last expression in a lambda is considered the return value:
        result
    })

    //如果编译器能够推断类型，lambda表达式可以不写类型
    val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })

    // Function references can also be used for higher-order function calls:
    val product = items.fold(1, Int::times)
}    
```

## 函数类型
kotlin使用函数类型来处理函数声明

函数类型满足以下条件
* 所有函数类型都有一个圆括号括起来的参数类型列表和一个返回类型：(A, B) -> C，参数类型列表可以为空：( ) -> A，Unit返回类型不能省略

* 函数类型可以有一个额外的接收类型：A.(B) -> C，函数文本内部可以调用接收对象中的方法  TODO:

* 挂起函数属于特殊的函数类型：suspend ( ) -> Unit

函数类型声明能够添加函数参数名称：(x: Int, y: Int) -> Int

函数类型需要注意的问题
* 函数类型可空: ((Int, Int) -> Int)?

* 函数类型可以使用括号组合: (Int) -> ((Int) -> Unit)

* 函数类型箭头符号为右结合性，(Int) -> (Int) -> Unit等价于(Int) -> ((Int) -> Unit)

可以为函数类型定义别名

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

## 实例化函数类型





