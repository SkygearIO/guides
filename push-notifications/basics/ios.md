---
title: Skygear Push Basics
---

[[toc]]

::: tips

You need to register the devices before sending push notifications to the devices.

:::

There are THREE platform specific steps to register for push notification on iOS to ensure Push Notification working with Skygear.

1. Registering device at Skygear and Apple Push Notification Service
2. Request authorization to display notifications
3. Update your device token on Skygear 

After completing these three steps, you will be able to receive push notification sent via Skygear.

<a id="registering-device"></a>
## Registering device at Apple Push Notification Service

Here's how.

It is suggested that you register a device when your app launches everytime. Since the Skygear container remembers the device ID, it will reuse the existing device ID whenever you try to register the current device more then once.

Now, you are able to request for a remote notification token at some point in your app.

This is an example on how you may register a remote notification when the application launches at `didFinishLaunchingWithOptions`.

```obj-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [[[SKYContainer defaultContainer] push] registerDeviceCompletionHandler:^(NSString *deviceID, NSError *error) {
        if (error) {
            NSLog(@"Failed to register device: %@", error);
            return;
        }

        // Anything you want to do in the callback can be added here
    }];

    // This will prompt the user for permission to send remote notification
    [application registerForRemoteNotifications];

    // Other application initialization logic here

    return YES;
}
```

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    SKYContainer.default().push.registerDeviceCompletionHandler { (deviceID, error) in
        if error != nil {
            print ("Failed to register device: \(error)")
            return
        }

        // Anything you want to do in the callback can be added here
    }

    // This will prompt the user for permission to send remote notification
    application.registerForRemoteNotifications()

    // Other application initialization logic here

    return true
}
```

<a id="request-authorization"></a>
## Request authorization to display notifications

You should make sure your app has the permission to send user push notifications when the app is in the background according to the [Apple's documentation][apple-doc-registerforremotenotifications].

::: caution

If you do not request and receive authorization for your app's interactions, the system delivers all remote notifications to your app silently. It is adviced to ask the user for permission before registering Apple Push Notification Service .

:::

Here's how you can request for notification handler events:

```obj-c

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {


	UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
	[center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert + UNAuthorizationOptionSound)
	   completionHandler:^(BOOL granted, NSError * _Nullable error) {
	      // Enable or disable features based on authorization.
	}];


    [[[SKYContainer defaultContainer] push] registerDeviceCompletionHandler:^(NSString *deviceID, NSError *error) {
        if (error) {
            NSLog(@"Failed to register device: %@", error);
            return;
        }

        // Anything you want to do in the callback can be added here
    }];

    // This will prompt the user for permission to send remote notification
    [application registerForRemoteNotifications];

    // Other application initialization logic here

    return YES;
}



```

```swift
let center = UNUserNotificationCenter.current()
center.requestAuthorization(options: [.alert, .sound]) { (granted, error) in
    // Enable or disable features based on authorization
}

```

## Update your device token on Skygear

When a device token is registered with Apple Push Notification Service, you
should register the device once again by updating the registration
with a device token at `didRegisterForRemoteNotificationsWithDeviceToken `

```obj-c
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSLog(@"Registered for Push notifications with token: %@", deviceToken.description);
    [[[SKYContainer defaultContainer] push] registerRemoteNotificationDeviceToken:deviceToken completionHandler:^(NSString *deviceID, NSError *error) {
        if (error) {
            NSLog(@"Failed to register device token: %@", error);
            return;
        }

        // You should put subscription creation logic in the following method
        [self addSubscriptions];
    }];
}
```

```swift
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    print ("Registered for Push notifications with token: \(deviceToken.description)")
    SKYContainer.default().push.registerRemoteNotificationDeviceToken(deviceToken) { (deviceID, error) in
        if error != nil {
            print ("Failed to register device token: \(error)")
            return
        }

        // You should put subscription creation logic in the following method
        addSubscriptions()
    }
}
```

When the device is registered on the remote server, a device ID will be
available to the client application. If you register the current device
using the above convenient methods, the device ID is returned from
`registeredDeviceID` property on the container.

## Sending push notification to users

Okay, now we are ready to send push notifiactions to the regiestered devices. If you have not registered your device on Skygear, please read the guide on device registeration.

You can send Push Notfication to all devices belongs to your selected users.

```obj-c
// send notification through APNS
SKYAPSNotificationInfo *apsInfo = [SKYAPSNotificationInfo notificationInfo];
apsInfo.alertBody = @"Hi iOS!";

SKYNotificationInfo *info = [SKYNotificationInfo notificationInfo];
info.apsNotificationInfo = apsInfo;

