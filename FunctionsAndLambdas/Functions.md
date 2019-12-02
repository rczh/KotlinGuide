# Functions
## 函数声明
kotlin使用fun关键字来声明函数，函数参数使用name: type定义，参数之间用逗号分隔，每个参数必须明确指定参数类型

```kotlin
fun powerOf(number: Int, exponent: Int) { /*...*/ }
```

### 参数默认值
kotlin允许为函数参数设置默认值

```kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) { /*...*/ }
```

当子类重写父类中带有默认参数值的函数时，子类函数的定义必须省略默认参数值

```kotlin
open class A {
    open fun foo(i: Int = 10) { /*...*/ }
}

class B : A() {
    override fun foo(i: Int) { /*...*/ }  // no default value allowed
}
```

如果函数定义中默认参数值在非默认参数值之前定义，函数调用时只能通过指定参数名称来使用默认参数值

```kotlin
fun foo(bar: Int = 0, baz: Int) { /*...*/ }

foo(baz = 1) // The default value bar = 0 is used
```

如果函数定义中最后一个参数为lambda表达式，函数调用时可以将lambda表达式的实现放在圆括号外面

```kotlin
fun foo(bar: Int = 0, baz: Int = 1, qux: () -> Unit) { /*...*/ }

foo(qux = { println("hello") }) // Uses both default values bar = 0 and baz = 1 
foo { println("hello") }        // Uses both default values bar = 0 and baz = 1
```

### 指定调用参数名称




