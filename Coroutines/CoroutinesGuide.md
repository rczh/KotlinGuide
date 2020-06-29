# Coroutines Guide
作为一个语言，Kotlin在它的标准库中只提供最低等级的api，以使各种其他库能够使用协程。不同于很多其他具有类似功能的语言，async和await在Kotlin中不是关键字，甚至不是标准库的一部分。此外，Kotlin的挂起函数为异步操作提供了比futures和promises更安全，更少出错的抽象概念

kotlinx.coroutines是JetBrains为协程开发的一个丰富的库。它包含很多本教程涉及的高级协程原语，比如launch和async等

这是一个关于kotlinx.coroutines核心特性的教程，它包含一系列被划分成不同主题示例

为了按照教程中的示例来使用协程，你需要添加kotlinx-coroutines-core模块的依赖

## Table of contents

* Basics
* Cancellation and Timeouts
* Composing Suspending Functions
* Coroutine Context and Dispatchers
* Asynchronous Flow
* Channels
* Exception Handling and Supervision
* Shared Mutable State and Concurrency
* Select Expression (experimental)

## Additional references

* Guide to UI programming with coroutines
* Coroutines design document (KEEP)
* Full kotlinx.coroutines API reference
