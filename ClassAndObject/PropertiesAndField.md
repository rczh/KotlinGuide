# Properties and Fields
## 属性声明
kotlin中使用var关键字声明变量，使用val关键字声明常量

### 属性定义格式
属性定义的完整语法格式：

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

如果能从初始值或者get方法的返回值中推断出类型，则可以省略属性类型

kotlin为每个属性提供默认的set和get方法，对于只读属性只能提供get方法，可变属性可以提供set和get方法。每次访问属性时get方法将被调用，每次为属性赋值时set方法将被调用

### 自定义set和get方法
用户在定义属性时可以自定义set和get方法

```kotlin
var stringRepresentation: String
    get() = this.toString()
    //通常情况下set方法的参数名为value，也可以为其他名字
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
    
//由于能从get方法返回值中推断属性类型，这里省略属性类型    
val isEmpty get() = this.size == 0  // has type Boolean
```

如果只修改set, get方法的可见性或者增加注解时，可以省略set,get方法的实现

```kotlin
var setterVisibility: String = "abc"
    private set // the setter is private and has the default implementation

var setterWithAnnotation: Any? = null
    @Inject set // annotate the setter with Inject
```

### 幕后字段
为了在set,get方法中访问属性本身，kotlin定义了幕后字段，用关键字field来表示。如果属性使用了至少一个默认的set,get方法实现，或者自定义的set,get方法中引用了field关键字，将为属性生成幕后字段

```kotlin
var counter = 0 // Note: the initializer assigns the backing field directly
    set(value) {
        if (value >= 0) field = value
    }

//不会为isEmpty生成幕后字段
val isEmpty: Boolean
    get() = this.size == 0
```

注意，field关键字只能在set,get方法中使用

### 幕后属性
对于不适合使用幕后字段的情况，可以自定义幕后属性

```kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```

## 编译时常量
对于在编译时已知的属性值，可以使用const关键字将其定义为编译时常量

编译时常量需要满足以下条件：
* Top-level成员，对象声明或者伴生对象的成员
* 使用原始类型或者字符串类型作为初始值
* 没有自定义get方法

可以在注解中使用编译时常量

```kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"
@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```

## 延迟初始化属性和变量
kotlin中可以使用lateinit关键字定义延迟初始化属性

```kotin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // dereference directly
    }
}
```

延迟初始化属性需要满足以下条件：

* 没有自定义set,get方法的var类型属性
* var属性需要为非空，并且类型不能为原始类型
* 类内部的成员变量，并且不能出现在主构造函数中
* top-level属性或者本地变量

注意，在初始化lateinit属性之前访问它将抛出一个异常

可以使用kotlin属性引用中的isInitialized字段来判断lateinit属性是否已经初始化

```kotlin
class Foo {
    lateinit var lateInitVar: String
    fun checkInit() {
        if(this::lateInitVar.isInitialized){

        }
    }
}
```
