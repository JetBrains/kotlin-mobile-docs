[//]: # (title: Make your Android application work on iOS – tutorial)
[//]: # (auxiliary-id: Integrate_KMM_into_an_existing_application)

Here you can learn how to make your existing Android application cross platform so that it works both on Android and iOS. 
You'll be able to write code and test it for Android and iOS only once, in one place.

This tutorial uses a [sample Android application](https://github.com/KaterinaPetrova/kmm-integrate-into-existing-app) with a single screen for entering a username and password. 
The credentials are validated and saved in an in-memory database.

If you aren't familiar with KMM, learn how to [create and configure KMM application from scratch](create-first-app.md).

## Prepare an environment for development

1. [Set up your environment for KMM development](setup.md) by installing the necessary tools on a macOS.

   >* For completing specific steps in this tutorial you need a Mac with macOS. These steps include writing iOS-specific code and running an iOS application.  
   >You won't be able to do this on other operating systems, such as Microsoft Windows – it's an Apple requirement.
   >* For this tutorial, use Android Studio 4.2 or higher.
   >
   {type="note"}

2. In Android Studio, create a new project from the version control: `https://github.com/KaterinaPetrova/kmm-integrate-into-existing-app`.
    
    > You can create a similar application yourself using the **Login Activity** template for a new project with some modifications.
    >
    {type="note"}

3. Switch to the **Project** view.

    ![Project view](project-view-for-integrate.png){width=200}

## Decide what code to make cross platform

Decide which code of your Android application is better to share for iOS and which keep native. 
A simple rule is share what you want to reuse as much as possible. 
The business logic is often the same for both Android and iOS, so it's a great candidate for reusing. 
You can get [more recommendations on cross-platform code](architect-kmm-app.md).

In your sample Android application, the business logic is stored in the package `com.jetbrains.simplelogin.androidapp.data`.
Your future iOS application will use the same logic, so make it cross platform.

![Business logic to share](business-logic-to-share.png){width=350}

## Create a shared module for cross-platform code

The cross-platform code that is used for both iOS and Android _lives_ in the shared module.
KMM provides a special wizard for creating such modules.

In your Android project, create a KMM shared module for your cross-platform code. Later you'll connect it to your existing Android and future iOS applications.

1. In Android Studio, click **New** | **New Module**.

2. In the list of templates, select **KMM Shared Module**, enter the module name `shared` and select the **Generate packForXcode Gradle task** checkbox.  
    This task is required for connecting the shared module to the iOS application.

   ![KMM shared module](kmm-module-wizard-1.png)

3. Click **Finish**.

The wizard creates the KMM shared module, updates configuration files, and creates files with classes that demonstrate multiplatform benefits.
You can learn more about the [KMM project structure](discover-kmm-project.md).
    
## Depend on the shared module in your Android application

To use cross-platform code in your Android application, connect the shared module to it, move the business logic code there, and make this code cross platform.

1. Ensure that `compileSdkVersion` and `minSdkVersion` in `build.gradle.kts` of the `shared` module are the same as `build.gradle` of your Android application in the `app` module.  
   If they are different, update them in `build.gradle.kts` of the shared module. Otherwise, you'll encounter a compile error.
   
2. Add a dependency on the shared module to `build.gradle` of your Android application.
   
    ```kotlin
    dependencies {
        implementation project(':shared')
    }
    ```

3. Check that the shared module is successfully connected to your application and greets you. Dump the `greeting()` function result to the log 
   by updating the method `onCreate()` of the class `LoginActivity`.

    ```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)

       Log.i("Login Activity", "Hello from shared module: " + (Greeting().greeting()))
   
    }
    ```
   
    Search for `Hello` in the log, and you'll find a greeting from the shared module.

    ![Greeting from the shared module](shared-module-greeting.png)

## Make the business logic cross platform

You can now extract the business logic code to the KMM shared module and make it platform independent. This is necessary for reusing it for both Android and iOS.
 
1. Move the business logic code `com.jetbrains.simplelogin.androidapp.data` from the `app` directory to the `com.jetbrains.simplelogin.shared` package in the `shared/commonMain` directory.
   You can drag and drop the package or refactor it by moving from one directory to another.
   
    ![Drag and drop the package with the business logic code](moving-business-logic.png){width=350}

2. When Android Studio asks what you'd like to do, select to move the package, and then approve refactoring.

    ![Refactor the buiness logic package](refactor-business-logic-package.png){width=500}

3. Ignore all warnings about platform-dependent code, and click **Continue**.

    ![Warnings about platform-dependent code](warnings-android-specific-code.png){width=450}

4. Remove Android-specific code by replacing it with cross-platform Kotlin code or connecting to Android-specific APIs using [`expect` and `actual` declarations](connect-to-platform-specific-apis.md).

### Replace Android-specific code with cross-platform code  {initial-collapse-state="collapsed"}

To make your code work well on both Android and iOS, replace all JVM dependencies with Kotlin dependencies wherever possible.

1. Replace `IOException` that is not available in Kotlin with `RuntimeException` in the `login()` function of the `LoginDataSource` class.

<compare> 

```kotlin
return Result.Error(IOException("Error logging in", e))
```

```kotlin
return Result.Error(RuntimeException("Error logging in", e))
```

</compare>

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
