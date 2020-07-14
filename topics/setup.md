[//]: # (title: Set up the mobile multiplatform environment)
[//]: # (auxiliary-id: Set_up_mobile_multiplatform_environment)

Before you start [creating your first application](create-first-app.md) that works on iOS and Android, set up your mobile 
multiplatform environment:

* Take a Mac with the macOS operating system. It is required for building iOS applications.
* Install the latest version of [Android Studio](https://developer.android.com/studio). The recommended version is 4.1 
or higher. You will use Android Studio for creating your multiplatform applications and running them on device simulators.
* Install the latest version of [Xcode](https://apps.apple.com/us/app/xcode/id497799835). The recommended version is 11.5 
or higher. 
Xcode is required for building your iOS applications. Most of the time, Xcode will work in the background. You will need
 it if you want to add Swift or Objective-C code to your iOS app and run your iOS application on a real device.
* Update the Kotlin plugin to the version 1.4.0 or higher.  
    In Android Studio, select **Tools** | **Kotlin** | **Configure Kotlin Plugin Updates** and update to the latest 
    version in the **Stable** update channel.
* Install the *Mobile Multiplatform* plugin.  
    In Android Studio, select  **Preferences** | **Plugins**, search for the plugin *Mobile Multiplatform* in 
    **Marketplace** and install it.
    
    ![Mobile Multiplatform plugin](mobile-multiplatform-plugin.png){width=500}
    
* Set `JAVA_HOME` for building mutliplatform applications.  
     
    1. In Android Studio, press COMMAND + ; to open **Project Structure** settings.
    2. Open **SDK Location** in the left pane and copy your JDK location. 
    ![JDK location](jdk-location.png)
    3. Run the following command in Terminal:  
        `export JAVA_HOME="YOUR-JDK-LOCATION"` 

Now it's time to [create your first mobile multiplatform application](create-first-app.md).