[//]: # (title: Add dependencies to Kotlin Multiplatform modules)
[//]: # (auxiliary-id: Add_dependencies_to_Kotlin_Multiplatform_modules)

## iOS dependencies

Apple SDK dependencies (such as Foundation or Core Bluetooth) are available as a set of prebuilt libraries in a Kotlin Multiplatform Mobile project. They do not require any additional configuration.

You can also reuse other libraries and frameworks from the iOS ecosystem in your iOS source sets. Kotlin supports interoperability with Objective-C dependencies and Swift dependencies if their API is exported to Objective-C with the `@objc` attribute. Pure Swift dependencies are not yet supported.

Integration with the CocoaPods dependency manager is also supported with the same limitation – you cannot use pure Swift pods. Using CocoaPods is the recommended way to handle iOS dependencies in Kotlin Multiplatform Mobile (KMM) projects. Use manual dependency management only if you want some specific tuning of the interop process or have other strong reasons to opt for it.

### With CocoaPods

1. Perform [initial CocoaPods integration setup](https://kotlinlang.org/docs/reference/native/cocoapods.html#install-the-cocoapods-dependency-manager-and-plugin)

2. Add dependency on a Pod library that you want to use from the CocoaPods repository with `pod()` to build script of your project.

    <tabs>
    <tab title="Groovy">
    
    ```Groovy
    kotlin {
       cocoapods {
           //..
           pod("AFNetworking", "~> 4.0.0")
       }
    }

    ```
    </tab>
    <tab title="Kotlin">
    
    ```Kotlin
    kotlin {
       cocoapods {
           //..
           pod("AFNetworking", "~> 4.0.0")
       }
    }

    ```
         
    </tab>
    </tabs>

3. Re-import the project.

To use the dependency in Kotlin code, import the package `cocoapods.<library-name>`. In the example above, that would be:
```kotlin
import cocoapods.AFNetworking.*
```

[More information about CocoaPods integration](https://kotlinlang.org/docs/reference/native/cocoapods.html).

### Without CocoaPods

If you don’t want to use CocoaPods, you can use the cinterop tool to create Kotlin bindings for Objective-C or Swift declarations. With such declarations, you will be able to call them from Kotlin code. You’ll need to:
* Download your dependency.
* Build it to get its binaries.
* Create a special `.def` file that describes this dependency to cinterop.
* And adjust your build script to generate bindings during the build.

> In Kotlin %kotlinVersion% configuring cinterop dependencies for [intermediate common source sets](https://kotlinlang.org/docs/reference/mpp-share-on-platforms.html#share-code-on-similar-platforms), such as `iosMain`, is not yet supported. 
> You will need to configure such dependencies for both platform source sets: `iosArm64Main` and `iosX64Main`.
>
>{type=”note”}

The sets of steps differ a bit for [libraries](#add-a-library-without-cocoapods) and [frameworks](#add-a-framework-without-cocoapods), but the main idea remains the same.

#### Add a library without CocoaPods:

1. Download the library source code and place it somewhere where you can reference it from your project. 

2. Build a library (library authors usually provide a guide on how to do this) and get a path to the binaries.

3. In your project, create a `.def` file, for example `DateTools.def`.

4. Add a first string to this file: `language = Objective-C`. If you want to use a pure C dependency, omit the language property.

5. Provide values for two mandatory fields:
    * `headers` – describes which headers will be processed by cinterop.
    * `package` – sets the name of the package these declarations should be put into.

    Example:
    ```properties
    headers = DateTools.h
    package = DateTools
    ```    

6. Add information about interoperability with this library to the build script:
    * Pass the path to the `.def` file. This path can be omitted if your `.def` file has the same name as cinterop and it is placed in the folder `src/nativeInterop/cinterop/`.
    * Tell cinterop where to look for header files using the `includeDirs` option.
    * Configure linking against library binaries.

    <tabs>
    <tab title="Groovy">
    
    ```Groovy
    kotlin {
       iosX64 {
           compilations.main {
               cinterops {
                   DateTools {
                       // Path to .def file
                       defFile("src/nativeInterop/cinterop/DateTools.def")
                  
                       // Directories for header search (an analogue of the -I<path> compiler option)
                       includeDirs("include/this/directory", "path/to/another/directory")
                   }
                   anotherInterop { /* ... */ }
               }
           }

           binaries.all {
               // Linker options required to link against the library.
               linkerOpts "-L/path/to/library/binaries", "-lbinaryname"
           }
       }
    }
    
    ```
    </tab>
    <tab title="Kotlin">
    
    ```Kotlin
    kotlin {
       iosX64() {
           compilations.getByName("main") {
               val DateTools by cinterops.creating {
                   // Path to .def file
                   defFile("src/nativeInterop/cinterop/DateTools.def")
     
                   // Directories for header search (an analogue of the -I<path> compiler option)
                   includeDirs("include/this/directory", "path/to/another/directory")
               }
               val anotherInterop by cinterops.creating { /* ... */ }
           }
      
           binaries.all {
               // Linker options required to link against the library.
               linkerOpts("-L/path/to/library/binaries", "-lbinaryname")
           }
       }
    }

    ```
         
    </tab>
    </tabs>

7. Build the project.

Now you can use this dependency from Kotlin code. To do that, import the package you’ve set up in the `package` property in the `.def` file. For the example above, that would be:
```kotlin
import DateTools.*
```

#### Add a framework without CocoaPods:

1. Download the framework source code and place it somewhere where you can reference it from your project.

2. Build the framework (framework authors usually provide a guide on how to do it), and get a path to the binaries.

3. In your project, create a `.def` file, for example `MyFramework.def`.

4. Add a first string to this file: `language = Objective-C`. If you want to use a pure C dependency, omit language property.

5. Provide values for two mandatory fields:
    * `modules` – the name of the framework that should be processed by the cinterop.
    * `package` – the name of the package these declarations should be put into.
    Example:
    ```properties
    modules = MyFramework
    package = MyFramework
    ```

6. Add information about interoperability with the framework to the build script:
    * Pass the path to the .def file. This path can be omitted if your `.def` file has the same name as cinterop and it is placed in the folder `src/nativeInterop/cinterop/`.
    * Pass the framework name to the compiler and linker using the `-framework` option.
    Pass the path to the framework sources and binaries to the compiler and linker using the `-F` option.

    <tabs>
    <tab title="Groovy">
    
    ```Groovy
    kotlin {
       iosX64 {
           compilations.main {
               cinterops {
                   DateTools {
                       // Path to .def file
                       defFile("src/nativeInterop/cinterop/MyFramework.def")
                  
                       compilerOpts("-framework", "MyFramework", "-F/path/to/framework/")
                   }
                   anotherInterop { /* ... */ }
               }
           }
      
           binaries.all {
               // Tell the linker where the framework is located.
               linkerOpts("-framework", "MyFramework", "-F/path/to/framework/")
           }
       }
    }

    ```
    </tab>
    <tab title="Kotlin">
    
    ```Kotlin
    kotlin {
       iosX64() {
           compilations.getByName("main") {
               val DateTools by cinterops.creating {
                   // Path to .def file
                   defFile("src/nativeInterop/cinterop/DateTools.def")

                   compilerOpts("-framework", "MyFramework", "-F/path/to/framework/"
               }
               val anotherInterop by cinterops.creating { /* ... */ }
           }
      
           binaries.all {
               // Tell the linker where the framework is located.
               linkerOpts("-framework", "MyFramework", "-F/path/to/framework/")
           }
       }
    }

    ```
         
    </tab>
    </tabs>

7. Build the project.

Now you can use this dependency from Kotlin code. To do that, import the package you’ve set up in the package property in the `.def` file. For the example above, that would be:
```kotlin
import MyFramework.*
```

[More information about Objective-C and Swift interop](https://kotlinlang.org/docs/reference/native/objc_interop.html)

[More information about configuring cinterop from Gradle](https://kotlinlang.org/docs/reference/mpp-dsl-reference.html#native-targets)


## Android dependencies

The workflow for adding Android-specific dependencies to a Kotlin Multiplatform module is the same as for pure Android projects: add a line declaring the dependency you need to your Gradle build script, import the project, and you’ll be able to use this dependency in Kotlin code.

The recommended way to add Android dependencies to Kotlin Mobile Multiplaform projects is by adding it to a specific Android source set:

<tabs>
<tab title="Groovy">
    
```Groovy
sourceSets {
    androidMain {
        dependencies {
            implementation 'com.example.android:app-magic:12.3'
        }
}

```
</tab>
<tab title="Kotlin">
    
```Kotlin
sourceSets["androidMain"].dependencies {
        implementation("com.example.android:app-magic:12.3")
}
```
         
</tab>
</tabs>

Moving what was a top-level dependency in a pure Android project to a specific source set in a KMM project might be difficult if the top-level dependency had some non-trivial configuration name. For example, to move а `debugImplementation` dependency from the top level of an Android project, you’ll need to add an implementation dependency to the source set named `androidDebug`.
To minimize such migration efforts, you can add a dependencies block inside the Android block:

<tabs>
<tab title="Groovy">
    
```Groovy
android {
   ...
  
   dependencies {
       implementation 'com.example.android:app-magic:12.3'
   }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
android {
   ...
  
   dependencies {   
       implementation("com.example.android:app-magic:12.3")
   }
}
```
</tab>
</tabs>

Dependencies declared here will be treated exactly the same as dependencies from the top-level block, but this way will also separate Android dependencies visually in your build script and make it less confusing.

Putting dependencies into standalone dependencies block at the end of the script, in a way that is idiomatic to pure Android projects, is also supported. However, we strongly recommend **against** using it, because when a build script with Android dependencies is configured in the top-level block and other targets’ dependencies are configured per source set, this is likely to cause confusion.

[More information about adding dependencies in Android documentation](https://developer.android.com/studio/build/dependencies)

## Multiplatform libraries

In addition to dependencies specific to iOS and Android ecosystems, some libraries have also adopted the Kotlin Multiplatform technology. Such dependencies can be used from different targets and often even from common code. Examples of multiplatform libraries are [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) or [SQLDelight](https://github.com/cashapp/sqldelight). Library authors usually provide guides for adding their dependency to the project. On this page we will cover some basic cases you are likely to run into while working on multiplatform projects.

* Kotlin standard library is added automatically to all multiplatform projects, you don’t have to do anything manually.

* If you want to use a library from all source sets you can add it only to the common source set. Multiplatform plugin will add corresponding parts to any other source sets automatically.

<tabs>
<tab title="Groovy">
    
```Groovy
kotlin {
    sourceSets {
        commonMain {
            dependencies {
                implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%'
            }
        }
        androidMain {
            dependencies {
                //dependency to platform part of kotlinx.coroutines will be added automatically
            }
        }
    }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
kotlin {
    sourceSets["commonMain"].dependencies {
        implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%")
    }
    sourceSets["androidMain"].dependencies {
        //dependency to platform part of kotlinx.coroutines will be added automatically
    }
}
```
</tab>
</tabs>

* If you want to use multiplatform library only for specific source sets, you can add it to those only. Library declarations will be available only in those source sets. Please note that you might need to use some platform-specific artefact in such cases (like SQLDelight `native-driver` in the example below). Exact name of the artefact you’ll need should be provided in library documentation.

<tabs>
<tab title="Groovy">
    
```Groovy
kotlin {
    sourceSets {
        commonMain {
            dependencies { 
            // kotlinx.coroutines will be available in all source sets
            implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%'
            }
        }
        androidMain {
            dependencies { }
        }
        iosMain {
            dependencies {
            // SQLDelight will be available only in iOS source set, but not in android or common
            implementation 'com.squareup.sqldelight:native-driver:%sqlDelightVersion%'
            }
        }
    }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
kotlin {
    sourceSets["commonMain"].dependencies {
        //kotlinx.coroutines will be available in all source sets
        implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%")
    }
    sourceSets["androidMain"].dependencies {
    }
    sourceSets["iosX64Main"].dependencies {
        //SQLDelight will be available only in iOS source set, but not in android or common
        implementation("com.squareup.sqldelight:native-driver:%sqlDelightVersion%)
    }
}
```
</tab>
</tabs>

* You can connect one multiplatform project to another as a dependency. To do that just add project dependency to the source set that needs it. If you want to use dependency in all source sets - add it to the common one, in this case other source sets will get their versions automatically.

<tabs>
<tab title="Groovy">
    
```Groovy
kotlin {
    sourceSets {
        commonMain {
            dependencies {
                implementation project(':some-other-multiplatform-module')
            }
        }
        androidMain {
            dependencies {
                //platform part of :some-other-multiplatform-module will be added automatically
            }
        }
    }
}
```
        
</tab>
<tab title="Kotlin">
    
```Kotlin
kotlin {
    sourceSets["commonMain"].dependencies {
        implementation(project(":some-other-multiplatform-module"))
    }
    sourceSets["androidMain"].dependencies {
        //platform part of :some-other-multiplatform-module will be added automatically
    }
}
```
</tab>
</tabs>

[More information about Multiplatform Gradle DSL for configuring dependencies](https://kotlinlang.org/docs/reference/mpp-dsl-reference.html#dependencies)

[Community-driven list of Kotlin Multiplatform libraries](https://libs.kmp.icerock.dev/)