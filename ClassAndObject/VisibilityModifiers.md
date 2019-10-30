# Visibility Modifiers
kotlin中提供四种可见性修饰符：private, protected, internal, public

如果没有明确指定可见性修饰符则默认为public

## top-level成员的可见性
函数、属性、类、对象和接口能够在包中直接声明

* 默认为public可见性，声明的成员在任何地方可见

* 指定private可见性，声明的成员只在声明文件内部可见 

* 指定internal可见性，声明的成员在当前模块中可见，模块为一系列一起编译的kotlin文件

* top-level成员不能使用protected可见性

```kotlin
// file name: example.kt
package foo

private fun foo() { ... } // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt
    
internal val baz = 6    // visible inside the same module
```

## 类成员的可见性

* 默认为public可见性，任何能够使用类的对象都可以使用类中的public成员

* 指定private可见性，声明的成员只能在当前类中可见 

* 指定internal可见性，模块中任何能够使用类的对象都可以使用类中的internal成员

* 指定protected可见性，声明的成员只能在当前类和子类中可见

注意，kotlin中外部类不能看到内部类或者内联类的private成员

```kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // public by default
    
    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible
    // b, c and d are visible
    // Nested and e are visible
    //覆盖的成员默认可见性为protected
    override val b = 5   // 'b' is protected
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible
    // o.c and o.d are visible (same module)
    // Outer.Nested is not visible, and Nested::e is not visible either 
}
```

主构造函数默认的可见性为public，可以使用如下格式指定主构造函数的可见性

```kotlin
class C private constructor(a: Int) { ... }
```

注意，局部变量，局部方法，局部类不能使用可见性修饰符
