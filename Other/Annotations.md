# Annotations
## 注解声明
注解用来将元数据附加到代码中。kotlin使用annotation关键字来声明注解，它必须放在class前面

```kotlin
annotation class Fancy
```

可以使用元注解为注解类指定附加的属性：

* @Target用来指定能够被注解类注解的元素类型，包括类、函数、属性、表达式等等

* @Retention表示注解的生命周期，用来指定是否注解被保存在class文件中以及是否注解在运行时通过反射可见，默认情况下两者都为true

* @Repeatable允许在同一元素上多次使用相同的注解

* @MustBeDocumented用来指定注解是公共api的一部分，并且应该被包含在api文档的类或者函数声明中

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```

### 注解使用
可以为类、函数、参数或者表达式添加注解

```kotlin
@Fancy class Foo {
    @Fancy fun baz(@Fancy foo: Int): Int {
        return (@Fancy 1)
    }
}
```

注解主构造函数时需要使用constructor关键字

```kotlin
class Foo @Inject constructor(dependency: MyDependency) { ... }
```

可以为属性访问函数添加注解

```kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```

### Constructors
kotlin中注解类可以有带参数的构造函数

```kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```

构造函数的参数类型可以包括：

* 与Java基本类型相对应的类型，比如Int，Long等

* 字符串类型

* 类引用，比如Foo::class

* 枚举类型

* 其他注解类型

* 以上类型的数组

注意，注解类构造函数参数不能为可空类型，因为jvm不支持将null作为注解类的属性值

如果将注解类做为另一个注解类构造函数的参数，它的名字不需要加@符

```kotlin
annotation class ReplaceWith(val expression: String)
annotation class Deprecated(val message: String, val replaceWith: ReplaceWith = ReplaceWith(""))
@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```

如果需要将类作为注解类构造函数的参数，可以使用KClass。编译器自动将KClass转换为java类，使java代码可以正常访问注解和参数

```kotlin
import kotlin.reflect.KClass
annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any>)
@Ann(String::class, Int::class) class MyClass
```

### Lambdas
可以在lambda表达式中使用注解，注解将被应用到执行lambda表达式的invoke方法

```kotlin
annotation class Suspendable
val f = @Suspendable { Fiber.sleep(10) }
```

## Annotation Use-site Targets
由于编译器会为kotlin类的属性生成set,get方法，为主构造函数参数生成相对应的属性和set,get方法。当注解属性或者主构造函数参数时，可以精确指定注解作用域

```kotlin
class Example(@field:Ann val foo,    //注解foo属性
              @get:Ann val bar,      //注解bar get方法
              @param:Ann val quux)   //注解quux构造函数参数
```

可以使用类似语法对整个文件注解

```kotlin
@file:JvmName("Foo")
package org.jetbrains.demo
```

为同一个元素定义多个注解时，可以将所有注解添加到中括号中从而避免重复定义注解

```kotlin
class Example {
     @set:[Inject VisibleForTesting]
     var collaborator: Collaborator
}
```

kotlin支持的精确注解作用域包含：

* file(注解文件)

* property(该注解对于java代码不可见)

* field(注解属性)

* get(注解属性get方法)

* set(注解属性set方法)

* receiver(注解扩展函数或者扩展属性的接收类型参数)

* param(注解构造函数参数)

* setparam(注解属性set方法的参数)

* delegate(注解委托代理对象属性)

注解扩展函数的接收类型参数

```kotlin
fun @receiver:Fancy String.myExtension() { ... }
```

如果没有精确指定注解作用域，则根据所使用注解的元注解@target来选择作用域。如果@target中包含多个可用作用域，则使用以下列表中的第一个可用作用域：

* param

* property

* field

## Java Annotations
java注解和kotlin完全兼容

```kotlin
import org.junit.Test
import org.junit.Assert.*
import org.junit.Rule
import org.junit.rules.*

class Tests {
    //对属性tempFolder的get方法使用java注解Rule
    @get:Rule val tempFolder = TemporaryFolder()

    @Test fun simple() {
        val f = tempFolder.newFile()
        assertEquals(42, getTheAnswer())
    }
}
```

由于java编写的注解没有定义参数顺序，使用注解时需要通过命名参数的形式来传递参数

```java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```

```kotlin
// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```

如果注解中包含名字为value的参数，传递参数时可以不指定名称

```java
// Java
public @interface AnnWithValue {
    String value();
}
```

```kotlin
// Kotlin
@AnnWithValue("abc") class C
```

### Arrays as annotation parameters
如果注解中包含名字为value的数组类型参数，传递参数时可以使用vararg类型

```java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```

```kotlin
// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") class C
```

对于名字为非value的数组类型参数，传递参数时可以使用数组字面形式(kotlin1.2开始)或者arrayOf函数

```java
// Java
public @interface AnnWithArrayMethod {
    String[] names();
}
```

```kotlin
// Kotlin 1.2+:
@AnnWithArrayMethod(names = ["abc", "foo", "bar"]) 
class C

// Older Kotlin versions:
@AnnWithArrayMethod(names = arrayOf("abc", "foo", "bar")) 
class D
```

### Accessing properties of an annotation instance
kotlin中通过属性来访问注解对象的参数值

```java
// Java
public @interface Ann {
    int value();
}
```

```kotlin
// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```

