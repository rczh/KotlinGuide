# Object Expressions and Declarations
## object表达式
object表达式用来创建匿名类

```kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { /*...*/ }
    override fun mouseEntered(e: MouseEvent) { /*...*/ }
})
```

匿名类能够同时继承父类和接口，父类和接口之间使用逗号分隔

```kotlin
open class A(x: Int) {
    public open val y: Int = x
}

interface B { /*...*/ }

val ab: A = object : A(1), B {
    override val y = 15
}
```

如果object表达式没有指定类名，编译器将默认创建一个类对象

```kotlin
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
}
```

注意，匿名对象只能在方法内部或者为私有声明使用做为返回类型。当匿名对象为公有方法或者公有属性使用做为返回类型时，公有方法或属性的实际类型为匿名类父类型或者Any，这时匿名类中新加入的成员将不可访问




