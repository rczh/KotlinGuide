# Sealed Classes
kotlin中的密闭类作为枚举类的扩展，用来表示类的层级结构

密闭类本身是抽象的，它能够有抽象的数据成员。密闭类没有公共的构造函数，它不能被直接实例化

## 声明密闭类
使用sealed关键字来声明密闭类，密闭类的子类必须和密闭类声明在同一个文件中，密闭类子类的继承类可以放在任何文件中

```kotlin
sealed class Expr
//从1.1开始，kotlin数据类能够继承其它类
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

## 密闭类的优点
密闭类的优点体现在使用when作为表达式时，如果能够确保已经覆盖了所有的情况，则不需要提供else语句

```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // the `else` clause is not required because we've covered all the cases
}
```
