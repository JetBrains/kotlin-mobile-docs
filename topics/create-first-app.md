[//]: # (title: Create your first multiplatform application)
[//]: # (auxiliary-id: Create_your_first_multiplatform_application)

Now that you've [set up an environment for KMM development](setup.md), it's time to create your first application.

1. In Android Studio, select **File** | **New** | **New Project**.
2. Select **KMM Application** in the list of project templates, and click **Next**.  

    ![Mobile Multiplatform project template](kmm-project-wizard-1.png)
    
3. Specify a name for your first application, and click **Next**.  

    ![Mobile Multiplatform project - general settings](kmm-project-wizard-2.png)

4. Keep the default names for the application and shared folders, select the checkbox to generate sample tests for your project, 
and click **Finish**.  

    ![Mobile Multiplatform project - additional settings](kmm-project-wizard-3.png)  
    
Now wait while your project is set up. It may take some time to download and set up the required components when you 
do this for the first time.
    
To view the complete structure of your mobile multiplatform project, switch the view from **Android** to **Project**. 
You can [discover what your project includes](discover-kmm-project.md) and how you can use this. 
    
![Select the Project view](select-project-view.png){width=200}  
    
## Run your application 

You can run your multiplatform application on Android and iOS.

### Run your application on Android

* In the list of run configurations, select **androidApp**, and click **Run**.  
    
    ![Run multiplatform app on Android](run-android.png){width=400}
    
    ![First mobile multiplatform app on Android](first-kmm-on-android-1.png){width=300}

#### Run on a different Android simulator

* In the list of Android simulators, select a simulator you want to use, and click **Run**.

    ![Run Android application on a different simulator](android-new-simulator.png){width=350}
    
### Run your application on iOS

* In the list of run configurations, select **iosApp**, and click **Run**.  
    
    ![Run multiplatform app on iOS](run-ios.png){width=450}
    
    ![First mobile multiplatform app on Android](first-kmm-on-ios-1.png){width=300}

#### Run on a different iOS simulator

By default, your iOS application runs in an iPhone 11 simulator. If you want to run it on a different simulator, you can add a 
new run configuraion.

1. In the list of run configurations, click **Edit Configurations**.

    ![Edit run configurations](ios-edit-configurations.png){width=450}

2. Click the **+** button above the list of configurations and select **iOS Application**.

    ![New run configuration for iOS application](ios-new-configuration.png)

4. Name your configuration.

5. Select the simulator in the list, and then click **OK**.

    ![New run configuration with iOS simulator](ios-new-simulator.png)
    
6. Click **Run** to run your application on the new simulator.
    
#### Run on a real device

If you want to run your iOS application on a real iPhone, [connect it to Xcode](https://developer.apple.com/documentation/xcode/running_your_app_in_the_simulator_or_on_a_device), 
 [create a run configuration](#run-your-application-on-a-different-ios-simulator) for it, and run your application.

> If your build fails, follow a workaround in [this issue](https://youtrack.jetbrains.com/issue/KT-40907).
>
{type="note"}

## Run tests

You can run tests to check that the shared code works correctly on both platforms. Of course, you can also write and run tests to check the 
platform-specific code.

### Run tests on iOS
    
1. Open the file `iosTest.kt` in `shared/src/iosTest/kotlin/com.example.shared`.  
    Directories with `Test` in their name contain tests.  
    This file includes a sample test for iOS.  
    
    ![iOS test Kotlin file](ios-test-kt.png)
   
2. Click the **Run** icon in the gutter next to the test.  

Tests run on a simulator without UI. Congratulations! The test has passed – see test results in the console.

![iOS test result](ios-test-result.png){width=300}

### Run tests on Android

For Android, follow a procedure that is very similar to the one for running tests on iOS.

1. Open the file `androidTest.kt` in `shared/src/androidTest/kotlin/com.example.shared`.

2. Click the **Run** gutter icon next to the test. 

## Update your application

1. Open the file `Greeting.kt` in `shared/src/commonMain/kotlin/com.example.shared`.  
    This directory stores the shared code for both platforms – Android and iOS. If you make changes to the shared code, you will see
    changes in both applications.

    ![Common Kotlin file](common-kotlin-file.png)
    
2. Update the shared code – use the Kotlin standard library function that works on all platforms and reverts text: `reversed()`.

    ```kotlin
    class Greeting {
        fun greeting(): String {
            return "Guess what it is! > ${Platform().platform.reversed()}!"
        }
    }
    ```

3. Run the updated application on Android.

    ![Updated mobile multiplatform app on Android](first-kmm-on-android-2.png){width=300}
    
4. Run the updated application on iOS.  

    ![Updated mobile multiplatform app on iOS](first-kmm-on-ios-2.png){width=300}
    
5. Run tests on Android and iOS.  
    As you see, the tests fail. Update the tests to pass. You know how to do this, right? ;)
    
    ![iOS test failed](ios-test-failed.png)