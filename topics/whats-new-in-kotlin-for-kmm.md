[//]: # (title: What's new in Kotlin for KMM)
[//]: # (auxiliary-id: Whats_new_in_Kotlin_for_KMM)

KMM is part of the larger Kotlin ecosystem and leverages Kotlin features and improvements for a better mobile developer experience. 
[Every Kotlin release](https://kotlinlang.org/releases.html#release-details) brings features and improvements that are helpful for mobile developers like you. 

Android Studio will recommend an automatic update to a new Kotlin release. You can also [update manually](https://kotlinlang.org/releases.html#updating-to-a-new-release).

Here you can find a short summary of the features Kotlin provides for developing multiplatform mobile applications.

## Kotlin 1.5 for KMM

[Kotlin 1.5](https://kotlinlang.org/docs/whatsnew15.html) introduces a number of improvements that are helpful for KMM:

* [Choosing a testing dependency for each platform has been simplified](https://kotlinlang.org/docs/whatsnew15.html#simplified-test-dependencies-usage-in-multiplatform-projects)
  and it is now done automatically by the Gradle plugin.
  
* A new [API for getting a char category is now available in multiplatform projects](https://kotlinlang.org/docs/whatsnew15.html#new-api-for-getting-a-char-category-now-available-in-multiplatform-code).

* [Kotlin/Native is now more performant and stable](https://kotlinlang.org/docs/whatsnew15.html#kotlin-native). 
  It is receiving a set of performance improvements that speed up both compilation and execution.

Learn more about [what's new in Kotlin 1.5](https://kotlinlang.org/docs/whatsnew15.html).

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
