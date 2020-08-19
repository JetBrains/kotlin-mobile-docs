[//]: # (title: Kotlin Native Concurrent Mutability)
[//]: # (auxiliary-id: Kotlin_Native_Concurrent_Mutability)

Kotlin/Native's state and concurrency model has two simple rules.

1. Mutable, non-frozen state is visible to only one thread at a time (i.e. [Thread confinement](https://dzone.com/articles/java-concurrency-thread-confinement))
2. Immutable, frozen state can be shared between threads.

The result of following these rules is that you can't change global state, and you can't change the same shared state from multiple threads. In many cases, simply changing how we're used to architecting code will work fine, and we don't need concurrent mutability. State was mutable from multiple threads in JVM code, but didn't *need* to be.

However, in many other cases, we'll need arbitrary thread access to state, or we'll have "service" objects that need to be available to the entire application. Maybe we simply don't want to go through the potentially costly exercise of rearchitecting existing code. Whatever the reason, **it will not always be feasible to constrain mutable state to a single thread**.

There are various techniques to allow you to work around these restrictions, each with their own pros and cons.

## Atomics

Kotlin/Native provides a set of Atomic classes which can be frozen, yet allow the value they contain to be changed. These classes implement a special-case handling of state in the Kotlin/Native runtime. To be clear, they allow you to change values inside frozen state.

The Kotlin/Native runtime includes a few different variations of Atomics.

### AtomicInt/AtomicLong

The first two are simple numerics: `AtomicInt` and `AtomicLong`. They allow your code to have a shared Int or Long that can be read and changed from multiple threads.

```kotlin
object AtomicDataCounter {
    val count = AtomicInt(3)
  
    fun addOne() {
        count.increment()
    }
}
```

The example above is a global `object`, which is frozen by default in Kotlin/Native, but you can change the value of `count`. It's important to note, you can change the value of `count` *from any thread*.

### AtomicReference

Arguably the more interesting class is `AtomicReference`. `AtomicReference` holds an object instance, and you can change that object instance. The object you put in `AtomicReference` must be frozen, but you can change the value that `AtomicReference` holds. For example, the following won't work in Kotlin/Native:

```kotlin
data class SomeData(val i: Int)

object GlobalData {
    var sd = SomeData(0)

    fun storeNewValue(i: Int) {
        sd = SomeData(i)
    }
}
```

According to the rules of global state, global `object` values are frozen in Kotlin/Native, so trying to modify `sd` will fail. You could implement that with `AtomicReference`:

```kotlin
data class SomeData(val i: Int)

object GlobalData {
    val sd = AtomicReference(SomeData(0).freeze())

    fun storeNewValue(i: Int) {5
        sd.value = SomeData(i).freeze()
    }
}
```

Some concepts above to highlight. The `AtomicReference` itself is frozen, which lets it live inside something that is frozen. The data we are passing into the `AtomicReference` instance is being explicitly frozen in the code above, but in the multiplatform libraries you'll likely use, the data will be frozen automatically. If you use the Kotlin/Native runtime's `AtomicReference`, you **must** remember to call `freeze()` explicitly.

### AtomicReference considerations

`AtomicReference` can be very useful when you need some shared state. There are some drawbacks to consider.

Accessing and changing values in an `AtomicReference` is very costly performance-wise *relative to* standard mutable state. *Relative* is the key word here. If performance is a concern, keep that in mind when evaluating `AtomicReference`.

There is also a potential issue with memory leaks. `AtomicReference` may leak memory if you don't explicitly clear it out. Keep the following in mind:

1. This only applies to situations where the object kept in the `AtomicReference` has cyclical references.
2. If you're keeping `AtomicReference` in a global object that never leaves scope, this won't matter (because the memory never needs to be reclaimed during the life of the process).
3. If you have state that may have cyclic references and needs to be reclaimed, you should use a nullable type in the `AtomicReference` and set it to null explicitly when you're done with it.

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

This memory leak issue is a current focus of the JetBrains team and should be resolved in the future.

Finally, there's also a consistency concern. Setting/getting values in `AtomicReference` is itself atomic, but if your logic requires a longer chain of thread exclusion, you'll need to implement that yourself. For example, if you have a list of values in an `AtomicReference` and you want to scan them first before adding a new one, you'll need to have some form concurrency management that `AtomicReference` alone does not provide.

The following won't protect against duplicate values in the list if called from multiple threads:

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

You will need to implement some form of locking or check-and-set logic to ensure proper concurrency.

## Thread-Isolated State

Rule #1 of Kotlin/Native state is mutable state is visible to only one thread. Atomics allow mutability from any thread. Isolating mutable state to a single thread, and allowing other threads to communite with that state, is an alternate method of concurrent mutability.

The concept is simple. Create a work queue that has exclusive access to a thread, and create some mutable state that lives in just that thread. Other threads communicate to the mutable thread by scheduling "work" on the work queue.

Data that goes in or comes out, if any, needs to be frozen, but the mutable state hidden in the worker thread remains mutable. Conceptually it looks like the following. One thread pushes some frozen state into the state worker, which stores it in the mutable state container. Another thread later schedules work that gets that state out

![State diagram](isostate-diagram.gif)

Implementing thread-isolated state is somewhat complex, but there are libraries available to provide that functionality.

### Thread-isolated state considerations

For simple values, `AtomicReference` will likely be an easier option. For cases with significant state, and potentially significant state changes, thread-isolated state may be a better choice. The main performance penalty is actually crossing over threads, but in performance tests with collections, for example, thread-isolated state significantly outperforms mutable state implemented with `AtomicReference`.

Thread-isolated state also avoids the consistency issues with `AtomicReference`. Because all operations happen in the state thread, and because you're scheduling work, you can perform operations with multiple steps and guarantee consistency without needing to manage thread exclusion (see "MyListCache" example above). Thread isolation is a design feature of the Koltin/Native state rules, and isolating mutable state works with those rules.

Thread-isolated state is also somewhat more flexible from the perspective that you can make mutable state concurrent. You can use any type of mutable state, rather than needing to create complex concurrent implementations.

## Lower Level Capabilities

Kotlin/Native has some more advanced ways of sharing concurrent state. If you are doing things that require high levels of performance you may occasionally need to avoid the concurrency rules altogether. ***However, this is a more advanced topic. You should have a deep understanding of how concurrency in Kotlin/Native works, under the hood, and be very careful when doing these kinds of things.***

### Platform-Native Code

Kotlin/Native runs on top of C++, and provides interop with C and Objective-C. If running on iOS, you can also pass lambda args into your shared code from Swift. All of this native code runs outside of the Kotlin/Native state restrictions. C, C++, etc, have no concept of "frozen" and do not enforce the Kotlin/Native state rules.

Effectively, that means you can implement concurrent mutable state in a native language and have Kotlin/Native talk to it.

You can [declare C code directly in a .def file](https://kotlinlang.org/docs/reference/native/concurrency.html#raw-shared-memory), or use [C and ObjC Interop](https://kotlinlang.org/docs/reference/native/c_interop.html) to access low level code. If on iOS specifically, as mentioned, you can use Swift to implement Kotlin interfaces or pass in lambdas that Kotlin code can call from any thread.

#### Platform-Native code considerations

There are benefits to platform-native. The obvious one being performance. An efficient C++ map implementation will probably be quite fast. Also, there are likely to be many native library options available, as C/C++ have been used extensively for decades.

On the negative side, you'll need to manage concurrency on your own. C does not know about `frozen`, but if you store state from Kotlin in C structures, and share them between threads, the Kotlin state definitely needs to be frozen. The same is true for the other languages. Kotlin/Native's runtime will generally warn you about issues, but it's possible to cause concurrency problems in native code that are very, very difficult to track down. It is also very easy to creat memory leaks.

Another concern, if you're targeting platforms other than native, such as the JVM or Javascript, you'll need alternate implementations for whatever you implement with platform native code. That will obviously take more work, but also introduce opportunities for platform inconsistencies.
