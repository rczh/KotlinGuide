# 内联函数
由于函数调用时会执行栈操作从而带来一定的运行开销，kotlin引入内联函数来避免函数调用的开销

## 定义
kotlin中使用inline关键字来定义内联函数

```kotlin
inline fun <T> lock(lock: Lock,  body: () -> T): T {
    lock.lock()
    try {
        return body()
    }finally {
        lock.unlock()
    }
}

fun foo(){
    println("foo")
}

fun main(){
    val l = ReentrantLock()
    lock(l) { foo() }
}    
```

编译时，编译器使用内联函数的实现来代替函数调用

```java
public static final void main() {
  //编译器生成的代码
  l.lock();
  try {
      foo();
  }
  finally {
      l.unlock();
  }
}
```

注意，使用内联函数会造成编译器生成的代码增加，尽量避免对大函数使用内联

内联函数会将函数本身和函数类型参数一起进行替换

## noinline
由于内联函数会将函数类型参数一起内联，如果内联函数中调用以该函数类型参数作为实参的普通函数时会产生编译错误

```kotlin
inline fun <T> lock(lock: Lock,  body: () -> T): T {
    lock.lock()
    try {
        //编译错误，otherMethod需要() -> T类型的函数类型实例，但是body已经被内联成为一个具体的函数值
        //return otherMethod(body)
    }finally {
        lock.unlock()
    }
}

fun <T> otherMethod(body: ()-> T): T{
    return body()
}
```

noinline关键字用来声明函数类型参数不内联

```kotlin
inline fun <T> lock(lock: Lock, noinline body: () -> T): T {
    lock.lock()
    try {
        //body是() -> T类型的函数类型实例，没有被内联
        return otherMethod(body)
    }finally {
        lock.unlock()
    }
}
```

内联的函数类型参数只能在内联函数内部调用或者作为实参传递给其他内联函数

## 非本地返回
默认情况下只允许使用带标签的return语句从lambda表达式中返回，如果接收lambda表达式参数的函数是内联的，则允许在lambda表达式中直接使用return语句

```kotlin
inline fun inlined(block: () -> Unit) {
    println("hi!")
}
fun foo() {
    inlined {
        return // OK: the lambda is inlined
    }
}
fun main() {
    foo()
}
```

这种直接使用return语句从lambda表达式中返回的形式称为非本地返回

## crossinline
如果内联函数内部不直接调用lambda表达式参数，而是将lambda表达式参数传递给其他成员，比如匿名类，这时lambda表达式中不允许使用非本地返回

```kotlin
inline fun f(crossinline body: () -> Unit) {
    val result = object: Runnable {
        override fun run() = body()
    }
    
    val run = Runnable { body() }
}

fun main(){
    f {
        println("hello")
        return
    }
}
```

使用crossinline关键字标识lambda表达式中不允许使用非本地返回

## reified
在运行时泛型类型不包含实际类型的任何信息，类型信息将被擦除，编译器禁止在运行时对泛型类型使用is类型检查

内联函数提供了具体化类型参数reified，使用reified关键字声明的类型参数能够像普通类一样允许使用as,is操作符

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

注意，普通函数不允许使用reified关键字

## 内联属性
可以将不带幕后字段属性的set,get方法声明为内联函数

```kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

当使用inline关键字声明内联属性时，内联属性中的set,get方法都是内联函数

```kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

## 公共内联函数的限制
由于公共的内联函数可能被其他模块调用，如果内联函数中调用本地模块的其它私有函数，当私有函数发生变化时必须重新编译调用内联函数的模块。为避免类似情况发生，kotlin不允许公共内联函数调用本地的私有函数

私有内联函数允许调用本地的私有函数

