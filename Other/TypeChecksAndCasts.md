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




