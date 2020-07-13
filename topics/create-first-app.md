[//]: # (title: Create your first application)
[//]: # (auxiliary-id: Create_First_Application)

## Set up the multiplatform environment
Before you start creating your first application that works on iOS and Android, set up the environment:

* Take a Mac with the macOS operating system. It is required for building iOS applications.
* Install the latest version of [Android Studio](https://developer.android.com/studio). The recommended version is 4.1 
or higher. You will use Android Studio for creating your multiplatform applications and running them on device simulators.
* Install the latest version of [Xcode](https://apps.apple.com/us/app/xcode/id497799835). The recommended version is ... or higher. Xcode is required for building your iOS 
applications. Most of the time, Xcode will work in the background. You will need to use it if you want to add Swift or 
Objective-C code to your iOS app and run your iOS application on a real device.
* Update the Kotlin plugin to the version 1.4.0 or higher.  
    In Android Studio, select **Tools** | **Kotlin** | **Configure Kotlin Plugin Updates** and update to the latest 
    version in the **Stable** update channel.
* Install the *Mobile Multiplatform* plugin.  
    Select  **Preferences** | **Plugins**, search for the plugin *Mobile Multiplatform* in **Marketplace** and install it.
    
    ![Mobile Multiplatform plugin](/mobile-multiplatform-plugin.png){width=400}

## Create your first multiplatform mobile application