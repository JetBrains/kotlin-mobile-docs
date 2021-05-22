[//]: # (title: Set up an environment for KMM development)
[//]: # (auxiliary-id: Set_up_an_environment_for_KMM_development)

在你开始[创建第一个在 iOS 和 Android 上均可运行的应用](create-first-app.md)前，先开始搭建 Kotlin 移动多平台（KMM）的开发环境：

1. 如果你打算编写共享代码或 Android 端代码，你可以在任何具有 [Android Studio](https://developer.android.com/studio) 支持的操作系统的电脑上工作。
   如果你同时还想写 iOS 端代码，并且在模拟器或者真机上运行 iOS 应用，请使用运行 macOS 的 Mac。基于苹果的要求，这些步骤（iOS）不能在其他操作系统上执行，例如微软的 Windows。

2. 安装 [Android Studio](https://developer.android.com/studio) – 4.2 及以上版本。
   你可以使用 Android Studio 来创建多平台应用以及在模拟器或真机上运行它们。

3. 如果你需要编写 iOS 端代码并运行应用，你需要安装 [Xcode](https://apps.apple.com/us/app/xcode/id497799835)  –  11.3 及以上版本。
   多数时候 Xcode 只会在后台工作，你会使用它来添加 Swift 或 Objective-C 代码到你的 iOS 应用。


4. 确保你安装了一个兼容的 [Kotlin 插件](kmm-plugin-releases.md#release-details)。
    在 Android Studio， 选择  **Tools** | **Kotlin** | **Configure Kotlin Plugin Updates** 并检查当前 Kotlin 插件版本。如果需要，请在 **Stable** 渠道升级到最新的版本。
    
5. 安装 *Kotlin Multiplatform Mobile* 插件。
    在 Android Studio，选择 **Preferences** | **Plugins**，在 **Marketplace** 搜索插件 *Kotlin Multiplatform Mobile* 并安装。
    
    ![Kotlin 移动多平台（KMM）插件](mobile-multiplatform-plugin.png){width=500}
    
    查看 [Kotlin 移动多平台（KMM）插件发布日志](kmm-plugin-releases.md)。
    
6. 安装 [JDK](https://www.oracle.com/java/technologies/javase-downloads.html) ，如果你之前未安装。
    想检查它是否安装, 可在终端运行命令 `java -version`。
     
现在是时候[创建你的第一个 Kotlin 移动多平台（KMM）应用](create-first-app.md) 了。
