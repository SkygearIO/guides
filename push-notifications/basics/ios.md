---
title: Skygear Push Basics
---

You need to register the devices before sending them push notifications.

<a id="registering-device"></a>
## Registering device

It is suggested that you register a device with remote server on every launch.
Since the container remembers the device ID when the device is registered
on the remote server, it will reuse an existing device ID whenever you try
to register the current device.

You should also request a remote notification token at some point in the
program. If it is appropriate to ask the user for permission for remote
notification, you can also do so when the application launches.

```obj-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [[SKYContainer defaultContainer] registerDeviceCompletionHandler:^(NSString *deviceID, NSError *error) {
        if (error) {
            NSLog(@"Failed to register device: %@", error);
            return;
        }
    
        // Anything you want to do in the callback can be added here
    }];

    // This will prompt the user for permission to send remote notification
    [application registerForRemoteNotifications];

    // Other application initialization logic here
}
```

When a device token is registered with Apple Push Notification Service, you
should register the device once again by updating the registration
with a device token.

```obj-c
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSLog(@"Registered for Push notifications with token: %@", deviceToken);
    [[SKYContainer defaultContainer] registerRemoteNotificationDeviceToken:deviceToken completionHandler:^(NSString *deviceID, NSError *error) {
        if (error) {
            NSLog(@"Failed to register device token: %@", error);
            return;
        }

        // Anything you want to do in the callback can be added here
    }];
}
```

When the device is registered on the remote server, a device ID will be
available to the client application. If you register the current device
using the above convenient methods, the device ID is returned from
`registeredDeviceID` property on the container.

## Send push notification from cloud code

To send push notifications through cloud code, please refer to the
[Cloud Code Guide: Push Notifications][doc-cloud-function-push-notifications].


## Sending push notification to users

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

## Sending push notification to devices

```
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

### TODO: Problems with the interface

1. There are no exported interfaces for SDK to fetch device ids. Sending to device
   is basically unusable.
2. Cannot really think of a valid use case to send notification from one device
   to another device. Sending to devices is more of a plugin thing: e.g. send
   notification to devices that have an outdated version of app.
3. The first array argument of `sendCompletionHandler` is always of `NSString`
   In the case of sending to users, it would be expected to be an array of
   `SKYUserRecordID`.

[doc-cloud-function-push-notifications]: /guide/cloud-function/calling-skygear-api/python/#push-notifications
