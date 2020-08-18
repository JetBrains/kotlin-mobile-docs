[//]: # "title: Kotlin Native Concurrency Overview"
[//]: # "auxiliary-id: Kotlin_Native_Concurrency_Overview"

Writing concurrent multithreaded code is important in many contexts. This is certainly true when writing shared Kotlin code for native mobile target platforms.

It is important to note at the beginning that for Kotlin/Native, the [concurrency and memory model are unique]([kotlin-native-concurrency-overview.md](https://github.com/JetBrains/kotlin-mobile-docs/blob/master/topics/kotlin-native-concurrency-overview.md)), but they are [also in flux](https://blog.jetbrains.com/kotlin/2020/07/kotlin-native-memory-management-roadmap/). As a result, writing concurrent multiplatform code that targets native, and more specifically Apple targets, is a more complicated and evolving situation than it will likely be a little down the road. However, it is certainly possible to do so, and in a way that will be compatible with future changes.

This document will largely cover using `kotlinx.coroutines` for writing concurrent code, but we will discuss some other options you can explore. Ultimately you will need to review the pros and cons of any solution, and pick what works best for your situation.

## Coroutines and Kotlin/Native

Using Coroutines for asynchronous and concurrent code is well documented for developers writing code for the JVM. For Native, the situation is in flux and is somewhat more complex.

For any reasonable use of coroutines in Kotlin, you will almost certainly need the `kotlinx.coroutines` library. It provides a number or features on top of the language-level coroutine primitives.

The first, and currently "official", version of `kotlinx.coroutines` for Kotlin/Native is for single-threaded use only. You cannot send work to other threads by changing the `Dispatcher` instance. As of Kotlin 1.4, that official version is `1.3.9`.

You can, of course, suspend execution, and work can be done on other threads, provided that a different mechanism schedules and manages that work. However, the main version of `kotlinx.coroutines` cannot change threads on it's own.

There is an alternate branch and set of releases for `kotlinx.coroutines` that provide support for using multiple threads. It is a separate branch for the reasons listed in the [concurrency model update post](https://blog.jetbrains.com/kotlin/2020/07/kotlin-native-memory-management-roadmap/), and will likely remain a separate branch until those updates are complete. However, with some considerations kept in mind, the multithreaded versions of `kotlinx.coroutines` works great, and can be used in production environments.

This is a somewhat complicated, but temporary situation. You should understand the various options available and choose which works best for you. This document will largely cover the multithreaded version of `kotlinx.coroutines`, but we'll also list some other options to explore.

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

With regards to using kotlinx.coroutines in Kotlin/Native, performing work on a different thread will follow that same basic pattern. Specify a different Dispatcher and a function block of work to execute. In general, switching Dispatchers and threads works the same as it does with the JVM, but there are a few critical differences when running on Kotlin/Native.

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

Please reference the more detailed Kotlin/Native concurrency docs to understand how this works and how to resolve issues. As it pertains to coroutines, just understand that they follow the same rules as everything else. Captured state will be frozen.

## Returned Data

Data returned from a different thread is also frozen. This is generally fine, as the data returned is usually some form of data object. Common best practice advises immutable data, but even if you intend to return mutable state, you'll genenrally figure out right away that your returned value cannot be changed.

```kotlin
val dc = runBlocking(Dispatchers.Default) {
    DataClass("Hello Again")
}

println("${dc.isFrozen}")
```

Where this can present a problem is the situation where you are keeping mutable state isolated to a single thread and communicating with it by way of coroutine threading operations. If you attempt to return data that retains a reference to the mutable state, it will freeze it by association.

Keeping thread-isolated state is beyond the scope of this document, but it's important to keep in mind that returned values are frozen.

## Using Multithreaded Coroutines

To use the multithreaded version of `kotlinx.coroutines` for Kotlin/Native, you have to add a specific version of the library. The current version for Kotlin 1.4 is `1.3.9-native-mt`. You can add the dependency by including the following in your `commonMain` dependency section.

```kotlin
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.9-native-mt"
```

When using other libraries that also depend on `kotlinx.coroutines`, such as Ktor, you should make sure that the multiplatform one is used. One way to do that is with `strictly`:

```kotlin
implementation ("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.9-native-mt"){
  version {
    strictly("1.3.9-native-mt")
  }
}
```

Because the "official" version of `kotlinx.coroutines` is the single-threaded one, any library that uses `kotlinx.coroutines` will almost certainly use the single-threaded release. If you see `InvalidMutabilityException` related to coroutine operation, it's very likely that your build is getting the wrong version.

## Special Notes

### Memory Leaks

Part of the reason for the multiple versions of native `kotlinx.coroutines` is that in some cases, using coroutines across threads can result in memory leaks. For simpler use cases, this isn't much of an issue, but may present a problem under load for more complex coroutine scenarios. We will update guidance on what to avoid in the near future.

### Ktor

At present, Ktor is a special case. The current version of Ktor running on native, and specifically Apple environments, was designed for the single-threaded version of `kotlinx.coroutines`. As a result, you need to be careful with how you integrate it into your projects.

Calls to ktor need to be initiated from the main thread. That can make some situations more complex, but in simpler cases, this is pretty straightforward. The more confusing situation is, the `CoroutineScope` used by Ktor cannot also have been used across threads. If you're going to use Ktor and coroutines that cross threads, you can keep a separate `CoroutineScope` for Ktor. As a more complex alternative, you can creat a child `CoroutineScope` with an isolated `Job`. If the `Job` gets frozen, Ktor will stop working. See [here](https://github.com/touchlab/KaMPKit/blob/master/shared/src/iosMain/kotlin/co/touchlab/kampkit/ktor/Platform.kt#L13) for an example.

There is an [issue](https://youtrack.jetbrains.com/issue/KTOR-499) to track the status of Ktor and multithreaded coroutines. Keep an eye on that for progress.

## Alternatives

For simpler background tasks, creating your own processor is always an option. As there does not appear to be a simple library available for this, we've published a basic one just for native mobile: [KMMWorker](https://github.com/touchlab/KMMWorker). It provides basic thread worker queues for Android and iOS/MacOS.

To use the library, add the dependency to `commonMain`:

```kotlin
dependencies {
    implementation("co.touchlab:kmmworker:0.1.1")
}
```

Create a  `Worker` in common code with the following:

```kotlin
val worker = Worker()
```

You can schedule work and get a `Future` returned, or specifically useful for native mobile, run `backgroundTask`, which takes 2 function block arguments. The first is a task to run on a background thread, and the second is run on the main thread with the result. The main thread function block is *not* frozen, assuming you call this function from the main thread.

```kotlin
worker.backgroundTask({
    SomeData("Hello")
}) { result ->
    when(result){
        is Success -> println("${result.result}")
        is Error -> println("${result.thrown}")
    }
}
```

This library is more of an example of how to create a simple job queue, but we may maintain it in the future if there's interest.

### CoroutineWorker

[`CoroutinesWorker`](https://github.com/Autodesk/coroutineworker) is a library published by AutoDesk that implements some features of coroutines across threads, using the single-threaded version of `kotlinx.coroutines`. For simple suspend functions this is a pretty good option, but does not support Flow and other structures.

### Reaktive

[Reaktive](https://github.com/badoo/Reaktive) is an Rx-like library, implementing Reactive Extensions for Kotlin Multiplatform. It has some coroutine extensions, but is designed around RX and threads primarily.

## Platform Concurrency

The "other" major option in practice for some production use cases is to rely on the platform to handle concurrency. This is more common in situations where the shared Kotlin code will be used for business logic or data operations rather than architecture. In general, this will work as expected. The only major consideration for Kotlin/Native is thread-safe data passing.

To share state in Kotlin/Native across threads, that state needs to be frozen. All of the concurrency libraries discussed in this document will freeze your data automatically. You will rarely, if ever, need to do so explicitly.

If you are returning data to the native platform, and it will be sharing data across threads, you will need to ensure that data is frozen before leaving the Kotlin/Native boundary.

Kotlin, on platforms other than Kotlin/Native, have no concept of frozen. To make `freeze` available in common code, you can create expect/actual implementations for `freeze`, or use `stately-common`, which provides this functionality for you. In Kotlin/Native, `freeze` will freeze your state, while on the JVM (or JS), it'll do nothing.

```kotlin
dependencies {
  implementation 'co.touchlab:stately-common:1.0.x'
}
```

See [stately-common](https://github.com/touchlab/Stately#stately-common) for more information.


