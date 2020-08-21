[//]: # "title: Concurrency with coroutines"
[//]: # "auxiliary-id: Concurrency_with_Coroutines"

When working with mobile platforms, you may need to write multi-threaded code that runs in parallel. For this purpose, 
you can use single-threaded or multi-threaded version of the `kotlinx.coroutines` library and alternative solutions.

Review the pros and cons of each solution and choose the one that works the best for your situation.

Learn more about [concurrency, current approach, and future improvements](concurrency-overview.md).

## Single-threaded coroutines

The `kotlinx.coroutines` library provides a number of features on top of the language-level coroutine primitives. Learn 
more about [coroutines]().

The current version of `kotlinx.coroutines` for Kotlin/Native including iOS supports only single-threaded usage. 
You cannot send work to other threads by changing the `Dispatcher` instance. For Kotlin %kotlinVersion%, the 
official version is `%coroutinesVersion%`.

You can suspend execution and do work on other threads while using a different mechanism for scheduling  
and managing that work. However, this version of `kotlinx.coroutines` cannot change threads on its own.

There is also [another version of `kotlinx.coroutines`](#multithreaded-coroutines) that provides support for multiple threads.

Get aquited with the main concepts of using coroutines:

* Asynchronous vs. parallel processing
* Dispatcher for changing threads
* Freezing captured data
* Freezing returned data

### Asynchronous vs. parallel processing

Let's make a distinction between asynchronous and parallel processing. Within a coroutine, the sequence of processing 
may be suspended and resumed later. This allows for asynchronous, non-blocking code, without using callbacks or promises. 
That is asynchronous, but everything related to that coroutine may happen in a single thread. 

The following code makes a network call using [Ktor](). In the main thread, the call is initiated 
and suspended, while the actual networking is performed by another underlying process. When completed, the code is resumed 
in the main thread.

```kotlin
val client = HttpClient
//Running in main thread, start a get call
client.get<String>("https://example.com/some/rest/call")
//The get call will suspend and let other work happen in the main thread, and resume when the get call completes
```

That is distinct from parallel code. Parallel code will need to be run in another thread. Depending on the libraries 
you use and what you need to do, it's possible you won't ever need to use multiple threads.

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

### Freezing captured data

To run code on a different thread, you pass a `functionBlock`, which gets frozen, and then run on another thread. 

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
Kotlin/Native must be frozen. A function argument is itself a state, which will be frozen, along with anything it captures.

Coroutine functions that cross threads use the same pattern. To allow function blocks to be executed on another thread, 
they are frozen.

In the following example, the data class instance `dc` will be captured by the function block, and frozen when crossing 
threads. The `println` statement will print `true`.

```kotlin
val dc = DataClass("Hello")
withContext(Dispatchers.Default) {
    println("${dc.isFrozen}")
}
```

As with other methods of running parallel code in Kotlin/Native, you'll need to be careful with the captured state. 
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

### Freezing returned data

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

Learn more about [thread-isolated state]().

## Multi-threaded coroutines

There is an alternate branch for `kotlinx.coroutines` that provide support for using multiple threads. 

It is a separate branch for the reasons listed in the [concurrency model update post](https://blog.jetbrains.com/kotlin/2020/07/kotlin-native-memory-management-roadmap/), 
and will likely remain a separate branch until those updates are complete. 
However, with some considerations kept in mind, the multithreaded version of `kotlinx.coroutines` works great, and can be 
used in production environments.

You can use multi-threaded version of `kotlinx.coroutines` for Kotlin/Native. The current version for Kotlin %kotlinVersion% 
is `%coroutinesVersion%-native-mt`. For the `commonMain` source set in the `build.gradle.kts`, add the dependency:

```kotlin
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%-native-mt"
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

Because the "official" version of `kotlinx.coroutines` is the single-threaded one, any library that uses `kotlinx.coroutines` 
will almost certainly use the single-threaded release. 
If you see `InvalidMutabilityException` related to coroutine operation, it's very likely that your build is getting the wrong version.

> Using multi-threaded coroutines may result in _memory leaks_. It may be a problem for complex coroutine scenarios under load.
> We are working on a solution for this.
{type="note"}

See a [complete example of using multithreaded coroutines in the KMM application](https://github.com/touchlab/KaMPKit).

### Ktor and coroutines

Follow these guidelines when integrating Ktor into your project:

* Make sure that you use the single-threaded version of `kotlinx.coroutines` as a dependency for Ktor. This is because Ktor 
for macOS and iOS was designed this way.
* Make calls to Ktor from the main thread. 
* If you're going to use Ktor and coroutines that cross threads, keep a separate `CoroutineScope` for Ktor. As a more complex 
alternative, you can create a child `CoroutineScope` with an isolated `Job`. If the `Job` gets frozen, Ktor will stop working. 
See [this example](https://github.com/touchlab/KaMPKit/blob/master/shared/src/iosMain/kotlin/co/touchlab/kampkit/ktor/Platform.kt#L13) for details.

There is an [issue](https://youtrack.jetbrains.com/issue/KTOR-499) to track the status of Ktor and multithreaded coroutines. 
Keep an eye on that for progress. The most recent version of Ktor also includes multithreaded coroutines support in the CIO client, but that currently does not support HTTPS: [ktor-1-4-0-now-available](https://blog.jetbrains.com/ktor/2020/08/18/ktor-1-4-0-now-available/).

## Alternatives to `kotlinx-coroutines`

### CoroutineWorker

[`CoroutinesWorker`](https://github.com/Autodesk/coroutineworker) is a library published by AutoDesk that implements some 
features of coroutines across threads, using the single-threaded version of `kotlinx.coroutines`. 

For simple suspend functions this is a pretty good option, but does not support Flow and other structures.

### Reaktive

[Reaktive](https://github.com/badoo/Reaktive) is an Rx-like library, implementing Reactive extensions for Kotlin Multiplatform. 
It has some coroutine extensions, but is designed around RX and threads primarily.

### Custom processor

For simpler background tasks, creating your own processor is always an option. You can create wrappers around platform-specific. 
See a [simple example](https://github.com/touchlab/KMMWorker).

### Platform concurrency

Another practical option for production use is to rely on the platform to handle concurrency. 
It could be helpful when the shared Kotlin code will be used for business logic or data operations rather 
than architecture. 

To share state in Kotlin/Native across threads, that state needs to be frozen. All of the concurrency libraries discussed here 
will freeze your data automatically. You will rarely, if ever, need to do so explicitly.

If you return data to the iOS platform that should be shared across threads, you need to ensure 
that data is frozen before leaving the Kotlin/Native boundary.

Kotlin, on platforms other than Kotlin/Native, have no concept of frozen. To make `freeze` available in common code, 
you can create expect and actual implementations for `freeze`, or use `stately-common`, which provides this functionality. 
In Kotlin/Native, `freeze` will freeze your state, while on the JVM (or JS), it'll do nothing.

```kotlin
dependencies {
  implementation 'co.touchlab:stately-common:1.0.x'
}
```

See [stately-common](https://github.com/touchlab/Stately#stately-common) for more information.


