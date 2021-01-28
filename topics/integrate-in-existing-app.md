[//]: # (title: Make your Android application multiplatform – tutorial)
[//]: # (auxiliary-id: Integrate_KMM_into_an_existing_application)

Here you can learn how to make your existing Android application multiplatform so that it works on both Android and iOS. 
You'll be able to write code and test it for Android and iOS only once, in the one place.

This tutorial uses a [sample Android application](https://github.com/KaterinaPetrova/kmm-integrate-into-existing-app) with a single screen for entering login and password. 
The credentials are validated and saved in the in-memory storage.

> You can create a similar application yourself using the **Login Activity** template for a new project with some modifications.
>
{type="note"}

If you aren't familiar with KMM, learn how to [create and configure KMM application from scratch](create-first-app.md).

### Decide what to make multiplatform

First decide which code of your Android application is better to share for iOS and which keep native. 
A simple rule is share what you want to reuse as much as possible. 
The business logic is often the same for both Android and iOS, so it's a great candidate for reusing. 
Get [more recommendations on sharing your code](architect-kmm-app.md).

In your sample Android application, the business logic is stored in the package `com.jetbrains.simplelogin.androidapp.data`.
Your future iOS application will use the same logic, so make it multiplatform.

### Create a shared module for multiplatform code

The multiplatform code that is used for both iOS and Android should _live_ in the shared module.
KMM provides a special wizard for creating such modules.

In your Android project, create a KMM shared module for your multiplatform code. Later you'll connect it to your existing Android and future iOS applications.

1. [Set up your environment for KMM development](setup.md) by installing the necessary tools on a macOS.
    
    >Since in this tutorial you'll write iOS-specific code and run an iOS application on a simulated device, use a Mac with a macOS.  
    >You won't be able to run the iOS application on other operating systems, such as Microsoft Windows. It's an Apple requirement.
    >
    {type="note"}

2. In Android Studio, click **New** | **New Module**. 

    >For this tutorial, use Android Studio 4.2 or higher.
    >
    {type="note"}

3. In the list of module types, select **KMM Shared Module** and then click **Next**.

    ![KMM shared module](kmm-module-wizard-1.png) 

4. Select the **Add sample tests** checkbox.

    ![KMM shared module configuration](kmm-module-wizard-2.png) 
   
5. Click **Finish**.

The wizard creates the KMM shared module, updates configuration files, and creates files with classes that demonstrate multiplatform benefits.
You can learn more about the [KMM project structure](discover-kmm-project.md).
    
### Use the shared module for your Android application

To use multiplatform code in your Android application, connect the shared module to it, move the business logic code there, and make this code multiplatform.

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


### Make the business logic multiplatform




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
