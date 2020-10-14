# Class and Inheritance
## Class
kotlin中使用class关键字声明类。类的声明包含类名、类头和类体，其中类头和类体是可选的

```kotlin
//省略类体
class Empty
```

### 构造函数
kotlin中类能包含一个主构造函数和多个二级构造函数，当主构造函数没有注解或者可见性修饰符时可以省略constructor关键字

```kotlin
class Person constructor(firstName: String) { /*...*/ }
//省略constructor关键字
class Person(firstName: String) { /*...*/ }
//注解和可见性修饰符放在constructor关键字前面
class Customer public @Inject constructor(name: String) { /*...*/ }
```

系统默认生成的主构造函数的可见性为public，可以通过声明空的主构造函数并且添加private修饰符来更改默认主构造函数的可见性

```kotlin
class DontCreateMe private constructor () { /*...*/ }
```

kotlin中主构造函数不能包含任何代码，初始化代码可以放在init代码块中。主构造函数的参数能够在init代码块和属性初始化代码中使用。在实例化对象时，init代码块和属性初始化代码按照它们在类中定义的顺序执行

```kotlin
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)
    init {
        println("First initializer block that prints ${name}")
    }
    val secondProperty = "Second property: ${name.length}".also(::println)
    init {
        println("Second initializer block that prints ${name.length}")
    }
}
```

kotlin提供了一种简单的方式用来在主构造函数中声明并且初始化变量

```kotlin
class Person(val firstName: String, val lastName: String, var age: Int) { /*...*/ }
```

如果主构造函数中的所有参数都有默认值，编译器将生成一个额外的使用默认值的无参数构造函数

### 二级构造函数
kotlin中可以定义多个以constructor为前缀的二级构造函数。如果类中有一个主构造函数，每个二级构造函数都需要直接或者间接的委托给主构造函数。如果类中没有定义主构造函数，每个二级构造函数都会隐含的委托给系统默认的主构造函数

```kotlin
class Person(val name: String) {
    var children: MutableList<Person> = mutableListOf<Person>();
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

init代码块实际上是主构造函数的一分部，由于每个二级构造函数在第一条语句调用主构造函数，因此所有的init代码块都会在二级构造函数之前执行

```kotlin
class Constructors {
    init {
        println("Init block")
    }

    constructor(i: Int) {
        println("Constructor")
    }
}
```

### 创建实例
kotlin中没有new关键字，创建实例时直接调用构造函数

```kotlin
val invoice = Invoice()
val customer = Customer("Joe Smith")
```

### 类的成员
kotlin中的类可以包含:
* 二级构造函数和init代码块
* 方法
* 属性
* 嵌套类和内联类
* 对象声明


## 继承
kotlin中所有类都有一个基类Any，可以在类头中使用冒号来明确的指定父类，并且父类必须被正确的初始化

```kotlin
open class Base(p: Int)
class Derived(p: Int) : Base(p)
```

如果子类没有主构造函数，需要在每个二级构造函数中初始化父类

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

### 方法覆盖
kotlin需要对父类中可被覆盖的方法使用open修饰符，对子类中覆盖的方法使用override修饰符

在一个final类中使用open修饰方法是无效的，默认情况下类是final的并且不能被继承，需要使用open声明类使它能够被继承

```kotlin
open class Shape {
    open fun draw() { /*...*/ }
    fun fill() { /*...*/ }
}

class Circle() : Shape() {
    override fun draw() { /*...*/ }
}
```

被override修饰的方法默认是open的并且能够被子类继承。可以使用final来禁止继承

```kotlin
open class Rectangle() : Shape() {
    final override fun draw() { /*...*/ }
}
```

### 属性覆盖
属性覆盖和方法覆盖类似，需要对父类中可被覆盖的属性使用open修饰符，子类中覆盖的属性使用override修饰符，属性的类型必须保持一致

```kotlin
open class Shape {
    open val vertexCount: Int = 0
}

class Rectangle : Shape() {
    override val vertexCount = 4
}
```

由于val类型属性只包含get方法，var类型属性同时包含get和set方法，可以使用var类型属性覆盖val类型属性，但val类型属性不能覆盖var类型属性

可以在子类的主构造函数中使用override关键字覆盖父类中属性

```kotlin
interface Shape {
    val vertexCount: Int
}

class Rectangle(override val vertexCount: Int = 4) : Shape // Always has 4 vertices

class Polygon : Shape {
    override var vertexCount: Int = 0  // Can be set to any number later
}
```

### 子类的初始化顺序
当创建子类对象实例时，首先执行父类的初始化逻辑，然后执行子类的初始化逻辑

```kotlin
open class Base(val name: String) {
    init { println("Initializing Base") }
    open val size: Int = 
        name.length.also { println("Initializing size in Base: $it") }
}

class Derived(
    name: String,
    val lastName: String
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {
    init { println("Initializing Derived") }
    override val size: Int =
        (super.size + lastName.length).also { println("Initializing size in Derived: $it") }
}
```

注意，由于在执行父类初始化逻辑时子类中的初始化并没有执行，所以不应该在父类的初始化逻辑中调用任何子类的属性或者方法

### 调用父类的实现
子类中可以使用super关键字调用父类的方法或者属性

```kotlin
open class Rectangle {
        open fun draw() { println("Drawing a rectangle") }
        val borderColor: String get() = "black"
    }

    class FilledRectangle : Rectangle() {
        override fun draw() {
            super.draw()
            println("Filling the rectangle")
        }
        val fillColor: String get() = super.borderColor
    }
```

在内联类中可以使用"super@外部类名"的形式来访问外部类中父类的成员

```kotlin
class FilledRectangle: Rectangle() {
    fun draw() { /* ... */ }
    val borderColor: String get() = "black"
    
    inner class Filler {
        fun fill() { /* ... */ }
        fun drawAndFill() {
            super@FilledRectangle.draw() // Calls Rectangle's implementation of draw()
            fill()
            println("Drawn a filled rectangle with color ${super@FilledRectangle.borderColor}") // Uses Rectangle's implementation of borderColor's get()
        }
    }
}
```

如果子类的父类或者接口中包含了相同的方法定义，子类必须覆盖并实现这个方法。子类中可以使用"super<父类名>"的形式来指定调用哪个父类的成员

```kotlin
open class Rectangle {
    open fun draw() { /* ... */ }
}

interface Polygon {
    fun draw() { /* ... */ } // interface members are 'open' by default
}

class Square() : Rectangle(), Polygon {
    // The compiler requires draw() to be overridden:
    override fun draw() {
        super<Rectangle>.draw() // call to Rectangle.draw()
        super<Polygon>.draw() // call to Polygon.draw()
    }
}
```

## 抽象类
kotlin中使用abstract来声明抽象类和抽象方法，抽象类和抽象方法不需要加open修饰符

可以使用abstract关键字来声明override方法

```kotlin
open class Polygon {
    open fun draw() {}
}

abstract class Rectangle : Polygon() {
    override abstract fun draw()
}
```

## 伴生对象
kotlin中并不提供静态方法，如果需要在不创建类对象的情况下访问类的方法或者属性，可以使用对象声明或者伴生对象


