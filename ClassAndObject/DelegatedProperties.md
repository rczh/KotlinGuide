# Delegated Properties
kotlin支持委托属性

```kotlin
class Example {
    var p: String by Delegate()
}

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

by关键字后面定义委托对象Delegate，属性p的set和get方法将被委托到Delegate对象的setValue和getValue方法。委托对象不必实现任何接口，但是必须提供相应的setValue和getValue方法(通过成员函数或者扩展函数)

从kotlin1..1开始，可以在方法内部或者代码块中声明委托属性

```kotlin
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)

    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
```

## 委托方法定义

```kotlin
operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
```

对于只读属性，委托对象需要提供getValue方法，它的返回值为属性类或子类的对象

* thisRef参数表示定义属性的类对象

* property参数为KProperty，表示属性对象的描述

```kotlin
operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
```

对于可变属性，委托对象需要提供setValue方法

* thisRef和property参数与getValue方法类似

* value参数为属性类或子类的对象，表示被设置的值

委托类可以实现ReadOnlyProperty或者ReadWriteProperty接口，接口中定义了相应的setValue和getValue方法

```kotlin
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```

## 实现原理
编译器为每个委托属性生成一个辅助属性并将set和get方法委托给辅助属性

```kotlin
class C {
    var prop: Type by MyDelegate()
}

//编译器生成的代码
class C {
    private val prop$delegate = MyDelegate()
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```

## 自定义生成委托对象逻辑









