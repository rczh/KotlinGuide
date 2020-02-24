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
