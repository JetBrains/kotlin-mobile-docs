[//]: # (title: Atomics with Stately)
[//]: # (auxiliary-id: Atomics_with_Stately)

Kotlin/Native imposes restrictions on state, to help ensure safe concurrency. The rules, in summary:

1. Mutable, non-frozen state is visible to only one thread at a time
2. Immutable, frozen state can be shared between threads.

Of course, sometimes multiple threads need to share some mutable state. There are [multiple ways to accomplish this](kotlin-native-concurrent-mutability.md). 
The most straightforward way to do this is with atomics.

Kotlin/Native provides a set of atomic classes that implement a special-case handling of mutable data. An instance of an 
atomic class can be frozen, and kept as a field of another object, but the value in the atomic can be changed.

It is important to understand. This is a special case in Kotlin/Native's state model. Atomic classes allow you to modify 
frozen state.

```kotlin
object FrozenObject {
    val counter = AtomicInt(0)
    
    fun plusOne(){
        counter.value++
    }
}

fun doCount(){
    background { 
        FrozenObject.plusOne()
    }
}
```

Kotlin/Native provides atomic primitive classes which you can use directly. We'll be using [Stately](https://github.com/touchlab/Stately), a utility library for multiplatform compatibility with Kotlin/Native-specific concurrency.

## Getting Started

Add the Stately concurrency module to your dependencies.

```kotlin
commonMain {
    dependencies {
        implementation 'co.touchlab:stately-concurrency:1.0.2'
    }
}
```

For our examples we'll be executing code on multiple threads. There are various ways to do that, but to make things simple 
we're going to create our own function called `background`, which takes a function block and returns a future.

```kotlin
fun <R> background(block: () -> R):MPFuture<R> 
```

Anything in the block is run on a different thread.

The example code is pretty basic, and exists just to illustrate how to use the atomic classes. If you'd like to run these 
examples, you can clone [StatelyAtomicsSample](https://github.com/touchlab/StatelyAtomicsSample). It contains the dependencies 
you'll need to run these examples, as well as the example code itself.

## Multiplatform

Kotlin/Native provides atomic primitive classes that you can use directly, but only in native code. 
Stately's goal is to facilitate multiplatfom code by providing common classes that delegate to each platform. The examples 
provided are written in common Kotlin code, but the state rules imposed by Kotlin/Native will only apply when running in 
a native instance. JS and the JVM don't know about frozen state, so although the code will run on JS and JVM, 
using atomics isn't strictly necessary on those platforms. We'll be running the samples on iOS because that is a Kotlin/Native platform.

## Atomic Types

There are atomic classes for `Int`  and `Long`, which provide basic numeric values, and `AtomicReference`, which can hold 
a reference to any arbitrary object instance[^1]. In the Kotlin/Native runtime, an atomic instance can be frozen, and shared 
with many threads, yet the value in that instance can be changed.

## AtomicInt and AtomicLong

Imagine a global object with a simple `Int` counter:

```kotlin
object CounterObject {
    var counter = 0

    fun plusOne(){
        counter++
    }
}
```

Global objects in Kotlin/Native are frozen by default. This code would actually cause a runtime error if you tried to call 
`plusOne()`. Solving this particular issue is easy with `AtomicInt`.

```kotlin
object CounterObject {
    val counter = AtomicInt(0)

    fun plusOne(){
        counter.value++
    }
}
```

If we interact with that object a bit:

```kotlin
fun countThings() {
    println("Count is: ${CounterObject.counter.value}")
    CounterObject.plusOne()
    println("Count is: ${CounterObject.counter.value}")
    background {
        CounterObject.counter.value = 42
    }.consume()

    println("Count is: ${CounterObject.counter.value}")
}
```

We'll see the following printed:

```
Count is: 0
Count is: 1
Count is: 42
```

`AtomicLong`, predictably, does the same thing, but with a `Long` ranged value.

### Concurrency Considerations

Using `AtomicInt/Long` will allow you change frozen values in Kotlin/Native, but that doesn't automatically make the data thread safe. Consider the following:

```kotlin
fun plusOne(){
    counter.value++
}
```

That will "work", but if you are calling `plusOne()` from multliple threads, you will have a race condition, because that's really 2 operations. There's nothing special about Kotlin/Native here. To avoid increment issues, there's a method ready to call.

```kotlin
fun plusOne(){
    counter.incrementAndGet()
}
```

This example is a simple case. Ensuring safe concurrency is a complex topic, and can be risky. That is why Kotlin/Native's rules exist, to help ensure safe concurrency. Being able to change shared state from multiple threads with atomics reintroduces some of that risk. Fully exploring how to properly use atomic variable is beyond the scope of this article, but bear in mind that modifying shared state means you need to consider these issues.

## AtomicReference

`AtomicReference` can hold an object instance. This will allow you to change arbitrary values in shared state. The `AtomicReference` itself is frozen, and anything put into it will be frozen as well.

```kotlin
data class SomeData(val s: String)

object GlobalReference {
    val myRef = AtomicReference(SomeData("hello"))
    fun updateRef(s: String) {
        myRef.value = SomeData(s)
    }
}
```

`GlobalReference` is a global object, so it and all of it's data is frozen. You can access and update `myRef` from any thread.

```kotlin
fun globalRefs(){
    println(GlobalReference.myRef.value)
    GlobalReference.updateRef("hello again")
    println(GlobalReference.myRef.value)
}
```

This will print:

```
SomeData(s=hello)
SomeData(s=hello again)
```

`AtomicReference` can be very useful in cases where you have service objects that are accessed globally, and in cases where rearchitecting existing code to be restricted to a single thread may not be disireable. However, `AtomicReference` access can have performance implications if used in performance critical code, relative to simple mutable state.

Here's another example, showing multiple threads accessing shared state with an `AtomicReference`.

```kotlin
data class Log(val count: Int, val currentTime: Long)

class LastLogTracker {
    private val logAtom = AtomicReference(Log(0, currentTimeMillis()))

    fun log(ct: Long) {
        while (true) {
            val prevLog = logAtom.value
            val nextLog = Log(prevLog.count + 1, ct)
            if (logAtom.compareAndSet(prevLog, nextLog))
                break
        }
    }

    val lastLog: Log
        get() = logAtom.value
}
```

There's a data class `Log` which has a `count` value that is the total number of log calls. `LastLogTracker` has the last log in an `AtomicReference`. You can call the `log` method from multiple threads.

If you're wondering why we have a loop, because we need the last `Log` object's count value, we need to do an atomic `compareAndSet` to ensure concurrent consistency. Alternatively, we could use some kind of a lock around this code. As mentioned previously, using atomics allows you to modify shared state, but if you do, you'll need to be aware of, and deal with, concurrency issues.

Finally, we call `log` 250k times, from the background and main thread.

```kotlin
fun manyLogs(){
    val lastLogTracker = LastLogTracker()
    val future = background {
        repeat(150_000){
            lastLogTracker.log(currentTimeMillis())
        }
    }

    repeat(100_000){
        lastLogTracker.log(currentTimeMillis())
    }

    future.consume()

    println("Total logs: ${lastLogTracker.lastLog.count}")
}
```



### Delegate

If you're using `AtomicReference` often, you may find the syntax cumbersome at times. You can make this easier to manage 
with Kotlin delegate properties. The following will help manage access to an `AtomicReference` instance.

```kotlin
class AtomicDelegate<T>(ival:T) {
    private val atom = AtomicReference(ival)
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return atom.value
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        atom.value = value
    }
}
```

The code from the previous example can then be simplified as follows.

```kotlin
object DelegateReference {
    var myRef :SomeData by AtomicDelegate(SomeData("hello"))
    fun updateRef(s: String) {
        myRef = SomeData(s)
    }
}

fun delegateRefs(){
    println(DelegateReference.myRef)
    DelegateReference.updateRef("hello again")
    println(DelegateReference.myRef)
}
```

`myRef` is now a `var` and can be changed. Under the hood, those changes delegate to an `AtomicReference`.

*We may add this directly to Stately in the future. We decided not to add it previously because of the memory leak issues mentioned below.*

### *Important Note*

Values held in `AtomicReference` may leak memory in certain cases. Keep the following in mind:

1. This only applies to situations where the object kept in the `AtomicReference` has cyclical references.
2. If you're keeping `AtomicReference` in a global object that never leaves scope, this won't matter (because the memory never needs to be reclaimed during the life of the process).
3. If you have state that may have cyclic references and needs to be reclaimed, you should use a nullable type in the `AtomicReference` and set it to null explicitly when you're done with it.

```kotlin
class SomeClass {
    val shared = AtomicReference<SomeData?>(SomeData("hello"))
    
    //Call this when you're done
    fun cleanup() {
        shared.value = null 
    }
}
```

This issue will likely be addressed in a future Kotlin release, but for now you should keep it in mind.

## Atomic-fu

There is another library that provides similar atomic support, directly from JetBrains, called Atomic-fu. It does a number of the same things that Stately does, but the Jetbrains team states that it is experimental and not really intended for public use ([see here](https://github.com/Kotlin/kotlinx.atomicfu/issues/90#issuecomment-597872907)):

> `atomicfu` is more or less experimental and was always developed for our own needs, not for public use.

We will be keeping an eye on the progress of Atomic-fu in the future and see if that progresses to something more geared towards public consumption.

[^1]: Stately also includes an `AtomicBoolean`. Under the hood that just uses `AtomicInt`.
