# basic type
kotlin中一切都是对象

## 数值
### 1.整数
kotlin提供四种类型表示整数

类型|大小bit|最小值|最大值
---|:--:|:--:|---:
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

类型|大小bit|有效位|指数位|小数位
---|:--:|:--:|:--:|---:
Float	|32|	24|	8|	6-7
Double|	64|	53|	11	|15-16

使用小数形式初始化时kotlin默认推断类型为Double。可以通过后缀f,F明确指定类型为Float，如果指定的Float小数值超过6位则四舍五入

```kotlin
val pi = 3.14 // Double
val eFloat = 2.7182818284f // Float, actual value is 2.7182817
```

kotlin不支持自动的类型转换，不能将Float，Int值自动转换为Double类型

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
整数常量表示支持十进制，二进制和十六进制

- 十进制
123
- 十六进制
0x0F
- 二进制
0b00001011

kotlin不支持八进制表示

可以使用下划线使数值常量更加可读

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

浮点数常量支持科学计数法

123.5e10

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

## 2.字符
