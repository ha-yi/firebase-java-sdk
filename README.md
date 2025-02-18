<h1 align="left">Firebase Java SDK <img alt="GitHub last commit" src="https://img.shields.io/github/last-commit/gitliveapp/firebase-java-sdk?style=flat-square"> <a href="https://git.live"><img src="https://img.shields.io/badge/collaborate-on%20gitlive-blueviolet?style=flat-square"></a></h1>
<img align="left" width="75px" src="https://avatars2.githubusercontent.com/u/42865805?s=200&v=4"> 
  <b>Built and maintained with 🧡 by <a href="https://git.live">GitLive</a></b><br/>
  <i>Development teams merge faster with GitLive</i><br/>
<br/>
<br/>
The Firebase Java SDK is a pure java port of the <a href="https://github.com/firebase/firebase-android-sdk">Firebase Android SDK</a> 
to run in clientside java environments such as the desktop.
<br/>
<br/>

Note this is different to the official [Firebase Admin Java SDK](https://github.com/firebase/firebase-admin-java) which enables access 
to Firebase services from privileged environments (such as servers or cloud).

It is used by the <a href="https://github.com/GitLiveApp/firebase-kotlin-sdk">Firebase Kotlin SDK</a> to support the JVM target for 
multiplatform projects, enabling you to use Firebase directly from your common source targeting <strong>Desktop</strong>, 
<strong>iOS</strong>, <strong>Android</strong> and <strong>Web</strong>, enabling the use of Firebase as a backend for 
<a href="https://www.jetbrains.com/lp/compose-multiplatform/">Compose Multiplatform</a>, for example.

It's currently in an alpha state, to be used at your own discretion, and contributions are very welcome! 

## Using in your projects

You can add the library via Gradle:

```kotlin
dependencies {
    implementation("dev.gitlive:firebase-java-sdk:0.1.2")
}
```

Or Maven:

```xml
<dependency>
    <groupId>dev.gitlive</groupId>
    <artifactId>firebase-java-sdk</artifactId>
    <version>0.1.2</version>
</dependency>
```

You can skip the above if you are using the SDK via the <a href="https://github.com/GitLiveApp/firebase-kotlin-sdk">Firebase Kotlin SDK</a> 

### Initializing the SDK

Before you can use the SDK you need to call the `FirebasePlatform.initializeFirebasePlatform` function to provide an implementation for 
logging, and persistent storage for simple key value pairs. This is used by the various Firebase products, for example, to persist the 
signed-in user in Firebase Auth.

Here's a simple example implementation in Kotlin that only persists in-memory:

```kotlin
FirebasePlatform.initializeFirebasePlatform(object : FirebasePlatform() {
    val storage = mutableMapOf<String, String>()
    override fun store(key: String, value: String) = storage.set(key, value)
    override fun retrieve(key: String) = storage[key]
    override fun clear(key: String) { storage.remove(key) }
    override fun log(msg: String) = println(msg)
})
```

It is also up to you to initialize the Firebase application object manually (unlike the Android SDK which is normally initialized via 
the configuration file). 

You first need to create a Firebase options object to hold the configuration data for the Firebase application. Full documentation for 
the options can be found in the [Android API reference documentation](https://firebase.google.com/docs/reference/android/com/google/firebase/FirebaseOptions.Builder).

The use of `FirebaseOptions.Builder` is shown in the following example:

```java
// Manually configure Firebase Options. The following fields are REQUIRED:
//   - Project ID
//   - App ID
//   - API Key
FirebaseOptions options = new FirebaseOptions.Builder()
        .setProjectId("my-firebase-project")
        .setApplicationId("1:27992087142:android:ce3b6448250083d1")
        .setApiKey("AIzaSyADUe90ULnQDuGShD9W23RDP0xmeDc6Mvw")
        // setDatabaseURL(...)
        // setStorageBucket(...)
        .build();
```

## Project status

The following libraries are available for the various Firebase products.

| Service or Product	                                                          | Port of Android version | 
|---------------------------------------------------------------------------------|:------------------------|
| [Authentication](https://firebase.google.com/docs/auth)                         |  N/A[^1]                |
| [Cloud Firestore](https://firebase.google.com/docs/firestore)                   | `24.1.2`                |
| [Realtime Database](https://firebase.google.com/docs/database)                  | `20.1.0`                |
| [Cloud Functions](https://firebase.google.com/docs/functions)                   | `20.1.0`                |

[^1]: Google has not open-sourced the Firebase Auth implementation for Android so a basic implementation using the Rest API is provided.

Is the Firebase library or API you need missing? [Create an issue](https://github.com/GitLiveApp/firebase-java-sdk/issues/new?labels=API+coverage&template=increase-api-coverage.md&title=Add+%5Bclass+name%5D.%5Bfunction+name%5D+to+%5Blibrary+name) to request additional API coverage or be awesome and [submit a PR](https://github.com/GitLiveApp/firebase-java-sdk/fork).

### Limitations

Currently, the following limitations are observed:

* Firebase Auth implementation is minimal and only supports a small subset of the API:
  - `signInAnonymously`
  - `signInWithCustomToken`
  - `currentUser`
  - `getAccessToken`
  - `getUid`
  - `addIdTokenListener`
  - `removeIdTokenListener`
  - `addAuthStateListener`
  - `removeAuthStateListener`
  - `signOut`
* Cloud Firestore does not support [Offline Persistence](https://firebase.google.com/docs/firestore/manage-data/enable-offline#configure_offline_persistence) or [SSL](https://firebase.google.com/docs/reference/android/com/google/firebase/firestore/FirebaseFirestoreSettings.Builder#setSslEnabled(boolean)), and should be setup as follows:
```java
FirebaseFirestoreSettings settings = new FirebaseFirestoreSettings.Builder(db.getFirestoreSettings())
    .setPersistenceEnabled(false)
    .setSslEnabled(false)
    .build();
db.setFirestoreSettings(settings);
```
* Realtime Database does not support [Disk Persistence](https://firebase.google.com/docs/database/android/offline-capabilities), and should be setup as follows:
```java
FirebaseDatabase.getInstance().setPersistenceEnabled(false)
```

## Building and contributing

This library is built with Gradle. 

Run `./gradlew build` to build, the first time you run build it will fail as it requires the output of the custom `copyAars` and `extractClasses` 
gradle tasks to be present in the build folder before the project will successfully compile. These tasks extract the jar files from the 
Firebase Android library AAR files to the `build/jar` folder and run on `./gradlew build`.

### Implementation details

Apart from Firebase Auth, the Firebase Android libraries and their dependencies are used as-is from the published maven artifacts available
at [Google's Maven Repository](https://maven.google.com). These are obviously designed to run on Android so to use them without modification we 
include a minimal Android emulation layer in the SDK only implementing the functionality used by the Firebase libraries at runtime.

Robolectric is included as a compile-only dependency to overcome compilation errors due to missing Android APIs as they are many more 
compile-time dependencies on the Android SDK than what is required for proper functioning at run-time. This makes development of the SDK 
somewhat challenging as the compiler is unable to assist you on what Android functionality needs to be emulated before the library will 
function correctly (instead the library will error at runtime usually with one of the [ReflectiveOperationException](https://docs.oracle.com/javase%2F7%2Fdocs%2Fapi%2F%2F/java/lang/ReflectiveOperationException.html) subclasses.)

The best way to support more of the Firebase SDKs is via writing unit tests to test the required functionality at 
runtime and assert the correct behavior.

