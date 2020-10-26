[//]: # (title: KMM releases)
[//]: # (auxiliary-id: KMM_releases)

Since KMM is now in [Alpha](kmm-evolution.md), we are continuously working on stabilizing the KMM plugin for Android Studio and regularly release new versions 
that include new features, improvements, and bug fixes. 

Make sure that you have the latest version of the KMM plugin!

## Update to a new release

Android Studio suggests updating to a new KMM plugin release once it is out. When you accept the suggestion, it automatically updates the KMM plugin to the new version. 
You need to restart Android Studio to complete plugin installation.

You can check the KMM plugin version and update the plugin manually in **Preferences** | **Plugins**.

You need to have compatible Kotlin version for the KMM plugin to work correctly. You can find compatible versions in the [release details](#release-details).
You can check the Kotlin version and update it in **Preferences** | **Plugins** or in **Tools** | **Kotlin** | **Configure Plugin Updates**.

>If you do not have compatible Kotlin version installed, the KMM plugin will be disabled. You will need to update your Kotlin 
>version, and then enable the KMM plugin in **Preferences** | **Plugins**.
>
{type="note"}

## Release details

The following table lists details of latest KMM plugin releases. 

<table> 
<tr>
<th>
Release info
</th>
<th>
Release highlights
</th>
<th>
Compatible Kotlin version
</th>
</tr>
<tr>
<td>

**0.1.3**

Released: October 2, 2020

</td>
<td>

* Added compatibility with iOS 14 and Xcode 12
* Fixed naming in platform tests created by the KMM Wizard

</td>
<td>

* [Kotlin 1.4.10](https://kotlinlang.org/releases.html#release-details)
* Kotlin 1.4.20

</td>
</tr>
<tr>
<td>

**0.1.2**

Released: September 29, 2020

</td>
<td>

 * Fixed compatibility with [Kotlin 1.4.20-M1](https://kotlinlang.org/eap/#build-details)
 * Enabled error reporting to JetBrains by default

</td>
<td>

* [Kotlin 1.4.10](https://kotlinlang.org/releases.html#release-details)
* Kotlin 1.4.20

</td>
</tr>

<tr>
<td>

**0.1.1**

Released: September 10, 2020

</td>
<td>

* Fixed compatibility with Android Studio Canary 8 and higher

</td>
<td>

* [Kotlin 1.4.10](https://kotlinlang.org/releases.html#release-details)
* Kotlin 1.4.20

</td>
</tr>
<tr>
<td>

**0.1.0**

Released: August 31, 2020

</td>
<td>

* The first version of the KMM plugin. Learn more in the [blog post](https://blog.jetbrains.com/kotlin/2020/08/kotlin-multiplatform-mobile-goes-alpha/).

</td>
<td>

* [Kotlin 1.4.10](https://kotlinlang.org/releases.html#release-details)
* Kotlin 1.4.20

</td>
</tr>

</table>