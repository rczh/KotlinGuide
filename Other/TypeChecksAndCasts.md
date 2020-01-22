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

满足以下规则可以使用自动类型转换：

* val本地变量：除了本地委托属性之外的val本地变量都可以使用自动类型转换

* val属性：除了open属性或者具有自定义get方法之外的val属性都可以使用自动类型转换

* var本地变量：除了在is检查和使用之间发生改变或者在修改它的lambda表达式中使用或者本地委托属性之外的var本地变量都可以使用自动类型转换

* var属性：由于var属性能够在任何时间被修改，所有var属性都不能使用自动类型转换

## "Unsafe" cast operator
如果无法进行转换时类型转换操作会抛出异常，通常称这种类型转换为不安全的类型转换。kotlin中使用中缀操作符as执行这种不安全的类型转换

```kotlin
//由于null不能转换成String，如果y为null则y as String会抛出异常
val x: String = y as String
```

对于y为null的情况，可以使用可空的转换类型

```kotlin
val x: String? = y as String?
```

## "Safe" (nullable) cast operator
为了避免出现异常可以使用安全类型转换操作符as?，如果转换失败as?返回null

```kotlin
//对于y为null的情况，虽然as?的转换类型为String，但是y as? String的结果为空
val x: String? = y as? String
```

## Type erasure and generic type checks






