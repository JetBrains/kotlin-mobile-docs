[//]: # (title: KMM evolution)
[//]: # (auxiliary-id: KMM_evolution)

Kotlin ecosystem components have [four stability levels](https://kotlinlang.org/docs/reference/evolution/components-stability.html): Experimental, Alpha, Beta, and Stable. 
The current stability level of KMM is **Alpha**. 

This means the Kotlin team is committed to KMM and the product will develop quickly. We will listen to your feedback and we will provide fixes and improvements as soon as possible. 
Unfortunately, sometimes we won’t be able to guarantee compatibility. 
However, we’ll try to avoid compatibility issues as much as possible by using feature flags and providing migration guides for new versions.

KMM’s stability level as an SDK is based on the lowest stability level of any of its existing core subcomponents. 
New KMM features and subcomponents are always considered **Experimental** in their first releases, and these do not affect the overall stability level.

|{width="30%"}**Component**|**Stability level**|
| ---- | --- |
|Kotlin/JVM|Stable|
|Kotlin/Native Runtime|Beta|
|KLib binaries|Alpha|
|expect/actual language feature|Beta|
|Multiplatform Gradle plugin|Beta|
|Kotlin/Native interop with C and Objective C|Beta|
|CocoaPods integration|Beta|
|Multiplatform IDE support|Alpha|
|KMM plugin for Android Studio|Experimental|

Learn more about the [stability of Kotlin components](https://kotlinlang.org/docs/reference/evolution/components-stability.html).

If you already have a multiplatform project, learn how to [migrate it to Kotlin 1.4.0](https://kotlinlang.org/docs/reference/migrating-multiplatform-project-to-14.html).
