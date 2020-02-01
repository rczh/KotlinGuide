# Equality
kotlin支持两种类型相等

* 结构相等：调用equals方法

* 引用相等：两个引用指向同一个对象

## 结构相等
使用==操作符判断结构相等，通常情况下a == b表达式被转换成以下形式：

```kotlin
//如果a不为空调用equals方法，否则判断b是否引用空值。当a为空时，a == b语句不会抛出异常
a?.equals(b) ?: (b === null)
```

注意，a == null将被自动转换成a === null

通过覆盖equals(other: Any?): Boolean方法可以自定义结构相等实现

## 单精度数值相等
当相等性检查能够明确判断为Float或者Double类型时，检查遵循IEEE754浮点算术标准。否则不能使用IEEE754浮点算术标准，在这种情况下-0.0f和0.0f不相等

```kotlin
class Person(val age: Any){
    override fun equals(other: Any?): Boolean {
        if(other is Person){
            return this.age == other.age
        }
        return false
    }
}

fun main(){
    val p = Person(-0.0f)
    println(p == Person(0.0f))
}    
```

## 引用相等
使用===操作符判断引用相等，如果a和b指向同一个对象则a === b返回true

注意，对于基本类型值比如Int，===等价于==

