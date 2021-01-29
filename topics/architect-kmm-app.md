[//]: # (title: Architect your KMM application)
[//]: # (auxiliary-id: Architect_your_KMM_application)

This document provides architecture guidelines on how to decide which layers of your application make multiplatform that work both on iOS and Android and which keep native.

A KMM application includes two layers:

* The **frontend** that defines the user interface and its behavior, including animations and transitions.
* The **backend** that defines the business logic, data management, network stack, and other components that support the frontend.

KMM presumes that the best user experience is when it is native to the platform itself, and therefore we don’t recommend sharing it in KMM.
However, you can share the code for the UI behavior that defines what happens with any user interaction and how the frontend communicates
with the backend with the help of [compatible architectural patterns](#architectural-patterns-for-sharing-ui-behavior).

On the backend, the logic includes many platform-specific features such as network requests, local database access,
hardware manipulation, and cryptographic storage. You can use a multiplatform library that does this for you or [connect to platform-specific APIs](connect-to-platform-specific-apis.md)
with the `expect` declaration, by providing the `actual` implementation for each platform.

## Architectural patterns for sharing UI behavior {initial-collapse-state="collapsed"}

You can choose to share the UI behavior using the [Model-View-Presenter (_MVP_)](#mvp-for-legacy-ui-frameworks)
or the [Model-View-Intent (_MVI_)](#mvi-for-declarative-ui-frameworks) pattern. These patterns:

*   Make a clear distinction between the UI and presentation layers.
*   Are completely decoupled from the UI platform.
*   Are easily testable without a UI environment.

### MVP for legacy UI frameworks 

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

### MVI for declarative UI frameworks 

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

## Best practices for the KMM app architecture

To summarize, we recommend sharing the following in your application:

| Layer | Recommendation on sharing|
| ----- | -------------- |
| Business logic | **Yes** |
| Platform access | **Yes/no**. You’ll still need to [use platform-specific APIs](connect-to-platform-specific-apis.md), but you can share the behavior.|
| Frontend behavior (reaction to inputs & communication with the backend) | **Yes/no**. Consider these [architectural patterns](#architectural-patterns-for-sharing-ui-behavior).|
| User interface (including animations & transitions) | **No**. It needs to be platform-specific.|

_We'd like to thank the [Kodein Koders team](https://twitter.com/kodeinkoders) for helping us write this article._


