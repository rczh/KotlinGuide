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

### 二级构造函数



