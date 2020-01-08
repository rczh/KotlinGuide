# List Specific Operations
kotlin为列表提供了一组通过索引访问元素的操作

## 获取某个元素
使用get函数或者简写形式[index]可以通过索引访问列表元素，如果索引值超出列表大小get函数将抛出异常

可以使用getOrElse或者getOrNull函数避免抛出异常

* getOrElse函数可以传递一个lambda表达式，如果索引值超出集合范围，getOrElse函数将返回lambda表达式的结果

* 如果索引值超出集合范围，getOrNull函数将返回空

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.get(0))
    println(numbers[0])
    //numbers.get(5)                         // exception!
    println(numbers.getOrNull(5))             // null
    println(numbers.getOrElse(5, {it}))        // 5
}
```

## 获取部分元素
subList函数以列表形式返回一个基于原始列表元素的视图。如果原始列表元素发生变化，subList函数的结果也会发生变化

```kotlin
fun main() {
    val numbers = (0..13).toMutableList()
    var sub = numbers.subList(3, 6)
    println(sub)
    
    numbers.set(3, 1)
    println(numbers)
     //原始列表元素发生变化，导致sub结果也发生变化
    println(sub)
}
```

## 查找元素位置





