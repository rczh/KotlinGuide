# Basic Types
kotlin中一切都是对象

## 一. 数值
### 1.整数
kotlin提供了四种数值类型来表示整数

类型|位数|最小值|最大值
---|:--:|---:|---:
Byte|	8	 |-128	|127
Short|16|	-32768|	32767
Int|	32|	-2,147,483,648  |	2,147,483,647 
Long|	64|	-9,223,372,036,854,775,808 |9,223,372,036,854,775,807

kotlin能够自动推断数值类型，如果初始值不超过Int的最大值则类型为Int，超过时类型为Long。

可以添加后缀L指定变量为Long类型

```kotlin
val one = 1 // Int
val threeBillion = 3000000000 // Long
val oneLong = 1L // Long
```

### 2.浮点数
kotlin提供两种类型浮点数

类型|位数|有效位|指数位|小数位
---|:--:|:--:|:--:|---:
Float	|32|	24|	8|	6-7
Double|	64|	53|	11	|15-16

使用小数形式初始化时kotlin默认推断类型为Double。可以通过后缀f或F明确指定类型为Float，如果指定的Float小数值超过6位则四舍五入

```kotlin
val pi = 3.14 // Double
val eFloat = 2.7182818284f // Float, actual value is 2.7182817
```

kotlin不支持自动的类型转换，不能将Float、Int值自动转换为Double类型

```kotlin
fun main() {
    fun printDouble(d: Double) { print(d) }

    val i = 1    
    val d = 1.1
    val f = 1.1f 

    printDouble(d)
//    printDouble(i) // Error: Type mismatch
//    printDouble(f) // Error: Type mismatch
}
```

### 3.数值常量
整数常量支持二进制、十进制和十六进制表示

- 二进制
0b00001011
- 十进制
123
- 十六进制
0x0F

注意，kotlin不支持八进制表示

可以使用下划线使数值常量更具有可读性

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

浮点数常量支持科学计数法

```
123.5e10
```

### 4.数值装箱
默认情况下数值被存储在JVM原始类型中，当使用可空类型时数值会被装箱

```kotlin
val a: Int = 10000
println(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
println(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
```

使用===会比较两个装箱对象，由于两个装箱对象地址不同所以结果为false

```kotlin
val a: Int = 10000
println(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
println(boxedA == anotherBoxedA) // Prints 'true'
```

使用==时编译器仅仅比较两个原始类型值

### 5.显示转换
kotlin不支持自动将小类型转换为大类型，下面的操作是非法的

```kotlin
fun main() {
    val b: Byte = 1 // OK, literals are checked statically
    val i: Int = b // ERROR
}
```

可以使用Byte.toInt方法明确的将Byte转换成Int

```kotlin
fun main() {
    val b: Byte = 1
    val i: Int = b.toInt() // OK: explicitly widened
    print(i)
}
```

### 6.运算符
kotlin支持标准的数字算数运算符，这些运算符对应相应类中的方法

注意，kotlin中没有单独的位运算符，位运算符是以中缀形式命名的方法

```kotlin
val x = (1 shl 2) and 0x000FF000
```

### 7.浮点数比较
如果已知操作数a和b为Float或者Double类型时，浮点数的比较遵循浮点算数标准。否则使用Float或者Double类的equals和compareTo方法进行比较

## 二. 字符
kotlin使用Char类型表示字符，在kotlin中字符不能直接作为数值使用，只能通过toInt方法进行转换

```kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```

使用单引号来表示字符'1'，特殊字符可以使用反斜杠转义\t, \b, \n, \r, \\', \\", \\\\, \\$

## 三. 布尔值
kotlin使用Boolean类型表示布尔值，支持三种操作符：||，&&，!

## 四. 数组
kotlin中使用Array类来表示数组，Array类中包含set和get方法，[]操作符会重载为相应的set和get方法

创建数组的几种方式：
* 使用arrayOf方法
```kotlin
var a = arrayOf(1, 2, 3)
```

* 使用arrayOfNulls方法
```kotlin
//create an array of a given size filled with null elements
var b = arrayOfNulls<Int>(3)
```

* 使用Array构造方法
```kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5) { i -> (i * i).toString() }
asc.forEach { println(it) }
```

kotlin中数组是不变的，不能将Array&lt;String>数组赋值给Array&lt;Any>

kotlin也提供了原始类型数组，比如ByteArray, ShortArray, IntArray等等，这些数组并不继承Array类，但是它们包含和Array类相同的方法和属性

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)

// Array of int of size 5 with values [0, 0, 0, 0, 0]
val arr = IntArray(5)
```

## 五. 无符号整形
注意，无符号整形目前处于实验状态，并且仅在kotin1.3中可用

无符号类型支持大部分有符号操作符

类型|位数|最大值
---|:--:|---:
kotlin.UByte|8| 255
kotlin.UShort|16|65535
kotlin.UInt|32| 2^32 - 1
kotlin.ULong|64|2^64 - 1

kotlin也提供原始类型的无符号数组，比如UByteArray,UShortArray,UIntArray,ULongArray

### 无符号常量
kotlin使用后缀u或者U来表示无符号常量，如果没有提供具体类型，kotlin将根据无符号常量值的大小来选择UInt或者ULong类型

```kotlin
val a1 = 42u // UInt: no expected type provided, constant fits in UInt
val a2 = 0xFFFF_FFFF_FFFFu // ULong: no expected type provided, constant doesn't fit in UInt
val a = 1UL // ULong, even though no expected type provided and constant fits into UInt
```

## 六. 字符串
kotlin使用String表示字符串，字符串是不变的，字符串中的元素是可以通过索引操作符s[i]访问的字符

可以使用for循环遍历字符串

```kotlin
fun main() {
val str = "abcd"
    for (c in str) {
        println(c)
    }
}
```

可以使用加号运算符连接字符串，运算符的第一个元素必须是字符串

```kotlin
fun main() {
    val s = "abc" + 1
    println(s + "def")
}
```

### 字符串常量
kotlin支持两种类型的字符串常量，转义字符串和raw原始字符串

raw字符串使用"""表示，raw字符串不支持转义字符，可以包含新行和任意其他字符

```kotlin
val s = "Hello, world!\n"
val text = """
    for (c in "foo")
        print(c)
"""
```

### 字符串模板
字符串常量能够包含模板表达式，模板表达式为$变量名，或者${表达式}形式

```kotlin
val i = 10
println("i = $i") // prints "i = 10"
val s = "abc"
println("$s.length is ${s.length}") // prints "abc.length is 3"
```

转义字符串和raw字符串都可以使用模板表达式，由于raw字符串不支持转义字符，可以使用模板表达式来显示$字符

```kotlin
val price = """
${'$'}9.99
"""
```


