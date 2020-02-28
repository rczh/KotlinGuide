# Reflection
反射是一组语言和库提供的功能，用来在运行时获取程序的结构

kotlin为使用反射功能提供了单独的运行组件kotlin-reflect.jar，目的是为了减小不使用反射功能的应用程序所需运行时库的大小。如果使用反射功能需要将kotlin-reflect.jar文件添加到项目的classpath中

## Class References
最基本的反射功能是获取kotlin类的运行时引用

```kotlin
//引用类型为KClass
val c = MyClass::class
```

注意，kotlin类引用与java类引用不同，可以使用KClass实例的java属性来获取java类引用

## Bound Class References (since 1.1)
可以通过对象实例获取指定类的引用

```kotlin
val widget: Widget = ...
//尽管widget的类型为Widget，widget::class的返回结果为GoodWidget类的引用
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

## Callable references
对函数、属性或者构造函数的引用能够做为函数类型的实例被调用或者使用

所有可调用引用的公共父类为KCallable<out R>，其中R为返回类型值，它可以是属性的属性类型，或者是构造函数的构造类型

### Function References

















