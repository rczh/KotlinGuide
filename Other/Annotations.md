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





















