# Delegation
kotlin语言本身支持委托代理模式

## 定义
kotlin使用by关键字定义委托代理

```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
    val b = BaseImpl(10)
    Derived(b).print()
}
```

by语句表示编译器将为Derived生成所有Base的方法，并将其转发给b对象




