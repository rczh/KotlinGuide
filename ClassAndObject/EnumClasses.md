# Enum Classes
kotlin支持定义枚举类，其中每一个枚举常量都是枚举类的一个实例

```kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

枚举类中可以定义成员函数或属性，每一个枚举常量可以覆盖或者定义新的成员函数或属性

注意，需要使用分号将枚举常量和枚举类中的成员分隔开

```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
        fun printWait() {
            print("wait")
        }
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
    var i = 1
}
```

枚举类不能继承父类但是可以实现接口，可以在枚举类中实现接口的所以成员函数，或者分别在每个枚举常量中实现接口的成员函数

```kotlin
enum class IntArithmetics : BinaryOperator<Int>, IntBinaryOperator {
    PLUS {
        override fun apply(t: Int, u: Int): Int = t + u
    },
    TIMES {
        override fun apply(t: Int, u: Int): Int = t * u
    };

    override fun applyAsInt(t: Int, u: Int) = apply(t, u)
}

fun main() {
    val a = 13
    val b = 31
    for (f in IntArithmetics.values()) {
        println("$f($a, $b) = ${f.apply(a, b)}")
    }
}
```

## 遍历枚举常量
可以使用values和valueOf方法来遍历和获取枚举常量

```kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

从kotlin1.1开始，可以使用泛型方法enumValues<T>() 和enumValueOf<T>()来遍历和获取枚举常量

```kotlin
println(enumValues<Color>().joinToString {
            //枚举常量中包含name和ordinal属性，表示枚举名称和位置
            it.name + "_" + it.ordinal
        }
    )
```

枚举常量默认实现了Comparable接口，顺序为定义枚举常量的自然顺序

```kotlin
if(Color.BLUE.compareTo(Color.GREEN) > 0){
        println("blue > green")
    }
```
