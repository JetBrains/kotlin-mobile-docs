[//]: # (title: What's new in Kotlin for KMM)
[//]: # (auxiliary-id: Whats_new_in_Kotlin_for_KMM)

KMM is part of the larger Kotlin ecosystem and leverages Kotlin features and improvements for a better mobile developer experience. 
[Every Kotlin release](https://kotlinlang.org/releases.html#release-details) brings features and improvements that are helpful for mobile developers like you. 

Android Studio will recommend an automatic update to a new Kotlin release. You can also [update manually](https://kotlinlang.org/releases.html#updating-to-a-new-release).

Here you can find a short summary of the features Kotlin provides for developing multiplatform mobile applications.

## Kotlin 1.5.20 for KMM

[Kotlin 1.5.20](https://kotlinlang.org/docs/whatsnew1520.html) introduces a number of improvements and features that are helpful for KMM:

* **Export of KDoc comments to generated Objective-C headers.**
  You can now set the Kotlin/Native compiler to export the [documentation comments (KDoc)](https://kotlinlang.org/docs/kotlin-doc.html) from Kotlin code
  to the Objective-C frameworks generated from it, making them visible to the frameworks’ consumers.

  This feature is experimental. We would appreciate your feedback on it in [YouTrack](https://youtrack.jetbrains.com/issue/KT-38600).

  Learn more about [exporting KDoc comments to generated Objective-C headers and how to opt in to this feature](https://kotlinlang.org/docs/whatsnew1520.html#opt-in-export-of-kdoc-comments-to-generated-objective-c-headers).

* **New framework-packing task for Kotlin/Native**.
  The [Kotlin Multiplatform Gradle plugin](https://kotlinlang.org/docs/mpp-dsl-reference.html) now includes the `embedAndSignAppleFrameworkForXcode` task, which can be used from Xcode to connect KMM modules to the iOS part of your project.

  Check out this [blog post](https://blog.jetbrains.com/kotlin/2021/07/multiplatform-gradle-plugin-improved-for-connecting-kmm-modules/) to learn about the new framework-packing task and how to remove from the `packForXcode` task from your build script.

Learn more about [what's new in Kotlin 1.5.20](https://kotlinlang.org/docs/whatsnew1520.html).

## Kotlin 1.5.0 for KMM

[Kotlin 1.5.0](https://kotlinlang.org/docs/whatsnew15.html) introduces a number of improvements and features that are helpful for KMM:

* **Simplified test dependency selection for each platform.**
  Now you can use the `kotlin-test` dependency to add dependencies for testing in the `commonTest` source set. The
  Gradle plugin will infer the corresponding platform dependencies for each test source set:
  * `kotlin-test-junit` for JVM source sets.
  * `kotlin-test-common` and `kotlin-test-annotations-common` for common source sets.

  iOS source sets use Kotlin/Native, which has everything built in, so they do not require any additional artifacts.

  You can also use the `kotlin-test` dependency in any shared or platform-specific source set.
  Learn more about [setting dependencies on test libraries](https://kotlinlang.org/docs/gradle.html#set-dependencies-on-test-libraries).

* **New API for getting a char’s Unicode category.** A variety of new character-related functions are available on all platforms and in the common code. They include several functions for checking whether a char is a letter or a digit, like `Char.isLetterOrDigit()`, as well as
  functions for checking the case of a char, like  `Char.isUpperCase()`. The property `Char.category` and the enum class `CharCategory` are available, as well.  
  Learn more about this [new API](https://kotlinlang.org/docs/whatsnew15.html#new-api-for-getting-a-char-category-now-available-in-multiplatform-code).

* **Improved Kotlin/Native performance and stability**. Kotlin/Native is receiving a set of performance improvements that speed up
  both compilation and execution.  
  Learn more about the [Kotlin/Native improvements](https://kotlinlang.org/docs/whatsnew15.html#kotlin-native).

Learn more about [what's new in Kotlin 1.5.0](https://kotlinlang.org/docs/whatsnew15.html).

## Kotlin 1.4.30 for KMM

[Kotlin 1.4.30](https://kotlinlang.org/docs/whatsnew1430.html) introduces a number of improvements that are helpful for KMM:

* **Improved compilation time for an iOS simulator**. Recompiling binaries for the iOS simulator after making changes in the code now requires much less time.
  You can see the most significant improvements when re-running unit tests or applications on the iOS simulator.
  For example, the time required to rebuild the framework in the [KMM Networking and data storage sample](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/tree/final) has decreased from 9.5 seconds (in 1.4.10) to 4.5 seconds (in 1.4.30).  
  These optimizations affect other scenarios as well.

* Support for **libraries delivered in Xcode 12.2**.

* Support for the new **watchosX64** target in Kotlin/Native. This target makes it possible to run the simulator on 64-bit architecture.

Learn more about [what's new in Kotlin 1.4.30](https://kotlinlang.org/docs/whatsnew1430.html).

## Kotlin 1.4.20 for KMM

[Kotlin 1.4.20](https://kotlinlang.org/docs/whatsnew1420.html) introduces a number of features, improvements, and bug fixes that are helpful for KMM:

* **CocoaPods plugin improvements**:
    * Rebuilding dependencies only when necessary.
    * Ability to add dependencies on libraries from a custom spec repository, Git repository, or archive, as well as on libraries with custom cinterop options.  
      Learn more about [adding CocoaPods dependencies](add-dependencies.md#with-cocoapods) and [these improvements](https://kotlinlang.org/docs/whatsnew1420.html#cocoapods-plugin-improvements).
     
* Support for **libraries delivered in Xcode 12**.

* **Escape analysis for Kotlin/Native**. A prototype of a new mechanism that gives a 10% iOS runtime performance improvement by allocating certain objects on the stack instead of the heap. 

* **Opt-in wrapping of Objective-C exceptions** in runtime to avoid crashes. Learn [how to opt in](https://kotlinlang.org/docs/whatsnew1420.html#opt-in-wrapping-of-objective-c-exceptions).

* **Updated structure of multiplatform library publications**. The library _root_ publication, which stands for the whole library, 
now includes metadata artifacts. These were published separately in earlier Kotlin versions.  
For compatibility, both multiplatform library authors and users must update to Kotlin 1.4.20. Learn more about [publishing a multiplatform library](https://kotlinlang.org/docs/mpp-publish-lib.html).

* **Deprecation of the Kotlin Android Extensions plugin**. The [`Parcelable` implementation generator](https://kotlinlang.org/docs/all-open-plugin.html#parcelable-implementations-generator) has been moved to a separate `kotlin-parcelize` plugin.

Learn more about [what's new in Kotlin 1.4.20](https://kotlinlang.org/docs/whatsnew1420.html).
