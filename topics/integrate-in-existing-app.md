[//]: # (title: Integrate in existing app)
[//]: # (auxiliary-id: How_to_integrate_KMM_in_your_existing_application)

## How to integrate Kotlin Multiplatform Mobile in your existing application

This guide will walk us through the steps that we need to take to empower our KMM experience in a mobile project. It will expose what we should abstract in a multiplatform library and how to abstract it, as well as how to deal with existing code and what are the best practices to embrace KMM.

Note that we assume that you already know [how to create and configure a KMM project](create-first-app.md)

## Deciding on what to abstract in a multiplatform library

### What should and should not be abstracted

You can divide an application architecture into two groups: the frontend and the backend.

The frontend contains the user interface and its behaviour, animations & transitions, etc.

The backend contains the business logic, persistence management, network stack, etc.

Let’s talk frontend. KMM assumes that the best user experience an app can deliver is heavily dependent on the platform itself, and therefore cannot be abstracted away.

However, that does not mean that the entire frontend cannot be abstracted. The UI behaviour (meaning what happens on any use interaction as well as how the communication with the backend is handled) can in fact be abstracted by using some compatible architectural pattern.

On the backend side, it’s easy to think that everything can be abstracted, since it’s only logic.

Well, it’s not. The logic itself uses a lot of platform features that are different from platforms to platforms, things like network requests, local database access, hardware manipulation, cryptographic storage, etc. KMM provides a way to abstract platform access with **expect**, but you still need to write each platform's **actual** **implementation (or use a multiplatform library that does it for you).

So, back to our question : what should or should not be abstracted?

If you see you’re application as a layered architecture, ranging from UI at the top and platform access at the bottom, then the answer is _anything in between_:

*   User Interface (including animations & transitions): **no**, needs to be platform specific.
*   Frontend behaviour (reaction to inputs & communication with backend): **yes**, with architectural patterns.
*   Business logic: **yes**, it’s only code !
*   Platform access: **yes and no**, you’ll still need to use platform specific APIs but you can abstract the behaviour.  

### How does my library can integrate with my existing code?

KMM compiles natively to each platform’s native format. This means that your library will be compiled to a **jar** or an **aar** archive for Android (dependent on the project configuration) and a **framework** directory for iOS, so it’s very easy to add them to your Android or iOS application project.

We recommend that you start from the backend of your application and work your way up.

Start with a pure logic module that requires as little as platform access as possible (for example, input validation, or data transformation). This lets you create a multiplatform project, its build (don’t forget to test!) and integration to application projects, which is enough work to not add platform specific features and tests!

Once you have a library that compiles and integrates, as well as a simple module that works, you can start working on modules that require platform access such as data persistence or network request.

A module API that is designed for multiplatform should represent a business unit by itself and easily represented by an interface. In essence, it should fulfill the same requirements as a module in dependency injection.

### Using trendy patterns to push the limits of a multiplatform library

Once you have abstracted your business code, you can also abstract your UI behaviour. Doing so ensures that most of your application (including its behaviour) is written and tested once. It also leaves to the platform the only thing that is truly specific to it: it’s UI.

For example, you may use either the MVP or its evolution the MVI pattern. Such patterns are adapted to multiplatform because they :

*   make a clear distinction between the UI layer and the presentation layer
*   are completely decoupled from the UI platform
*   are easily testable without a UI environment

#### Model View Presenter for “legacy” UI frameworks

MVP forces you to create an API for both the Presenter (that receives inputs) and the view (that displays outputs), allowing you to test each independently.

Here is an example of a simple MVP presenter:


```Kotlin
class LoginPresenter {
    interface View {
        fun displayError(code: Int)
        fun displayLoading(loading: Boolean)
        fun goToNextScreen()
    }

    private var view: View? = null
    private var lastCommand: View.() -> Unit =
        { displayLoading(false) }
    private fun commandView(command: View.() -> Unit) {
        lastCommand = command
        view?.command()
    }

    fun attach(view: View) { this.view = view.apply(lastCommand) }
    fun detach() { this.view = null }

    fun login(username: String, password: String) {
        MainScope().launch {
            try {
                commandView { displayLoading(true) }
                getNetwork().login(username, password) // suspending
                commandView { goToNextScreen() }
            } catch (ex: LoginException) {
                commandView {
                    displayLoading(false)
                    displayError(ex.code)
                }
            }
        }
    }
}
```


Note the commandView and lastCommand mechanism that allow a view to detach and re-attach for things such as configuration changes on Android.

#### Model View Intent for declarative UI frameworks

MVI is the natural evolution of MVP when working with declarative UI frameworks (although it also works with “legacy” frameworks). It is therefore recommended with Swift UI or Jetpack Compose.

In MVI, the entire UI structure must be described in one tree structure. Here is an example of a simple MVI presenter:

```Kotlin
class LoginPresenter {
    sealed class Model {
        object Form : Model()
        object Loading : Model()
        data class Error(val code: Int): Model()
        object GoToNextScreen : Model()
    }
    interface View {
        fun displayModel(model: Model)
    }

    private var view: View? = null
    private var lastModel: Model = Model.Form
    private fun displayModel(model: Model) {
        lastModel = model
        view?.displayModel(model)
    }

    fun attach(view: View) {
        this.view = view.apply { displayModel(lastModel) }
    }
    fun detach() { this.view = null }

    sealed class Intent {
        data class Login(val username: String, val password: String)
            : Intent()
    }

    fun process(intent: Intent) {
        when (intent) {
            is Intent.Login -> {
                MainScope().launch {
                    try {
                        displayModel(Model.Loading)
                        getNetwork().login(
                            intent.username, intent.password) // suspending
                        displayModel(Model.GoToNextScreen)
                    } catch (ex: LoginException) {
                        displayModel(Model.Error(ex.code))
                    }
                }
            }
        }
    }
}
```


Note the displayModel and lastModel mechanism that allow a view to detach and re-attach for things such as configuration changes on Android.

## Moving code from a platform project into a multiplatform library

### Abstracting business code

As we saw earlier, business logic is the easiest thing to port to KMM.

There is, however, a paradigm shift that takes a bit of time, for an Android developer, to integrate. You need to go from “It needs to work on Android” to “it needs to work”. To that end, we cannot emphasize enough how much testing is important !

#### Step 1: Modularize your current application

Before even creating a multiplatform project, you need to be able to extract part of your Android application. To that end, you first need to refactor your application into independent modules that can work on their own. The Dependency Injection design pattern is very useful to create such architecture.

A module should :
*   Be reduced to a simple interface describing its inputs and outputs.
*   Be in charge of a simple describable responsibility, such as “database access”, “file management”, “network API” or “credentials management”.

A good way to know if a module is ready to be abstracted is to answer two questions:
*   Can it be tested with mock dependencies?
*   Can it be mocked?

If both answers are yes, then you got yourself a module ready to be abstracted!

#### Step 2: Identify business and platform modules

Business module may have platform specific code, and that’s OK! They can be abstracted with little effort.

However, a module that mainly contains platform code should not be abstracted. It will probably be a lot easier to simply maintain a specific version of the module for each targeted platform.

For example, a module to read or write from the device storage will probably contain a lot of platform code (such as APIs to access files), and will be a lot easier to maintain if each platform project maintains its own version of the module by implementing the same interface.

On the other hand, a “Credentials management” module may contain some platform access (for example, for encryption) but will mainly contain business “universal” logic, which makes it perfect for extraction.

#### Step 3: Extract

You can now create a multiplatform library project with only one target : Android.

Port one of your business logic modules into it. The code should go into the “common” sourceset.

If there is some Android specific code in there, it needs to go to the android specific sourceset and be abstracted in common with the [expect / actual](connect-to-platform-specific-apis.md) mechanism.

You should now be able to compile an Android AAR, import it into your android specific project, and use it in your application.

#### Step 4: Implement actual declarations for iOS platform

Add the iOS platform to your multiplatform project and implement each needed actual declaration. See the example of the basic project configuration on the [discover kmm project](discover-kmm-project.md) page.

#### Step 5: Test it!

Test matters. Kotlin provides a [multiplatform testing library](https://kotlinlang.org/api/latest/kotlin.test/). You should write unit tests to prove your code works as intended!

You must launch your tests on each platforms, which ensures that each of your actual declarations work the same way on each platform.

### Deploying to mobile projects

#### Android

Deploying to Android is very straightforward : you simply need to add the `maven-publish` Gradle plugin, and deploy to a maven repository.

You don’t even need an external maven repository. You can run `./gradlew publishToMavenLocal` to put the library in your local repository (a directory on your computer).

Once into the repository, you can have your application project depend on the library with a simple dependency in the `build.gradle.kts`:

```Kotlin
implementation("com.mycompany.mylib:business-lib:1.0.0")
```

#### iOS

There are two ways you can deploy your library to an iOS application:

*   Use the [Kotlin/Native cocoapods plugin](https://kotlinlang.org/docs/reference/native/cocoapods.html), which allows you to use a Kotlin Multiplatform project with native targets as a CocoaPods dependency (Kotlin Pod)
*   [Configure your project to create an iOS framework](https://kotlinlang.org/docs/tutorials/native/apple-framework.html) and manually copy it into the iOS application project.


_We'd like to thank the [Kodein Koders team](https://twitter.com/kodeinkoders) for helping us write this article._
