[//]: # (title: Networking with Ktor)
[//]: # (auxiliary-id: Networking_with_Ktor)

The standard approach in modern client-server architectures is to use the HTTP protocol for 
transferring data between the server and the client. An HTTP client is a mandatory tool for mobile
app allowing it to interact with the server. One of the options for a network client for the Kotlin
Mobile Multiplatform (*KMM*) project is **Ktor** - a framework that allows you to create
asynchronous clients and servers. As part of a multiplatform project, the Ktor Library can also be
used as an HTTP client. One of Ktor's features is its use of **Kotlin Coroutines** and suspend
functions in the external UI of asynchronous network operations. For more detailed information, see
the Official [Documentation](https://ktor.io/).

This article discusses connection of a Ktor client to the multiplatform project, creation and
configuring of an HTTP client, and using it to perform basic network queries.

## Connecting a Ktor client to the KMM project

A Ktor HTTP client can be used in the Kotlin Multiplatform project under the platform and common
code. The library is connected using Gradle DSL, by adding the required dependencies in the
`dependencies` block in the `build.gradle` file of the module.

To work with Ktor in the common code, you need to add an artifact to the list of dependencies in the
`commonMain` set in the `build.gradle` file of the KMM module:

```groovy
commonMain {
    dependencies {
        implementation "io.ktor:ktor-client-core:$ktor_version"
    }
}
```

To connect the library for the Android platform in the KMM module to the corresponding set of
sources, you need to add the `ktor-client-android` dependency:

```groovy
androidMain {
    dependencies {
        implementation "io.ktor:ktor-client-android:$ktor_version"
    }
}
```

To connect to the iOS platform, you need to add the `ktor-client-ios` dependency to the required set
of sources:

```groovy
iosArm64Main {
    dependencies {
        implementation "io.ktor:ktor-client-ios:$ktor_version"
    }
}
iosX64Main {
    dependencies {
        implementation "io.ktor:ktor-client-ios:$ktor_version"
    }
}
```

Instead of `$ktor_version`, you need to indicate the required version of the library.

After synchronization of the Gradle project, the library will be connected and available for use.

For more information on connecting a Ktor client to the multiplatform project, see the
[Documentation](https://ktor.io/clients/http-client/multiplatform.html).

## Creating and configuring the HTTP Client

To create an HTTP client with default settings, you just need to employ the `HttpClient()` function:

```kotlin
val httpClient: HttpClient = HttpClient()
```

This method of creating a client can be used in both the common and platform codes.

Specific implementation of the HTTP engine, which is published in separate Gradle artifacts, may be
transferred to the argument of the `HttpClient()` function. Therefore, you must separately connect
the appropriate Gradle dependency to use an alternative engine. If you call up the function of
creating a client without an argument, then one of the engines available to Ktor will be
automatically selected at the moment of project compilation. For the full list of supported HTTP
engines, refer to the [Documentation](https://ktor.io/clients/http-client/engines.html).

Since the configuration process can be implemented in both the common and platform code, there are
several approaches to creating a client. You can either transfer the link to the library with a
common code for the object of a ready-made HTTP client or send links to engines if the version with
the default engine is not a good fit.

Ktor also gives you an option of using a special HTTP engine to test `MockEngine`, which simulates
the execution of queries without actual connection to the API endpoint:

```kotlin
val httpClient: HttpClient = HttpClient(MockEngine)
```

Just as all other engines, `MockEngine` is distributed as a separate Gradle artifact, so the
connection must be performed separately. For more detailed information on testing, refer to the
[Documentation](https://ktor.io/clients/http-client/testing.html#).

Client configuration can be done through a lambda expression with the receiver. In other words, the
receiver object of the `HttpClientConfig` class for a specific HTTP engine through which the entire
configuration is performed will be transferred to the lambda, which is transferred as an argument to
the `HttpClient()` function.

The following is an example of HTTP client configuration:

```kotlin
val httpClient = HttpClient {
    expectSuccess = false
    ResponseObserver { response ->
        println("HTTP status: ${response.status.value}")
    }
}
```

In this case, it is indicated with regard to the HTTP client that no exceptions must be created when
receiving HTTP statuses indicating errors or redirecting using the `expectSuccess` property. Also,
`ResponseObserver` is used to add an observer for all received responses, in which a string with a
status code is included in its standard output.

After you finish working with the HTTP client, you need to free up the resources that it uses (pool
threads, busy connections, and `CoroutineScope` for coroutines). To do this, call up the `close`
function in `HttpClient`:

```kotlin
httpClient.close()
```

If you need to use `HttpClient` for one single query, then you can use an extension function - `use`
- which will automatically call up `close` after executing the code block:

```kotlin
val status = HttpClient().use { httpClient ->
    // ...
}
```

Bear in mind that the function `close` function prohibits the creation of new queries, but does not
terminate currently active ones. Resources will only be released after all client queries have been
completed.

For more information on creating and configuring a client, see the
[Documentation](https://ktor.io/clients/http-client/quick-start/client.html).

### Adding a Feature

Ktor allows for connecting additional HTTP client functionality during the configuration stage. To
do this, use the `install` function, which admits heirs of the `HttpClientFeature` UI into the
argument.

For example, you can use the `ResponseObserver` class to set up an observer for responses. At the
beginning of the article, an observer was added using the `ResponseObserver{}` builder function,
which internally calls up the `install` function. An observer as additional functionality can be
explicitly added as follows:

```kotlin
val httpClient = HttpClient {
    install(ResponseObserver) {
        onResponse { response ->
            println("HTTP status: ${response.status.value}")
        }
    }
}
```

Other functions for the HTTP client, such as logging all queries and responses or automatic
serialization of objects, can also be connected this way. Ktor comes with a kit of ready-made
additional functionalities, which are connected as separate Gradle artifacts in the `dependencies{}`
block. For description of the entire list, refer to the
[Documentation](https://ktor.io/clients/http-client/features.html).

## Creating HTTP queries

The main function for creating queries is `request` - an extension function for the `HttpClient`
class. All query settings are generated using the `HttpRequestBuilder` class. The function `request`
is marked with the `suspend` modifier, so all queries must be executed in coroutines.

For example, a `GET` query, whose result comes as a string, may look as follows:

```kotlin
val htmlContent = httpClient.request<String> {
    url("https://en.wikipedia.org/wiki/Main_Page")
    method = HttpMethod.Get
}
```

You can use the `headers` extension function to add headers to the HTTP query; therefore, just add
the `headers{}` block inside the `request` builder block, e.g.:

```kotlin
val htmlContent = httpClient.request<String> {
    url("https://en.wikipedia.org/wiki/Main_Page")
    method = HttpMethod.Get

    headers {
        append("Accept", "application/json")
        append("Authorization", "oauth token")
    }
}
```

To indicate the query body, you must assign a value to the public `body` property in the
`HttpRequestBuilder` class. You can assign a string or inheritance class objects `OutgoingContent`
to this property. For example, sending data with a `text/plain` text MIME type can be implemented as
follows:

```kotlin
val htmlContent = httpClient.request<String> {
    url("http://127.0.0.1:8080/")
    method = HttpMethod.Post

    body = TextContent(
        text = "Body content",
        contentType = ContentType.Text.Plain
    )
}
```

Instead of `String`, to obtain more information on query results, such as HTTP status, you can use
the `HttpResponse` type:

```kotlin
val response = httpClient.request<HttpResponse> {
    url("https://en.wikipedia.org/wiki/Main_Page")
    method = HttpMethod.Get
}
if (response.status == HttpStatusCode.OK) {
    // HTTP-200
}
```

For more information about the `HttpResponse`, refer to the
[Documentation](https://ktor.io/clients/http-client/quick-start/responses.html).

You can also obtain query results in the form of a byte array:

```kotlin
val response = httpClient.request<ByteArray> {
    url("https://en.wikipedia.org/wiki/Main_Page")
    method = HttpMethod.Get
}
```

The Library is equipped with extension functions for the `HttpClient` class for carrying out basic
HTTP methods: `get`, `post`, `put`, `patch`, `delete`, `options`, `head`.

Executing a `GET` query based on the corresponding extension function with an additional header will
look as follows:

```kotlin
val response = httpClient.get<HttpResponse>("http://127.0.0.1:8080/") {
    headers {
        append("Accept", "application/json")
    }
}
```

A `POST` query can be implemented as follows:

```kotlin
val response = httpClient.post<HttpResponse>("http://127.0.0.1:8080/") {
    headers {
        append("Authorization", "token")
    }
    body = "Command"
}
```

### Concurrency

The external UI of the Ktor Library is based on `suspend` functions, so Kotlin Coroutines are used
when working with asynchronous queries. Therefore, all network queries must be executed in
coroutines, which will suspend their execution while awaiting a response.

For concurrent execution of two or more queries, you can use coroutine builders: `launch` or `async`.
E.g., sending two concurrent queries using `async` might look as follows:

```kotlin
suspend fun parallelRequests() = coroutineScope<Unit> {
    val httpClient = HttpClient()

    val firstRequest = async { httpClient.get<ByteArray>("https://127.0.0.1:8080/a") }
    val secondRequest = async { httpClient.get<ByteArray>("https://127.0.0.1:8080/b") }

    val bytes1 = firstRequest.await()    // Suspension point.
    val bytes2 = secondRequest.await()   // Suspension point.

    httpClient.close()
}
```

### Multipart queries

To send Multipart when constructing a query, a `MultiPartFormDataContent` type object must be
transferred to the `body` property, which object takes on the `parts: List<PartData>` argument in
the constructor. To create this list, the `FormBuilder` builder class is used, in which multiple
variations of the `append` functions for adding the necessary data are announced. Work with the
builder is simplified by the `formData` builder function, which accepts a lambda with the
`FormBuilder` receiver.

An example of creating a simple POST query with Multipart data may look as follows:

```kotlin
val request: String = httpClient.post("http://127.0.0.1:8080/") {
    body = MultiPartFormDataContent(
        formData {
            append("key", "value")
        }
    )
}
```
