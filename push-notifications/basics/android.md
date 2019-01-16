---
title: Skygear Push Basics
---

[[toc]]

::: note

To setup android app with FCM SDK, please use Skygear Android SDK v1.6.5 or later.

:::

::: caution

For migrating to FCM from the older version Skygear Android SDK, please read [Migrate to FCM from GCM][android-migrate-fcm]

:::

Once you have [setup FCM in skygear skygear][setup-fcm-in-skygear] properly, you will need to enable receiving push notifications in the client app on Android. You need to:

1. Add Firebase to the project
2. Setup `FirebaseMessagingService` to register token and receive message

## Add Firebase to the project

- In the General tab of project setting, select the android icon to register app.

   ![Add android app in Firebase Console][fcm-add-android-app]

    Complete the steps to add your app into the Firebase project. Download the
    `google-services.json` file during the registration process. You can get back this file
    anytime in the General tab. Put the `google-services.json` into your Android app
    module root directory.

- Add the Google services plugin into your project by modifying the `build.gradle` files.
    The plugin will load the `google-services.json` file.

    **Project-level `build.gradle` (`<project>/build.gradle`):**
    ```
    buildscript {
        dependencies {
            // Add this line
            classpath 'com.google.gms:google-services:4.0.1'
        }
    }
    ```

    **App-level `build.gradle` (`<project>/<app-module>/build.gradle`):**
    ```
    dependencies {
        // Add this line
        implementation 'com.google.firebase:firebase-core:16.0.6'
    }
    ...
    // Add to the bottom of the file
    apply plugin: 'com.google.gms.google-services'
    ```

## Setup `FirebaseMessagingService` to register token and receive message

There are two ways to setup `FirebaseMessagingService`:

1. Use Skygear default `FirebaseMessagingService`, it will help you to handle
    the token registration and simply display the message in android default Notification.
2. Create your own `FirebaseMessagingService`, use Skygear SDK for sending token to skygear and
    customize code in `onMessageReceived`.


### Option 1: Use Skygear default `FirebaseMessagingService`

- Update `AndroidManifest.xml` file in your application, include the following in the manifest:

    ```
    <service android:name="io.skygear.skygear.fcm.FirebaseMessagingService">
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT" />
        </intent-filter>
    </service>
    ```

### Option 2: Create your own `FirebaseMessagingService`

- Create your own `FirebaseMessagingService` class, extends `com.google.firebase.messaging.FirebaseMessagingService`. Override method
`onNewToken` and `onMessageReceived`.

    ```java
    @Override
    public void onNewToken(String token) {
        // send registration to skygear server
        Container container = Container.defaultContainer(this.getApplicationContext());
        container.getPush().registerDeviceToken(token);
    }

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        super.onMessageReceived(remoteMessage);
        // handle notification
    }
    ```
    ```kotlin
    override fun onNewToken(token: String?) {
        // send registration to skygear server
        val container = Container.defaultContainer(applicationContext)
        container.getPush().registerDeviceToken(token)
    }

    override fun onMessageReceived(remoteMessage: RemoteMessage?) {
        // handle notification
    }
    ```

- Update `AndroidManifest.xml` file in your application, include the following in the manifest:

    ```
    <service android:name=".YourFirebaseMessagingService">
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT" />
        </intent-filter>
    </service>
    ```

[fcm-add-android-app]:/assets/push-notifications/fcm-add-android-app.png
[setup-fcm-in-skygear]: /guides/push-notifications/config/android/
[android-migrate-fcm]: /guides/push-notifications/android-migrate-fcm/
[firebase-console]: https://console.firebase.google.com
[setup-fcm-android-client]: https://firebase.google.com/docs/cloud-messaging/android/client
