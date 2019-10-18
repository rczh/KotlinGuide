# command line compiler
## 1.下载编译器
编译器下载地址：https://github.com/JetBrains/kotlin/releases

![kotlin-compiler.png](https://github.com/rczh/KotlinGuide/blob/master/Basic/kotlin_compiler.png)

## 2.创建hello.kt

```kotlin
fun main(args: Array<String>) {
    println("Hello, World!")
}
```

## 3.使用kotlin编译器编译程序

```
kotlinc hello.kt -include-runtime -d hello.jar
```

-include-runtime选项用来将kotlin运行库打包到jar中，通过java -jar hello.jar直接运行程序。如果jar包中不会包含kotlin运行库，只能通过kotlin -classpath hello.jar HelloKt运行程序

