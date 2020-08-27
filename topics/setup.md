[//]: # (title: Set up an environment for KMM development)
[//]: # (auxiliary-id: Set_up_an_environment_for_KMM_development)

Before you begin [creating your first application](create-first-app.md) to work on both iOS and Android, you will need to 
start by setting up an environment for Kotlin mobile multiplatform (KMM) development:

1. If you are going to work with shared code or Android-specific code, you can work on any computer with an operating 
   system supported by [Android Studio](https://developer.android.com/studio).  
   If you also want to write iOS-specific code and run an iOS application on a simulated or real device, use a Mac with a 
   macOS.  
2. Install [Android Studio](https://developer.android.com/studio) – version 4.1 (RC 1 or higher). 
    You can also use version 4.0 but the KMM plugin for this version has some limitations such as limited configuration options 
    in the [KMM Application Wizard](create-first-app.md) and lack of the KMM Shared Module Wizard that is helpful for [integrating KMM in an existing application](integrate-in-existing-app.md).  
    You will use Android Studio for creating your multiplatform applications and running them on simulated or hardware devices.
3. If you need to write iOS-specific code and run an iOS application, install [Xcode](https://apps.apple.com/us/app/xcode/id497799835) 
    –  version 11.3 or higher.                                                                                                                                                                                                                                                                                                                          
    Most of the time, Xcode will work in the background. You will use it to add Swift or Objective-C code to your iOS application.
4. Update the Kotlin plugin to version 1.4.0 or higher.  
    In Android Studio, select **Tools** | **Kotlin** | **Configure Kotlin Plugin Updates** and update to the latest 
    version in the **Stable** update channel.
5. Install the *Kotlin Multiplatform Mobile* plugin.  
    In Android Studio, select  **Preferences** | **Plugins**, search for the plugin *Kotlin Multiplatform Mobile* in 
    **Marketplace** and install it.
    
    ![Kotlin Multiplatform Mobile plugin](mobile-multiplatform-plugin.png){width=500}
    
6. Install the [JDK](https://jdk.java.net/14/) if you haven't already done so.  
    To check if it's installed, run the command `java -version` in the Terminal.       
     
Now it's time to [create your first KMM application](create-first-app.md).