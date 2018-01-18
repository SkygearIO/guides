---
title: Setting up FCM for Android Push Notification
---

[[toc]]

You can send Push Notification to your user's devices via Skygear Push Notification. Skygear uses the Firebase Cloud Messaging(FCM) interface to send push messages to client Android applications.

To enable sending Push Notifications through FCM, you need:

* Have Skygear Android SDK installed. [Learn how to install SDK][doc-android-quickstart] if you haven't.
* Configure the FCM Server Key on Skygear Portal

![Overview of Skygear Push Notifications][push-overview-fcm]

## Obtain the server key from Google Firebase Console

You can obtain the server key from Google Firebase Console.
Open [Firebase Console][firebase-console] and log in using your Google account.

## Create a New Firebase Project

In the dashboard of Firebase Console, click on Create New project. 

![Create a new Firebase project][fcm-new-project]

After creating a project, go to Project Settings.

![Go to project Settings of your project][fcm-firebase-settings]

In the Cloud Messaging tab, you will find your server keys in place.

![Find your server Keys in the Cloud Messaging tab][fcm-project-settings]

You can copy existing and create new Server Keys under the Project credentials section. 

## Filling in the Server Key in Skygear Portal

Copy the Server Key you want to use and fill in [Skygear Portal][skygear-portal] at *App > Push Notification > Firebase Cloud Messaging (FCM)*. Click the Update button to save the key.

![Fill in the Server in Skygear Portal][fcm-fill-in-server-key]

::: note

To start sending message via FCM, remember to turn on `Enable Firebase Cloud Messaging`.

:::


[doc-android-quickstart]: /guides/intro/quickstart/android/
[fcm-doc]: https://firebase.google.com/docs/cloud-messaging/
[fcm-fill-in-server-key]:/assets/push-notifications/fcm-fill-in-server-key.png
[fcm-firebase-settings]:/assets/push-notifications/fcm-firebase-settings.png
[fcm-new-project]:/assets/push-notifications/fcm-new-project.png
[fcm-project-settings]:/assets/push-notifications/fcm-project-settings.png
[firebase-console]: https://console.firebase.google.com/
[push-overview-fcm]:/assets/push-notifications/push-overview-fcm.png
[skygear-portal]: https://portal.skygear.io
