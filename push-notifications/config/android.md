---
title: Setting up FCM for Android Push Notification
---

[[toc]]

You can send Push Notification to your user's devices via [Skygear Push API][doc-push-basic-android] or CMS Push Manager. Skygear uses the Firebase Cloud Messaging(FCM) service to send push messages to client Android applications.

To enable sending Push Notifications through FCM, you need:

* Install Skygear Android SDK on your app. [Learn how to install SDK][doc-android-quickstart] if you haven't.
* Setup credentials for FCM in [Skygear Portal][skygear-portal]

![Overview of Skygear Push Notifications][push-overview-fcm]

## Setup Firebase Project

In the dashboard of [Firebase Console][firebase-console], click on Create New project. 

![Create a new Firebase project][fcm-new-project]

After creating a project, go to Project Settings.

![Go to project Settings of your project][fcm-firebase-settings]

## Obtain server credential from Google Firebase Console

There are TWO ways setting up FCM:

1. Use service account (Firebase Admin SDK)
1. Use server key (FCM legacy HTTP API)

::: note

Using service account is available in Skygear Server v1.6.4 or later. We
recommend using service account approach which use the updated HTTP v1 API. 

:::

### Option 1: Using Service Account (v1.6.4 onward)

#### Download service account private key

In the Service accounts tab, click **Generate new private key** to download
the service account private key JSON file.

![Download service account private key for Firebase Admin SDK][fcm-service-account]

#### Upload service account private key in Skygear Portal

Go to [Skygear Portal][skygear-portal] at *App > Push Notification > Firebase Cloud Messaging (FCM)*, turn **ON** *Enable Firebase Cloud Messaging* and turn **OFF** *Use Legacy FCM Server API*. Upload the service account file and click *Save Changes*.

![Upload service account file in Skygear Portal][fcm-upload-service-account-key]

### Option 2: Use Server Key (All version)

In the Cloud Messaging tab, you will find your server keys in place.

![Find your server Keys in the Cloud Messaging tab][fcm-project-settings]

You can copy existing and create new Server Keys under the Project credentials section. 

#### Filling in the Server Key in Skygear Portal

Go to [Skygear Portal][skygear-portal] at *App > Push Notification > Firebase Cloud Messaging (FCM)*, turn **ON** *Enable Firebase Cloud Messaging* and turn **ON** *Use Legacy FCM Server API*. Paste the server key and click *Save Changes*.

![Fill in the Server in Skygear Portal][fcm-fill-in-server-key]

::: note

To start sending message via FCM, remember to turn on `Enable Firebase Cloud Messaging`.

:::

[doc-push-basic-android]: /guides/push-notifications/basics/android/
[doc-android-quickstart]: /guides/intro/quickstart/android/
[fcm-doc]: https://firebase.google.com/docs/cloud-messaging/
[fcm-fill-in-server-key]:/assets/push-notifications/fcm-fill-in-server-key.png
[fcm-upload-service-account-key]:/assets/push-notifications/fcm-upload-service-account-key.png
[fcm-firebase-settings]:/assets/push-notifications/fcm-firebase-settings.png
[fcm-new-project]:/assets/push-notifications/fcm-new-project.png
[fcm-project-settings]:/assets/push-notifications/fcm-project-settings.png
[fcm-service-account]:/assets/push-notifications/fcm-service-account.png
[push-overview-fcm]:/assets/push-notifications/push-overview-fcm.png
[firebase-console]: https://console.firebase.google.com/
[skygear-portal]: https://portal.skygear.io
