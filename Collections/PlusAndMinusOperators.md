# plus and minus Operators
kotlin为集合定义了plus和minus操作符，plus和minus操作符的第一个操作数为集合，第二个操作数可以为元素或者另一个集合，返回结果为一个新的只读集合

## plus操作符
plus操作符的结果包括原始集合元素和第二个操作数元素

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
    val plusList = numbers + "five"
    println(plusList)
}
```

## minus操作符
minus操作符的结果包含原始集合元素减去第二个操作符元素

* 如果减数是一个元素，只从原始集合中删除该元素的第一个匹配项

* 如果减数为一个集合，集合中的所有元素都会从原始集合中删除

```kotlin
fun main(){
    var numbers = listOf("one", "two", "three", "four", "three", "four")
    val minusList = numbers - listOf("three", "four")
    val minusElement = numbers - "three"
    println(minusList)
    println(minusElement)
}
```

kotlin也为集合定义了plusAssign和minusAssign操作符

* 对于只读集合，自增和自减操作符相当于先执行plus和minus操作，然后将结果重新赋值给变量。因此只能对var类型只读集合执行自增和自减操作

* 对于可写集合，自增和自减操作会更新原始集合。只能对val类型可写集合执行自增和自减操作，对var类型可写集合执行自增和自减操作会引起歧义产生编译错误

```kotlin
fun main(){
    /**
     *  1. val, 只读集合：编译错误，无法将结果赋值
     *  2. var, 只读集合：ok
     *  3. val, 可写集合: ok，修改原有集合
     *  4. var, 可写集合：编译错误，编译器无法判断使用plus，或者plusAssign
     */
    var numbers = listOf("one", "two")
    numbers += "six"
    println(numbers)
    
    val mutNumber = mutableListOf("one", "two")
    mutNumber += "three"
    println(mutNumber)
}
```

