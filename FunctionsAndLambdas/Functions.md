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
kotlin允许在调用函数时指定参数名称，通过指定参数名称可以使函数调用代码更可读

```kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
/*...*/
}

reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
)
```

如果函数调用中同时使用了位置参数和命名参数，位置参数需要放在命名参数前面

```kotlin
fun foo(bar: Int = 0, baz: Int) { /*...*/ }

foo(1, baz = 1)
```

可以使用扩展操作符来传递指定名称的vararg参数

```kotlin
fun foo(vararg strings: String) { /*...*/ }

foo(strings = *arrayOf("a", "b", "c"))
```

注意，由于java字节码不保存函数参数名称，所以不能使用指定参数名称的形式来调用java函数

### Unit返回类型
如果函数不返回任何值，可以使用Unit返回类型

```kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    //可以不明确返回Unit    
    // `return Unit` or `return` is optional
}
```

函数声明中可以省略Unit返回类型

```kotlin
fun printHello(name: String?) { ... }
```

### 表达式返回类型
kotlin允许使用表达式作为函数的返回结果

```kotlin
fun double(x: Int): Int = x * 2
```

如果编译器能够推断返回类型，函数声明时可以省略返回类型

```kotlin
fun double(x: Int) = x * 2
```

### 明确指定返回类型
由于带有块体的函数可能包含复杂的逻辑，kotlin不能推测带有块体的函数返回类型，除非带有块体的函数返回类型为Unit，否则必须明确指定返回类型

### 可变数量参数vararg
kotlin允许使用vararg关键字定义可变数量参数，在函数内部vararg参数类型为Array

```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
//可以一个接一个传递参数
asList(1, 2, 3)

//可以使用扩展操作符将数组内容传给vararg参数
val a = arrayOf(1, 2, 3)
asList(-1, 0, *a, 4)
```

函数中只能定义一个vararg参数，通常位于最后位置。如果vararg参数没有定义在最后位置，vararg后面的参数可以使用指定参数名称的形式传递，如果最后位置为函数类型参数，可以在圆括号外传递lambda表达式

### 中缀表示法
kotlin允许使用infix关键字定义函数中缀表示法，使用中缀表示法调用函数时可以省略点和圆括号

中缀函数必须满足以下条件
* 中缀函数必须是成员函数或者扩展函数

* 中缀函数必须有一个参数

* 中缀函数参数不能有默认值，不能是vararg

```kotlin
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2

// is the same as
1.shl(2)
```

中缀函数的优先级低于算术运算符，类型转换和rangeTo操作符，高于布尔运算符和is,in操作符

中缀函数调用时必须明确指定接收方和参数

```kotlin
class MyStringCollection {
    infix fun add(s: String) { /*...*/ }
    
    fun build() {
        this add "abc"   // Correct
        add("abc")       // Correct
        //add "abc"        // Incorrect: the receiver must be specified
    }
}
```

## 函数作用域
kotlin中函数能够声明在文件顶级，函数内部，或者作为成员函数和扩展函数

### 内部函数
kotlin能够在函数内部定义函数

```kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: MutableSet<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

内部函数能够访问外部函数的本地变量

```kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

## 尾递归函数


