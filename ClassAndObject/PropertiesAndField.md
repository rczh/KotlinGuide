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




