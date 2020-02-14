# Null Safety
## 空类型和非空类型
kotlin类型系统的目标是从代码上解决空引用带来的问题，在很多编程语言中最常见的问题是访问空引用的成员会导致空指针异常

代码中以下情况能够引起空指针异常：

* 抛出NullPointerException的显示调用

* 使用!!操作符

* 某些数据和初始化不一致

  * 一个未被初始化只在构造函数中可用的this对象被传递并在某处被使用，这种情况称为leaking this
  
  * 父类构造函数调用在子类中实现的open成员，由于父类构造函数的执行优于子类构造函数，该open成员没有被初始化

* java程序交互

  * 访问平台类型空引用的成员
  
  * 用于java交互带有不正确可空性的泛型类型。java代码可能添加null到MutableList<String>类型，这种情况应该使用MutableList<String?>类型
  
  * 外部Java代码引起的其他问题

kotlin中类型系统区分了可以包含空和不能包含空的引用

```kotlin
fun main(){
    //普通字符串类型不能包含null
    var a: String = "abc"
    a = null // compilation error
    
    //可空字符串类型可以包含null
    var b: String? = "abc"
    b = null // ok
    print(b)
    
    //对于非空类型对象可以直接访问属性，不会引起空指针异常
    val lenA = a.length
    //对于可空类型对象不能直接访问属性
    val lenB = b.length // error: variable 'b' can be null
}
```

## 在条件中检查空
可以明确的检查b是否为空，并且分别处理两种不同情况。编译器将跟踪检查的执行并且允许在if语句中访问length属性

```kotlin
fun main() {
    val b: String? = "Kotlin"
    val l = if (b != null) b.length else -1
    
    if (b != null && b.length > 0) {
        print("String of length ${b.length}")
    } else {
        print("Empty string")
    }
}
```

注意，这种方式只适用于b是不可变的情况(例如在检查和使用之间没有被修改的本地var变量或者val变量)。如果b是可变的，在检查后b可能变成空

## 安全调用操作符


```kotlin
fun main() {
    val a = "Kotlin"
    val b: String? = null
    println(b?.length)
    println(a?.length) // Unnecessary safe call
}
```




