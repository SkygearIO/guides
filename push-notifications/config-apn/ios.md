---
title: Setting up APN for iOS Push Notification
---

[[toc]]

You can send Push Notification to your user's devices via Skygear Push Notification. Skygear uses the Apple Push Notification(APN) interface to send push messages to client iOS applications.

To enable sending Push Notifications through APN, you need to:

* Have Skygear iOS SDK installed. [Learn how to install SDK][doc-ios-quickstart] if you haven't.
* Set up Apple Push Notification Authentications in [Skygear Portal][skygear-portal]
* Have a provisioning profile for the App ID of your testing app

![Overview of Skygear Push Notifications][push-overview-apn]

There are TWO ways of setting up Apple Push Notification Authentications:

1. Use Token-base Auth
2. Upload Push Certificate and Key

These credentials can be obtained from [Apple Developer Member Center](apple-developer-member-center). 

::: note

We recommend Token Base Authentication which involves less steps. However We will guide you through both ways.

:::

## Option 1: Using Token-base Auth

Skygear Push Notification can use Apple Push Notification Authentication Key to send Push Notifications to the application identified by the App ID.

### Obtaining an Auth Key and Team ID

You may find the **Team ID** at [membership](https://developer.apple.com/account/#/membership/) area at member center.

![View your team ID][apn-view-team-id]

Then you can obtain an **Apple Push Notification Authentication Key** for your [Apple Developer account][apple-developer-account].

![View you Auth Keys][apn-view-auth-key]

### Setting the credentials on Skygear Portal

Firstly, Enable APN in [Skygear Portal][skygear-portal]: *App > Push Notification > Apple Push Notification (APN)*

::: tips

Please turn on the **Use Token-based Apple Push Notification** option to enable using Auth Key.

:::

Now, specify the Team ID, Auth Key ID and upload the Auth Key file in the corresponding fields.

![Filling in Auth Key on Skygear Portal][apn-fill-in-auth-key]

Click on Update button to apply changes.

::: note

To send message to apps that are already on App Store you need to turn on `Send Push Notification in Production Environment`.

:::

## Option 2: Uploading Push Certificate and Key

You will need to upload the followings to enable APN:

1. Push Certificate
2. Private Key for the Certificate

### Obtaining the Apple Push Notification Service certificate

You can request an Apple Push Notification Service certificate from the [Apple Developer Member Center][apple-developer-member-center].

Skygear requires the certificate and private key files in *PEM format*. Please follow [this guide][generating-pem-guide] in detail to generate PEM files for your key.

### Setting the credentials on Skygear Portal

Firstly, Enable APN in [Skygear Portal][skygear-portal]: *App > Push Notification > Apple Push Notification (APN)*

Then, upload the Apple Push Notification Service certificate and private key file. 

![Upload Push Certificate to Skygear Portal][apn-upload-cert]

## Provisioning profile for Apps under Development

To test your Apps under development which is not on App Store yet, you will need a Provisioning Profile for the specific App ID.

1. Go to the [Apple Developer Member Center][apple-developer-member-center].
2. Go Certificates, Identifiers and Profiles and select *Provisioning Profiles > All*.
3. Click the + button to create a new Provisioning Profile.
4. Select iOS App Development as provisioning profile type, then click Continue.
5. In the drop down menu, select the App ID you want to use, then click Continue.
6. Select the iOS Development certificate of the App ID you have chosen in the previous step, then click Continue.
7. Select the iOS devices that you want to include in the Provisioning Profile, then click Continue. Make sure to select all the devices you want to use for your testing.
8. Input a name for this provisioning profile, then click Generate.
9. Download and open the Provisioning Profile file to install it on your Mac.

[apn-download-authkey]:/assets/push-notifications/apn-download-authkey.png[apn-fill-in-auth-key]:/assets/push-notifications/apn-fill-in-auth-key.png[apn-upload-cert]:/assets/push-notifications/apn-upload-cert.png[apn-view-auth-key]:/assets/push-notifications/apn-view-auth-key.png[apn-view-team-id]:/assets/push-notifications/apn-view-team-id.png
[apple-developer-account]: https://developer.apple.com/account.
[apple-developer-member-center]: https://developer.apple.com/membercenter/index.action
[doc-ios-quickstart]: /guides/intro/quickstart/ios/
[generating-pem-guide]: https://blog.krishan711.com/generating-ios-push-certificates
[push-overview-apn]:/assets/push-notifications/push-overview-apn.png
[skygear-portal]: https://portal.skygear.io

