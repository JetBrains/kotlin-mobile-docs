[//]: # (title: Integrate KMM in an existing application)
[//]: # (auxiliary-id: Integrate_KMM_in_an_existing_application)

Here you can learn how you can integrate KMM in an existing Android application and make it multiplatform so that it works on both Android and iOS. 
You will understand what code is good to share and how to use shared code in your existing application.

Sharing code saves your time and effort on writing code and testing it for Android and iOS – you do this once in one place.

Before you begin, learn how to [create and configure a KMM application](create-first-app.md)

## Decide what to share

An application includes two layers:

* The **frontend** that defines the user interface and its behavior, including animations & transitions.
* The **backend** that defines the business logic, data management, network stack, and other components that support frontend.

KMM assumes that the best user experience is heavily dependent on the platform itself, and therefore should not be shared. 
However, you can share code for the UI behavior that defines what happens on any user interaction and how the frontend communicates 
with the backend with the help of compatible architectural patterns.

On the backend, the logic itself includes many platform-specific features such as network requests, local database access, 
hardware manipulation, and cryptographic storage. KMM provides a way to [connect to platform-specific APIs](connect-to-platform-specific-apis.md) 
with the `expect` declaration by providing the  `actual` implementation for each platform (or using a multiplatform library that does this 
for you).

To summarize what we recommend that you share in your application:

| Layer | Recommendation |
| ----- | -------------- |
| User Interface (including animations & transitions) | **No**, needs to be platform specific|
| Frontend behavior (reaction to inputs & communication with the backend) | **Yes**, with [architectural patterns](#mvp-for-legacy-ui-frameworks) |
| Business logic | **Yes** |
| Platform access | **Yes/no**. You’ll still need to [use platform-specific APIs](connect-to-platform-specific-apis.md) but you can share the behavior.|

## Integrate KMM into an existing application

1. [Modularize your existing Android application](#modularize-your-current-application).

2. [Identify modules to share](#identify-modules-to-share).

3. [Create a KMM shared module](#create-a-kmm-shared-module).

4. [Extract modules to the KMM shared module](#extract-modules).

5. [Implement `actual` declarations for iOS](#implement-actual-declarations-for-ios).

6. [Test your KMM shared module](#test-your-shared-module).

### Modularize your current application

Refactor your application into independent modules that can work on their own. The [Dependency Injection design pattern](https://developer.android.com/training/dependency-injection) 
is very useful to create such architecture. 

A module should:
*   Be a simple interface describing its inputs and outputs.
*   Be in charge of a simple describable responsibility, such as database access, file management, network API or 
credentials management.

A good way to check if a module is ready for sharing is to answer two questions:
*   Can it be tested with mock dependencies?
*   Can it be mocked?

If both answers are _yes_, then you can share the module!

### Identify modules to share

You can share business modules that have some platform-specific code. However, don't share a module that 
mainly contains platform-specific code – it is easier to maintain specific versions of the module for Android and iOS.

For example, a module that reads or writes from the device storage contains much platform code like APIs to access files. 
So it's easier if each platform project maintains its own version of the module by implementing the same interface.

On the other hand, a module that manages credentials may contain some platform-specific code (for example, for encryption) 
but it mostly provides the logic that is common for both platforms. That's why it's a perfect candidate for sharing.

You can also choose to share the UI behavior using the [Model View Presenter (_MVP_)](#mvp-for-legacy-ui-frameworks) 
or its evolution [Model View Intent (_MVI_)](#mvi-for-declarative-ui-frameworks) pattern. These patterns:

*   Make a clear distinction between the UI and presentation layers.
*   Are completely decoupled from the UI platform.
*   Are easily testable without an UI environment.

#### MVP for legacy UI frameworks {initial-collapse-state="collapsed"}

_Model View Presenter_ (_MVP_) forces you to create an API for both the Presenter that receives inputs and the View that displays outputs, allowing 
you to test each independently.

Here is an example of a simple MVP presenter:

```kotlin
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

Pay attention to the `commandView` and `lastCommand` mechanism that allows a view to detach and re-attach, for example for configuration changes on Android.

#### MVI for declarative UI frameworks {initial-collapse-state="collapsed"}

_Model View Intent_ (_MVI_) is the natural evolution of MVP when working with declarative UI frameworks (although it also works with “legacy” frameworks). 
It is therefore recommended with Swift UI or Jetpack Compose.

In MVI, the entire UI structure is described in one tree structure. Here is an example of a simple MVI presenter:

```kotlin
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

Pay attention to the `displayModel` and `lastModel` mechanism that allows a view to detach and re-attach, for example for configuration changes on Android.

### Create a KMM shared module

In your Android project, create a KMM shared module for your shared code.

1. In Android Studio, click **New** | **New Module**. 

2. In the list of module types, select **KMM Shared Module** and click **Next**.

    ![KMM shared module](kmm-module-wizard-1.png) 

3. Select a checkbox to add sample tests and click **Finish**.

    ![KMM shared module configuration](kmm-module-wizard-2.png) 
    
### Extract modules

You can now extract modules to a KMM shared module starting from the backend of your application and working the way up.
Start with a pure logic module that requires as little as platform access as possible and continue working with modules 
that require platform access such as data storage and network request. 
 
For each business logic module:
* Add the shared code to the `commonMain` source set.
* Add Android-specific code to an Android-specific source set and share it in `commonMain` with [`expect` and `actual` declarations](connect-to-platform-specific-apis.md).

### Implement `actual` declarations for iOS

For `expect` declarations in the shared code, add required `actual` implementations for iOS. 

### Test your shared module

Kotlin provides a [multiplatform testing library](https://kotlinlang.org/api/latest/kotlin.test/) that you can use for writing unit tests. 
Launch your tests on each platform to ensure that your actual declarations work the same way on Android and iOS.

As an example, you can use sample tests that are added to the `commonTest`, `androidTest`, and `iosTest` source sets. Learn more about [running sample tests](create-first-app.md#run-tests).

## Next steps

Using a shared module as part of your KMM application project is good for beginning and getting experience with KMM. 
However, when you continue working with the shared module with your team and decide to use it in other projects, you can move it to a separate
project as a multiplatform library, publish it, and use in your projects as a dependency.

Learn more about [publishing a library for Android](https://kotlinlang.org/docs/reference/mpp-publish-lib.html) and [using your library as a dependency](https://kotlinlang.org/docs/reference/mpp-add-dependencies.html).

For iOS, you can:
* [Build your library as an iOS framework](https://kotlinlang.org/docs/reference/mpp-build-native-binaries.html) and use it in other projects.
* [Publish a library with the iOS target and use it as a CocoaPods dependency](https://kotlinlang.org/docs/reference/native/cocoapods.html) (known as a Kotlin Pod).


_We'd like to thank the [Kodein Koders team](https://twitter.com/kodeinkoders) for helping us write this article._
