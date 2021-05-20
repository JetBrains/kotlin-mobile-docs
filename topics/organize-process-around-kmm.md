[//]: # (title: Organize a process around KMM)
[//]: # (auxiliary-id: Organize_a_process_around_KMM)

Applications that target iOS and Android are usually developed independently for each platform, and synchronization is
done at the level of requirements and the server API. Kotlin Multiplatform allows you to share code between multiple
platforms. This means that native developers have to adjust their workflows to take into account that some parts of the
code are written once and then shared between the platforms. In this article, we will look at different ways to
organize responsibilities with respect to the development of the common code.

We'll mainly discuss Android and iOS mobile development, but some points may be relevant for other platforms as well.

## Multiplatform impact on development process

Project development occurs in the following stages:

1. Task decomposition
2. Planning
3. Development
4. Debugging
5. Review
6. Testing

Let's take a look at how adding a shared Kotlin Multiplatform module affects these different development stages. It's
important to take into account how much of the project’s common code you're planning to share between the platforms.

**Task decomposition:** At this stage, try to determine what functionality you want to move to the shared library
considering that Android and iOS are no longer fully independent.

**Planning:** This step includes assigning tasks to developers. Note that certain tasks may depend on each other. For
example, shared code tasks may create bottlenecks for some of the platform-specific tasks, so make sure to implement the
shared features first.

**Development:** Coordinate all public interface changes in the shared library. Otherwise, native developers may have
difficulty with an inconvenient/inaccessible API.

**Review:** Ask your iOS and Android developers to review each other's code. This allows your team to look at the code
from the perspective of both platforms, which will help reduce potential errors both in the UI and under the hood.

**Testing:** Manual testing remains the same. If QA engineers come across the same problem on both platforms, chances
are the bug is in the shared code.

Keep in mind that Android and iOS developers use different approaches to organizing project architecture. Discuss the
shared module architecture and agree on what exactly you are going to bring into the shared library, as well as what
public API will be used for the shared code.

## Approaches to organizing teamwork for Multiplatform projects

There are three distinct approaches to organizing the way teams work on an Multiplatform library:

1. [The Android team implements the Android application and the Multiplatform module, which iOS developers use as a black box](#only-android-developers-work-on-the-multiplatform-module).
2. [Android and iOS developers work on the Multiplatform library simultaneously](#android-and-ios-developers-work-on-the-multiplatform-module).
3. [A dedicated team is allocated to work on the Multiplatform library, and both Android and iOS developers use it as a black box](#the-multiplatform-module-is-developed-by-a-dedicated-team).

To choose the right strategy for your project, consider how much of the planned functionality you're going to place in
the shared module:
* If you want to create a module with little or purely utilitarian functionality, the first option will work just fine
for you. It will be less costly for iOS developers to adopt Kotlin, and you won’t have to spend time re-building the
framework for iOS.
* If you implement the business logic of your application in the shared module, engage iOS developers at least at the
design stage in order to build the right public interface. But it’s still better to attract developers of all target
platforms so that they can all understand how the application works.
* If you are planning to implement a significant number of features as shared code in a large project, you can build a
team of developers willing to develop both for iOS and Android or develop applications in separate teams using the Multiplatform
library.

### Only Android developers work on the Multiplatform module

When the shared module is developed by a team of Android developers experienced with Kotlin and Gradle, several things
should be taken into account:

* Kotlin Multiplatform compiles the shared code to different targets in different ways. Bear in mind that the same code
will be executed on multiple platforms and operating systems. 
* Android developers will have to become familiar with Kotlin/Native, which is used for iOS. For example, they will need
to know about the compatibility limitations between Kotlin and Swift/Objective-C ([find more details on compatibility here](https://kotlinlang.org/docs/native-objc-interop.html)). Or if concurrency is used in the shared code, they will have to deal with the Kotlin/Native Memory
Model, which [differs significantly](concurrency-overview.md) from the Java Memory Model.
* Android developers may not be able to independently write all the expect/actual code in the shared module as they
could lack iOS development experience. That's why you may need to involve iOS developers, at least as advisors, or use
abstract interfaces in the module leaving the implementation to library users.
* To compile the Multiplatform library for iOS, you need to have macOS. The lack of direct access can be overcome by using a
build automation system that supports continuous integration.

### Android and iOS developers work on the Multiplatform module

When both the Android and the iOS teams work on the Multiplatform module together, your team may encounter the following problems:

* The iOS developers may have no experience using Kotlin or Kotlin Multiplatform. In this case, they would have to
receive training or learn the language on their own. 
* The iOS developers may have no experience with Gradle, which is used for building Multiplatform projects. That's why either
they would have to master Gradle, or you would have to hand project configuration tasks over to Android developers. 
* As Android and iOS developers approach their work differently, the project participants will need to communicate a lot
in order to create a correct and convenient library API for both platforms. 
* With this team structure, it's particularly important to consider that Android and iOS developers approach their work
differently. Therefore, communication is vital when working together on the shared code. Make sure to provide your team
with opportunities to communicate online and hold face-to-face meetings.

### The Multiplatform module is developed by a dedicated team

If you choose to allocate a dedicated team of Android and iOS developers to work on the shared code, consider the
following:

* Longer release cycles for the shared library and the need to support multiple versions for different projects may hold up
application development.
* Different versions of Multiplatform libraries with different Kotlin versions will not work in the same project, since not all KMM components have reached [strict binary compatibility rules](https://kotlinlang.org/docs/kotlin-evolution.html#evolving-the-binary-format)
* As native development teams work independently, it's particularly important to create and maintain detailed
documentation for the shared modules in order to simplify the integration of the shared code and reduce the number of
questions frequently posed to the library authors.

_We'd like to thank the [IceRock team](https://icerockdev.com/) for helping us write this article._
