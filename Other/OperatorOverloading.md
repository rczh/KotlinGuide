# Operator overloading
kotlin允许为操作符提供重载，可以使用带有指定名称的成员函数或者扩展函数来实现操作符重载，操作符重载函数需要使用operator标识

## 一元操作符
### 一元前缀操作符
| 表达式 | 重载函数 |
| --- | --- |
| +a | 	a.unaryPlus() |
| -a | a.unaryMinus() |
| !a | a.not() |

对于表达式+a，编译器运行时执行以下步骤：

* 确定a的类型，假设为T

* 查找成员函数或者接收类型为T的扩展函数是否包含unaryPlus()函数

* 如果函数不存在或者有歧义提示编译错误

* 如果函数存在并且返回类型为R，则表达式+a类型为R

注意，这些操作符已经为基本类型进行了优化不会产生函数调用开销

```kotlin
data class Point(val x: Int, val y: Int)
operator fun Point.unaryMinus() = Point(-x, -y)
val point = Point(10, 20)

fun main() {
   println(-point)  // prints "Point(x=-10, y=-20)"
}
```

### 自增自减操作符
| 表达式 | 重载函数 |
| --- | --- |
| a++ | 	a.inc() |
| a-- | a.dec() |

inc()和dec()函数需要返回一个值，该返回值将被赋给使用++或者--操作符的变量。它们不应该改变调用inc或dec的对象

对于表达式a++，编译器执行以下解析步骤：

* 确定a的类型，假设为T

* 查找成员函数或者接收类型为T的扩展函数是否包含inc()函数

* 检查函数的返回类型是否为T的子类型

计算表达式执行以下步骤：

* 将a的初始值保存到临时变量a0

* 将inc()函数的结果赋值给a

* 返回a0作为表达式的结果

对于表达式++a，编译器执行的解析步骤相同，计算表达式执行以下步骤：

* 将inc()函数的结果赋值给a

* 返回a的新值作为表达式的结果

## 二元操作符
### 算数操作符
| 表达式 | 重载函数 |
| --- | --- |
| a + b | 	a.plus(b) |
| a - b	| a.minus(b) |
| a \* b | a.times(b) |
| a / b | a.div(b) |
| a % b | a.rem(b), a.mod(b) (deprecated) |
| a .. b | a.rangeTo(b) |

注意，从kotlin1.1开始支持rem函数，kotlin1.0使用mob函数

```kotlin
data class Counter(val dayIndex: Int) {
    operator fun plus(increment: Int): Counter {
        return Counter(dayIndex + increment)
    }
}
```

### in操作符
| 表达式 | 重载函数 |
| --- | --- |
| a in b | 	b.contains(a) |
| a !in b	| !b.contains(a) |

in和!in使用相同的函数，注意b为contains函数对象，a为函数参数

```kotlin
    var a = listOf(1,2,3)
    var b = 1
    if(b in a){
        println("b in a")
    }
```

### 索引访问操作符
| 表达式 | 重载函数 |
| --- | --- |
| a[j] | 	a.get(i) |
| a[i , j] | a.get(i, j) |
| a[i_1, ..., i_n] | 	a.get(i_1, ..., i_n) |
| a[i] = b	| a.set(i, b) |
| a[i, j] = b | a.set(i, j, b) |
| a[i_1, ..., i_n] = b	| a.set(i_1, ..., i_n, b) |

方括号被转换成使用适当数量参数的set和get函数调用

### invoke操作符
| 表达式 | 重载函数 |
| --- | --- |
| a() | 	a.invoke() |
| a(i) | a.invoke(i) |
| a(i, j) | 	a.invoke(i, j) |
| a(i_1, ..., i_n) | a.invoke(i_1, ..., i_n) |

圆括号被转换成使用适当数量参数的invoke函数调用

### Augmented assignments
| 表达式 | 重载函数 |
| --- | --- |
| a += b | 	a.plusAssign(b) |
| a -= b	| a.minusAssign(b) |
| a \*= b | 	a.timesAssign(b) |
| a /= b	| a.divAssign(b) |
| a %= b | 	a.remAssign(b), a.modAssign(b) (deprecated) |










