[//]: # "title: Concurrent mutability"
[//]: # "auxiliary-id: Concurrent_Mutability"

When it comes to working with iOS, [Kotlin/Native's state and concurrency model](concurrency-overview.md) has [two simple rules](concurrency-overview.md#rules-for-state-sharing).

对于iOS系统而言，Kotlin/Native的[状态共享和并发模型](concurrency-overview.md)的[两个规范](concurrency-overview.md#rules-for-state-sharing)可以简单归纳为如下：

1. 一个可变的非冻结的状态同一时刻只对一个线程可见
2. 一个不可变得冻结的状态可以被多线程并发访问

基于这两个规范，开发者不要尝试修改[global state](concurrency-overview.md#global-state)，也不要尝试在多个线程中去并发修改同一共享状态。针对这些并发可变性状态的使用场景，很多时候只需要修改代码设计就可以规避。比如JVM虽然支持在多线程中并发修改可变状态，但开发者需要在使用时考虑这些用法是否必要。

当然在很多场景中，不可避免的需要实现任意线程对状态的并发访问，比如一个 _service_ 对象可能在整个应用内被其他多个模块使用，再比如通过代码设计规避并发可变性成本高昂。所以不论什么原因，现实中将可变状态限制为单线程访问是不现实的。

针对这些不可避免的并发使用场景，有多种解决方案可供选择，当然开发者需要根据这些方案的优劣结合现实情况进行选择：

- [Atomics](#atomics)
- [Thread-isolated states](#thread-isolated-state)
- [Low-level capabilities](#low-level-capabilities)

## Atomics

Kotlin/Native提供了一系列原子类，针对Kotlin/Native运行时通过内部的特殊处理，支持在其被冻结的同时可以对其内部持有的对象进行修改。Kotlin/Native运行时提供了一些Atomics的相关变体，开发者可以直接使用这些变体或者通过引入一些依赖库进行扩展。

比如官方提供的原子库 [`kotlinx.atomicfu`](https://github.com/Kotlin/kotlinx.atomicfu)，kotlinx.atomicfu目前还处于早期版本的实验阶段，更多在内部使用。除了这个官方库外，还可以使用[Stately](https://github.com/touchlab/Stately)，Stately是由[Touchlab](https://touchlab.co)开发的专用于Kotlin/Native并发的实用库，具有较好的跨平台兼容性。

### `AtomicInt`/`AtomicLong`

首先介绍两个基础数据类型的原子实现： `AtomicInt` 和 `AtomicLong`。开发者可以使用它们来获得可以正确进行多线程读写的 `Int` 或 `Long`。

```kotlin
object AtomicDataCounter {
    val count = AtomicInt(3)
  
    fun addOne() {
        count.increment()
    }
}
```

示例代码定义了一个全`glabal object` , 通常全局对象在Kotlin/Native中是冻结的，但在示例中 `count` 可以通过 `AtomicInt` 的 `increment()` 方法修改内部持有的数值，而且该操作可以在任意线程完成。

### `AtomicReference`

`AtomicReference` 持有一个对象的实例，该实例本身是冻结的, 但是`AtomicReference` 可以修改具体持有哪个对象实例 。在 Kotlin/Native 直接修改全局对象并不生效，如下示例代码:

```kotlin
data class SomeData(val i: Int)

object GlobalData {
    var sd = SomeData(0)

    fun storeNewValue(i: Int) {
        sd = SomeData(i) //Doesn't work
    }
}
```

根据 [全局状态规范](concurrency-overview.md#global-state),  `global object` 在 Kotlin/Native中是冻结的，因此对 `sd` 的修改将会失败，此时可以借助 `AtomicReference` 达成目的:

```kotlin
data class SomeData(val i: Int)

object GlobalData {
    val sd = AtomicReference(SomeData(0).freeze())

    fun storeNewValue(i: Int) {
        sd.value = SomeData(i).freeze()
    }
}
```

 `AtomicReference` 本身是冻结的，这让它可以进行全局定义，而上方示例代码中的 `AtomicReference` 持有的对象也要求是冻结的。在跨平台库中数据可以自动冻结，但在Kotlin/Native运行时中则需要开发者主动调用 `freeze()` 进行冻结。 

`AtomicReference` 对于多线程进行状态共享非常有用，但是开发者需要进行权衡，因为获取和修改`AtomicReference` 相对于标准实现会带来额外的性能损耗。如果开发者对性能敏感，可以考虑下一小节的 [thread-isolated state](#thread-isolated-state)。此外，开发者还需要考虑潜在的内存泄漏风险，因为通过全局的 `AtomicReference` 持有对象可能存在循环引用，而开发者不能及时清理就会带来内存泄漏问题:

* 如果 `AtomicReference` 中可能存在循环引用，开发者可以在使用完成后显式设置其值为null。
* 当然如果开发者可以保证 `AtomicReference` 被持有和使用能贯穿整个生命周期，就无需担心内存泄漏，因为此时该全局对象生存周期本身就和进程的生命周期匹配。

```kotlin
class Container(a:A) {
    val atom = AtomicReference<A?>(a.freeze())

    /**
     * Call when you're done with Container
     */
    fun clear(){
        atom.value = null
    }
}
```

最后，这里还有一个一致性问题需要尽心探讨。 `AtomicReference` 可以保证`get/set` 进行对象获取和写入本身是原子的，但是后续的操作逻辑就无法保证其一致性。比如通过  `AtomicReference` 持有一个 `list` 对象，开发者希望每次添加新数据时先进行查找，以此保证不会添加重复数据。而这一系列操作无法通过原子类来保证数据一致性， 因为多线程情景下，这一系列代码对应的指令可能会被多个线程交叉执行。此时就需要引入其他并发机制进行保护，比如通过某种形式的锁或者类似GPU中的栅栏逻辑。

下方示例代码无法保证多线程场景下`list` 不存在重复数据，也即存在一致性问题:

```kotlin
object MyListCache {
    val atomicList = AtomicReference(listOf<String>().freeze())
    fun addEntry(s:String){
        val l = atomicList.value
        val newList = mutableListOf<String>()
        newList.addAll(l)
        if(!newList.contains(s)){
            newList.add(s)
        }
        atomicList.value = newList.freeze()
    }
}
```

## Thread-isolated state

正如规则一揭示的那样，Kotlin/Native要求一个可变状态只能对一个线程可见。通过Atomics支持，可变状态可以被多线程访问。另外一种解决思路是：将可变状态隔离到单个线程，其余线程通过线程间通信操作获取该状态。

线程隔离状态的设计通常是创建一个可供其他线程进行排他访问持有可变状态线程的工作队列，通过该工作队列调度即可完成对线程隔离可变状态的跨线程访问。当然从该工作队列写入和读取的状态副本是冻结的，而线程内的可变状态可根据情况返回其副本或者使用外部冻结状态进行更新。简单归纳：一个线程将冻结状态通过工作队列传递给目标线程，由其更新内部持有的可变状态，随后其他线程通过该工作队列获取该状态副本。

![Thread-isolated state](isolated-state.png){animated="true"}

实现线程隔离状态有些复杂，但有提供此功能的库。

### `AtomicReference` vs. thread-isolated state

对于简单的值， `AtomicReference` 可能是一个不错的选择；但是对于那些频繁变动的复杂的状态，使用线程隔离的状态可能是一个更好的选，虽然跨线程调用通常被认为会带来更大的性能损耗，但是大量性能测试数据表明很多情况下线程隔离状态优于 `AtomicReference` ，比如`collections` 。

此外，使用线程隔离状态可以避免 `AtomicReference` 带来的一致性问题。因为所有的代码指令都是在工作线程内完成，也即所有的操作都是单线程执行的，此时不需要进行大量并发保护来保证一致性。线程隔离的状态是Kotlin/Native的一个设计特性，可以很好的满足状态共享和可变状态的规范。

最后，线程隔离状态也更加灵活，因为可以使可变状态并发，此时可以使用任何类型的可变状态，而无需考虑复杂的并发保护。

## Low-level capabilities

为了更好利用系统提供的并发特性，需要往往需要借助原生系统实现，此时就会绕过Kotlin/Native提供的并发实现。

> 这是一个更深入的话题，你需要对Kotlin/Native的工作原理有深入的了解，使用时需要多加小心，更多内容可以参考[concurrency](https://kotlinlang.org/docs/reference/native/concurrency.html)。
>
> {type="note"}

Kotlin/Native运行在C++层上，支持与C和Objective-C的互操作。如果运行在 iOS操作系统上，开发者可以通过传递lambda表达式在Swift语法层面完成调用，这些系统原生语法的代码运行不受Kotlin/Native的状态规范限制。

这意味着开发者可以通过原生语法实现并发的可变状态，并通过与Kotlin/Native的交互获取或改写该状态。开发者可以通过[Objective-C interop](https://kotlinlang.org/docs/reference/native/c_interop.html) 访问底层代码，也即可以通过Swift实现Kotlin接口或者传入lambda表达式与Kotlin/Native进行交互，此时也就支持在任意线程访问可变状态。

使用平台原生语法的好处是可以获得更好的性能，但是需要开发者自己完成并发状态的管理。Objective-C本身不会关心状态的“冻结”，但将Kotlin获取的状态存储在Objective-C中并在多线程间进行共享，此时还是要充分考虑对Kotlin/Native本身的适配。

虽然Kotlin/Native运行时通常会对这种并发用法的部分异常进行告警，但是依旧可能出现极难追踪的异常，而且这种用法极易造成内存泄漏。

此外KMM应用需要适配JVM，采用这种原生系统相关的并发实现往往需要多端适配，而这些额外增加的工作量也需要进行权衡，也可能导致各端实现不一致。

_This material was prepared by [Touchlab](https://touchlab.co/) for publication by JetBrains._

