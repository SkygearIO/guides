---
title: Skygear Push Basics
---

To enable receiving push notifications from Skygear, you need to:

1. Delegate Skygear SDK to manage the GCM services
2. Provide your GCM Sender ID
3. Trigger GCM Token Registration

<a id="delegate-skygear"></a>
## Delegate Skygear SDK to manage the GCM services

You need to update the `AndroidManifest.xml` file in your application to add
necessary permission and delegate Skygear SDK to manage the GCM services.

Please be reminded that you need to replace all occurrences of
`your.app.package` to the package name of your application.

```html
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

<a id="provide-gcm-sender-id"></a>
## Provide your GCM Sender ID

After delegating Skygear SDK to manage the GCM services, you also need to
provide the GCM Sender ID to Skygear. You may find your GCM Sender ID on
[Firebase Console][firebase-console].

You can add your GCM Sender ID when configuring Skygear.

```java
Configuration config = new Configuration.Builder()
    .endPoint("https://<your-app-name>.skygeario.com/")
    .apiKey("<your-api-key>")
    .gcmSenderId("<your-gcm-sender-id>")
    .build();

Container.defaultContainer(this).configure(config);
```

If you are extending `SkygearApplication`, you may override `getGcmSenderId()`
to provide your GCM Sender ID.

```java
public class MyApplication extends SkygearApplication {

    /* Other overriding */

    @Override
    public String getGcmSenderId() {
        return "<your-gcm-sender-id>";
    }
}
```

<a id="trigger-registration"></a>
## Trigger GCM Token Registration

After all the configuration, you may also need to trigger the GCM token
registration as early as possible, in terms of your application flow.

You are suggested to trigger the registration at the creation of your
application or the creation of the first activity of your application.

The following code snippet shows how to trigger GCM token registration:

```java
Container skygear = Container.defaultContainer(this);
if (skygear.getGcmSenderId() != null) {
    Intent gcmTokenRegisterIntent = new Intent(this, RegistrationIntentService.class);
    this.startService(gcmTokenRegisterIntent);
}
```

<a id="override-handling"></a>
## Override Notification Handling

Skygear SDK has a default handling for GCM notifications, which will show the
`title` and the `body` under the notification payload.

You may also handle the notification by yourself. To do that, you may update
the `AndroidManifest.xml` and declaring your `ListenerService`:

```html
<?xml version="1.0" encoding="utf-8"?>
<manifest ...>
    <application ...>
        <!-- Your Activity Declarations -->
        ...

        <!-- Other GCM Services -->
        ...

        <service
            android:name="your.package.gcm.MyListenerService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            </intent-filter>
        </service>
    </application>
</manifest>
```

And you need to extend `com.google.android.gms.gcm.GcmListenerService` to
handle the GCM notification:

```java
public class MyListenerService extends com.google.android.gms.gcm.GcmListenerService {
    @Override
    public void onMessageReceived(String s, Bundle bundle) {
        super.onMessageReceived(s, bundle);

        /* Handle the GCM notification */
    }
}

```
[firebase-console]: https://console.firebase.google.com
