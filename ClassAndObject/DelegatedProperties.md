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
通过定义provideDelegate方法，可以自定义创建委托属性对象的逻辑

```kotlin
class ResourceID<T>(val value: T){
    companion object {
        val image_id = ResourceID<Int>(100)
        val text_id = ResourceID<Int>(200)
    }
}

class ResourceDelegate<T>(val v: T) : ReadOnlyProperty<MyUI, T> {
    override fun getValue(thisRef: MyUI, property: KProperty<*>): T {
        println("property name: ${property.name}")
        return v
    }
}

class ResourceLoader<T>(val id: ResourceID<T>) {
    //创建委托属性对象时调用provideDelegate方法
    operator fun provideDelegate(
        thisRef: MyUI,
        prop: KProperty<*>
    ): ResourceDelegate<T> {
        checkProperty(thisRef, prop.name)
        // create delegate
        return ResourceDelegate(id.value)
    }

    private fun checkProperty(thisRef: MyUI, name: String): Boolean {
        return true
    }
}

class MyUI {
    fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> {
        return ResourceLoader(id)
    }

    val image by bindResource(ResourceID.image_id)
    val text: Int by ResourceDelegate<Int>(ResourceID.text_id.value)
}
```

编译器调用provideDelegate方法生成委托属性对象

```kotlin
class C {
    var prop: Type by MyDelegate()
}

//编译器生成的代码
class C {
    // calling "provideDelegate" to create the additional "delegate" property
    private val prop$delegate = MyDelegate().provideDelegate(this, this::prop)
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```

## 标准库提供的属性委托
kotlin标准库提供了几种常用的委托属性方法

### lazy
lazy方法需要一个lambda表达式参数，当第一次调用委托属性get方法时执行lambda表达式并且保存结果，之后再调用get方法时直接返回保存结果

```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}
```

lazy方法支持LazyThreadSafetyMode参数
* LazyThreadSafetyMode.SYNCHRONIZED，默认情况下lambda表达式的执行是同步的

* LazyThreadSafetyMode.PUBLICATION，允许多个线程同时执行lambda表达式，但是只保存第一个线程的执行结果

* LazyThreadSafetyMode.NONE，不做任何线程安全保护

### observable
observable方法需要两个参数，初始值和lambda表达式。每次对委托属性赋值之后都会调用lambda表达式

```kotlin
class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main() {
    val user = User()
    user.name = "first"
    user.name = "second"
}
```

与observable相对应的是vetoable方法，每次对委托属性赋值之前会调用lambda表达式

### 委托属性到map
能够使用map作为委托属性对象，委托属性使用属性名作为key从map中取值

```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
fun main() {
    val user = User(mapOf(
        "name" to "John Doe",
        "age"  to 25
    ))
    println(user.name) // Prints "John Doe"
    println(user.age)  // Prints 25
}
```

注意，对于var属性需要使用MutableMap作为委托属性对象










