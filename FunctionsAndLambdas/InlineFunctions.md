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




