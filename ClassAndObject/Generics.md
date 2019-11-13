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
java中通过泛型类型参数? extends E来声明接收泛型E或者E的子类型，这里的E为泛型类型的上界，也就是说List&lt;String>是List&lt;? extends Object>的子类型，可以将List&lt;String>赋值给List&lt;? extends Object>

带有上界的泛型类型参数使得泛型类型协变

```java
class Fruit {}
class Apple extends Fruit {}
class Jonathan extends Apple {}
class Orange extends Fruit {}
```

由于编译器无法确定传给apples参数的具体泛型类型，apples的实际参数类型可能是List&lt;Apple>或者List&lt;Jonathan>，所以编译器无法执行add操作

由于泛型类型参数? extends Apple的上界为Apple，apples中的任何数据项都是Apple或者Apple的子类，所以编译器可以正常执行get操作，get操作的返回结果可以赋值给Apple或者Apple的父类

```java
static void readFrom(List<? extends Apple> apples) {
    //apples.add(new Apple());  // 编译错误
    //apples.add(new Fruit());  // 编译错误
    //apples.add(new Object());  // 编译错误

    //向上get
    Apple apple = apples.get(0);
    //Jonathan jonathan = apples.get(0);  // 编译错误
    Fruit fruit = apples.get(0);
} 

readFrom(new ArrayList<Jonathan>());
```

就是说，协变只支持读操作，不支持写操作

java中的数组默认是协变的
//TODO:

## 逆变
java中通过泛型类型参数? super E来声明接收泛型E或者E的父类型，这里的E为泛型类型的下界，也就是说List&lt;? super String>是List&lt;Object>的父类型，可以将List&lt;Object>赋值给List&lt;? super String>

带有下界的泛型类型参数使得泛型类型逆变

由于编译器无法确定传给apples参数的具体泛型类型，apples的实际参数类型可能是List&lt;Apple>或者List&lt;Fruit>，所以编译器无法确定get操作返回结果的具体类型，get操作只能将结果赋值给Object对象

由于泛型类型参数? super Apple的下界为Apple，apples中的任何数据项都是Apple或者Apple的父类，所以编译器可以正常执行add操作，将Apple或者Apple的父类数据项添加到apples中

```java
static void writeTo(List<? super Apple> apples) {
    //向下add
    apples.add(new Apple());
    apples.add(new Jonathan());
    //apples.add(new Fruit());  // 编译错误

    //只能get Object
    Object obj = apples.get(0);
    //Fruit fruit = apples.get(0); // 编译错误
}

writeTo(new ArrayList<Fruit>());
```

就是说，逆变只支持写操作，不支持读操作

## PECS



