[//]: # (title: What's new in Kotlin for KMM)
[//]: # (auxiliary-id: Whats_new_in_Kotlin_for_KMM)

KMM is part of the large Kotlin ecosystem and leverages Kotlin features and improvements for better mobile developer experience. 
[Every Kotlin release](https://kotlinlang.org/releases.html#release-details) brings features and improvements that are helpful for you, mobile developers. 

Here you can find a short summary of Kotlin features that you can use when developing multiplatform mobile applications.

## Kotlin 1.4.20 for KMM

[Kotlin 1.4.20](https://kotlinlang.org/docs/reference/whatsnew1420.html) brings a number of features, improvements, and bug fixes that are helpful for KMM:

* **CocoaPods plugin improvements and changes**:
    * Rebuilding dependencies only when it's necessary.
    * Ability to add dependencies on a library from a custom spec repository, Git repository, or archive as well as on a library with custom cinterop options.
    * Adding dependencies on libraries requires updating the Podfile for correct integration with Xcode.
     
* Support for **libraries delivered in Xcode 12**.

* **Escape analysis**. A prototype of the new mechanism that improves the runtime performance by 10% via allocating certain objects on the stack instead of the heap. 

* **Opt-in wrapping of Objective-C exceptions** in runtime to avoid crashes.

* **Updated structure of mutliplatform library publications**. The library _root_ publication, which stands for the whole library, 
now includes metadata artifacts, which were published separately in earlier Kotlin versions.  
For compatibility, both multiplatform library authors and users must update to Kotlin 1.4.20.

* **Performance improvements and bug fixes** in various components, including the ones added in Kotlin 1.4.0.

* **Deprecation of the Kotlin Android Extensions plugin**. The `Pacrelable` implementation generator moves to a separate `kotlin-parcelize` plugin.

Android Studio will suggest you to update to Kotlin 1.4.20 automatically. You can also [update manually](https://kotlinlang.org/releases.html#updating-to-a-new-release).

Learn more about [what's new in Kotlin 1.4.20](https://kotlinlang.org/docs/reference/whatsnew1420.html). 

