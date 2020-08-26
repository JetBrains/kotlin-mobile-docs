[//]: # (title: KMM evolution)
[//]: # (auxiliary-id: KMM_evolution)

Kotlin ecosystem components have [four stability levels](https://kotlinlang.org/docs/reference/evolution/components-stability.html): Experimental, Alpha, Beta and Stable. 
Current KMM stability level is **Alpha**. 

It means that Kotlin team is committed to KMM, and the product will develop quickly. We’ll listen to your feedback and provide fixes and improvements as soon as possible. Unfortunately, sometimes it means that we won’t be able to guarantee compatibility. However, we’ll try to avoid compatibility issues as much as possible by using feature flags and providing migration guides for new versions.

KMM stability level is mostly defined by stability level of its existing core subcomponents: the least stable one defines the stability of the whole SDK. New KMM features and subcomponents are always considered Experimental in first releases - and don’t affect overall stability level.

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

[More information about Stability of Kotlin Components](https://kotlinlang.org/docs/reference/evolution/components-stability.html)

[Migrating Kotlin Multiplatform Projects to 1.4.0](https://kotlinlang.org/docs/reference/migrating-multiplatform-project-to-14.html)

