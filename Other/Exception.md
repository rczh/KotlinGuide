# Exception
## 异常类
kotlin中所有异常类都继承于Throwable，每个异常都有一个消息、跟踪栈和一个可选原因

使用throw表达式可以抛出异常

```kotlin
fun main() {
    throw Exception("Hi There!")
}
```

使用try表达式可以捕获异常

```kotlin
try {
    // some code
}
catch (e: SomeException) {
    // handler
}
finally {
    // optional finally block
}
```

可以有零个或者多个catch块，finally块可以省略，但是至少应该有一个catch或者finally块

### try表达式
kotlin中try是表达式，它可以有返回值

```kotlin
val a: Int? = try { parseInt(input) } catch (e: NumberFormatException) { null }
```

try表达式的返回值为try块中的最后一个表达式值或者catch块中的最后一个表达式值，finally块的内容不会影响try表达式的返回值

## 异常检查
在java中某些函数定义可能会抛出异常，比如Appendable接口中的append函数

```java
Appendable append(CharSequence csq) throws IOException;
```

java中任何调用append函数的方法都需要捕获IOException，这将导致程序中包含很多这样的代码

```java
try {
    log.append(message)
}
catch (IOException e) {
    // Must be safe
}
```

实际上，异常检查对于小型程序可以提高开发人员的工作效率和代码质量，但是对于大型程序异常检查会降低工作效率并且几乎不会提高代码质量

kotlin中不支持异常检查(函数定义时不会抛出异常)

## Nothing类型
kotlin中throw是一个表达式，它可以用于elvis操作符中。throw表达式的类型是Nothing类型，该类型没有值用来标记无法到达的代码位置

```kotlin
val s = person.name ?: throw IllegalArgumentException("Name required")
```

可以使用Nothing去标识一个不会返回的函数

```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

编译器不会在调用该函数之后继续执行

```kotlin
val s = person.name ?: fail("Name required")
println(s)     //执行到这里时s已经被初始化
```

Nothing?类型的值只可能是null，当使用null初始化一个推断类型值时，由于没有其他可用于确定具体类型的信息，编译器将推断为Nothing?类型

```kotlin
val x = null           // 'x' has type `Nothing?`
val l = listOf(null)   // 'l' has type `List<Nothing?>
```

