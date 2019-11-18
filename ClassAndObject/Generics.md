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

也就是说，协变只支持读操作，不支持写操作

### 数组是协变的
java中的数组默认是协变的，可以将String[]赋值给Object[]

```java
Fruit[] fruit = new Apple[10];
fruit[0] = new Apple();
fruit[1] = new Jonathan();
```

## 逆变
java中通过泛型类型参数? super E来声明接收泛型E或者E的父类型，这里的E为泛型类型的下界，也就是说List&lt;? super String>是List&lt;Object>的父类型，可以将List&lt;Object>赋值给List&lt;? super String>

带有下界的泛型类型参数使得泛型类型逆变

由于编译器无法确定传给apples参数的具体泛型类型，apples的实际参数类型可能是List&lt;Apple>或者List&lt;Fruit>，所以编译器可以正常执行add操作，将Apple或者Apple的子类添加到apples中

由于泛型类型参数? super Apple的下界为Apple，apples中的任何数据项都是Apple或者Apple的父类，所以编译器无法确定get操作返回结果的具体类型，get操作只能将结果赋值给Object对象

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

也就是说，逆变只支持写操作，不支持读操作

## PECS
PECS表示为Producer-Extends, Consumer-Super，协变只从生产者读，逆变只从消费者写

kotlin中通过使用out,in修饰符来定义协变和逆变

## 声明型型变
kotlin允许在类定义时对泛型类型声明型变，这种定义方式称为声明型型变

### 声明型协变
在定义类时通过泛型类型参数out T来声明泛型T是协变的，类中只能定义返回类型为T的读方法，不能定义参数为T的写方法，可以将Source&lt;String>赋值给Source&lt;Any>

```kotlin
interface Source<out T> {
    fun nextT(): T
    //fun add(t: T)// 编译错误
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
}
```

### 声明型逆变
同理，在定义类时可以通过泛型类型参数in T来声明泛型T是逆变的，类中只能定义参数为T的写方法，不能定义返回类型为T的读方法，可以将Comparable&lt;Number>赋值给Comparable&lt;Double>

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0)
    // Number is supertype of Double ,we can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

## 使用型型变
某些类中既需要参数为T的写方法，又需要返回类型为T的读方法，对于这样的类并不适合使用声明型型变

```kotlin
class Array<T>(val size: Int) {
    fun get(index: Int): T { ... }
    fun set(index: Int, value: T) { ... }
}
```

copy方法用来把from中的数据复制到to，由于无法使用声明型协变(Array&lt;T>是不变的)，当from的实际参数类型为Array&lt;Int>时会出现类型不匹配错误

```kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}

fun demo(){
    val ints: Array<Int> = arrayOf(1, 2, 3)
    val any = Array<Any>(3) { "" } 
    //copy(ints, any)
    //   ^ type is Array<Int> but Array<Any> was expected
}
```

通过在copy方法中使用out修饰符为from参数定义泛型类型协变，实现将Array&lt;Int>赋值给Array&lt;out Any>，这种定义方式称为使用型协变

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) { ... }
```

使用型协变也限制了copy方法中只能调用from对象中的读方法，从而避免在copy方法中对from对象的随意写入，这里的from对象不仅仅是一个普通的Array对象，而是一个受限制的投影对象，因此kotlin中的使用型型变也叫做类型投影

同理，可以在方法中使用in修饰符定义泛型类型逆变

注意，java中不支持声明型型变，只能够使用使用型型变

### 星型投影
kotlin能够为泛型类型定义星型投影

* 对于Foo&lt;out T : TUpper>类型，Foo&lt;*>相当于Foo&lt;out TUpper>

* 对于Foo&lt;in T>类型，Foo&lt;*>相当于Foo&lt;in Nothing>

* 对于Foo&lt;T : TUpper>类型，Foo&lt;*>相当于Foo&lt;out TUpper>

* 对于Foo&lt;out T>类型，Foo&lt;*>相当于Foo&lt;out Any?>

```kotlin
interface Foo<out T : Apple>{
//    fun set(t: T)// 编译错误
    fun get(): T
}
//Foo<*>相当于Foo<out Apple>
fun readFoo(from: Foo<*>){
    from.get()
}

interface Foo2<T : Apple>{
    fun set(t: T)
    fun get(): T
}
//Foo<*>相当于Foo<out Apple>
fun readFoo2(from: Foo2<*>){
    from.get()
//    from.set(Jonathan())// 编译错误
}
```

如果方法中定义了多个泛型类型参数，每个参数都可以独立投影

## 泛型函数
kotlin中可以定义泛型函数，泛型类型放在函数名之前

```kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

fun <T> T.basicToString(): String {  // extension function
    // ...
}
```

调用泛型函数时需要在函数名后面指定泛型类型参数，如果编译器能够推断出泛型类型时可以省略泛型类型参数

```kotlin
val l = singletonList<Int>(1)

val m = singletonList(1)
```

## 泛型约束
如果所有泛型类型集合都能够被一个特定类型参数替换，这个类型参数称为泛型约束。最常见的约束类型为上界

### 上界
定义泛型时可以在冒号后面指定上界。如果没有指定，默认上界为Any?

```kotlin
fun <T : Comparable<T>> sort(list: List<T>) {  ... }

sort(listOf(1, 2, 3)) // OK. Int is a subtype of Comparable<Int>
sort(listOf(HashMap<Int, String>())) //编译错误，HashMap不是Comparable的子类型
```

一个泛型类型只能指定一个上界。如果一个泛型类型需要指定多个上界，需要使用where语句

```kotlin
//泛型类型T必须是CharSequence和Comparable的子类
fun <T> copyWhenGreater(list: List<T>, threshold: T): List<String>
    where T : CharSequence,
          T : Comparable<T> {
    return list.filter { it > threshold }.map { it.toString() }
}
```

## 泛型类型擦除
Kotlin仅在编译时对泛型类型执行类型安全检查，在运行时泛型类型不包含实际类型的任何信息，类型信息将被擦除。Foo<Bar>和Foo<Baz>在运行时被擦除为Foo<*>


















