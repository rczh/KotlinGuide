# Returns and Jumps
kotlin支持三种结构跳转表达式
* return

　　默认情况下，从最近的封闭函数或匿名函数返回

* break

　　终止最近的封闭循环

* continue

　　执行最近的封闭循环的下一个步

 以上三种跳转表达式能够作为其它表达式的一部分，它们的类型为Nothing类型
 
 ## 标签
 kotlin中的任何语句都可以加标签，标签的格式为：标识符@
 
 ```kotlin
 mylabel@ println("hello")
 ```
 
 ## Break label
 带有loop标签的break语句跳转到loop标签标识的for语句之后
 
 ```kotlin
 loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
 ```
 
 ## Return label
 ### 从最近的封闭函数返回
 默认情况下，嵌套在lambda表达式中的return语句从最近的封闭函数返回
 
 ```kotlin
 fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        //返回到foo方法的调用者
        if (it == 3) return // non-local return directly to the caller of foo()
        print(it)
    }
    println("this point is unreachable")
}
 ```
 
 ### 从lamdba表达式返回
  如果需要从lamdba表达式返回，可以使用标签标识lamdba表达式，然后使用带有标签的return语句返回到lambda表达式的调用者
 
 ```kotlin
 fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with explicit label")
}
 ```
 
 使用隐含标签从lamdba表达式返回，隐含标签的名称和使用lamdba表达式的函数名称相同
 
 ```kotlin
 fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with implicit label")
}
 ```
 
 ### 从匿名函数返回
 匿名函数中的return语句返回到匿名函数的调用者 
 
 ```kotlin
 fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // local return to the caller of the anonymous fun, i.e. the forEach loop
        print(value)
    })
    print(" done with anonymous function")
}
 ```
 
 ### 使用嵌套的lambda表达式模拟break
 嵌套lamdba表达式中带标签的return语句返回到run函数的调用者
  
 ```kotlin
 fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}
 ```
 
当return语句同时带有标签和返回值时，解析器优先解析标签
 
 ```kotlin
 //在标签a处返回1
 return@a 1
 ```
 
