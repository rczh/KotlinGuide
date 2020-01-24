# This Expression
kotlin中使用this表达式表示当前接收类型对象

* 在类的成员函数中，this表示当前的类对象

* 在扩展函数或者带有接收类型的字面函数中，this表示点左侧的接收类型对象

默认情况下不带标签的this引用最内部的密闭作用域，使用带标签的this可以引用其他作用域

## Qualified this
使用this访问外部作用域，比如类、扩展函数、带有标签和接收类型的字面函数，可以使用this@label形式，其中lable为外部作用域的标签

```kotlin
class A { // implicit label @A
    inner class B { // implicit label @B
        fun Int.foo() { // implicit label @foo
            val a = this@A // A's this
            val b = this@B // B's this

            //默认情况下，this引用最内部作用域，这里为foo函数的接收类型Int
            val c = this // foo()'s receiver, an Int
            val c1 = this@foo // foo()'s receiver, an Int

            val funLit = lambda@ fun String.() {
                val d = this@lambda // funLit's receiver
            }
            
            //由于lambda表达式没有任何接收类型对象，这里this引用foo函数的接收类型Int
            val funLit2 = { s: String ->
                // foo()'s receiver, since enclosing lambda expression
                // doesn't have any receiver
                val d1 = this
            }
        }
    }
}
```

