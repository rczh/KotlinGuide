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

系统默认生成的主构造函数的可见性为public，可以通过声明空的主构造函数并且添加private修饰符来更改默认可见性

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
kotlin中可以定义多个以constructor为前缀的二级构造函数。如果类中有一个主构造函数，每个二级构造函数都需要直接或者间接的调用主构造函数。如果类中没有定义主构造函数，每个二级构造函数都会隐含的调用系统默认的主构造函数

```kotlin
class Person(val name: String) {
    var children: MutableList<Person> = mutableListOf<Person>();
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

init代码块实际上作为主构造函数的一分部，由于每个二级构造函数在第一条语句调用主构造函数，因此所有的init代码块都会在二级构造函数之前执行

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


