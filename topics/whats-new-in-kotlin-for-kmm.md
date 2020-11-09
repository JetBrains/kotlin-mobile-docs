[//]: # (title: What's new in Kotlin for KMM)
[//]: # (auxiliary-id: Whats_new_in_Kotlin_for_KMM)

KMM is part of the large Kotlin ecosystem and leverages Kotlin features and improvements for better developer experience. 
Every Kotlin release brings features, improvements, and bug fixes that are important for you, mobile developers. 

Here you can find a short summary of Kotlin features that you can use when developing multiplatform mobile applications.

## Kotlin 1.4.20 for KMM

[Kotlin 1.4.20](https://kotlinlang.org/docs/reference/whatsnew1420.html) brings a number of features, improvements, and bug fixes that are helpful for KMM:

* **CocoaPods plugin improvements**:
    * Rebuilding dependencies only when it's necessary.
    * Extended DSL that supports adding dependencies on: 
        * A library from a custom spec repository.
        * A remote library from a Git repository.
        * A library from an archive (also available by arbitrary HTTP address).
        * A static library.
        * A library with custom cinterop options.
    * Integration with Xcode requires updating the Podfile when adding dependencies on libraries.
     
* Support for **libraries delivered in Xcode 12**.

* **Escape analysis**. A prototype of the new mechanism that improves the runtime performance by 10% via allocating certain objects on the stack instead of the heap. 

* **Opt-in wrapping of Objective-C exceptions** in runtime to avoid crashes.

* **Updated structure of mutliplatform library publications**. The library _root_ publication, which stands for the whole library 
and is automatically resolved to the appropriate platform-specific artifacts when added as a dependency to the common source set, 
now includes metadata artifacts, which were published separately in earlier Kotlin versions.  
For compatibility, both multiplatform library authors and users must update to Kotlin 1.4.20.

* **Performance improvements and bug fixes** in various components, including the ones added in Kotlin 1.4.0.

* **Deprecation of the Kotlin Android Extensions plugin**. The `Pacrelable` implementation generator moves to a separate `kotlin-parcelize` plugin.

Android Studio will suggest you to update to Kotlin 1.4.20 automatically. You can also [update manually](https://kotlinlang.org/releases.html#updating-to-a-new-release).

Learn more about [what's new in Kotlin 1.4.20](https://kotlinlang.org/docs/reference/whatsnew1420.html). 

