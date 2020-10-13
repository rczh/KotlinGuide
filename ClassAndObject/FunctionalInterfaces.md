# Functional (SAM) interfaces
只有一个抽象方法的接口称为函数接口，或单个抽象方法(SAM)接口。函数接口可以有多个非抽象成员，但只能有一个抽象成员

可以使用fun关键字声明函数接口

```kotlin
fun interface KRunnable {
   fun invoke()
}
```

## SAM conversions
对于函数接口，通过使用lambda表达式你可以使用SAM转换使代码更加简洁和可读

你可以使用lambda表达式来实现类，而不是手动创建实现函数接口的类。通过SAM转换，Kotlin可以将任何与接口抽象方法相匹配的lambda表达式转换为实现该接口的类的实例

参考以下kotlin函数接口

```kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}
```

如果不使用SAM转换，则需要编写这样的代码

```kotlin
// Creating an instance of a class
val isEven = object : IntPredicate {
   override fun accept(i: Int): Boolean {
       return i % 2 == 0
   }
}
```

通过使用SAM转换，你可以编写以下代码来实例化函数接口类

```kotlin
// Creating an instance using lambda
val isEven = IntPredicate { it % 2 == 0 }
```

使用一个lambda表达式来替换所有冗余代码

```kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}

val isEven = IntPredicate { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven.accept(7)}")
}
```

你还可以对Java接口使用SAM转换

## Functional interfaces vs. type aliases
函数接口和类型别名用于不同的目的。类型别名只是现有类型的名称，它们不会创建新类型而函数接口可以

类型别名只能有一个成员，而函数接口可以有多个非抽象成员和一个抽象成员。函数接口还可以实现和扩展其他接口

综上所述，函数接口比类型别名更灵活，提供的功能也更多

