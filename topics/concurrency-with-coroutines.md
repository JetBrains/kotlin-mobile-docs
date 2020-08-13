[//]: # "title: Kotlin Native Concurrency Overview"
[//]: # "auxiliary-id: Kotlin_Native_Concurrency_Overview"

Using Coroutines for asynchronous and concurrent code is well documented for developers writing code for the JVM. For Native, the situation is in flux and is somewhat more complex. The library which is essential for using coroutines, 'kotlinx.coroutines', has two separate supported versions for Kotlin/Native. The "official" version cannot share data across threads. The other version implements data sharing across threads, but will remain an unmerged separate version for the forseeable future. Likely until the [concurrency model update](https://blog.jetbrains.com/kotlin/2020/07/kotlin-native-memory-management-roadmap/) is completed.

What does that mean for writing concurrent Kotlin code that will run on native? Well, it's complicated. You'll need to evaluate the various options and pick which works best for you, but once you do, concurrent code in Kotlin Native is straightforward to create.

We're going to cover using the multithreaded release of 'kotlinx.coroutines', and will include links to other options that you can explore.

## Asynchronous vs Concurrent

First, let's make a distinction between asynchronous and concurrent processing. Within a coroutine, the sequence of processing may be suspended, and resumed later. This allows for asynchronous, non-blocking code, without using callbacks or promises. That is asynchronous, but everything related to that coroutine may happen in a single thread. 

As a concrete example, the following code makes a network call using Ktor. In the "main thread", the call is initiated, and suspended while the actual networking is performed by another underlying process. When complete, the code is resumed in the main thread.

```kotlin
val client = HttpClient
//Running in main thread, start a get call
client.get<String>("https://example.com/some/rest/call")
//The get call will suspend and let other work happen in the main thread, and resume when the get call completes
```

That is distinct from concurrent code. Concurrent code will need to be run in another thread. Depending on the libraries you use and what you need to do, it's possible you won't ever need to use multiple threads.

Assuming you do, let's dive into how that works with Kotlin/Native coroutines.

## Dispatcher and Context

Coroutines are executed by a Dispatcher. The Dispatcher defines what thread the coroutine will be executed on. There are a number of ways in which you can specify the Dispatcher, or change the one you're currently running in. For example: 

```kotlin
suspend fun differentThread() = withContext(Dispatchers.Default){
        println("Different thread")
    }
```

`withContext` takes a Dispatcher argument and a code block. Code within that block is executed by the thread defined by the Dispatcher. There are a number of other ways you can switch the Dispatcher, but we won't go through them in detail. They all generally take a Dispatcher argument and a block to be executed. See [Coroutine Context and Dispatchers](https://kotlinlang.org/docs/reference/coroutines/coroutine-context-and-dispatchers.html) for a far more detailed description of how Distatchers work in general.

With regards to using kotlinx.coroutines in Kotlin/Native, performing work on a different thread will follow that same basic pattern. Specify a different Dispatcher and a function block of work to execute. In general, that works the same as the JVM, but there are a few critical differences when running on Kotlin/Native.

## Captured and Frozen Data

All concurrency libraries running on Kotlin/Native follow the same basic pattern. To run some code on a different thread, you pass a function block, which gets frozen, then run on another thread. 

```kotlin
fun <R> runOnDifferentThread(functionBlock: () -> R)
```

You would call that function as follows:

```kotlin
runOnDifferentThread { 
    //Code run in another thread
}
```

As discussed in our [concurrency overview](kotlin-native-concurrency-overview.md), all state shared between threads in Kotlin/Native must be frozen. A function argument is itself state, which will be frozen, along with anything it captures.

Coroutine functions that cross threads use the same pattern. To allow function blocks to be executed on another thread, they are frozen.

In the following example, the data class instance `dc` will be captured by the function block, and frozen when crossing threads. The `println` statement will print true.

```kotlin
val dc = DataClass("Hello")
runBlocking(Dispatchers.Default) {
    println("${dc.isFrozen}")
}
```

As with other methods of running concurrent code in Kotlin/Native, you'll need to be careful with captured state. Sometimes it's obvious when state will be captured, but other situations are more subtle. For example:

```kotlin
class SomeModel(val id:IdRec){
    suspend fun saveData() = withContext(Dispatchers.Default){
        saveToDb(id)
    }
}
```

The code inside `saveData` will be run on another thread. That will freeze `id`, but becuause `id` is a field in the parent class, it will also freeze the parent class.

Please reference the more detailed Kotlin/Native concurrency docs to understand how this works and how to resolve issues. As it pertains to coroutines, just understand that they follow the same rules as everything else.

## Returned Data

Data returned from a different thread is also frozen. This is generally fine.

```kotlin
val dc = runBlocking(Dispatchers.Default) {
    DataClass("Hello Again")
}

println("${dc.isFrozen}")
```

This can become a problem if you are keeping some data unfrozen, and attempting to return something that retains a reference to it. That type of situation isn't very common, but be aware that returned values are frozen.

## Using Multithreaded Coroutines

To use the multithreaded version of coroutines for Kotlin/Native, you have to add a specific version of the library. The current version for the 1.4 release candidate is:

```kotlin
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.8-native-mt-1.4.0-rc"
```

When using other libraries that also depend on `kotlinx.coroutines`, such as Ktor, you should make sure that the multiplatform one is used. One way to do that is with `strictly`:

```kotlin
implementation ("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.8-native-mt-1.4.0-rc"){
  version {
    strictly("1.3.8-native-mt-1.4.0-rc")
  }
}
```

## Special Notes

### Memory Leaks

Part of the reason for the multiple versions of native `kotlinx.coroutines` is that in some cases, using coroutines across threads can result in memory leaks. For simpler use cases, this isn't much of an issue, but may present a problem under load for more complex coroutine scenarios. We will update guidance on what to avoid in the near future.

### Ktor

At present, Ktor is a special case. The current version of Ktor running on native, and specifically Apple environments, was designed for the single-threaded version of `kotlinx.coroutines`. As a result, you need to be careful with how you integrate it into your projects.

Calls to ktor need to be initiated from the main thread. That can make some situations more complex, but in simpler cases, this is pretty straightforward. The more confusing situation is, the `CoroutineScope` used by Ktor cannot also have been used across threads. If you're going to use Ktor and coroutines that cross threads, you can keep a separate `CoroutineScope` for Ktor. As a more complex alternative, you can creat a child `CoroutineScope` with an isolated `Job`. If the `Job` gets frozen, Ktor will stop working. See [here](https://github.com/touchlab/KaMPKit/blob/master/shared/src/iosMain/kotlin/co/touchlab/kampkit/ktor/Platform.kt#L13) for an example.

## Alternatives

For simpler background tasks, creating your own processor is always an option. For multiplatform between the JVM and Native, using `ExecutorService` and `Worker`, respectively. is reasonably simple to implement.

### CoroutineWorker

[`CoroutinesWorker`](https://github.com/Autodesk/coroutineworker) is a library published by AutoDesk that implements some features of coroutines across threads, using the single-threaded version of `kotlinx.coroutines`.

### Reaktive

[Reaktive](https://github.com/badoo/Reaktive) is an Rx-like library, implementing Reactive Extensions for Kotlin Multiplatform.


