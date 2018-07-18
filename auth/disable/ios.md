---
title: Disabling User Accounts
description: Learn how to disable and enable user account, and set an auto-enable period
---

[[toc]]

## Disabling User Accounts

It is possible to disable a user account so that the user is not able to access
resources on the server. For example, a user who has violated your service
agreement can be banned from accessing services provided by your Skygear Server.

When disabled, a user cannot log in to their account. Disabled user cannot
request Skygear API as doing so will return an error instead.

You can disable or enable a user account by calling a Skygear API. You can
create a mobile or web app so this functionality is available to admins.

User account can be disabled indefinitely or can be automatically enabled
after a certain period. When disabling a user account, you can also set
a message to be shown to the user so that the user knows why their account is
disabled.

### Disable a user

To disable a user, you need the User ID of the user to be disabled.
You can disable a user by calling
`adminDisableUserWithUserID:message:expiry:completion:`:

```obj-c
NSString *userID = @"923b3d99-f21f-4e7c-a383-ffa319fe04c7";
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] adminDisableUserWithUserID:userID
                                message:nil
                                 expiry:nil
                             completion:^(NSString *userID, NSError *error) {
                                 if (error) {
                                    NSLog(@"error disabling user: %@", error);
                                 }
                                 NSLog(@"user is disabled");
                             }];
```

```swift
let userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7"
SKYContainer.default().auth.adminDisableUser(withUserID: userID, message: nil, expiry: nil) { (userID, error) in
    if error != nil {
        print("error disabling user: \(error)")
        return
    }

    print("user id disabled")
}
```

### Enable a disabled user account

You can enable a disabled user account by calling
`adminEnableUserWithUserID:completion:`:

```obj-c
NSString *userID = @"923b3d99-f21f-4e7c-a383-ffa319fe04c7";
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] adminEnableUserWithUserID:userID
                                 completion:^(NSString *userID, NSError *error) {
                                     if (error) {
                                        NSLog(@"error enabling user: %@", error);
                                     }
                                     NSLog(@"user is enabled");
                                 }];
```

```swift
let userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7"
SKYContainer.default().auth.adminEnableUser(withUserID: userID) { (userID, error) in
    if error != nil {
        print("error enabling user: \(error)")
        return
    }

    print("user id enabled")
}
```

### Disable a user with a message and expiry period.

You can disable a user and attach a login message to the user account. The
login message can be shown to the user when the user logs in.

You can also set an expiry period. After the expiry period, the user will be
automatically enabled.

```obj-c
NSString *userID = @"923b3d99-f21f-4e7c-a383-ffa319fe04c7";
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] adminDisableUserWithUserID:userID
                                message:@"Account disabled due to TOS violations."
                                 expiry:[[NSDate date] dateByAddingTimeInterval:60*5];
                             completion:^(NSString *userID, NSError *error) {
                                 if (error) {
                                    NSLog(@"error disabling user: %@", error);
                                 }
                                 NSLog(@"user is disabled");
                             }];
```

```swift
let userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7"
SKYContainer.default().auth.adminDisableUser(withUserID: userID,
                                             message: "Account disabled due to TOS violations.",
                                             expiry: NSDate().addingTimeInterval(5*60)) { (userID, error) in
    if error != nil {
        print("error disabling user: \(error)")
        return
    }

    print("user id disabled")
}
```

### Tell user that their account is disabled

When a user is disabled, server may return `UserDisabled` error. This will only
happen if the server action requires an authenticated user. To reliably check if
the user is disabled, call
[`getWhoAmIWithCompletionHandler:`].

To find out how to handle Skygear Error, check out [Error Handling in SDK].

The `localizedDescription` property of the returned error will explain to the
user that their account is disabled. If you want to handle the error
differently, you can get the disable message from error info dictionary.

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] getWhoAmIWithCompletionHandler:^(SKYRecord *user, NSError *error) {
    if (error.code == SKYErrorUserDisabled) {
        // the user is disabled
        NSLog(@"disable message: %@", error.info[@"message"]);
        return;
    } else if (error != nil {
        // other error has occurred
        return;
    }

    // the user is not disabled
}];
```

```swift
SKYContainer.default().auth.getWhoAmI { (user, error) in
    if let error = error as? SKYError {
        if error.code == SKYError.userDisabled {
            // the user is disabled
            print("disable message: \(error.userInfo["message"]\)")
        }
        return
    }

    // the user is not disabled
}
```

[Error Handling in SDK]: /guides/advanced/sdk-error-handling/ios/
[`getWhoAmIWithCompletionHandler:`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYAuthContainer.html#/c:objc(cs)SKYAuthContainer(im)getWhoAmIWithCompletionHandler:
