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

## noinline参数




