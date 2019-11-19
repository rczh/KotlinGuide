# Nested and Inner Classes
## 嵌套类
kotlin中类能够嵌套在另一个类中，嵌套类不能够访问外部类的成员

```kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
        fun t(){
            //bar = 3//编译错误
        }
    }
}
```

## 内部类
使用inner关键字标识的嵌套类称为内部类，内部类可以访问外部类的成员

```kotlin
class Outer {
    private val bar: Int = 1
    
    inner class Inner {
        fun foo() = bar
    }
}
```

## 匿名内部类
kotlin中使用object关键字创建匿名内部类

```kotlin
fun main(){
    var i = 2
    //匿名内部类可以访问外部成员
    addClickListener(object : OnClickListener{
        override fun onClick() {
            i = 5
        }
    })
}
```

对于java函数式接口，可以使用lambda表达式创建对象

```
public interface OnClickListener {
    public void onClick();
}

fun addClickListener(listener: OnClickListener){
    listener.onClick()
}

fun main(){
    var i = 2
    //使用lamdba表达式创建OnClickListener对象
    var listener = OnClickListener{ i = 6 }
    addClickListener(listener)
}
```
