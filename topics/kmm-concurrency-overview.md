[//]: # (title: KMM concurrency overview)
[//]: # (auxiliary-id: KMM_Concurrency_Overview)

When you extend your development experience from Android to Kotlin Multiplatform Mobile, you will face a different state 
and concurrency model for iOS. This is a Kotlin/Native model. [Kotlin/Native](https://kotlinlang.org/docs/reference/native-overview.html) 
is a technology for compiling Kotlin code to native binaries, which can run without a virtual machine, for example, on iOS. 

Mutable memory available to multiple threads at the same time, if unrestricted, is known to be risky and error prone. 
Languages like Java, C++, Swift/Objective-C, let multiple threads access the same state in unrestricted fashion. It is up to 
you to avoid causing problems. Concurrency issues are unlike other programming issues in that they are 
often very difficult to reproduce. You may not see them locally while developing, or they only happen once in a while. 
You may only see them in production under load.

In short, just because your tests pass does not mean your code is OK.

Not all languages are designed this way. Javascript simply does not 
allow you to access the same state concurrently. On the other end of the spectrum, Rust's 
language-level concurrency and state management makes it very popular. 

## Rules on sharing state

Kotlin/Native introduces rules on sharing state between threads. These rules exist to prevent unsafe shared 
access to mutable state. If you are coming from a JVM background and write concurrent code, you may need to change the way 
you architect your data, but you can achieve the same results without risky side effects.

It is also important to point out, there are [ways to work around these rules](kotlin-native-concurrent-mutability.md). 
The intent is to make working around these rules something that you rarely, if ever do.

There are just two simple rules around state and concurrency.

### Rule 1: Mutable state == 1 thread

If your state is mutable, only one thread can _see_ it at a time. Any regular class state that 
you would normally use in Kotlin is considered by the Kotlin/Native runtime as _mutable_. If you aren't doing any concurrency, 
Kotlin/Native behaves the same as any other Kotlin, with the exception of [global state](#global-state).

```kotlin
data class SomeData(var count:Int)

fun simpleState(){
    val sd = SomeData(42)
    sd.count++
    println("My count is ${sd.count}") // It will be 43
}
```

If there's only one thread, you don't have concurrency issues. Technically this is referred 
to as _thread confinement_, which means that you cannot change the UI from a background thread. Kotlin/Native's state rules 
formalize that concept for all threads.

### Rule 2: Immutable state == many threads

If state can't be changed, multiple threads can safely access it.
In Kotlin/Native, _immutable_ doesn't mean everything is a `val`. It means _frozen state_.

## Immutable and frozen state

This example is immutable, by its definition – it has 2 `val` elements, both of final immutable types.

```kotlin
data class SomeData(val s:String, val i:Int)
```

The next example may be immutable or mutable. It is not clear what `SomeInterface` may do internally at compile time. 
In Kotlin, determining deep immutability, statically at compile time, is not possible.

```kotlin
data class SomeData(val s:String, val i:SomeInterface)
```

Kotlin/Native needs to verify that some piece of state is really immutable, at runtime. The runtime could simply walk 
through all the state and verify that each is deeply immutable, but that would be inflexible, and if you needed 
to do that every time the runtime wanted to check mutability, it would have significant performance consequences.

Kotlin/Native defines a new runtime state called _frozen_. Any instance of an object may be frozen. If an object is frozen:

1. You cannot change any of its state. Attempting to do so will result in a runtime exception: `InvalidMutabilityException`. 
A frozen object instance is 100%, runtime-verified, immutable.
2. Everything it references is also frozen. All other objects it has a reference to are guaranteed to be frozen. That means, 
when the runtime needs to determine if an object can be shared with another thread, it only needs to check if that object 
is frozen. If it is, the whole graph is also frozen and safe to be shared.

The Native runtime adds an extension function `freeze()` to all classes. Calling `freeze()` will freeze an object and everything 
referenced by the object, recursively.

```kotlin
data class MoreData(val strData: String, var width: Float)
data class SomeData(val moreData: MoreData, var count: Int)
//...
val sd = SomeData(MoreData("abc", 10.0), 0)
sd.freeze()
```

![Freezing state](freezing-state.gif)

* `freeze()` is a one-way operation. You can't _unfreeze_ something.
* `freeze()` is not available in shared Kotlin code, but several libraries provide expect and actual declarations
 for using it in shared code. However, if you're using a concurrency library, like [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines), it will 
 likely freeze data that crosses thread boundaries automatically. This is convenient, but may be a source of errors and 
 confusion, especially if you are new to Kotlin/Native. There are techniques to avoid this and debug it when it happens.

`freeze` is not unique for Kotlin. You can also find it in [Ruby](https://www.honeybadger.io/blog/when-to-use-freeze-and-frozen-in-ruby/) and [Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze).

## Global state

Kotlin allows you to define some globally available state. If left simply mutable, global state would violate [_Rule 1_](#rule-1-mutable-state-1-thread).  
To conform to Kotlin/Native's state rules, global state has some special conditions. 
These conditions make the state frozen or only visible to a single thread.

### Global `object`

Global `object` instances are frozen by default. That means all threads can access them, but they are immutable. The following won't work.

```kotlin
object SomeState{
    var count = 0
    fun add(){
        count++ //This will throw an exception
    }
}
```

Trying to change `count` will throw an exception because `SomeState` is frozen (which means all of its data is frozen).

You can make a global object thread _local_, which will allow it to be mutable and give each thread a copy of its state. 
Annotate it with `@ThreadLocal`.

```kotlin
@ThreadLocal
object SomeState{
  var count = 0
  fun add(){
    count++ //👍
  }
}
```

If different threads read `count`, they'll get different values, because each thread has its own copy.

These global object rules also apply to companion objects.

```kotlin
class SomeState{
    companion object{
        var count = 0
        fun add(){
            count++ //This will throw an exception
        }
    }
}
```

### Global properties

Global properties are a special case. *They are only available to the main thread*, but are mutable. Accessing them from 
other threads will throw an exception.

```kotlin
val hello = "Hello" //Only main thread can see this
```

You can annotate with :

* `@SharedImmutable`, which will make them globally available but frozen.
* `@ThreadLocal`, which will give each thread its own mutable copy.

This rule applies to global properties with backing fields. Computed properties and global functions do not have the main 
thread restriction.

## Current and future models

Kotlin/Native's concurrency rules will require some adjustment in architecture design, but with the help of libraries and
 new best practices, day to day development is basically unaffected. In fact, adhering to Kotlin/Native's rules with 
multiplatform code, will result in safer concurrency across the KMM application. 

In the KMM application, you have Android and iOS targets with different state rules. Some teams, generally with 
larger applications, are sharing code for very specific functionality, and often managing concurrency in the host platform. 
This will require explicit freezing of state returned from Kotlin, but otherwise is straghtforward. 

A more extensive model, where concurrency is managed in Kotlin 
and the host is communicating on its main thread to shared code, is simpler from the state management perspective. 
Concurrency libraries, like [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines), 
will help automate freezing. You'll also be able to leverage the power of [coroutines](https://kotlinlang.org/docs/reference/coroutines/coroutines-guide.html) 
in your code, and gain more efficiencies by sharing more code.

However, the current Kotlin/Native concurrency model has a number of deficiencies. For example, mobile developers are used to freely 
share their objects between threads, and they have already developed a number of approaches and architectural patterns to 
avoid data races while doing so. It is possible to write efficient applications that do not block the main thread using 
Kotlin/Native, but the ability to do so comes with a steep learning curve.

That's why we are working on the new memory manager and concurrency model for Kotlin/Native that will help us remove these 
drawbacks. Learn more on [what we are doing in this direction](https://blog.jetbrains.com/kotlin/2020/07/kotlin-native-memory-management-roadmap/).

_We'd like to thank the [Touchlab team](https://twitter.com/touchlabhq) for helping us write this article._