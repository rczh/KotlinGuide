# Opt-in Requirements
注意，opt-in注解@RequiresOptIn和@OptIn是实验性质的。@RequiresOptIn和@OptIn注解在1.3.70版本中被引入用来代替之前使用的@Experimental和@UseExperimental注解，同时，-Xopt-in编译选项用来代替-Xuse-experimental

Kotlin标准库提供了一种要求并且明确同意使用某些api元素的机制，该机制允许库开发人员将api中需要opt-in的特定条件告知用户，例如，一个API处于实验状态并且将来可能会改变

为了防止潜在的问题，编译器会就这些条件向此类api的用户发出警告，并要求他们在使用API之前进行选择

## Opting in to using API
如果库作者将库的API声明标记为需要opt-in，你应该明确同意在代码中使用它。有几种使用此类api的方法，所有方法都可以使用，没有任何技术限制。你可以自由选择最适合你的方式

### Propagating opt-in
当你在三方代码库中使用API时，你可以将API的opt-in需求传递到你的代码库中。使用API中的opt-in注解来注解你的声明，这样你可以在代码库中使用这个注解标注的API

```kotlin
// library code
@RequiresOptIn(message = "This API is experimental. It may be changed in the future without notice.")
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class MyDateTime // Opt-in requirement annotation

@MyDateTime                            
class DateProvider // A class requiring opt-in

// client code
fun getYear(): Int {  
    val dateProvider: DateProvider // Error: DateProvider requires opt-in
    // ...
}

@MyDateTime
fun getDate(): Date {  
    val dateProvider: DateProvider // OK: the function requires opt-in as well
    // ...
}

fun displayDate() {
    println(getDate()) // error: getDate() requires opt-in
}
```

正如你在本例中看到的，带注解的函数看起来是@MyDateTime API的一部分。因此，这样的opt-in注解将opt-in需求传递到三方代码库中。代码库的用户将看到同样的警告信息，并且也需要同意使用。如果需要使用多个opt-in API，使用他们所有的opt-in注解来标注声明

### Non-propagating use
在不公开自己API的模块中，比如应用程序，你可以使用opt-in api，而无需将opt-in需求传递到你的代码中。在这种情况下，使用@OptIn标注你的声明，并且将opt-in注解作为它的参数

```kotlin
// library code
@RequiresOptIn(message = "This API is experimental. It may be changed in the future without notice.")
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class MyDateTime // Opt-in requirement annotation

@MyDateTime                            
class DateProvider // A class requiring opt-in

//client code
@OptIn(MyDateTime::class)
fun getDate(): Date { // Uses DateProvider; doesn't propagate the opt-in requirement
    val dateProvider: DateProvider
    // ...
}

fun displayDate() {
    println(getDate()) // OK: opt-in is not required
}
```

当用户调用getDate函数时，他们不会被通知api中的opt-in需求

要在一个文件的所有函数和类中使用一个opt-in api，将文件级注解@file:OptIn添加到文件的顶部，包引用之前

```kotlin
//client code
 @file:OptIn(MyDateTime::class)
```

### Module-wide opt-in
如果你不想注解每个使用opt-in的api，你可以为整个模块引入它们。为了在模块中使用opt-in api，使用-Xopt-in参数编译模块，并且指定你使用的opt-in api的完整目录名。比如：-Xopt-in=org.mylibrary.OptInAnnotation

使用此参数编译的效果与模块中每个声明都使用注解@OptIn(OptInAnnotation::class)的效果相同

如果使用Gradle构建模块，你可以这样添加参数：

```kotlin
tasks.withType<KotlinCompile>().all {
    kotlinOptions.freeCompilerArgs += "-Xopt-in=org.mylibrary.OptInAnnotation"
}
```

如果你的Gradle模块是一个多平台模块，使用useExperimentalAnnotation方法：

```kotlin
sourceSets {
    all {
        languageSettings.useExperimentalAnnotation("org.mylibrary.OptInAnnotation")
    }
}
```

如果使用Maven构建模块：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-plugin</artifactId>
            <version>${kotlin.version}</version>
            <executions>...</executions>
            <configuration>
                <args>
                    <arg>-Xopt-in=org.mylibrary.OptInAnnotation</arg>                    
                </args>
            </configuration>
        </plugin>
    </plugins>
</build>
```

要在模块级别上加入多个opt-in api，可以为模块中使用的每个opt-in需求标注添加一个描述参数

## Requiring opt-in for API
### Opt-in requirement annotations
如果你需要用户明确同意使用你模块的API，你可以创建一个注解类作为opt-in注解。这个类必须使用@RequiresOptIn标注

```kotlin
@RequiresOptIn
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class MyDateTime
```

opt-in注解必须满足几个需求

* Retention必须为BINARY

* Target中没有EXPRESSION和FILE

* 没有参数

opt-in可以使用以下两个严重级别之一

* RequiresOptIn.Level.ERROR：opt-in的默认级别是强制性的。如果用户没有明确同意使用opt-in api，代码将无法编译

* RequiresOptIn.Level.WARNING：opt-in不是强制性的。如果用户没有明确同意使用opt-in api，编译器将产生一个警告

可以使用@RequiresOptIn注解的level参数设置所需要的级别

另外，你可以提供一个消息通知API用户关于使用API的特定条件。编译器将为使用API而没有提供opt-in的用户显示该消息

```kotlin
@RequiresOptIn(level = RequiresOptIn.Level.WARNING, message = "This API is experimental. It can be incompatibly changed in the future.")
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class ExperimentalDateTime
```

如果你发布了多个需要opt-in的独立功能，为每个功能声明一个注解。这使用户能更加安全的使用API。他们能仅仅使用他们明确接受的功能。你还可以从独立的功能中删除opt-in

### Marking API elements
为了使API具有opt-in，可以使用opt-in注解标注它的声明

```kotlin
@MyDateTime
class DateProvider

@MyDateTime
fun getTime(): Time {}
```

## Opt-in requirements for experimental APIs
如果你为在实验状态中的功能使用opt-in，请小心处理API升级以避免破坏客户端代码

一旦你的实验功能完成并以稳定的状态发布，请从声明中删除它的opt-in注解。客户端可以不受限制的使用它们。但是你应该在模块中保留opt-in注解类，以便和现有客户端代码保持兼容

为了让客户端相应地更新他们的代码(从代码中删除注解并重新编译)，将注解标记为@Deprecated并且在弃用消息中提供解释

```kotlin
@Deprecated("This opt-in requirement is not used anymore. Remove its usages from your code.")
@RequiresOptIn
annotation class ExperimentalDateTime
```

## Experimental status of the opt-in requirements
opt-in机制在kotlin 1.3中尚处于实验性质。在未来的版本中可能会以不兼容的方式进行更改

为了让使用@OptIn和@RequiresOptIn注解的用户意识到他们的实验状态，当编译带有这些注解的代码时编译器会发出警告。比如：This class can only be used with the compiler argument '-Xopt-in=kotlin.RequiresOptIn'

可以添加编译参数-Xopt-in=kotlin.RequiresOptIn来删除警告

