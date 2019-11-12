# Generics
kotlin中的类能够定义泛型

```kotlin
class Box<T>(t: T) {
    var value = t
}

//创建对象时需要提供类型参数
val box: Box<Int> = Box<Int>(1)

//如果编译器能够推断出泛型类型，可以省略类型参数
val box = Box(1)
```

## 不变
默认情况下，java中的泛型类型是不变的，也就是说List&lt;String>并不是List&lt;Object>的子类型，不能将List&lt;String>赋值给List&lt;Object>

```java
List<String> strs = new ArrayList<String>();

//编译错误
List<Object> objs = strs;
```

## 协变
java中通过泛型类型参数? extends E来声明接收泛型E或者E的子类型，这里的E为泛型类型的上界，也就是说List&lt;String>是List&lt;? extends Object>的子类型

带有上界的泛型类型参数使得泛型类型协变

```java
class Fruit {}
class Apple extends Fruit {}
class Jonathan extends Apple {}
class Orange extends Fruit {}
```

对于

```java


    /**
     * 协变支持get读,不支持add写
     * @param apples
     */
    static void readFrom(List<? extends Apple> apples) {
//        apples.add(new Apple());  // 编译错误
//        apples.add(new Fruit());  // 编译错误
//        apples.add(new Object());  // 编译错误

        //向上get
        Apple apple = apples.get(0);
//        Jonathan jonathan = apples.get(0);  // 编译错误
        Fruit fruit = apples.get(0);
    } 

readFrom(new ArrayList<Jonathan>());
```

java中的数组默认是协变的

## 逆变
