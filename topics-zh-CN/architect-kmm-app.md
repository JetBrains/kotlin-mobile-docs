[//]: # (title: 设计 KMM 应用程序)
[//]: # (auxiliary-id: Architect_your_KMM_application)

本文档提供了架构指南来指导如何决定应用程序的哪些层可以跨平台开发，哪些层可以保持原生开发。

在后端，安卓和 iOS 的业务逻辑相似，因此它是跨平台的绝佳选择。
对于特定于平台的功能，如网络请求、本地数据库访问、硬件操作和加密存储，可以使用已经实现好这些功能的多平台库 [连接到平台特定的 API 接口](connect-to-platform-specific-apis.md)，或者通过使用 `expect` 声明，并在每个平台使用 `actual` 实现相关功能。

KMM 认为使用原生代码开发的 UI 才有最好的用户体验。所以不建议使用 KMM 开发跨平台的 UI 代码。
但是，有了 [兼容的架构模式](#architectural-patterns-for-sharing-ui-behavior)的帮助，UI 行为中定义用户交互逻辑以及前端如何通信的代码也可以复用。

## KMM 应用架构的最佳实践

总之，我们建议在应用程序中使用以下层级进行跨平台开发:

| 层级 | 关于跨平台的建议 |
| ----- | -------------- |
| 业务逻辑 | **是** |
| 平台访问 | **是/否**。 仍需要使用 [连接到平台特定的 API 接口](connect-to-platform-specific-apis.md)，但可以跨平台开发这些行为 |
| 前端行为 （响应输入以及和后端的通信）) | **是/否**。 考虑这些[架构模式](#architectural-patterns-for-sharing-ui-behavior) |
| 用户界面（包括动画和过渡） | **否**。 这部分需要特定于平台开发 |

### 复用 UI 行为的架构模式
{initial-collapse-state="collapsed"}

通过使用 [Model-View-Presenter (_MVP_)](#mvp-for-legacy-ui-frameworks) 或者 [Model-View-Intent (_MVI_)](#mvi-for-declarative-ui-frameworks) 架构模式可以复用 UI 行为代码。这些架构模式：

* 明确区分 UI 和表示层。
* 与 UI 平台完全分离。
* 无需 UI 环境即可轻松测试。

#### 传统 UI 框架的 MVP 架构模式

_Model-View-Presenter_ (_MVP_) 强制为接受输入的 presenter 和显示输出的 view 创建一个 API 接口，使每部分可以独立测试。

下面是一个简单的 MVP presenter 的例子:


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

注意 `commandViwe` 和 `lastCommand` 机制，它允许视图分离和重新连接，例如在 Android 上的配置更改。

#### 声明式 UI 框架的 MVI 架构模式

_Model-View-Intent_ (_MVI_) 是 MVP 架构模式在使用声明式 UI 框架时的自然演变，（尽管它也适用于传统 UI 框架）。因此建议与 Swift UI 或者 Jetpack Compose 配合使用。

在 MVI 模式中，整个 UI 结构被描述在一棵树上。下面是一个简单的 MVI presenter 的例子:

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

注意 `displayModel` 和 `lastModel` 机制，它允许视图分离和重新连接，例如在Android上的配置更改。

### 感谢贡献者

_我们要感谢 [ Kodein Koders 团队](https://twitter.com/kodeinkoders) 帮助我们撰写了这篇文章。_
