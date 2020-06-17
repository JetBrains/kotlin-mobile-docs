[//]: # (title: Publishing Kotlin Multiplatform Mobile Applications)
[//]: # (auxiliary-id: Publishing_Kotlin_Multiplatform_Mobile_Applications)

The last stage prior to users gaining access to an application is publishing it in stores.
This article explains how to prepare Kotlin Multiplatform Mobile (**KMM**) applications for
publication and what points of this process deserve special attention.

## Publishing Android apps

Since [Kotlin is the main language for Android development](https://developer.android.com/kotlin),
a Kotlin Multiplatform Library with shared code is the go-to Gradle module that is basically no
different from other Android libraries. Therefore, **KMM** has no obvious effect on compiling the
project and building the application. And the publication of Android apps is no different from the
usual process and will be carried out in the usual way, as described in the
[Official Documentation](https://developer.android.com/studio/publish).

## Publishing iOS apps

When working with Kotlin Multiplatform Mobile, the main stages of publishing an iOS app are the same
as described in the [Official Documentation](https://developer.apple.com/ios/submit/).

The main feature that KMM adds is the linking of the shared Kotlin Library to an Xcode project. The
main technical points of integrating the Library with the shared code is solved by the Kotlin
Project Wizard for IDEA and Android Studio. But if a KMM project is implemented without using the
project wizard, bear in mind the following when building and bundling the iOS project in Xcode:

* The Kotlin Multiplatform Library with a shared app code compiles down to the native Framework,
which is separately linked to the iOS project.
* You need to connect the framework compiled for the specific platform to the Xcode project.
* In the Xcode project settings, you must specify the path to the Framework to search for the build
system.
* After building the project, you should launch and test the app to make sure that there are no
issues when working with the Kotlin Library in the Runtime mode.

For more information on connecting the Kotlin Multiplatform Library to the iOS project in Xcode, see
the [Documentation](https://kotlinlang.org/docs/tutorials/native/apple-framework.html#xcode-for-ios-targets).

### Working with crash reporting tools

If the project uses libraries of crash reporting tools that require the `.dSYM` file, then it should
be noted that if errors occur in the Kotlin code in the Multiplatform Library, the statically
compiled Framework will lack tracing for the Kotlin code, which will complicate the error analysis
process. To ensure that the reports contain all the necessary tracing, build a dynamic Framework
from the shared Kotlin module. In this case, a correct `.dSYM` file will be created, which must be
uploaded to the app crash analytics services. Details on creating `.dSYM` for the Kotlin code can be
found in the [Kotlin Native Documentation](https://kotlinlang.org/docs/reference/native/ios_symbolication.html).
