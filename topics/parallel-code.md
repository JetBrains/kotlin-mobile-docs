[//]: # (title: Parallel code)
[//]: # (auxiliary-id: Parallel_code)

When working with mobile platforms, you may need to write multithreaded code that runs in parallel. For this purpose, 
you can use the `kotlinx.coroutines` library ([standard](#coroutines) or [multithreaded version](#multithreaded-coroutines)) 
and [alternative solutions](#alternatives-to-kotlinx-coroutines).

Review the pros and cons of each solution and choose the one that works the best for your situation.

Learn more about [concurrency, current approach, and future improvements](concurrency-overview.md).

## Coroutines

Coroutines are light-weight threads that allow you to write asynchronous non-blocking code. Kotlin provides the 
[`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines) library with a number of high-level coroutine-enabled primitives.

The current version of `kotlinx.coroutines`, which can be used for iOS, supports usage only in a single thread. 
You cannot send work to other threads by changing a [dispatcher](#dispatcher-for-changing-threads). For Kotlin %kotlinVersion%, the 
official version is `%coroutinesVersion%`.

You can suspend execution and do work on other threads while using a different mechanism for scheduling  
and managing that work. However, this version of `kotlinx.coroutines` cannot change threads on its own.

There is also [another version of `kotlinx.coroutines`](#multithreaded-coroutines) that provides support for multiple threads.

Get acquainted with the main concepts for using coroutines:

* [Asynchronous vs. parallel processing](#asynchronous-vs-parallel-processing)
* [Dispatcher for changing threads](#dispatcher-for-changing-threads)
* [Frozen captured data](#frozen-captured-data)
* [Frozen returned data](#frozen-returned-data)

### Asynchronous vs. parallel processing

Asynchronous and parallel processing are different terms. 

Within a coroutine, the sequence of processing may be suspended and resumed later. This allows for asynchronous, 
non-blocking code, without using callbacks or promises. That is asynchronous processing, but everything related to that 
coroutine can happen in a single thread. 

The following code makes a network call using [Ktor](https://ktor.io/). In the main thread, the call is initiated 
and suspended, while another underlying process performs actual networking. When completed, the code resumes 
in the main thread.

```kotlin
val client = HttpClient
//Running in main thread, start a `get` call
client.get<String>("https://example.com/some/rest/call")
//The get call will suspend and let other work happen in the main thread, and resume when the get call completes
```

That is different from parallel code that needs to be run in another thread. Depending on your purpose 
and used libraries, you may never need to use multiple threads.

### Dispatcher for changing threads

Coroutines are executed by a dispatcher that defines which thread the coroutine will be executed on. 
There are a number of ways in which you can specify the dispatcher, or change the one for the coroutine. For example: 

```kotlin
suspend fun differentThread() = withContext(Dispatchers.Default){
    println("Different thread")
}
```

`withContext` takes a dispatcher as an argument and a code block, which will be executed by the thread defined by 
the dispatcher. Learn more about [coroutine context and dispatchers](https://kotlinlang.org/docs/reference/coroutines/coroutine-context-and-dispatchers.html).

To perform work on a different thread, specify a different dispatcher and a code block to execute. In general, 
switching dispatchers and threads works similar to JVM, but there are differences related to freezing
captured and returned data.

### Frozen captured data

To run code on a different thread, you pass a `functionBlock`, which gets frozen, and then runs in another thread. 

```kotlin
fun <R> runOnDifferentThread(functionBlock: () -> R)
```

You will call that function as follows:

```kotlin
runOnDifferentThread {
    //Code run in another thread
}
```

As described in the [concurrency overview](concurrency-overview.md), a state shared between threads in 
Kotlin/Native must be frozen. A function argument is a state itself, which will be frozen, along with anything it captures.

Coroutine functions that cross threads use the same pattern. To allow function blocks to be executed on another thread, 
they are frozen.

In the following example, the data class instance `dc` will be captured by the function block and will be frozen when crossing 
threads. The `println` statement will print `true`.

```kotlin
val dc = DataClass("Hello")
withContext(Dispatchers.Default) {
    println("${dc.isFrozen}")
}
```

When running parallel code, be careful with the captured state. 
Sometimes it's obvious when the state will be captured, but not always. For example:

```kotlin
class SomeModel(val id:IdRec){
    suspend fun saveData() = withContext(Dispatchers.Default){
        saveToDb(id)
    }
}
```

The code inside `saveData` run on another thread. That will freeze `id`, but because `id` is a property of the parent class, 
it will also freeze the parent class.

### Frozen returned data

Data returned from a different thread is also frozen. Even though it's recommended that you return immutable data, you can 
return a mutable state in a way that doesn't allow changing a returned value.

```kotlin
val dc = withContext(Dispatchers.Default) {
    DataClass("Hello Again")
}

println("${dc.isFrozen}")
```

It may be a problem if a mutable state is isolated in a single thread and coroutine threading operations are used for 
communication. If you attempt to return data that retains a reference to the mutable state, it will also freeze the data by 
association.

Learn more about [thread-isolated state](concurrent-mutability.md#thread-isolated-state).

## Multithreaded coroutines

A [special branch](https://github.com/Kotlin/kotlinx.coroutines/tree/native-mt) of the `kotlinx.coroutines` library 
provides support for using multiple threads. It is a separate branch for the reasons listed in the [future concurrency model blog post](https://blog.jetbrains.com/kotlin/2020/07/kotlin-native-memory-management-roadmap/). 

However, you can still use the multithreaded version of `kotlinx.coroutines` in production taking into account its specifics. 
The current version for Kotlin %kotlinVersion% is `%coroutinesVersion%-native-mt`. 

To use the multithreaded version, add a dependency for the `commonMain` source set in `build.gradle.kts`:

```kotlin
commonMain {
    dependencies {
        implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%-native-mt"
    }
}
```

When using other libraries that also depend on `kotlinx.coroutines`, such as Ktor, make sure that they use the single-threaded 
version of `kotlinx-coroutines` that works for all platforms. You can do this with `strictly`:

```kotlin
implementation ("org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%-native-mt"){
    version {
        strictly("%coroutinesVersion%-native-mt")
    }
}
```

Because the main version of `kotlinx.coroutines` is a single-threaded one, libraries will almost certainly rely on this version. 
If you see `InvalidMutabilityException` related to a coroutine operation, it's very likely that you use wrong version.

> Using multithreaded coroutines may result in _memory leaks_. It may be a problem for complex coroutine scenarios under load.
> We are working on a solution for this.
>
{type="note"}

See a [complete example of using multithreaded coroutines in the KMM application](https://github.com/touchlab/KaMPKit).

### Ktor and coroutines

Follow these guidelines when integrating Ktor into your project:

* Make sure that you use the single-threaded version of `kotlinx.coroutines` as a dependency for Ktor. Ktor 
for macOS and iOS requires this version.
* Make calls to Ktor from the main thread. 
* If you're going to use Ktor and coroutines that cross threads, keep a separate `CoroutineScope` for Ktor. As a more complex 
alternative, you can create a child `CoroutineScope` with an isolated `Job`. If `Job` gets frozen, Ktor will stop working. 
See [this example](https://github.com/touchlab/KaMPKit/blob/master/shared/src/iosMain/kotlin/co/touchlab/kampkit/ktor/Platform.kt#L13) for details.

## Alternatives to `kotlinx-coroutines`

There are a few alternatives ways to run parallel code.

### CoroutineWorker

[`CoroutinesWorker`](https://github.com/Autodesk/coroutineworker) is a library published by AutoDesk that implements some 
features of coroutines across threads using the single-threaded version of `kotlinx.coroutines`. 

For simple suspend functions this is a pretty good option, but does not support Flow and other structures.

### Reaktive

[Reaktive](https://github.com/badoo/Reaktive) is an Rx-like library, implementing Reactive extensions for Kotlin Multiplatform. 
It has some coroutine extensions, but is designed around RX and threads primarily.

### Custom processor

For simpler background tasks, you can create your own processor with wrappers around platform specifics. 
See a [simple example](https://github.com/touchlab/KMMWorker).

### Platform concurrency

In production, you can also rely on the platform to handle concurrency. 
It could be helpful when the shared Kotlin code will be used for business logic or data operations rather 
than architecture. 

To share state in iOS across threads, that state needs to be [frozen](concurrency-overview.md#immutable-and-frozen-state). The concurrency libraries mentioned here 
will freeze your data automatically. You will rarely need to do so explicitly, if ever.

If you return data to the iOS platform that should be shared across threads, ensure 
that data is frozen before leaving the iOS boundary.

Kotlin has the concept of frozen only for Kotlin/Native platforms including iOS. To make `freeze` available in common code, 
you can create expect and actual implementations for `freeze`, or use [`stately-common`](https://github.com/touchlab/Stately#stately-common), which provides this functionality. 
In Kotlin/Native, `freeze` will freeze your state, while on the JVM it'll do nothing.

To use `stately-common`, add a dependency for the `commonMain` source set in `build.gradle.kts`:

```kotlin
commonMain {
    dependencies {
        implementation "co.touchlab:stately-common:1.0.x"
    }
}
```

_We'd like to thank the [Touchlab team](https://twitter.com/touchlabhq) for helping us write this article._