# ControlFlow
## if表达式
在kotlin中if是一个带有返回值的表达式，由于if表达式完全能够代替三元操作符，kotlin中不提供三元操作符

```kotlin
var max: Int
//作为语句
if (a > b) {
    max = a
} else {
    max = b
}

//作为表达式
val max = if (a > b) a else b
```

if表达式可以用块来表示，块中最后一个表达式的值就是块的值

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

注意：当使用if作为表达式为变量赋值时需要提供else分支

## when表达式
when表达式用来代替java中的switch语句。when表达式按顺序比较所有分支，直到某个分支条件与参数相匹配

使用when表达式为变量赋值时需要提供else分支

```kotlin
var result = when (x) {
        1 -> print("x == 1")
        2 -> print("x == 2")
        else -> { // Note the block
            print("x is neither 1 nor 2")
        }
    }
```

当多个分支条件使用相同的处理语句时，可以将多个条件用逗号分隔

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

可以使用表达式作为分支条件

```kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

可以使用检查值是否在某个范围，或者集合中的in语句作为分支条件

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    //validNumbers为集合
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

可以使用检查值是否为某个特定类型的is语句作为分支条件，kotlin能够直接访问相应类型的方法或属性而不需要进行类型转换

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

when表达式可以用来代替if-else-if语句链，当when语句不带参数时，每个分支条件都应该是一个布尔表达式。when表达式顺序比较所有分支条件，当某个分支条件为true时执行相应的处理语句

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

从kotlin1.3开始when语句的参数允许使用变量定义的方式，变量的作用域将被限定在when语句中

```kotlin
fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
```

## for循环
可以使用for循环遍历目标集合的所有内容

```kotlin
    var ints= listOf<Int>(1, 2, 3)
    for (item: Int in ints) {
        println(item)
    }
```

List集合提供了iterator方法，它返回一个Iterator迭代器对象，Iterator迭代器对象中包含next和hasNext方法

使用for循环遍历范围表达式

```kotlin
for (i in 1..3) {
    println(i)
}
for (i in 6 downTo 0 step 2) {
    println(i)
}
```

for循环中使用索引遍历数组或者列表

```kotlin
    val array = arrayOf("a", "b", "c")
    //方式一
    for (i in array.indices) {
        println(array[i])
    }
    //方式二
    for ((index, value) in array.withIndex()) {
        println("the element at $index is $value")
    }
```

## while循环
kotlin支持while和do-while循环

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```

## Break and continue
Kotlin支持循环中的break和continue操作符
