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

```kotlin
open class MyClass{
    open var i = 1
}

class C {
    // Private function, so the return type is the anonymous object type
    private fun foo() = object {
        val x: String = "x"
    }

    fun foo2() {
        //方法内部使用匿名类做为返回类型
        val adHoc = object {
            var x: Int = 0
            var y: Int = 0
        }
        print(adHoc.x + adHoc.y)
    }

    // Public function, so the return type is Any, 匿名类中新加入的成员x不能被访问
    fun publicFoo() = object {
        val x: String = "x"
    }

    // Public properties, so the return type is MyClass, 新加入的j成员j不能被访问
    var myClass = object : MyClass(){
        var j = 1
    }

    fun bar() {
        val x1 = foo().x        // Works
//        val x2 = publicFoo().x  // ERROR: Unresolved reference 'x'
//        val x3 = myClass.j    //ERROR
    }
}
```

object表达式声明的匿名类为匿名内部类，在匿名类内部可以访问外部类的成员

## object声明
kotlin语言原生支持单例，使用object声明来创建单例。kotlin保证object声明的初始化是线程安全的

```kotlin
class BaseManager{
}

//object声明能够继承父类
object DataProviderManager : BaseManager{
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }
}
```

使用object声明的名字来引用成员函数

```kotlin
DataProviderManager.registerDataProvider(...)
```

注意，object声明不能定义在方法内部，object声明能够嵌套在其他object声明或者非内部类中

### 伴生对象




