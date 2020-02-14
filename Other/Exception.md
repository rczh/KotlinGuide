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

## Checked Exceptions
