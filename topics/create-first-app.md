[//]: # (title: Create your first multiplatform application)
[//]: # (auxiliary-id: Create_first_multiplatform_application)

Once you've [set up your mobile multiplatform environment](setup.md), it's time to create your first application.

1. In Android Studio, select **File** | **New** | **New Project**.
2. Select **Mobile Application** in the list of project templates, and click **Next**.  

    ![Mobile Multiplatform project template](kmm-project-wizard-1.png)
    
3. Specify a name for your first application, and click **Next**.  

    ![Mobile Multiplatform project - general settings](kmm-project-wizard-2.png)

4. Keep default names for application and shared folders, select a checkbox to generate sample tests for your project, 
and click **Finish**.  

    ![Mobile Multiplatform project - additional settings](kmm-project-wizard-3.png)  
    
    Wait a bit until your project is set up. It may take some time to download and set up required components when you 
    do this for the first time.
    
## Run your application 

You can run your multiplatform application on Android and iOS.

### Run your application on Android

* In the list of run configurations, select **androidApp**, and click **Run**.  
    
    ![Run multiplatform app on Android](run-android.png){width=400}
    
    ![First mobile multiplatform app on Android](first-kmm-on-android-1.png){width=300}
    
### Run your application on iOS

* In the list of run configurations, select **iosApp**, and click **Run**.  
    
    ![Run multiplatform app on iOS](run-ios.png){width=450}
    
    > Due to technical limitations, iOS device names are shown in the first list. The second list is disabled for **iosApp**.
    >
    {type="note"}   
    
    ![First mobile multiplatform app on Android](first-kmm-on-ios-1.png){width=300}

## Run tests

You can run tests to check that shared code works correctly on both platforms. Of course, you can also write and run tests to check 
platform-specific code.

### Run tests in iOS

1. Switch the view from **Android** to **Project**.  
    The **Project** view correctly displays the structure of your mobile multiplatform project.  
    
    ![Select the Project view](select-project-view.png){width=200}  
    
2. Open the file `iosTest.kt` in **shared** | **src** | **iosTest** | **kotlin**.  
    Directories with **Test** in their name contain tests.  
    This file includes a sample test for iOS.  
    
    ![iOS test Kotlin file](ios-test-kt.png)
   
 
3. Click the **Run** gutter icon next to the test.  

    ![Run iOS test](run-ios-test.png)

Congratulations! The test has passed.

![iOS test result](ios-test-result.png){width=300}

### Run test on Android

Follow a similar procedure to running tests on iOS.

1. Open the file `androidTest.kt` in **shared** | **src** | **androidTest** | **kotlin**.

2. Click the **Run** gutter icon next to the test. 

## Update your application

1. Open the file `common.kt` in **shared** | **src** | **commonMain** | **kotlin**.  
    This file stores shared code for both platforms - Android and iOS. If you make changes to the common code, you will see
    changes in both applications.

    ![Common Kotlin file](common-kotlin-file.png)
    
2. Update the common code - use a Kotlin standard library function that works on all platforms and reverts text - `reversed()`.

    ```kotlin
    expect class Platform() {
        val platform: String
    }
    
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
    As you see, tests fail. Update the tests to pass. You know how to do this, right? ;)
    
    ![iOS test failed](ios-test-failed.png){width=300}
    