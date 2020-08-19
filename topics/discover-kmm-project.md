[//]: # (title: Discover KMM project)
[//]: # (auxiliary-id: Discover_KMM_project)

The purpose of the Kotlin Multiplatform Mobile (_KMM_) technology is unifying the development of applications with common 
logic for Android and iOS platforms. To make this possible, KMM uses a mobile-specific structure of
[Kotlin multiplatform](https://kotlinlang.org/docs/reference/multiplatform.html) projects.
On this page, we’ll describe the structure of a basic KMM project. Note that this structure isn’t the only
possible way to organize a KMM project; however, we recommend it as a starting point.

A basic Kotlin Mobile Multiplatform (KMM) project consists of three components:

* _Shared module_ - a Kotlin module that contains common logic for both Android and iOS applications.
Builds into an Android library and an iOS framework. Uses Gradle as a build system.
* _Android application_ - a Kotlin module that builds into the Android application.
Uses Gradle as a build system.
* _iOS application_ - an Xcode project that builds into the iOS application.

<img src="basic-project-structure.png" alt="Basic KMM project structure" width="700"/>

This is the structure of a KMM project that you create with a Project Wizard in IntelliJ IDEA or Android Studio.
Real-life projects can have more complex structure; we consider these three components essential for a KMM project.

Let’s take a closer look at the basic project and its components.

## Root project

The root project is a Gradle project that holds the shared module and the Android application as its subprojects.
They are linked together via the [Gradle multi-project mechanism](https://docs.gradle.org/current/userguide/multi_project_builds.html). 

<tabs>
<tab title="Groovy">
    
```Groovy
// gradle.settings
include ':shared'
include ':androidApp'
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
// gradle.settings.kts
include(":shared")
include(":androidApp")
 ```
         
</tab>
</tabs>

The iOS application is produced from an Xcode project. It’s stored in a separate directory within the root project.
Xcode uses its own build system; thus, the iOS application project isn’t connected with other parts of the KMM project
via Gradle. Instead, it uses the shared module as an external artifact - framework. For details on integration between
the shared module and the iOS application, see [iOS application](#ios-application).

This is a basic structure of a KMM project:

<img src="basic-project-dirs.png" alt="Basic KMM project directories" width="500"/>

The root project doesn’t hold source code. You can use it to store global configuration in its `build.gradle(.kts)` or 
`gradle.properties`, for example, add repositories or define global configuration variables.

For more complex projects, you can add more modules into the root project by creating them in the IDE and linking via
`include` declarations in the Gradle settings.

## Shared module

Shared module contains the core application logic used in both target platforms: classes, functions, and so on.
This is a [Kotlin multiplatform](https://kotlinlang.org/docs/reference/multiplatform.html) module that that compiles
into an Android library and an iOS framework. It uses Gradle with the Kotlin multiplatform plugin applied and 
has targets for Android and iOS.

<tabs>
<tab title="Groovy">
    
```Groovy
plugins {
    id 'org.jetbrains.kotlin.multiplatform' version '<version-placeholder>'
    //..
}

kotlin {
    android()
    iosX64()
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
plugins {
    kotlin("multiplatform") version "<version-placeholder>"
    // ..
}

kotlin {
    android()
    iosX64()
}
 ```
         
</tab>
</tabs>

### Source sets

The shared module contains the code that is common for Android and iOS applications. However, to implement the same logic
 on Android and iOS, you sometimes need to write two platform-specific versions of it. 
 To handle such cases, Kotlin offers the [expect/actual](connect-to-platform-specific-apis.md) mechanism.
 The source code of the shared module is organized in three source sets accordingly:

* `commonMain` stores the code that works on both platforms, including the `expect` declarations
* `androidMain` stores Android-specific parts, including `actual` implementations
* `iosMain` stores iOS-specific parts, including `actual` implementations

Each source set has its own dependencies. By default, these are dependencies to the Kotlin standard library and
platform core libraries. Note that the iOS source set has an implicit dependency on the standard library:
you don’t need to declare it in the build script.

<tabs>
<tab title="Groovy">
    
```Groovy
kotlin {
    sourceSets {
        commonMain {
            dependencies {
                implementation kotlin('stdlib-common')
            }
        }
        androidMain {
            dependencies {
                implementation kotlin('stdlib-jdk7')
                implementation 'androidx.core:core-ktx:1.2.0'
            }
        }
        iosMain {
             //stdlib is added by default for native platforms
        }

        // ...
    }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(kotlin("stdlib-common"))
            }
        }
        val androidMain by getting {
            dependencies {
                implementation(kotlin("stdlib-jdk7"))
                implementation("androidx.core:core-ktx:1.2.0")
            }
        }
        val iosMain by getting
        //stdlib is added by default for native platforms
 
        // ...
    }
}
 ```
         
</tab>
</tabs>

When you write your code, add the dependencies you need to the corresponding source sets.
Read [Building Multiplatform Projects wih Gradle](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#adding-dependencies) for more information.

Along with `*Main` source sets, there are three matching test source sets:

* `commonTest`
* `androidTest`
* `iosTest`

Use them to store unit tests for common and platform-specific source sets accordingly.
By default, they have dependencies on Kotlin test library, providing you with means for Kotlin unit testing:
annotations, assertion functions and other. You can add dependencies on other test libraries you need.

<tabs>
<tab title="Groovy">
    
```Groovy
kotlin {
    sourceSets {
        //...

        commonTest {
            dependencies {
                implementation kotlin('test-common')
                implementation kotlin('test-annotations-common')
            }
        }
        androidTest {

        }
        iosTest {

        }
    }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
kotlin {
    sourceSets {
        // ...
        val commonTest by getting {
            dependencies {
                implementation(kotlin("test-common"))
                implementation(kotlin("test-annotations-common"))
            }
        }
        val androidTest by getting
        val iosTest by getting
    }

}
 ```
         
</tab>
</tabs>

The main and test source sets described above are default. The Kotlin multiplatform plugin generates them
automatically upon target creation. In your project, you can add more source sets for specific purposes.
For more information, see [Building Multiplatform Projects wih Gradle](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#configuring-source-sets).

### Android library

The configuration of the Android library produces from the shared module is typical for Android projects.
To learn about Android libraries creation, see [Create an Android library](https://developer.android.com/studio/projects/android-library)
in the Android developer documentation.

To produce the Android library, two more Gradle plugins are used in addition to Kotlin multiplatform:

* Android library
* Kotlin Android extensions

<tabs>
<tab title="Groovy">
    
```Groovy
plugins {
    // ...
    id 'com.android.library'
    id 'kotlin-android-extensions'
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
plugins {
    // ...
    id("com.android.library")
    id("kotlin-android-extensions")
}
 ```
         
</tab>
</tabs>

The configuration of Android library is stored in the `android {}` top-level block of the shared module’s build script.

<tabs>
<tab title="Groovy">
    
```Groovy
android {
    compileSdkVersion 29
    defaultConfig {
        minSdkVersion 24
        targetSdkVersion 29
        versionCode 1
        versionName '1.0'
    }
    buildTypes {
        release {
            minifyEnabled false
        }
    }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
android {
    compileSdkVersion(29)
    defaultConfig {
        minSdkVersion(24)
        targetSdkVersion(29)
        versionCode = 1
        versionName = "1.0"
    }
    buildTypes {
        getByName("release") {
            isMinifyEnabled = false
        }
    }
}
 ```
         
</tab>
</tabs>

It’s typical for any Android project. You can edit it to suit your needs.
To learn more, see the [Android developer documentation](https://developer.android.com/studio/build#module-level).

### iOS framework

For using in iOS applications, the shared module compiles into a framework - a kind of hierarchical directory
with shared resources used on the Apple platforms.
This framework connects to the Xcode project that builds into an iOS application.

The framework is produced via the [Kotlin/Native](https://kotlinlang.org/docs/reference/native-overview.html) compiler.
The framework configuration is stored in the `ios {}` block of the build script within `kotlin {}`.
It defines the output type `framework` and the string identifier `baseName` that is used to form the name
of the output artifact. Its default value matches the Gradle module name. 
For a real project, it’s likely that you’ll need a more complex configuration of the framework production.
For details, see [Building Multiplatform Projects wih Gradle](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#building-final-native-binaries).

<tabs>
<tab title="Groovy">
    
```Groovy
kotlin {
    // ...
    iosX64() {
        binaries {
            framework {
                baseName = 'shared'
            }
        }
    }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
kotlin {
    // ...
    iosX64() {
        binaries {
            framework {
                baseName = "shared"
            }
        }
    }
}
 ```
         
</tab>
</tabs>

Additionally, there is a Gradle task that exposes the framework to the Xcode project from which the iOS application is built.
It uses the configuration of the iOS application project to define the build mode (`debug` or `release`) and provide
the appropriate framework version to the specified location.

<tabs>
<tab title="Groovy">
    
```Groovy
task(packForXcode, type: Sync) {
    group = 'build'
    def mode = System.getenv('CONFIGURATION') ?: 'DEBUG'
    def framework = kotlin.targets.ios.binaries.getFramework(mode)
    inputs.property('mode', mode)
    dependsOn(framework.linkTask)
    def targetDir = new File(buildDir, 'xcode-frameworks')
    from({ framework.outputDirectory })
    into(targetDir)
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
val packForXcode by tasks.creating(Sync::class) {
    group = "build"
    val mode = System.getenv("CONFIGURATION") ?: "DEBUG"
    val framework = kotlin.targets.getByName<KotlinNativeTarget>("ios").binaries.getFramework(mode)
    inputs.property("mode", mode)
    dependsOn(framework.linkTask)
    val targetDir = File(buildDir, "xcode-frameworks")
    from({ framework.outputDirectory })
    into(targetDir)
}
 ```
         
</tab>
</tabs>

The task executes upon each build of the Xcode project to provide the latest version of the framework for the iOS application.
For details, see [iOS application](#ios-application).

<tabs>
<tab title="Groovy">
    
```Groovy
tasks.getByName('build').dependsOn(packForXcode)
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
tasks.getByName("build").dependsOn(packForXcode)
 ```
         
</tab>
</tabs>

## Android application

The Android application part of a KMM project is a typical Android application written in Kotlin. In a basic KMM project, it uses three Gradle plugins: 

* Kotlin Android
* Android Application
* Kotlin Android Extensions

<tabs>
<tab title="Groovy">
    
```Groovy
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android' version '<version_placeholder>'
    id 'kotlin-android-extensions'
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
plugins {
    id("com.android.application")
    kotlin("android") version "<version_placeholder>"
    id("kotlin-android-extensions")
} 
```
         
</tab>
</tabs>

To access the shared module code, the Android application uses it as a project dependency.

<tabs>
<tab title="Groovy">
    
```Groovy
dependencies {
    implementation project(':shared')
    //..
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
dependencies {
    implementation(project(":shared"))
    //..
} 
```
         
</tab>
</tabs>

Except this dependency, the Android application uses the Kotlin standard library and some common Android dependencies:

<tabs>
<tab title="Groovy">
    
```Groovy
dependencies {
    //..
    implementation 'androidx.core:core-ktx:1.2.0'
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation 'org.jetbrains.kotlin:kotlin-stdlib-jdk7'
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
dependencies {
    //..
    implementation("androidx.core:core-ktx:1.2.0")
    implementation("androidx.appcompat:appcompat:1.1.0")
    implementation("androidx.constraintlayout:constraintlayout:1.1.3")
    implementation(kotlin("stdlib-jdk7"))
} 
```
         
</tab>
</tabs>

Add your project’s Android-specific dependencies to this block.
The build configuration of the Android application is located in the `android {}` top-level block of the build script:

<tabs>
<tab title="Groovy">
    
```Groovy
android {
    compileSdkVersion 29
    defaultConfig {
        applicationId 'org.example.androidApp'
        minSdkVersion 24
        targetSdkVersion 29
        versionCode 1
        versionName '1.0'
    }
    buildTypes {
        'release' {
            minifyEnabled false
        }
    }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
android {
    compileSdkVersion(29)
    defaultConfig {
        applicationId = "org.example.androidApp"
        minSdkVersion(24)
        targetSdkVersion(29)
        versionCode = 1
        versionName = "1.0"
    }
    buildTypes {
        getByName("release") {
            isMinifyEnabled = false
        }
    }
}
```
         
</tab>
</tabs>

It’s typical for any Android project. You can edit it to suit your needs.
To learn more, see the [Android developer documentation](https://developer.android.com/studio/build#module-level).

## iOS application

The iOS application is produced from an Xcode project generated automatically by the Project Wizard in the IntelliJ IDEA.
It resides in a separate directory within the root KMM project. This is a basic Xcode project configured to use the
framework produced from the shared module.

<img src="basic-xcode-project.png" alt="Basic KMM Xcode project" width="400"/>

To make the declarations from the shared module available in the source code of the iOS application,
the framework is added to the project on the **General** tab of the project settings.

<img src="framework-in-project-settings.png" alt="Framework in the Xcode project settings" width="800"/>

For each build of the iOS application, the project obtains the latest version of the framework.
To do this, it uses a **Run Script** build phase that executes the [packForXcode](#ios-framework)
Gradle task from the shared module.

<img src="packforxcode-in-project-settings.png" alt="Execution of packForXcode in the Xcode project settings" width="800"/>

Finally, another build phase **Embed Framework** takes the framework from the specified location and
embeds it into the application build. This makes the shared code available in the iOS application.

<img src="embed-framework-in-xcode-project-settings.png" alt="Embedding the framework in the Xcode project settings" width="800"/>

In other aspects, the Xcode part of a KMM project is a typical iOS application project.
To learn more about creating iOS application, see the [Xcode documentation](https://developer.apple.com/documentation/xcode#topics).