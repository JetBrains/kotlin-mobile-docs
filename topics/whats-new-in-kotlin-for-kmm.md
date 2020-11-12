[//]: # (title: What's new in Kotlin for KMM)
[//]: # (auxiliary-id: Whats_new_in_Kotlin_for_KMM)

KMM is part of the larger Kotlin ecosystem and leverages Kotlin features and improvements for a better mobile developer experience. 
[Every Kotlin release](https://kotlinlang.org/releases.html#release-details) brings features and improvements that are helpful for mobile developers like you. 

Here you can find a short summary of the features Kotlin provides for developing multiplatform mobile applications.

## Kotlin 1.4.20 for KMM

[Kotlin 1.4.20](https://kotlinlang.org/docs/reference/whatsnew1420.html) introduces a number of features, improvements, and bug fixes that are helpful for KMM:

* **CocoaPods plugin improvements**:
    * Rebuilding dependencies only when necessary.
    * Ability to add dependencies on libraries from a custom spec repository, Git repository, or archive, as well as on libraries with custom cinterop options.  
      Learn more about [adding CocoaPods dependencies](add-dependencies.md#with-cocoapods) and [these improvements](https://kotlinlang.org/docs/reference/whatsnew1420.html#cocoapods-plugin-improvements).
     
* Support for **libraries delivered in Xcode 12**.

* **Escape analysis for Kotlin/Native**. A prototype of a new mechanism that gives a 10% iOS runtime performance improvement by allocating certain objects on the stack instead of the heap. 

* **Opt-in wrapping of Objective-C exceptions** in runtime to avoid crashes. Learn [how to opt in](https://kotlinlang.org/docs/reference/whatsnew1420.html#opt-in-wrapping-of-objective-c-exceptions).

* **Updated structure of multiplatform library publications**. The library _root_ publication, which stands for the whole library, 
now includes metadata artifacts. These were published separately in earlier Kotlin versions.  
For compatibility, both multiplatform library authors and users must update to Kotlin 1.4.20. Learn more about [publishing a multiplatform library](https://kotlinlang.org/docs/reference/mpp-publish-lib.html).

* **Deprecation of the Kotlin Android Extensions plugin**. The [`Parcelable` implementation generator](https://kotlinlang.org/docs/reference/compiler-plugins.html#parcelable-implementations-generator) has been moved to a separate `kotlin-parcelize` plugin.

Android Studio will recommend an automatic update to Kotlin 1.4.20. You can also [update manually](https://kotlinlang.org/releases.html#updating-to-a-new-release).

Learn more about [what's new in Kotlin 1.4.20](https://kotlinlang.org/docs/reference/whatsnew1420.html).
