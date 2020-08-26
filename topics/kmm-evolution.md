[//]: # (title: KMM evolution)
[//]: # (auxiliary-id: KMM_evolution)

Kotlin ecosystem components have [four stability levels](https://kotlinlang.org/docs/reference/evolution/components-stability.html): Experimental, Alpha, Beta and Stable. 
Current KMM stability level is **Alpha**. 

It means that Kotlin team is committed to KMM, but the product has not reached the final shape. 
We're eager to listen to your feedback and provide fixes and improvements as soon as possible - but unfortunately it means that we can't provide strict compatibility guarantees yet (because they would hinder deployment of new features and bugfixes). That being said, we still try to avoid compatibility issues as much as possible, use feature flags where necessary and provide migration guides for new versions.

KMM stability level is mostly defined by stability level of its core subcomponents: the least stable one defines the stability of the whole SDK. 

|{width="30%"}**Component**|**Stability level**|
| ---- | --- |
|Kotlin/JVM|Stable|
|Kotlin/Native Runtime|Beta|
|KLib binaries|Alpha|
|expect/actual language feature|Beta|
|Multiplatform Gradle plugin|Beta|
|Kotlin/Native interop with C and Objective C|Beta|
|CocoaPods integration|Beta|
|Multiplatform IDE support|Alpha<sup>(*)</sup>|

<sup>(*)</sup> [KMM plugin for Android Studio](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform-mobile) will be considered Experimental in first releases, but its functionality will always be supported in at least one IDE our adopters are used to. That’s why generally IDE support is considered Alpha.

[More information about Stability of Kotlin Components](https://kotlinlang.org/docs/reference/evolution/components-stability.html)

[Migrating Kotlin Multiplatform Projects to 1.4.0](https://kotlinlang.org/docs/reference/migrating-multiplatform-project-to-14.html)

