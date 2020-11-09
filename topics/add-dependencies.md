[//]: # (title: Add dependencies to KMM modules)
[//]: # (auxiliary-id: Add_dependencies_to_KMM_modules)

Every application requires a set of libraries in order to operate successfully. 
A KMM application can depend on multiplatform libraries that work on both iOS and Android, and it can depend on platform-specific iOS and Android libraries. 
Here you can learn how to add:
* [iOS dependencies](#ios-dependencies)
* [Android dependencies](#android-dependencies)
* [Multiplatform dependencies](#multiplatform-libraries)

## iOS dependencies

Apple SDK dependencies (such as Foundation or Core Bluetooth) are available as a set of prebuilt libraries in a Kotlin Multiplatform Mobile project. They do not require any additional configuration.

You can also reuse other libraries and frameworks from the iOS ecosystem in your iOS source sets. Kotlin supports interoperability with Objective-C dependencies and Swift dependencies if their API is exported to Objective-C with the `@objc` attribute. Pure Swift dependencies are not yet supported.

Integration with the CocoaPods dependency manager is also supported with the same limitation – you cannot use pure Swift pods. We recommend using CocoaPods to handle iOS dependencies in Kotlin Multiplatform Mobile (KMM) projects. We only suggest managing dependencies manually if you want to tune the interop process specifically or if you have some other strong reason to do so.

### With CocoaPods

1. Perform [initial CocoaPods integration setup](https://kotlinlang.org/docs/reference/native/cocoapods.html#install-the-cocoapods-dependency-manager-and-plugin)

2. Add a dependency on a Pod library from the CocoaPods repository that you want to use by including `pod()` in the build script of your project.

    <tabs>
    <tab title="Groovy">

    ```Groovy
    kotlin {
        cocoapods {
            //..
            pod("AFNetworking") {
                version = "~> 4.0.1"
            }
        }
    }
    ```

    </tab>
    <tab title="Kotlin">

    ```Kotlin
    kotlin {
        cocoapods {
            //..
            pod("AFNetworking") {
                version = "~> 4.0.1"
            }
        }
    }
    ```

    </tab>
    </tabs>

3. Re-import the project.

To use the dependency in your Kotlin code, import the package `cocoapods.<library-name>`. In the example above, that would be:
```kotlin
import cocoapods.AFNetworking.*
```

Learn more about [CocoaPods integration](https://kotlinlang.org/docs/reference/native/cocoapods.html).

### Without CocoaPods

If you don’t want to use CocoaPods, you can use the cinterop tool to create Kotlin bindings for Objective-C or Swift declarations. This will allow you to call them from Kotlin code. To do this:
1. Download your dependency.
2. Build it to get its binaries.
3. Create a special `.def` file that describes this dependency to cinterop.
4. Adjust your build script to generate bindings during the build.

> Configuring cinterop dependencies for [intermediate common source sets](https://kotlinlang.org/docs/reference/mpp-share-on-platforms.html#share-code-on-similar-platforms), such as `iosMain`, is not yet supported in Kotlin %kotlinVersion%. 
> You will need to configure such dependencies for both platform source sets: `iosArm64Main` and `iosX64Main`.
>
>{type=”note”}

The steps differ a bit for [libraries](#add-a-library-without-cocoapods) and [frameworks](#add-a-framework-without-cocoapods), but the idea remains the same.

#### Add a library without CocoaPods 

1. Download the library source code and place it somewhere where you can reference it from your project. 

2. Build a library (library authors usually provide a guide on how to do this) and get a path to the binaries.

3. In your project, create a `.def` file, for example `DateTools.def`.

4. Add a first string to this file: `language = Objective-C`. If you want to use a pure C dependency, omit the language property.

5. Provide values for two mandatory properties:
    * `headers` describes which headers will be processed by cinterop.
    * `package` sets the name of the package these declarations should be put into.

    For example:
    ```properties
    headers = DateTools.h
    package = DateTools
    ```

6. Add information about interoperability with this library to the build script:
    * Pass the path to the `.def` file. This path can be omitted if your `.def` file has the same name as cinterop and is placed in the `src/nativeInterop/cinterop/` directory.
    * Tell cinterop where to look for header files using the `includeDirs` option.
    * Configure linking to library binaries.

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
                // Linker options required to link to the library.
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
                // Linker options required to link to the library.
                linkerOpts("-L/path/to/library/binaries", "-lbinaryname")
            }
        }
    }
    ```

    </tab>
    </tabs>

7. Build the project.

Now you can use this dependency in your Kotlin code. To do that, import the package you’ve set up in the `package` property in the `.def` file. For the example above, this will be:
```kotlin
import DateTools.*
```

#### Add a framework without CocoaPods

1. Download the framework source code and place it somewhere that you can reference it from your project.

2. Build the framework (framework authors usually provide a guide on how to do this) and get a path to the binaries.

3. In your project, create a `.def` file, for example `MyFramework.def`.

4. Add the first string to this file: `language = Objective-C`. If you want to use a pure C dependency, omit the language property.

5. Provide values for these two mandatory properties:
    * `modules` – the name of the framework that should be processed by the cinterop.
    * `package` – the name of the package these declarations should be put into.
    For example:
    ```properties
    modules = MyFramework
    package = MyFramework
    ```

6. Add information about interoperability with the framework to the build script:
    * Pass the path to the .def file. This path can be omitted if your `.def` file has the same name as the cinterop and is placed in the `src/nativeInterop/cinterop/` directory.
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

Now you can use this dependency in your Kotlin code. To do this, import the package you’ve set up in the package property in the `.def` file. For the example above, this will be:

```kotlin
import MyFramework.*
```

Learn more about [Objective-C and Swift interop](https://kotlinlang.org/docs/reference/native/objc_interop.html) and 
[configuring cinterop from Gradle](https://kotlinlang.org/docs/reference/mpp-dsl-reference.html#cinterops).


## Android dependencies

The workflow for adding Android-specific dependencies to a KMM module is the same as it is for pure Android projects: add a line to your Gradle build script declaring the dependency you need and import the project. You’ll then be able to use this dependency in your Kotlin code.

We recommend adding Android dependencies to KMM projects by adding them to a specific Android source set:

<tabs>
<tab title="Groovy">
    
```Groovy
sourceSets {
    androidMain {
        dependencies {
            implementation 'com.example.android:app-magic:12.3'
        }
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

Moving what was a top-level dependency in an Android project to a specific source set in a KMM project might be difficult if the top-level dependency had a non-trivial configuration name. For example, to move а `debugImplementation` dependency from the top level of an Android project, you’ll need to add an implementation dependency to the source set named `androidDebug`.
To minimize the effort you have to put in to deal with migration problems like this, you can add a `dependencies` block inside the `android` block:

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

Dependencies declared here will be treated exactly the same as dependencies from the top-level block, but declaring them this way will also separate Android dependencies visually in your build script and make it less confusing.

Putting dependencies into a standalone `dependencies` block at the end of the script, in a way that is idiomatic to Android projects, is also supported. However, we strongly recommend **against** doing this because configuring a build script with Android dependencies in the top-level block and other target dependencies in each source set is likely to cause confusion.

Learn more about [adding dependencies in Android documentation](https://developer.android.com/studio/build/dependencies).

## Multiplatform libraries

In addition to dependencies specific to the iOS and Android ecosystems, some libraries have also adopted Kotlin Multiplatform technology. Dependencies on these libraries can be used from different targets and often even from common code. Examples of multiplatform libraries are [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) or [SQLDelight](https://github.com/cashapp/sqldelight). The authors of the libraries usually provide guides for adding their dependencies to your project. This page covers basic cases that will be helpful when working on multiplatform projects.

* The Kotlin standard library is added automatically to all multiplatform projects, you don’t have to do anything manually.

* If you want to use a library from all source sets, you can add it only to the common source set. The Kotlin Multiplatform Mobile plugin will add the corresponding parts to any other source sets automatically.

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

* If you want to use a multiplatform library just for specific source sets, you can add it exclusively to them. The specified library declarations will then be available only in those source sets. Please note that you might need to use a platform-specific name in such cases, like SQLDelight `native-driver` in the example below. You should be able to find the exact name in the library’s documentation.

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
            // SQLDelight will be available only in the iOS source set, but not in Android or common
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
        //SQLDelight will be available only in the iOS source set, but not in Android or common
        implementation("com.squareup.sqldelight:native-driver:%sqlDelightVersion%)
    }
}
```
</tab>
</tabs>

* You can connect one multiplatform project to another as a dependency. To do this, simply add a project dependency to the source set that needs it. If you want to use a dependency in all source sets, add it to the common one. In this case, other source sets will get their versions automatically.

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

Learn more about [configuring dependencies](https://kotlinlang.org/docs/reference/using-gradle.html#configuring-dependencies).

Check a [community-driven list of Kotlin Multiplatform libraries](https://libs.kmp.icerock.dev/).
