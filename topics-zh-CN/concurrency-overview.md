[//]: # "title: Concurrency overview"
[//]: # "auxiliary-id: Concurrency_Overview"

当你把 Android 的开发经验套用在 Kotlin 移动多平台（KMM）时，会发现在 iOS 平台是不同的状态和并发模型。也就是 Kotlin/Native 模型。[Kotlin/Native](https://kotlinlang.org/docs/reference/native-overview.html) 是一种将 Kotlin 代码编译为原生二进制文件的技术，可以在没有虚拟机的情况下运行，例如在 iOS 上运行。

众所周知，不加限制地让多个线程同时使用可变内存有风险且易出错。例如 Java， C++ 和 Swift/Objective-C 这些语言，允许多个线程不受限制的访问同一状态。并发问题与其他编程问题不同，常常很难复现。在本地开发时也许没有问题，或者只是零星出现。有时它只在生产环境下，加以一定负载才出现。

总之，就算测试通过了，也不意味着你的代码没有问题。

当然，也不是所有语言都采用这样设计。Javascript 简单的禁止了并发访问同一状态。另一个极端是 Rust，它提供语言级别的并发和状态管理，这使得它非常流行。

## <a name="rules-for-state-sharing"/>状态共享的规则 

Kotlin/Native 为线程间共享状态引入了一些规则。希望通过这些规则，防止不安全的共享访问可变状态。如果你有 JVM 的开发背景还写过并发代码，你可能需要改变数据架构的方式，但是这可以让你在没有风险副作用的情况下，实现相同的效果。

这里需要强调一下，有[绕过这些规则的方法](concurrent-mutability.md)。我们希望让你尽量避免绕开这些规则，条件允许的话。

关于状态和并发只有两个简单的规则。

### 规则 1： 可变状态 == 单线程

如果你的状态是可变的，那么它同一时间只对一个线程 _可见_。任何在 Kotlin 中正常使用的普通类状态都会被 Kotlin/Native 运行时当作 _可变的_ 。如果你没有使用并发，Kotlin/Native 的行为与其他 Kotlin 代码相同，除了[全局状态](#global-state)。

```kotlin
data class SomeData(var count:Int)

fun simpleState(){
    val sd = SomeData(42)
    sd.count++
    println("My count is ${sd.count}") // 结果是 43
}
```

如果仅有一个线程，你不会遇到并发问题。技术上，这被称为 _线程限制_ ，意味着你不能在后台线程修改 UI。Kotlin/Native 的状态规则为所有线程确立了这个概念。

### 规则 2: 不可变状态 == 多线程

如果一个状态不能被改变，多线程可以安全地访问它。在 Kotlin/Native 中，_不可变_ 并不是说所有东西都是 `val` 的。它意味着 _冻结（frozen）状态_

## 不可变和冻结状态

下面的示例被定义成不可变的 — 它有两个 `val` 元素，都是 final 不可变类型。

```kotlin
data class SomeData(val s:String, val i:Int)
```

接下来这个示例可能是不可变的，也可能是可变的。`SomeInterface` 内部的操作在编译期是未知的。Kotlin 不能在编译期静态地确定深层不可变性。

```kotlin
data class SomeData(val s:String, val i:SomeInterface)
```

Kotlin/Native 需要验证状态的某些部分在运行时确实是不可变的。运行时可以简单地遍历整个状态，依次验证其每一部分的深度不可变性，但这样做很死板。并且，如果每次运行时想要检查可变性时，都要这样做的话，会带来严重的性能问题。

Kotlin/Native 定义了一个新的运行时状态 _冻结（frozen）_。一个对象的任一实例都可能被冻结。如果一个对象是冻结的：

1. 你不能改变其状态的任何部分。试图这样做会触发运行时异常：`InvalidMutabilityException`。一个被冻结的对象实例，百分百是运行时验证过不可变的。
2. 它所引用的一切也都是被冻结的。它所引用的所有其他对象都被保证是冻结状态。也就是说当运行时需要确定一个对象是否可以与其他线程共享时，只需要检查该对象是否被冻结。如果是的话，整个图也被冻结了，可以安全地共享。

Native 运行时给所有的类添加了一个扩展函数 `freeze()`。调用 `freeze()` 函数会冻结一个对象，并递归地冻结该对象所引用的内容。

```kotlin
data class MoreData(val strData: String, var width: Float)
data class SomeData(val moreData: MoreData, var count: Int)
//...
val sd = SomeData(MoreData("abc", 10.0), 0)
sd.freeze()
```

![Freezing state](freezing-state.png){animated="true"}

* `freeze()` 是一个单向操作。你没法 _解冻（unfreeze）_。
* `freeze()` 在共享的 Kotlin 代码中不可用，不过有几个库提供了 expect 和 actual 声明，以便在共享的代码里使用。不过，如果你使用了一个并发库，例如 [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines)，它可能会自动冻结跨越线程边界的数据。

`freeze` 并不是 Kotlin 特有的。你可以在 [Ruby](https://www.honeybadger.io/blog/when-to-use-freeze-and-frozen-in-ruby/) 和 [Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) 里找到它。

## <a name="global_state"/>全局状态

Kotlin 允许定义全局可用的状态。如果只是简单地保留可变性，这个全局状态就会违反 [_规则 1_](#rule-1-mutable-state-1-thread)。
为了符合 Kotlin/Native 的状态规则，全局状态有一些特殊条件。这些条件可以冻结状态，或者使其只对单个线程可见。

### 全局 `对象`

全局 `对象` 默认是冻结的。也就是所有线程都能访问，不过它是不可变的。以下代码不能正常工作。

```kotlin
object SomeState{
    var count = 0
    fun add(){
        count++ // 这里会抛异常
    }
}
```

尝试修改 `count` 会触发异常，因为 `SomeState` 是被冻结的（它所引用的所有数据也是被冻结的）。

你可以把全局对象变成线程 _局部的_，这就允许它是可变的，每一个线程持有一份状态的拷贝。
对它使用 `@ThreadLocal` 注解。

```kotlin
@ThreadLocal
object SomeState{
    var count = 0
    fun add(){
        count++ //👍
    }
}
```

不同线程读取到的 `count` 是不同的，因为每个线程都有自己的副本。

全局对象的规则同样适用于伴生对象。

```kotlin
class SomeState{
    companion object{
        var count = 0
        fun add(){
            count++ // 这里会抛异常
        }
    }
}
```

### 全局属性

全局属性是一个特例，*它们只能被主线程访问*，是可变的。从其他线程访问全局属性会抛异常。

```kotlin
val hello = "Hello" // 只有主线程可见
```

你可以对全局属性使用以下注解：

* `@SharedImmutable`，标记为全局可见，不过是冻结状态。
* `@ThreadLocal`，每个线程持有一份拷贝。

这条规则适用于拥有支持字段（backing filed）的全局属性。计算属性和全局函数不受主线程的限制。

## 并发模型的现状和未来

Kotlin/Native 的并发规则需要对架构设计进行一些调整，但在库和新的最佳实践的帮助下，日常开发基本不会受到影响。实际上，遵循 Kotlin/Native 并发规则的跨平台代码会让 Kotlin 移动多平台（KMM）应用的并发性更加安全。你可以从这个[上手教程](https://play.kotlinlang.org/hands-on/Kotlin%20Native%20Concurrency/)开始尝试 Kotlin/Native 并发。

Kotlin 移动多平台（KMM）应用中，Android 和 iOS 遵循各自平台的状态规则。一些大型应用的开发团队，会共享特别具体的功能代码，常在宿主平台进行并发性管理。这种情况需要直接冻结 Kotlin 返回的状态，但除此之外，还是简单直接的。

一个更广泛的模型，即在 Kotlin 中管理并发，宿主平台在其主线程上与共享代码进行通信，从状态管理的角度来看，这种模式更简单。使用 [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines)之类的并发库，可以帮助自动化冻结。你也可以利用[协程](https://kotlinlang.org/docs/reference/coroutines/coroutines-guide.html)，共享更多的代码来提升效率。

然而，目前 Kotlin/Native 的并发模型仍有一些不足之处。例如，移动开发者已经习惯了在线程之间自由地共享状态，并开发了一系列的方法和架构模式避免数据竞争。虽然使用 Kotlin/Native 可以在不阻塞主线程的情况下编写高效应用程序，但是学习成本过高。

所以我们正在为 Kotlin/Native 开发一个全新的内存管理和并发模型，以便消除这些弊端。了解更多关于[我们的进展](https://blog.jetbrains.com/kotlin/2020/07/kotlin-native-memory-management-roadmap/)。

_This material was prepared by [Touchlab](https://touchlab.co/) for publication by JetBrains._
