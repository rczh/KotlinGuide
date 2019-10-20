# Package and Import
## Package
kotlin包的定义在文件开始处，如果没有定义包kotlin将使用默认包

kotlin不需要匹配包和目录结构，源文件可以放在任意位置。但是为了兼容java规范，通常应该匹配包和目录结构

kotlin为每一个文件导入一组默认包
* kotlin.*
* kotlin.annotation.*
* kotlin.collections.*
* kotlin.comparisons.* (since 1.1)
* kotlin.io.*
* kotlin.ranges.*
* kotlin.sequences.*
* kotlin.text.*

## Import
如果导入的类名字有冲突，可以使用as关键字重命名冲突类

```kotlin
import org.example.Message // Message is accessible
import org.test.Message as testMessage // testMessage stands for 'org.test.Message'
```

## top-level declaration
kotlin支持声明top-level方法或者变量，方法和变量不必像java那样必须声明在类中

```kotlin
fun sum(a: Int, b: Int) = a + b

fun main() {
    println("sum of 19 and 23 is ${sum(19, 23)}")
}
```
