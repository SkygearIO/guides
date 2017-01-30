---
title: Query Subscriptions
---

<a name="subscription"></a>
## Subscribe to Query Change

Creating a subscription requires your application to register the current device
on remote server. This is required for subscription to notify the client that
a subscription is triggered.

When a subscription is triggered, remote server notifies the client through
the publish-subscribe (pubsub) mechanism). It is also recommended that your
application requests a remote notification through the
`-registerForRemoteNotifications`. When a device token is available for
a device, the remote server also send a remote notification through Apple Push
Notification Service.

Having registered a device, your application should create a subscription by
specifying a query. The container will associate the device to the subscription
when you call the `-saveSubscription:completionHandler:` on the database.

### Registering device

Please refer to [Registering device][doc-registering-device] section to register the device first.

After you have registered device, you can then create a subscription.

In the register device callback, you should then add the subscriptions.

```obj-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [[SKYContainer defaultContainer] registerDeviceCompletionHandler:^(NSString *deviceID, NSError *error) {
        if (error) {
            NSLog(@"Failed to register device: %@", error);
            return;
        }
    
        // You should put subscription creation logic in the following method
        [self addSubscriptions];
    }];

    // This will prompt the user for permission to send remote notification
    [application registerForRemoteNotifications];

    // Other application initialization logic here
}
```
```swift
func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    SKYContainer.default().registerDeviceCompletionHandler { (deviceID, error) in
        if error != nil {
            print ("Failed to register device: \(error)")
            return
        }
        
        // You should put subscription creationg logic in the following method
        addSubscriptions()
    }
    
    // This will prompt the user for permission to send remote notification
    application.registerForRemoteNotifications()
    
    // Other application initialization logic here
}
```
#### Register for Push Notifications
```obj-c
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSLog(@"Registered for Push notifications with token: %@", deviceToken.description);
    [[SKYContainer defaultContainer] registerRemoteNotificationDeviceToken:deviceToken completionHandler:^(NSString *deviceID, NSError *error) {
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
    SKYContainer.default().registerRemoteNotificationDeviceToken(deviceToken) { (deviceID, error) in
        if error != nil {
            print ("Failed to register device token: \(error)")
            return
        }
        
        // You should put subscription creation logic in the following method
        addSubscriptions()
    }
}
```


### Adding subscription

```obj-c
SKYQuery *query = [[SKYQuery alloc] initWithRecordType:@"note" predicate:nil];
SKYSubscription *subscription =
[[SKYSubscription alloc] initWithQuery:query subscriptionID:@"my notes"];
[[[SKYContainer defaultContainer] privateCloudDatabase] saveSubscription:subscription
                                                       completionHandler:^(SKYSubscription *subscription, NSError *error) {
    if (error) {
        NSLog(@"Failed to subscribe for my note: %@", error);
        return;
    }

    NSLog(@"Subscription successful.");
}];
```

```swift
let query = SKYQuery(recordType: "note", predicate: nil)
let subscription = SKYSubscription(query: query, subscriptionID: "my notes")
SKYContainer.default().privateCloudDatabase.save(subscription) { (subscription, error) in
    
    if error != nil {
        print ("Failed to subscribe for my note: \(error)")
        return
    }
    
    print ("Subscription successful.")
}
```

### Implementing `SKYContainerDelegate` to receive notification

In Objective-C, add protocol declaration in `AppDelegate`.

```obj-c
@interface AppDelegate : UIResponder <UIApplicationDelegate, SKYContainerDelegate>
```

In Swift, you should add an extention to the end of `AppDelegate`.

```swift
extension AppDelegate: SKYContainerDelegate {
    
}
```

**Note: An complier error "AppDelegate does not conform to protocol SKYContainerDelegate" may occur. Just ignore it. The error will be solved after adding the delegate method.**

Implement the delegate method

```obj-c
- (void)container:(SKYContainer *)container didReceiveNotification:(SKYNotification *)notification
{
    NSLog(@"received notification = %@", notification);
    // do more with the notification (not implemented)
}
```

```swift
func container(_ container: SKYContainer!, didReceive notification: SKYNotification!) {
    print ("received notification = \(notification)")
    // do more with the notification (not implemented)
}
```

Set `AppDelegate` as container's delegate

```obj-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [SKYContainer defaultContainer].delegate = self;
    // ...
}
```

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    SKYContainer.default().delegate = self
    // ...
}
```

[doc-registering-device]: /guides/push-notifications/basics/ios/#registering-device
