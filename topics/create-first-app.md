[//]: # (title: Create your first multiplatform application)
[//]: # (auxiliary-id: Create_first_multiplatform_application)

Once you've [setup your mobile multiplatform environment](setup.md), it's time to create your first application.

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
    
    ![First mobile multiplatform app on Android](first-kmm-on-android.png){width=300}
    
### Run your application on iOS

* In the list of run configurations, select **iosApp**, and click **Run**.  
    
    ![Run multiplatform app on iOS](run-ios.png){width=450}
    
    > Due to technical limitations, iOS device names are shown in the first list. The second list is disabled for **iosApp**.
    >
    {type="note"}   
    
    IMAGE

## Update your application

1. Switch the view from **Android** to **Project**.  
    The **Project** view correctly displays the structure of your mobile multiplatform project.  
    
    ![Select the Project view](select-project-view.png){width=200}  
    
    ![Project view](project-view.png){width=200}

2. Open the file `common.kt` in **shared** | **src** | **commonMain** | **kotlin**.  
    This file stores shared code for both platforms - Android and iOS. If you make changes to the common code, you will see
    changes in both applications.

    ![Common Kotlin file](common-kotlin-file.png)
    
3. Update the common code - use a Kotlin standard library function that works on all platforms and reverts text - `reversed()`.

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

4. Run the updated application on Android.

    ![Updated mobile multiplatform app on Android](first-kmm-on-android-2.png){width=300}
    
5. Run the updated application on iOS.  

    IMAGE