# Detox for Android

## Breaking Changes :warning:

**If you are installing Detox for Android for the first time, you can skip right over to the setup section.**

> Follow our [Migration Guide](Guide.Migration.md) for instructions on how to upgrade from older versions.

* **In version 11 we switched to using Android Espresso of Android's new [androidx.\*  support libraries](https://developer.android.com/jetpack/androidx/).** We did this in order to stay up to date with Google's latest features and bug fixes, in the hopes of using them to improve our own Android support (which gets better every day!).

* **In version 10, we've made [Kotlin](https://kotlinlang.org/) mandatory for integrating Detox into your Android project.** In the very least, you must include the Kotlin gradle plugin in your project, as we shall see later on. Nevertheless, this is a breaking change so bear that in mind when upgrading. In any case, worry not of the impact on your app, as - unless you effectively use Kotlin in your own native code, **there will be no impact on the final APK**, in terms of size and methods count.

* **As of version 7** we require Android gradle plugin 3.0.0 or newer. This is a breaking change that makes it impossible to support previous Android gradle plugin versions.

  https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html

  For older Android gradle plugin support use `detox@6.x.x` instead ([previous setup guide here](https://github.com/wix/detox/blob/97654071573053def90e8207be8eba011408f977/docs/Introduction.Android.md)).


**Note: As a rule of thumb, we consider all old major versions discontinued; We only support the latest Detox major version.**

## Setup :gear:

### 1. Preliminary

Run through the basic steps of the [Getting Started guide](Introduction.GettingStarted.md), such as the environment and tools setup.

### 2. Apply Detox Configuration

Whether you've selected to apply the configuration in a  `.detoxrc.json` or bundle it into your project's `package.json` (under the `detox` section), this is what the configuration should roughly look like for Android:

```json
{
  "configurations": {
      "android.emu.debug": {
          "binaryPath": "android/app/build/outputs/apk/debug/app-debug.apk",
          "build":
            "cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug && cd ..",
          "type": "android.emulator",
          "device": {
            "avdName": "Pixel_API_28"
          }
      },
      "android.emu.release": {
          "binaryPath": "android/app/build/outputs/apk/release/app-release.apk",
          "build": "cd android && ./gradlew assembleRelease assembleAndroidTest -DtestBuildType=release && cd ..",
          "type": "android.emulator",
          "device": {
            "avdName": "Pixel_API_28"
          }
      }
  }
}
```

>For a comprehensive explanation of Detox configuration, refer to the [dedicated API-reference guide](APIRef.Configuration.md).

Pay attention to `-DtestBuildType`, set either to `debug` or `release` according to the main apk type.


Following device types could be used to control Android devices:

- `android.emulator`. Boot stock Android-SDK emulator (AVD) with provided `name`, for example `Pixel_API_28`.

- `android.attached`. Connect to already-attached android device. The device should be listed in the output of `adb devices` command under provided `name`.
  Use this type to connect to Genymotion emulator.

For a complete, working example, refer to the [Detox example app](/examples/demo-react-native/detox.config.js).

#### 2a. Using product flavors

If you are using custom [productFlavors](https://developer.android.com/studio/build/build-variants#product-flavors) the config needs to be applied a bit differently. This example shows how a `beta` product flavor would look for both debug and release build types:

```json
"detox" : {
    "configurations": {
        "android.emu.beta.debug": {
        "binaryPath": "android/app/build/outputs/apk/beta/debug/app-beta-debug.apk",
        "build": "cd android && ./gradlew assembleBetaDebug assembleBetaDebugAndroidTest -DtestBuildType=debug && cd ..",
        "type": "android.emulator",
        "device": {
          "avdName": "Pixel_API_28"
        }
      },
      "android.emu.beta.release": {
        "binaryPath": "android/app/build/outputs/apk/beta/release/app-beta-release.apk",
        "build": "cd android && ./gradlew assembleBetaRelease assembleBetaReleaseAndroidTest -DtestBuildType=release && cd ..",
        "type": "android.emulator",
        "device": {
          "avdName": "Pixel_API_28"
        }
      }
    }
}
```

### 3. Add the Native Detox dependency

> **Starting Detox 12.5.0, Detox is shipped as a precompiled `.aar`.**
> To configure Detox as a _compiling dependency_, nevertheless -- refer to the _Setting Detox up as a compiling dependency_ section at the bottom.

In your *root* buildscript (i.e. `build.gradle`), register both `google()` _and_ detox as repository lookup points in all projects:

```groovy
// Note: add the 'allproject' section if it doesn't exist
allprojects {
    repositories {
        // ...
        google()
        maven {
            // All of Detox' artifacts are provided via the npm module
            url "$rootDir/../node_modules/detox/Detox-android"
        }
    }
}
```



In your app's buildscript (i.e. `app/build.gradle`) add this in `dependencies` section:

```groovy
dependencies {
	  // ...
    androidTestImplementation('com.wix:detox:+')
}
```

... and add this to the `defaultConfig` subsection:

```groovy
android {
  // ...
  
  defaultConfig {
      // ...
      testBuildType System.getProperty('testBuildType', 'debug')  // This will later be used to control the test apk build type
      testInstrumentationRunner 'androidx.test.runner.AndroidJUnitRunner'
  }
}
```
Please be aware that the `minSdkVersion` needs to be at least 18.



### 4. Add Kotlin

If your project does not already support Kotlin, add the Kotlin Gradle-plugin to your classpath in the root build-script (i.e.`android/build.gradle`):

```groovy
buildscript {
    // ...
    ext.kotlinVersion = '1.3.0' // (check what the latest version is!)
    dependencies {
        // ...
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion"
    }
}
```

_Note: most guides advise of defining a global `kotlinVersion` constant - as in this example, but that is not mandatory._

***Note that Detox has been tested for version 1.1.0 of Kotlin, and higher!***



### 5. Create a Detox-Test Class

Add the file `android/app/src/androidTest/java/com/[your.package]/DetoxTest.java` and fill as in [the detox example app for NR](../examples/demo-react-native/android/app/src/androidTest/java/com/example/DetoxTest.java). **Don't forget to change the package name to your project's**.



### 6. Enable clear-text (unencrypted) traffic for Detox

Starting Android SDK v28, Google have disabled all clear-text network traffic by default. Namely, unless explicitly configured, all of your application's outgoing unencrypted traffic (i.e. non-TLS using HTTP rather than HTTPS) is blocked by the device.

For Detox to work, Detox test code running on the device must connect to the test-running host through it's virtual localhost interface<sup>(*)</sup> using simple HTTP traffic. Therefore, the following network-security exemption configuration must be applied --

*In an xml resource file, e.g. `android/app/src/main/res/xml/network_security_config.xml`:*

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">10.0.2.2</domain>
        <domain includeSubdomains="true">localhost</domain>
    </domain-config>
</network-security-config>
```

*In the app's `AndroidManifest.xml`*

```xml
<manifest>
  <application 
        ...
        android:networkSecurityConfig="@xml/network_security_config">
  </application>
</manifest>
```

> *Refer to the [Detox example app](https://github.com/wix/Detox/tree/master/examples/demo-react-native/android/app/src/main) for an example on this is effectively implemented.* 

**Note: if properly configured, this in no way compromises the security settings of your app.**

For full details, refer to [Android's security-config guide](https://developer.android.com/training/articles/security-config), and the dedicated article in the [Android developers blog](https://android-developers.googleblog.com/2016/04/protecting-against-unintentional.html). 

> *(\*) 10.0.2.2 for Google emulators, 10.0.3.2 for Genymotion emulators.*



## Proguard (Minification)

In apps running [minification using Proguard](https://developer.android.com/studio/build/shrink-code), in order for Detox to work well on release builds, please enable some Detox proguard-configuration rules by applying the custom configuration file on top of your own. Typically, this is defined using the `proguardFiles` statement in the minification-enabled build-type in your `app/build.gradle`:

```groovy
    buildTypes {
        // 'release' is typically the default proguard-enabled build-type
        release {
            minifyEnabled true

            // Typical pro-guard definitions
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            // Detox-specific additions to pro-guard
            proguardFile "${rootProject.projectDir}/../node_modules/detox/android/detox/proguard-rules-app.pro"
        }
    }

```



## Setting Detox up as a compiling dependency

This is an **alternative** to the setup process described under the previous section, on adding Detox as a dependency.



In your project's `settings.gradle` add:

```groovy
include ':detox'
project(':detox').projectDir = new File(rootProject.projectDir, '../node_modules/detox/android/detox')
```



In your *root* buildscript (i.e. `build.gradle`), register `google()` as a repository lookup point in all projects:

```groovy
// Note: add the 'allproject' section if it doesn't exist
allprojects {
    repositories {
        // ...
        google()
    }
}
```



In your app's buildscript (i.e. `app/build.gradle`) add this in `dependencies` section:

```groovy
dependencies {
  	// ...
    androidTestImplementation(project(path: ":detox"))
}
```



In your app's buildscript (i.e. `app/build.gradle`) add this to the `defaultConfig` subsection:

```groovy
android {
  // ...
  
  defaultConfig {
      // ...
      testBuildType System.getProperty('testBuildType', 'debug')  // This will later be used to control the test apk build type
      testInstrumentationRunner 'androidx.test.runner.AndroidJUnitRunner'
  }
}
```

Please be aware that the `minSdkVersion` needs to be at least 18.



## Troubleshooting

### Problem: `Duplicate files copied in ...`

If you get an error like this:

```sh
Execution failed for task ':app:transformResourcesWithMergeJavaResForDebug'.
> com.android.build.api.transform.TransformException: com.android.builder.packaging.DuplicateFileException: Duplicate files copied in APK META-INF/LICENSE
```

You need to add this to the `android` section of your `android/app/build.gradle`:

```groovy
packagingOptions {
    exclude 'META-INF/LICENSE'
}
```



### Problem: Kotlin stdlib version conflicts

The problems and resolutions here are different if you're using Detox as a precompiled dependency artifact (i.e. an `.aar`) - which is the default, or compiling it yourself.

#### Resolving for a precompiled dependency (`.aar`)

Of all [Kotlin implementation flavours](https://kotlinlang.org/docs/reference/using-gradle.html#configuring-dependencies), Detox assumes the most recent one, namely `kotlin-stdlib-jdk8`. If your Android build fails due to conflicts with implementations coming from other dependencies or even your own app, consider adding an exclusion to either the "other" dependencies or detox itself, for example:

```diff
dependencies {
-    androidTestImplementation('com.wix:detox:+')
+    androidTestImplementation('com.wix:detox:+') { 
+        exclude group: 'org.jetbrains.kotlin', module: 'kotlin-stdlib-jdk8'
+    }
}
```

Detox should work with `kotlin-stdlib-jdk7`, as well.

A typical error output formed by `Gradle` in this case is as provided, for example, in [#1380](https://github.com/wix/Detox/issues/1380):

```
Could not determine the dependencies of task ':detox:compileDebugAidl'.
> Could not resolve all task dependencies for configuration ':detox:debugCompileClasspath'.
   > Could not resolve org.jetbrains.kotlin:kotlin-stdlib:1.3.0.
     Required by:
         project :detox
      > Cannot find a version of 'org.jetbrains.kotlin:kotlin-stdlib' that satisfies the version constraints:
           Dependency path 'OurApp:detox:unspecified' --> 'com.squareup.okhttp3:okhttp:4.0.0-alpha01' --> 'org.jetbrains.kotlin:kotlin-stdlib:1.3.30'
           Dependency path 'OurApp:detox:unspecified' --> 'com.squareup.okio:okio:2.2.2' --> 'org.jetbrains.kotlin:kotlin-stdlib:1.2.60'
           Dependency path 'OurApp:detox:unspecified' --> 'org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.3.0' --> 'org.jetbrains.kotlin:kotlin-stdlib:1.3.0'
           Dependency path 'OurApp:detox:unspecified' --> 'com.facebook.react:react-native:0.59.5' --> 'com.squareup.okhttp3:okhttp:4.0.0-alpha01' --> 'org.jetbrains.kotlin:kotlin-stdlib:1.3.30'
           Dependency path 'OurApp:detox:unspecified' --> 'com.facebook.react:react-native:0.59.5' --> 'com.squareup.okio:okio:2.2.2' --> 'org.jetbrains.kotlin:kotlin-stdlib:1.2.60'
           Dependency path 'OurApp:detox:unspecified' --> 'org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.3.0' --> 'org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.0' --> 'org.jetbrains.kotlin:kotlin-stdlib:1.3.0'
           Constraint path 'OurApp:detox:unspecified' --> 'org.jetbrains.kotlin:kotlin-stdlib' strictly '1.3.0' because of the following reason: debugRuntimeClasspath uses version 1.3.0
           Constraint path 'OurApp:detox:unspecified' --> 'org.jetbrains.kotlin:kotlin-stdlib' strictly '1.3.0' because of the following reason: debugRuntimeClasspath uses version 1.3.0

   > Could not resolve org.jetbrains.kotlin:kotlin-stdlib-common:1.3.0.
     Required by:
         project :detox
      > Cannot find a version of 'org.jetbrains.kotlin:kotlin-stdlib-common' that satisfies the version constraints:
           Dependency path 'OurApp:detox:unspecified' --> 'com.squareup.okhttp3:okhttp:4.0.0-alpha01' --> 'org.jetbrains.kotlin:kotlin-stdlib:1.3.30' --> 'org.jetbrains.kotlin:kotlin-stdlib-common:1.3.30'
           Constraint path 'OurApp:detox:unspecified' --> 'org.jetbrains.kotlin:kotlin-stdlib-common' strictly '1.3.0' because of the following reason: debugRuntimeClasspath uses version 1.3.0
```
(i.e. the project indirectly depends on different versions of `kotlin-stdlib`, such as `1.3.0`, `1.3.30`, `1.2.60`)

#### Resolving for a compiling subproject

Detox requires the Kotlin standard-library as it's own dependency. Due to the [many flavours](https://kotlinlang.org/docs/reference/using-gradle.html#configuring-dependencies) by which Kotlin has been released, multiple dependencies often create a conflict.

For that, Detox allows for the exact specification of the standard library to use using two Gradle globals: `detoxKotlinVersion` and `detoxKotlinStdlib`. You can define both in your  root build-script file (i.e.`android/build.gradle`):

```groovy
buildscript {
    // ...
    ext.detoxKotlinVersion = '1.3.0' // Detox' default is 1.2.0
    ext.detoxKotlinStdlib = 'kotlin-stdlib-jdk7' // Detox' default is kotlin-stdlib-jdk8
}
```



### Problem: The app loads but tests fail to start in SDK >= 28

As reported in issue [#1450](https://github.com/wix/Detox/issues/1450), sometimes the application under test would properly launch on an emulator/device when running Detox, but the test runner will hang and will not start running the actual tests.

More specifically, when this happens:

1. Detox and the tests runner launch successfully, alongside the app being run (unless `launchApp: false` has been passed to `detox.init()`), but the first test simply hangs forever (as explained).
2. Eventually, the test runner would time-out.
3. The last reported Detox-logs before time-out would indicate the device failing to connect to the Detox tester on the host. For example:

```sh
detox[12345] DEBUG: [DetoxServer.js/CANNOT_FORWARD] role=testee not connected, cannot fw action (sessionId=11111111-2222-3333-4444-555555555555)
```

* The main step for getting this fixed is to **revisit [step 6](#6-enable-clear-text-unencrypted-traffic-for-detox) in this guide**, which discusses network-security.

* Alternatively, the `android:usesCleartextTraffic="true"` attribute can be configured in the `<application>` tag of the app's `AndroidManifest.xml`, but **that is highly discouraged**.

