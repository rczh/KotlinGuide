# data classes
kotlin中使用数据类来保存数据

## 声明数据类
使用data关键字来声明数据类，数据类的声明必须满足以下条件：

* 主构造函数中至少需要定义一个参数

* 所有主构造函数的参数需要使用val或者var标注

* 数据类不能是抽象的，公开的，密闭的，或者内联的

* 在kotlin1.1之前，数据类只能实现接口。从kotlin1.1开始，数据类能够继承其他类

```kotlin
data class User(val name: String, val age: Int)
```

## 编译器自动为数据类生成以下成员函数

* equals()，hashCode()

* toString()

* componentN()

component函数用来将一个对象分解成多个变量，按照主构造函数中声明属性的顺序，每个属性对应一个component函数

```kotlin
val jane = User("Jane", 35) 
val (name, age) = jane
println("$name, $age years of age") // prints "Jane, 35 years of age"
```

* copy()

copy函数用来复制一个对象并且改变其中的一些属性

```kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

## 自动生成的成员函数满足以下规则

* 如果数据类中明确实现了equals(), hashCode(),toString()函数，或者在父类中这些函数的实现使用了final关键字，这时编译器将不会再生成这些函数

* 如果父类中已经声明了相应的component函数，编译器会生成并且覆盖父类中的component函数。如果无法覆盖父类中的component函数将会报错

* 从1.3开始，kotlin禁止从已经声明了copy()函数的父类中继承数据类

* 不允许为componentN()和copy()函数提供明确的实现


注意：如果数据类需要一个无参数的构造函数，必须为数据类的所有参数指定默认值

```kotlin
data class User(val name: String = "", val age: Int = 0)
```

## 在数据类中定义属性
编译器只为主构造函数中定义的参数生成成员函数，对于数据类中定义的属性编译器不会生成成员函数

```kotlin
data class Person(val name: String) {
    var age: Int = 0
}
fun main() {
    val person1 = Person("John")
    val person2 = Person("John")
    person1.age = 10
    person2.age = 20
    //尽管两个person对象中的age属性值不相同，equals方法仍然输出true
    println("person1 == person2: ${person1 == person2}")
    println("person1 with age ${person1.age}: ${person1}")
    println("person2 with age ${person2.age}: ${person2}")
}
```






