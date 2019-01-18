---
title: Migrate to FCM from GCM
---

[[toc]]

As of April 10, 2018, [Google has deprecated GCM][google-migrate-fcm-faq]. For apps that are using GCM in Skygear for Android push notifications, you can follow the below steps to migrate from GCM to FCM.

1. Update Skygear server to v1.7.0+
1. Migrate from GCM setup to FCM in android client

## Update Skygear server

After updating Skygear server to v1.7.0+, Skygear server will use the configured server key with FCM API automatically. There's no need to reconfigure.

## Migrate from GCM setup to FCM in android client

1. Add Firebase to existing GCM project

    - Go to [Firebase Console][firebase-console], click *Add project*.

    ![Firebase Console][migrate-fcm-firebase-console]

    - Select the existing project and add firebase.

    ![Create FCM project][migrate-fcm-create-project]

1. Select **Add Firebase to your android app**

    ![Select add Firebase to your android app][migrate-fcm-select-add-firebase-to-app]

1. Complete the **Add Firebase to your android app** setup flow

    ![Add Firebase to your android app][migrate-fcm-add-firebase-to-app-setup]

    - Fill in the form and download the `google-services.json` file.
    - Put the `google-services.json` into your Android app module root directory.
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

1. Update the Skygear Android SDK to v1.7.0+

1. Replace the GCM with FCM setup in your code

    - Remove GCM sender id setting

        If you are using `Configuration.Builder`:

        ```java
        Configuration config = new Configuration.Builder()
            .endPoint("https://<your-app-name>.skygeario.com/")
            .apiKey("<your-api-key>")
            // Remove `gcmSenderId`
            .gcmSenderId("<your-gcm-sender-id>")
            .build();

        Container.defaultContainer(this).configure(config);
        ```

        If you are extending `SkygearApplication`:
        ```java
        public class MyApplication extends SkygearApplication {

            // Remove `getGcmSenderId` method
            @Override
            public String getGcmSenderId() {
                return "<your-gcm-sender-id>";
            }
        }
        ```

    - Remove GCM Token Registration Triggering

        Remove the GSM token registration codes. We will use `FirebaseMessagingService`
        to obtain device token.
        ```java
        Container skygear = Container.defaultContainer(this);
        if (skygear.getPush().getGcmSenderId() != null) {
            Intent gcmTokenRegisterIntent = new Intent(this, RegistrationIntentService.class);
            this.startService(gcmTokenRegisterIntent);
        }
        ```

    - Update the `AndroidManifest.xml`

        **Case 1: If you haven't override notification**

        Before
        ```
        <?xml version="1.0" encoding="utf-8"?>
        <manifest
            xmlns:android="http://schemas.android.com/apk/res/android"
            package="your.app.package">

            <!-- other permissions -->
            <uses-permission android:name="android.permission.WAKE_LOCK" />

            <permission
                android:name="your.app.package.permission.C2D_MESSAGE"
                android:protectionLevel="signature" />
            <uses-permission android:name="your.app.package.permission.C2D_MESSAGE" />

            <application ...>
                <!-- Your Activity Declarations -->
                ...

                <!-- Delegate Skygear SDK to manage GCM Services -->
                <receiver
                    android:name="com.google.android.gms.gcm.GcmReceiver"
                    android:exported="true"
                    android:permission="com.google.android.c2dm.permission.SEND">
                    <intent-filter>
                        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                        <category android:name="your.app.package" />
                    </intent-filter>
                </receiver>
                <service
                    android:name="io.skygear.skygear.gcm.ListenerService"
                    android:exported="false">
                    <intent-filter>
                        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                    </intent-filter>
                </service>
                <service
                    android:name="io.skygear.skygear.gcm.InstanceIDListenerService"
                    android:exported="false">
                    <intent-filter>
                        <action android:name="com.google.android.gms.iid.InstanceID" />
                    </intent-filter>
                </service>
                <service
                    android:name="io.skygear.skygear.gcm.RegistrationIntentService"
                    android:exported="false">
                </service>

            </application>
        </manifest>
        ```

        After
        ```
        <?xml version="1.0" encoding="utf-8"?>
        <manifest
            xmlns:android="http://schemas.android.com/apk/res/android"
            package="your.app.package">

            <application ...>
                <!-- Your Activity Declarations -->
                ...
                <service android:name="io.skygear.skygear.fcm.FirebaseMessagingService">
                    <intent-filter>
                        <action android:name="com.google.firebase.MESSAGING_EVENT" />
                    </intent-filter>
                </service>

            </application>
        </manifest>
        ```

        **Case 2: If you have override notification**

        Similar with case 1, but replacing your `GcmListenerService` with new `FirebaseMessagingService`.

        Before
        ```
        public class MyListenerService extends com.google.android.gms.gcm.GcmListenerService {
            @Override
            public void onMessageReceived(String s, Bundle bundle) {
                super.onMessageReceived(s, bundle);

                /* Handle the GCM notification */
            }
        }
        ```

        After
        ```
        public class MyFirebaseMessagingService extends com.google.firebase.messaging.FirebaseMessagingService {
            @Override
            public void onNewToken(String token) {
                // send registration to skygear server
                Container container = Container.defaultContainer(this.getApplicationContext());
                container.getPush().registerDeviceToken(token);
            }


            public void onMessageReceived(RemoteMessage remoteMessage) {
                super.onMessageReceived(remoteMessage);
                // Handle the FCM notification
            }
        }
        ```

        Refer to case 1, update the `AndroidManifest.xml`. The only difference is `FirebaseMessagingService`,
        use your own messaging service.

        After
        ```
        <?xml version="1.0" encoding="utf-8"?>
        <manifest
            xmlns:android="http://schemas.android.com/apk/res/android"
            package="your.app.package">

            <application ...>
                <!-- Your Activity Declarations -->
                ...
                <!-- Use your messaging service -->
                <service android:name=".MyFirebaseMessagingService">
                    <intent-filter>
                        <action android:name="com.google.firebase.MESSAGING_EVENT" />
                    </intent-filter>
                </service>

            </application>
        </manifest>
        ```

[migrate-fcm-firebase-console]: /assets/push-notifications/migrate-fcm-firebase-console.png
[migrate-fcm-create-project]: /assets/push-notifications/migrate-fcm-create-project.png
[migrate-fcm-select-add-firebase-to-app]: /assets/push-notifications/migrate-fcm-select-add-firebase-to-app.png
[migrate-fcm-add-firebase-to-app-setup]: /assets/push-notifications/migrate-fcm-add-firebase-to-app-setup.png
[firebase-console]: https://console.firebase.google.com
[google-migrate-fcm-faq]: https://developers.google.com/cloud-messaging/faq