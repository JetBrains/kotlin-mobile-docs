[//]: # (title: Integrate KMM into an existing application)
[//]: # (auxiliary-id: Integrate_KMM_into_an_existing_application)

Here you can learn how to integrate Kotlin Multiplatform Mobile (KMM) into an existing Android application and make it multiplatform so that it works on both Android and iOS. 
You will understand what kind of code is good to share and how to use shared code in your existing application.

Sharing code saves you time and effort on writing code and testing it for Android and iOS as you only need to do it once, in the one place.

Before you begin, learn how to [create and configure a KMM application](create-first-app.md).

## Decide what to share

An application includes two layers:

* The **frontend** that defines the user interface and its behavior, including animations and transitions.
* The **backend** that defines the business logic, data management, network stack, and other components that support the frontend.

KMM presumes that the best user experience is when it is native to theheavily dependent on the platform itself, and therefore we don’t recommend sharing it in KMMshould not be shared. 
However, you can share the code for the UI behavior that defines what happens with any user interaction and how the frontend communicates 
with the backend with the help of compatible architectural patterns.

On the backend, the logic includes many platform-specific features such as network requests, local database access, 
hardware manipulation, and cryptographic storage. KMM provides a way to [connect to platform-specific APIs](connect-to-platform-specific-apis.md) 
with the `expect` declaration, by providing the `actual` implementation for each platform (or using a multiplatform library that does this for you).

To summarize, we recommend sharing the following in your application:

| Layer | Recommendation on sharing|
| ----- | -------------- |
| Business logic | **Yes** |
| Platform access | **Yes/no**. You’ll still need to [use platform-specific APIs](connect-to-platform-specific-apis.md), but you can share the behavior.|
| Frontend behavior (reaction to inputs & communication with the backend) | **Yes/no**. Consider these [architectural patterns](#mvp-for-legacy-ui-frameworks).|
| User interface (including animations & transitions) | **No**. It needs to be platform-specific.|

## Integrate KMM into an existing application

1. [Modularize your existing Android application](#modularize-your-current-application).

2. [Identify modules to share](#identify-modules-to-share).

3. [Create a KMM shared module](#create-a-kmm-shared-module).

4. [Extract code to the KMM shared module](#extract-code).

5. [Make your application work on iOS](#make-your-application-work-on-ios).

6. [Test your KMM shared module](#test-your-shared-module).

### Modularize your current application

Refactor your application into independent modules that can work on their own. The [Dependency Injection design pattern](https://developer.android.com/training/dependency-injection) 
is very useful to create such an architecture. 

A module should:
*   Be a simple interface describing its inputs and outputs.
*   Be in charge of a simple describable responsibility, such as database access, file management, a network API, or 
credentials management.

To check if a module is ready for sharing, answer these two questions:
*   Can it be tested with mock dependencies?
*   Can it be mocked?

If you can answer _yes_ to both, then you can share the module.

### Identify modules to share

You can share business modules that have some platform-specific code. However, don't share a module that 
mainly contains platform-specific code, because it would be easier to maintain two specific versions of the module, one for Android and one for iOS.

For example, a module that reads or writes from the device storage contains a lot of platform code like APIs to access files. 
So it is better if each platform project maintains its own version of the module by implementing the same interface.

On the other hand, a module that manages credentials may contain some platform-specific code (such as for encryption), 
but it mostly provides the logic that is common for both platforms. That's why it's a perfect candidate for sharing.

You can also choose to share the UI behavior using the [Model-View-Presenter (_MVP_)](#mvp-for-legacy-ui-frameworks) 
or the [Model-View-Intent (_MVI_)](#mvi-for-declarative-ui-frameworks) pattern. These patterns:

*   Make a clear distinction between the UI and presentation layers.
*   Are completely decoupled from the UI platform.
*   Are easily testable without a UI environment.

#### MVP for legacy UI frameworks {initial-collapse-state="collapsed"}

_Model-View-Presenter_ (_MVP_) forces you to create an API for both the Presenter that receives inputs and the View that displays outputs, allowing 
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

Note the `commandView` and `lastCommand` mechanism, which allows a view to detach and re-attach, for example for configuration changes on Android.

#### MVI for declarative UI frameworks {initial-collapse-state="collapsed"}

_Model-View-Intent_ (_MVI_) is the natural evolution of MVP when working with declarative UI frameworks (although it also works with legacy UI frameworks). 
It is therefore recommended to be used with Swift UI or Jetpack Compose.

In MVI, the entire UI structure is described in one tree. Here is an example of a simple MVI presenter:

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

Note the `displayModel` and `lastModel` mechanism, which allows a view to detach and re-attach, for example for configuration changes on Android.

### Create a KMM shared module

In your Android project, create a KMM shared module for your shared code.

> The KMM Shared Module Wizard is available in Android Studio version 4.1 or higher.
>
{type="note"}

1. In Android Studio, click **New** | **New Module**. 

2. In the list of module types, select **KMM Shared Module** and then click **Next**.

    ![KMM shared module](kmm-module-wizard-1.png) 

3. Select the **Add sample tests** checkbox.

    ![KMM shared module configuration](kmm-module-wizard-2.png) 
4. Click **Finish**.
    
### Extract code

You can now extract code to a KMM shared module starting from the backend of your application and working your way up.
Start with a pure logic module that requires as little platform access as possible, and then continue working with modules 
that require platform access such as data storage and network requests. 
 
For each module:
1. Add the shared code to the `commonMain` source set.
2. Add Android-specific code to an Android-specific source set and share it in `commonMain` with [`expect` and `actual` declarations](connect-to-platform-specific-apis.md).
3. Run the application on Android to ensure that everything works correctly.

### Make your application work on iOS

For `expect` declarations in the shared code, add the required [`actual` implementations for iOS](connect-to-platform-specific-apis.md).

If you don't have an iOS application, create an Xcode project and specify the path to it in `gradle.properties`. 
If you already have an Xcode project, simply specify a relative or absolute path to the project.

```kotlin
xcodeproj=./iosApp
```

Once you're done, run your application on iOS to ensure that it works correctly.  

### Test your shared module

Kotlin provides a [multiplatform testing library](https://kotlinlang.org/api/latest/kotlin.test/) that you can use for writing unit tests. 
Launch your tests on each platform to ensure that your actual declarations work the same way on Android and iOS.

As an example, you can use sample tests that are added to the `commonTest`, `androidTest`, and `iosTest` source sets. 
Learn more about [running sample tests](create-first-app.md#run-tests).

## Next steps

Using a shared module as part of your KMM application project is good for getting started with KMM and finding your way around. 
Later, when you’ve worked with the shared module with your team and you decide to use it in other projects, you can move it to a separate
project as a multiplatform library, [publish your library](https://kotlinlang.org/docs/reference/mpp-publish-lib.html), and [use it in your projects as a dependency](https://kotlinlang.org/docs/reference/mpp-add-dependencies.html).

_We'd like to thank the [Kodein Koders team](https://twitter.com/kodeinkoders) for helping us write this article._
