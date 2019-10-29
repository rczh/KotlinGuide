# interface
## 接口定义
kotlin中使用interface关键字定义接口，接口中可以包含抽象方法和方法实现。一个类能够实现多个接口

```kotlin
interface MyInterface {
    //默认接口中的方法为抽象方法
    fun bar()
    fun foo() {
      // optional body
    }
}

class Child : MyInterface {
    override fun bar() {
        // body
    }
}
```

接口中可以定义抽象属性或者为属性提供访问器实现，访问器实现中不能包含幕后字段

```kotlin
interface MyInterface {
    val prop: Int // abstract
    
    //get方法中不能包含幕后字段
    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```

## 接口继承
接口能够从其他接口继承，子接口可以为父接口提供实现并且定义新的属性和方法

```kotlin
interface Named {
    val name: String
}

interface Person : Named {
    val firstName: String
    val lastName: String
    
    override val name: String get() = "$firstName $lastName"
}

data class Employee(
    // implementing 'name' is not required
    override val firstName: String,
    override val lastName: String,
    val position: Position
) : Person
```

## 解决冲突
当从多个接口继承时，多个接口可能定义相同的方法或者属性，子类必须重载这些属性或者方法并且明确指定调用哪个接口实现


```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        //明确指定调用B接口bar方法
        super<B>.bar()
    }
}
```



