# Collections Overview
kotlin支持以下集合类型：

* List是一个能够通过索引访问元素的有序集合，List中的元素可以重复

* Set是一个不包括重复元素的无序集合

* Map是一组key-value集合，其中每个key对应一个value。Map中的key是唯一的，value可以重复

## 集合类型
kotlin标准库实现了基本集合类型set,list,map，并为每个基本集合类型提供了两个接口

* 只读接口提供了读取集合元素的方法

* 可变接口继承于只读接口，提供了写入集合元素的方法

注意，只读集合类型默认是协变的，因为协变不能进行add操作，而只读集合类型不提供add方法。map集合类型只在value类型上是协变的

由于可变集合类型提供add方法，对于可变集合类型默认不支持协变

```kotlin
open class Shape
class Rectangle : Shape()

fun main(){
    var listShape: List<Shape>
    var listRect: List<Rectangle> = listOf()
    //只读集合默认支持协变
    listShape = listRect
    
    //可变集合默认不支持协变。可变集合提供了add方法，但是协变不能add
    var mutShapeList: MutableList<Shape>
    var mutRectList: MutableList<Rectangle> = mutableListOf<Rectangle>()
//    mutShapeList = mutRectList//编译错误
}
```

//TODO: 图

## List
List集合按顺序存储元素，并且提供索引访问集合元素。另外，List集合元素允许重复

如果两个List集合拥有同样的大小，并且在相同位置拥有equals相同的元素，则认为两个List是相等的

注意，这里的两个List相等相当于equals相等。如果判断两个List对象引用相等需要使用===

```kotlin
data class Person(var name: String, var age: Int)

fun main() {
    val bob = Person("Bob", 31)
    val people = listOf<Person>(Person("Adam", 20), bob, bob)
    val people2 = listOf<Person>(Person("Adam", 20), Person("Bob", 31), bob)
    //
    println(people == people2)
    bob.age = 32
    println(people == people2)
}
```

MutableList是一个带有写方法的List集合

```kotlin
fun main() {
    val numbers = mutableListOf(1, 2, 3, 4)
    numbers.add(5)
    numbers.removeAt(1)
    numbers[0] = 0
    numbers.shuffle()
    println(numbers)
}
```

kotlin中默认的List实现为ArrayList

## Set


