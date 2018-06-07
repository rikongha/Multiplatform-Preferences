# Multiplatform Settings

This is a Kotlin library for Multiplatform mobile apps, so that common code can persist key-value data. It stores things using SharedPreferences on Android and UserDefaults on iOS. 

## Adding to your project
First, add the multiplatform-settings bintray url to the `repositories` block of any module using it.

    repositories {
        ...
        maven { url = 'https://dl.bintray.com/russhwolf/multiplatform-settings' }
    }

In your `kotlin-platform-common` module, add the dependency

    implementation "com.russhwolf:multiplatform-settings-common:0.1-alpha2"
    
In your `kotlin-platform-android` module, add an `expectedBy` dependency on the common module as well as the dependency

    implementation "com.russhwolf:multiplatform-settings-android:0.1-alpha2"
    
In your `konan` module, add an `expectedBy` dependency on the common module as well as separate artifacts for the targets `ios_arm64` (physical device) and `ios_x64` (emulator). The syntax here is not particularly well-documented, but here's an example to illustrate. Assume you want to expose a framework named `MyKotlinFramework` to your ios project.

    konanArtifacts {
        framework('MyKotlinFramework_ios_arm64', targets: ['ios_arm64']) {
            enableMultiplatform true
            artifactName 'MyKotlinFramework'
            dependencies {
                artifactMyKotlinFramework_ios_arm64 "com.russhwolf:multiplatform-settings-ios_arm64:0.1-alpha2"
            }
        }
        framework('MyKotlinFramework_ios_x64', targets: ['ios_x64']) {
            enableMultiplatform true
            artifactName 'MyKotlinFramework'
            dependencies {
                artifactMyKotlinFramework_ios_x64 "com.russhwolf:multiplatform-settings-ios_x64:0.1-alpha2"
            }
        }
    }

See also the sample project, which uses this structure.

## Usage

A `Settings` instance can be created using a platform-specific factory. On Android, this factory needs a `Context` parameter

    val context: Context = ...
    val factory: Settings.Factory = Settings.Factory(context)
    val settings: Settings = factory.create("my_settings_name")
    
On iOS, the factory can be instantiated without passing any parameter

    val factory: Settings.Factory = Settings.Factory()
    val settings = factory.create("my_settings_name")
    
On both platforms, the name argument to `Factory.create()` can be omitted, and a platform-specific default will be used.
    
Once the `Settings` instance is created, you can store values by calling the various `putXXX()` methods, or their operator shortcuts

    settings.putInt("key", 3)
    settings["key"] = 3
    
You can retrieve stored values via the `getXXX()` methods or their operator shortcuts. If a key is not present, then the supplied default will be returned instead.

    val a: Int = settings.getInt("key")
    val b: Int = settings.getInt("key", defaultValue = -1) 
    val c: Int = settings["key", -1]
    
The `getXXX()` and `putXXX()` operation for a given key can be wrapped using a property delegate. This has the advantage of ensuring that the key is always accessed with a consistent type.

    val a: Int by settings.int("key")
    val b: Int by settings.int("key", defaultValue = -1)
    
    val c: Int? by settings.nullableInt("another_key")
    
Existence of a key can be queried
     
    val a: Boolean = settings.hasKey("key")
    val b: Boolean = "key" in settings
     
 Values can also be removed by key
  
    settings.remove("key")
    settings -= "key"  
  
 Finally, all values in a `Settings` instance can be removed
      
    settings.clear()

## Project Structure
The library logic lives in the module `multiplatform-settings` and its `common`, `android`, and `ios` submodules. The common module holds `expect` declarations for the `Settings` class, which can persist values of the `Int`, `Long`, `String`, `Float`, `Double`, and `Boolean` types. It also holds property delegate wrappers and other operator functions for cleaner syntax and usage. The android and ios modules then hold `actual` declarations, delegating to `SharedPreferences` or `NSUserDefaults`.

Some simple unit tests are defined which can be run via `./gradlew test`. These use Robolectric on Android to mock out the platform-specific behavior, and use the `macos` target to run the native tests.

There is also a sample project to demonstrate usage, which is configured as a separate IDEA/gradle project in the `sample` directory. It includes a `shared` module with `common`, `android`, and `ios` submodules, to demo a shared logic layer consuming the library. It also includes an `app-android` module which consumes `shared:android` and defines an Android UI, as well as an Xcode project in the `app-ios` directory which consumes the framework exported from `shared:ios` and holds an iOS UI.

In order to build the library, you must supply a file in the root directory called `keys.properties`, which defines the properties `bintrayUser` and `bintrayKey`. These are used to upload build artifacts to bintray. The sample project can be built separately and shouldn't need any additional configuration.

## License
        
    Copyright 2018 Russell Wolf
    
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
       http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