SKYSendPushNotificationOperation *operation = [SKYSendPushNotificationOperation operationWithNotificationInfo:info userIDsToSend:@[kenji, rick]];
operation.sendCompletionHandler = ^(NSArray *userIDs, NSError *error) {
    if (error) {
        NSLog(@"error sending push notification");
        return;
    }

    NSLog(@"Sent %@ notification to 2 users", @(userIDs.count));
};

[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
// send notification through APNS
let apsInfo = SKYAPSNotificationInfo()
apsInfo.alertBody = "Hi iOS!"

let info = SKYNotificationInfo()
info.apsNotificationInfo = apsInfo

let operation = SKYSendPushNotificationOperation(notificationInfo: info, userIDsToSend: [kenji, rick])
operation.sendCompletionHandler = { (userIDs, error) in
    if error != nil {
        print ("error sending push notification")
        return
    }

    print ("Sent \(userIDs?.count) notification to 2 users")
}

SKYContainer.default().add(operation)
```

## Sending push notification to devices

You can send Push Notfications to selected devices by give `deviceId`s.

```obj-c
// send notification through both APNS and GCM
SKYAPSNotificationInfo *apsInfo = [SKYAPSNotificationInfo notificationInfo];
apsInfo.alertBody = @"Hi iOS!";

SKYGCMNotificationInfo *gcmInfo = [SKYGCMNotificationInfo notificationInfo];
gcmInfo.collapseKey = @"hello";
gcmInfo.notification.title = @"Greetings from Skygear";
gcmInfo.notification.body = @"Hi Android!";

SKYNotificationInfo *info = [SKYNotificationInfo notificationInfo];
info.apsNotificationInfo = apsInfo;
info.gcmNotificationInfo = gcmInfo;

SKYSendPushNotificationOperation *operation = [SKYSendPushNotificationOperation operationWithNotificationInfo:info deviceIDsToSend:@[@"device0", @"device1"]];
operation.sendCompletionHandler = ^(NSArray *deviceIDs, NSError *error) {
    if (error) {
        NSLog(@"error sending push notification");
        return;
    }

    NSLog(@"Sent %@ notification to %@ devices", @(deviceIDs.count));
};

[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
// send notification through both APNS and GCM
let apsInfo = SKYAPSNotificationInfo()
apsInfo.alertBody = "Hi iOS!"

let gcmInfo = SKYGCMNotificationInfo()
gcmInfo.collapseKey = "hello"
gcmInfo.notification.title = "Greetings from Skygear"
gcmInfo.notification.body = "Hi Android!"

let info = SKYNotificationInfo()
info.apsNotificationInfo = apsInfo
info.gcmNotificationInfo = gcmInfo

let operation = SKYSendPushNotificationOperation(notificationInfo: info, deviceIDsToSend: ["device0", "device1"])
operation.sendCompletionHandler = { (deviceIDs, error) in
    if error != nil {
        print ("error sending push notification")
        return
    }

    print ("Sent \(deviceIDs?.count) notification to \(deviceIDs?.count) devices")
}
```

## Sending push notification from cloud code

To send push notifications through cloud code, please refer to the
[Cloud Code Guide: Push Notifications][doc-cloud-function-push-notifications].


## Unregistering device

When a device is registered, it is associated with the authenticated user.
Therefore it is necessary to unregister the device when the user is no longer
associated with the device, for example when the user logs out.

```obj-c
[container unregisterDeviceCompletionHandler:^(NSString *deviceID, NSError *error) {
    if (error != nil) {
        NSLog(@"Cannot unregister device");
        return;
    }
    [container logoutWithCompletionHandler:^(SKYUser *user, NSError *error) {
        // handle logout result
    }];
}];
```

```swift
container.unregisterDeviceCompletionHandler({ (deviceID, error) in
    if error != nil {
        print ("Cannot unregister device")
        return
    }
    container.logout(completionHandler: { (user, error) in
        // handle logout result
    })
})
```

<!--

### TODO: Problems with the interface

1. There are no exported interfaces for SDK to fetch device ids. Sending to device
   is basically unusable.
2. Cannot really think of a valid use case to send notification from one device
   to another device. Sending to devices is more of a plugin thing: e.g. send
   notification to devices that have an outdated version of app.
3. The first array argument of `sendCompletionHandler` is always of `NSString`
   In the case of sending to users, it would be expected to be an array of
   `SKYUserRecordID`.

-->

[doc-cloud-function-push-notifications]: /guides/cloud-function/calling-skygear-api/python/#push-notifications
[apple-doc-registerforremotenotifications]:https://developer.apple.com/documentation/uikit/uiapplication/1623078-registerforremotenotifications