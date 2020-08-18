[//]: # (title: Kotlin/Native Concurrency Overview)
[//]: # (auxiliary-id: Kotlin_Native_Concurrency_Overview)

One of the more confusing things about Kotlin/Native for new developers is the state and concurrency model. In general, Kotlin developers come to Native with experience from the JVM, and expect everything to be the same. It is not. State shared between threads on Native has restrictions imposed by the runtime that it would not in the JVM.

## Why?

Mutable memory available to multiple threads at the same time, if unrestricted, is known to be risky and error prone. Languages like Java, C++, Swift/Objc, let multiple threads access the same state in an unrestricted fashion. It is up to you, the developer, to avoid causing problems. Concurrency issues are unlike other programming issues in that they are often very difficult to reproduce. You may not see them locally while developing, or they only happen once in a while. You may only see them in production under load.

In short, just because your tests pass does not mean your code is OK.

Not all languages are designed this way. Most developers would be familiar with Javascript. Javascript simply does not allow you to access the same state concurrently. On the other end of the spectrum, one of the reasons for Rust's popularity is it's language-level concurrency and state management. It is enough of a problem that the Mozilla team created a new language rather than use C/C++.

Kotlin/Native introduces rules around how state is shared between threads. These rules exist to prevent unsafe shared access to mutable state. If you are coming from a JVM background and write concurrent code, the way you architect your data may need some modification, but in general, you can achieve the same results without the risky side effects.

It is also important to point out, if absolutely necessary, there are ways to work around these rules. The intent is to make working around these rules something that you rarely, if ever do.

## Two Rules

As mentioned, Kotlin/Native introduces new rules around state and concurrency. They are simple, and there are basically two of them.

### Rule 1: Mutable state == 1 thread

If your state is mutable, only one thread can "see" it at a time. We'll explain this all in more detail later. For now, undertand that all regular class state that you would normally use in Kotlin is considered by the Kotlin/Native runtime as "mutable". If you aren't doing any concurrency, Kotlin/Native behaves the same as any other Kotlin, with the exception of global state.

```kotlin
data class SomeData(var count:Int)

fun simpleState(){
  val sd = SomeData(42)
  sd.count++
  println("My count is ${sd.count}") // It will be 43
}
```

The goal of rule 1 is simple. If there's only 1 thread, you don't have concurrency issues. Technically this is referred to as "thread confinement". For native mobile and desktop UI developers, this should be familiar. You cannot change the UI from a background thread. Kotlin/Native's state rules formalize that concept for all threads.

### Rule 2: Immutable state == many threads

If state can't be changed, multiple threads can safely have access to it. This is also conceptually simple. In Kotlin/Native, "immutable" doesn't mean everything is a `val`. It means frozen state (see below).

### That's it. Two rules.

How they are implemented, and their implications, are obviously more involved. However, the basic concepts underlying Kotlin/Native and concurrency are, as promised, very simple.

## Mutable State and Frozen State

When we talk about immutable state, we tend to mean properties with `val` and simple types. We'll call that *source-immutable*.

```kotlin
data class SomeData(val s:String, val i:Int)
```

That is immutable, by it's definition. 2 `val` elements, both of final immutable types.

Of course, what if we did this?

```kotlin
data class SomeData(val s:String, val i:SomeInterface)
```

Is that immutable? Maybe. We have no idea at compile-time what `SomeInterface` may do internally. As the language is currently defined, determining deep immutability, statically at compile time, is not possible.

Kotlin/Native needs to verify that some piece of state is really immutable, at runtime. The runtime could simply walk through all the state and verify that each is deeply immutable, but that would be somewhat inflexible, and if you needed to do that every time the runtime wanted to check mutability, it would have significant performance consequences.

Kotlin/Native defines a new runtime state called "frozen". Any instance of an object may be frozen. If an object is frozen, that means the following:

1. You cannot change any of its state. Attempting to do so will result in a runtime exception: `InvalidMutabilityException`. A frozen object instance is 100%, runtime-verified, immutable.
2. Everything it references is also frozen. All other objects it has a reference to are guaranteed to be frozen. That means, when the runtime needs to determine if an object can be shared with another thread, it only needs to check if that object is frozen. If it is, the whole graph is also frozen and safe to be shared.

The Native runtime adds an extension function `freeze()` to all classes. Calling `freeze()` will make an object frozen. If will also freeze everything referenced by the object, recursively.

```kotlin
data class MoreData(val strData: String, var width: Float)
data class SomeData(val moreData: MoreData, var count: Int)
//...
val sd = SomeData(MoreData("abc", 10.0), 0)
sd.freeze()
```



![Freeze Annimation](freezesmall2.gif)

`freeze()` is a one-way operation. You can't "unfreeze" something.

`freeze()` is not available in common Kotlin code, but several libraries exist to provide expect/actuals to allow you to use it in common code. However, if you're using a concurrency library, like kotlinx.coroutines, it will likely freeze data that crosses thread boundaries automatically. This is convenient, but often a source of errors and confusion, especially for developers new to Kotlin/Native. There are techniques to avoid this and debug it when it happens, which are important skills to learn.

For JVM-focused developers, `freeze()` seems unique and possibly an odd choice, but this is not a new concept. You'll find `freeze()` in [Ruby](https://www.honeybadger.io/blog/when-to-use-freeze-and-frozen-in-ruby/) and [Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze), for example.

## Global State

Kotlin allows you to define some globally available state. To conform to Kotlin/Native's state rules, global state has some special conditions.

### object

Global `object` instances are frozen by default. That means all threads can access them, but they are immutable. The following won't work.

```kotlin
object SomeState{
  var count = 0
  fun add(){
    count++ //This will throw an exception
  }
}
```

Trying to change `count` will throw an exception because `SomeState` is frozen (which means all of it's data is frozen).

You can make a global object thread-local, which will allow it to be mutable and give each thread a copy of it's state. Annotate it with `@ThreadLocal`.

```kotlin
@ThreadLocal
object SomeState{
  var count = 0
  fun add(){
    count++ //👍
  }
}
```

Of course, if different threads read `count`, they'll get different values, because each thread has it's own copy.

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

### Properties

Global properties are a special case. ***They are only available to the main thread***, but are mutable. Accessing them from other threads will throw an exception.

```kotlin
val hello = "Hello" //Only main thread can see this
```

You can annotate with `@SharedImmutable`, which will make them globally available but frozen, or `@ThreadLocal`, which will give each thread it's own mutable copy.

This rule applies to global properties with backing fields. Computed properties and global functions do not have the main thread restriction.

### Why Special Rules?

Class instances that you create are only accessible to the thread you've created them on, unless you transfer them to another thread, or attempt to put them somewhere that is already visible to multiple threads. That means they conform to Rule 1. Your class instances only need to be frozen when you *intentionally* try to share them with other threads.

Global state, as the name implies, is globally available. Any thread can reference global state. All state in Kotlin/Native must adhere to the 2 state rules, and global state is no exception. If left simply mutable, global state would violate Rule 1. As a result, global state needs special rules. The global state rules either make the state frozen or only visible to a single thread, which complies with the Kotlin/Native runtime's requirements.

## Concurrency In Practice

Kotlin/Native's concurrency rules will require some adjustment in architecture design, but with the help of libraries and some new best practices, day to day development is basically unaffected. In fact, adhering to Kotlin/Native's rules with common multliplatform and JVM code, will result in safer concurrency across the application. Thread confinement is a common technique for concurrency management. As mentioned above, modern UI frameworks tend to enforce a "main thread" rule for modifying the UI.

For applications that are written entirely in Kotlin/Native, you'll almost certainly use a concurrency library which will automate most of the freezing of state and scheduling of work across threads. You'll need to be aware of frozen state and how to debug issues as they arise, but having a single model will make the situation somewhat simpler.

For shared multipatform code, there will be multiple platform targets, with different state rules. For native specifically, there are general integration patterns emerging. Some teams, generally with larger applications, are sharing code for very specific functionality, and often managing concurrency in the host platform. This will require explicit freezing of state returned from Kotlin, but otherwise is straghtforward. A more extensive model, where concurrency is managed in Kotlin and the host is communicating on it's main thread to shared code, is simpler from the state management perspective. Concurrency libraries, like `kotlinx.coroutines`, will help automate freezing. You'll also be able to leverage the power of coroutines in your code, and gain more efficiencies by sharing more code.

However you structure your code sharing and concurrency, understanding Kotlin/Native's rules and libraries is very important. There are some new concepts to learn, but nothing too complicated. In fact, adopting these rules and patterns will improve the safety of concurrent code for all of your target platforms.
