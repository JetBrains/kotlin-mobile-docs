[//]: # (title: Samples)
[//]: # (auxiliary-id: Samples)

This is a curated list of Kotlin Multiplatform Mobile (KMM) samples.  

Do you have a great idea for a sample, or one you would like to add to the list?  
Feel free to [reach out to us](mailto:kmm.feedback@kotlinlang.org) and tell us about it!

## KMM sample

This sample project from the Kotlin team demonstrates the Kotlin Multiplatform Mobile (KMM) basic concepts. 
Learn how to configure Gradle to build a simple KMM project with Android and iOS targets, how to share code in a common source set, and how to use expect and actual declarations in cases where code cannot be shared: [https://github.com/Kotlin/kmm-sample](https://github.com/Kotlin/kmm-sample).

## KMM RSS Reader production sample

This project demonstrates how you can use KMM in production. The sample contains a common Kotlin Multiplatform module, an Android project, and an iOS project.  
The production sample has already been published, and you can download it from the [App Store](https://apps.apple.com/ru/app/kmm-rss-reader/id1563922264) or from [Google Play](https://play.google.com/store/apps/details?id=com.github.jetbrains.rssreader.androidApp).

Look through the code to get a better understanding of how to create and prepare your application for publication: [https://github.com/Kotlin/kmm-production-sample](https://github.com/Kotlin/kmm-production-sample).

## KMM sample for an Android application

This sample project from the Kotlin team demonstrates how to make your existing Android application work on iOS. It builds on a simple Android application with a single screen for entering a username and password.

Complete [this step-by-step tutorial](integrate-in-existing-app.md) or check out the finished project on GitHub: [https://github.com/Kotlin/kmm-integration-sample/tree/final](https://github.com/Kotlin/kmm-integration-sample/tree/final).

## CocoaPods integration samples

These samples from the Kotlin team demonstrate how to use CocoaPods in KMM projects.  
* Learn how to add CocoaPods dependencies to a Kotlin project: [https://github.com/Kotlin/kotlin-with-cocoapods-sample](https://github.com/Kotlin/kotlin-with-cocoapods-sample)  
* Learn how to connect a Kotlin framework to an Xcode project using CocoaPods: [https://github.com/Kotlin/multitarget-xcode-with-kotlin-cocoapods-sample](https://github.com/Kotlin/multitarget-xcode-with-kotlin-cocoapods-sample)

## KaMP Kit

This application demonstrating KMM concepts uses some popular KMM libraries and has a simple set of features you can use as a reference.  
This is a good place to start if you’ve learned everything you can from basic KMM examples and want to dig a bit deeper: [https://github.com/touchlab/KaMPKit](https://github.com/touchlab/KaMPKit).

## Moko template

This “not-so-simple“ template covers Multiplatform Gradle DSL configuration, modular architecture, and the use of [moko libraries](https://moko.icerock.dev/). Also, Views are native  in this sample, but ViewModels are multiplatform.  
If you're interested in sharing not only the business logic layer, but also a presentation layer, this project could give you some inspiration: [https://github.com/icerockdev/moko-template](https://github.com/icerockdev/moko-template).

## PeopleInSpace

This minimalistic but powerful sample demonstrates how KMM can be integrated with modern UI frameworks, such as Swift UI and Jetpack Compose. It also accompanies a series of interesting blog posts and even has a nice bonus – watchOS target usage: [https://github.com/joreilly/PeopleInSpace](https://github.com/joreilly/PeopleInSpace).

## GitFox SDK

This library for creating client applications for GitLab servers is a nice example of an open-source production-ready Multiplatform SDK. By looking through its code, you can learn how to work with networks and authorization, how to integrate your Multiplatform SDK with iOS and Flutter apps, and how to bypass some of the most common pitfalls users encounter while working with KMM:
[https://gitlab.com/terrakok/gitlab-client/](https://gitlab.com/terrakok/gitlab-client).  
_Fun fact: this project was created by one of our team members, who was a mobile developer before coming to the KMM team!_
