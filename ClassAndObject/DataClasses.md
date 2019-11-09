# data classes
kotlin中使用数据类来保存数据

使用data关键字来声明数据类，数据类的声明必须满足以下条件：

* 主构造函数中至少需要定义一个参数

* 所有主构造函数的参数需要使用val或者var标注

* 数据类不能是抽象的，公开的，密闭的，或者内联的

* 在kotlin1.1之前，数据类只能实现接口。从kotlin1.1开始，数据类能够继承其他类

```kotlin
data class User(val name: String, val age: Int)
```

编译器自动为数据类生成以下成员函数：

* equals()，hashCode()

* toString()

* componentN()

主构造函数中按照声明属性的顺序，每个属性对应一个component函数

* copy()

自动生成的成员函数满足以下规则：

* 如果数据类中明确实现了equals(), hashCode(),toString()函数，或者在父类中这些函数的实现使用了final关键字，这时编译器将不会再生成这些函数

* 如果父类中已经声明了相应的component函数，编译器会生成并且覆盖父类中的component函数。如果无法覆盖父类中的component函数将会报错

* 从1.3开始，kotlin禁止从已经声明了copy()函数的父类中继承数据类

* 不允许为componentN()和copy()函数提供明确的实现


注意：如果数据类需要一个无参数的构造函数，必须为数据类的所有参数指定默认值

```kotlin
data class User(val name: String = "", val age: Int = 0)
```

