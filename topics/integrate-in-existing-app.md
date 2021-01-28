[//]: # (title: Make your Android application multiplatform – tutorial)
[//]: # (auxiliary-id: Integrate_KMM_into_an_existing_application)

Here you can learn how to make your existing Android application multiplatform so that it works on both Android and iOS.

Sharing code saves you time and effort on writing code and testing it for Android and iOS as you only need to do it once, in the one place.

Before you begin, learn how to [create and configure a KMM application](create-first-app.md).

### Create a KMM shared module

In your Android project, create a KMM shared module for your shared code.

> The KMM Shared Module Wizard is available in Android Studio version 4.2 or higher.
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
