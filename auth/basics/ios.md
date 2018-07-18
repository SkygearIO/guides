---
title: User Authentication Basics
description: User sign up, login, logout / access token / username, email and password management in iOS
---

[[toc]]

## Signing up

### Signing up with email or username

A user can sign up using a username or an email, along with a password.
It is done using either [`signupWithUsername:password:completionHandler:`] or
[`signupWithEmail:password:completionHandler:`].

Skygear does not allow duplicated usernames or emails. Signing up with a
duplicated identifier will give the error `Duplicated`.

While each of the sign-up functions is resolved with a user record,
in most cases you need not deal with it because
you can access the currently logged-in user using [`currentUser`].

[`signupWithUsername:password:completionHandler:`] sample code:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] signupWithUsername:@"john.doe"
                            password:@"verysecurepasswd"
                   completionHandler:^(SKYRecord *user, NSError *error) {
    if (error) {
        NSLog(@"error signing up user: %@", error);
        return;
    }

    NSLog(@"sign up successful");
    NSLog(@"user record: %@", user);
    NSLog(@"username: %@", user[@"username"]);
}];
```

```swift
SKYContainer.default().auth.signup(withUsername: "john.doe", password: "verysecurepasswd") { (user, error) in
    if error != nil {
        print("error signing up user: \(error)")
        return
    }

    print("sign up successful")
    print("user record: \(user)")
    print("username: \(user!["username"])")
    // do something else
}
```

[`signupWithEmail:password:completionHandler:`] sample code:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] signupWithEmail:@"john.doe@example.com"
                  password:@"verysecurepasswd"
         completionHandler:^(SKYRecordRecordID *user, NSError *error) {
    if (error) {
        NSLog(@"error signing up user: \(error)");
        return;
    }

    NSLog(@"sign up successful");
    NSLog(@"user record: %@", user);
    NSLog(@"email: %@", user[@"email"]);
}];
```

```swift
SKYContainer.default().auth.signup(withEmail: "john.doe@example.com", password: "verysecurepasswd") { (user, error) in
    if error != nil {
        print("error signing up user: \(error)")
        return
    }

    print("sign up successful")
    print("user record: \(user)")
    print("email: \(user!["email"])")
}
```

It is common to add other data to user record when signing up, you can do that
by specifying the data for user record in the third parameter of the signup
functions:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] signupWithEmail:@"john.doe@example.com"
                  password:@"verysecurepasswd"
                  profileDictionary:@{
                      @"interest": @"reading"
                  }
         completionHandler:^(SKYRecordRecordID *user, NSError *error) {
    if (error) {
        NSLog(@"error signing up user: \(error)");
        return;
    }

    NSLog(@"sign up successful");
    NSLog(@"user record: %@", user);
    NSLog(@"interest: %@", user[@"interest"]);
}];
```

```swift
SKYContainer.default().auth.signup(withEmail: "john.doe@example.com",
                                   password: "verysecurepasswd",
                                   profileDictionary: ["interest": "reading"]) { (user, error) in
    if error != nil {
        print("error signing up user: \(error)")
        return
    }

    print("sign up successful")
    print("user record: \(user)")
    print("intereest: \(user!["interest"])")
}
```

## Logging in

The login functions are similar to the sign-up ones.

If the credentials are incorrect, it will give the error of:

- `InvalidCredentials` if the password is incorrect;
- `ResourceNotFound` if the email or username is not found.

While each of the login functions is resolved with a user record,
in most cases you need not deal with it because
you can access the currently logged-in user using [`currentUser`].

### Logging in using a username

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] loginWithUsername:@"john.doe"
                           password:@"verysecurepasswd"
                  completionHandler:^(SKYRecord *user, NSError *error) {
    if (error && (error.code == SKYErrorResourceNotFound || error.code == SKYErrorInvalidCredentials)) {
        // incorrect username or password
        return;
    } else if (error) {
        // other kinds of error
        return;
    }

    NSLog(@"login successful");
}];
```

```swift
SKYContainer.default().auth.login(withUsername: "john.doe", password: "verysecurepasswd") { (user, error) in
    if let error = error as? SKYError {
        if (error.code == SKYError.resourceNotFound || error.code == SKYError.invalidCredentials) {
            // incorrect username or password
            return
        } else {
            // other kinds of error
            return
        }
    }

    print("login successful")
}
```

### Logging in using an email

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] loginWithEmail:@"john.doe"
                           password:@"verysecurepasswd"
                  completionHandler:^(SKYRecord *user, NSError *error) {
    if (error && (error.code == SKYErrorResourceNotFound || error.code == SKYErrorInvalidCredentials)) {
        // incorrect email or password
        return;
    } else if (error) {
        // other kinds of error
        return;
    }

    NSLog(@"login successful");
}];
```

```swift
SKYContainer.default().auth.login(withEmail: "john.doe", password: "verysecurepasswd") { (user, error) in
    if let error = error as? SKYError {
        if (error.code == SKYError.resourceNotFound || error.code == SKYError.invalidCredentials) {
            // incorrect email or password
            return
        } else {
            // other kinds of error
            return
        }
    }

    print("login successful")
}
```

## Logging out

Logging out the current user is simple using the [`logoutWithCompletionHandler:`] method.

Upon successful logout, the SDK will clear the current user and the access token
from the local storage.

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] logoutWithCompletionHandler:^(SKYRecord *user, NSError *error) {
    if (error != nil) {
        NSLog(@"Logout error: %@",error);
        return;
    }
    NSLog(@"Logout success");
}];
```

```swift
SKYContainer.default().auth.logout { (user, error) in
    if error != nil {
    	print("Logout error: \(error)")
    	return
    }
    print("Logout success")
}
```

## Getting the current user

You can retrieve the current user from [`currentUser`].

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
SKYRecord *user = [[container auth] currentUser]; // if not logged in, it will be null
```

```swift
let user = SKYContainer.default().auth.currentUser // if not logged in, it will be null
```

If there is an authenticated user, it will give you a [`SKYRecord`] which
is the user record. The user record looks like this:

```json
{
  '_id': 'abcdef',
  'username': 'Ben',
  'email': 'ben@skygeario.com',
}
```

Please be reminded that the [`currentUser`]
 object persist locally, and the data in the user record
 might not sync with the server if it was changed remotely.

To get the latest information of the current user,
you can call [`getWhoAmIWithCompletionHandler:`]:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] getWhoAmIWithCompletionHandler:^(SKYRecord *user, NSError *error) {
    if (error != nil) {
        // Error handling...
        return;
    }

    NSLog(@"Oh. I am %@", user[@"username"]);
}];
```

```swift
SKYContainer.default().auth.getWhoAmI { (user, error) in
    if error != nil {
        // Error handling...
        return
    }

    print("Oh. I am \(user[@"username"])")
}
```

## Observing user changes

The preferred way for your app to handle any logged-in user change is to
add observer for the [`SKYContainerDidChangeCurrentUserNotification`] notification.
The notification is pushed whenever the user is changed, i.e.
when any of the followings happens:

1. a user logs in;
2. a user logs out (or as logged out due to an expired access token);

The callback will receive the new user record as the argument.

```obj-c
NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
[center addObserverForName:SKYContainerDidChangeCurrentUserNotification
                    object:nil
                     queue:[NSOperationQueue mainQueue]
                usingBlock:^(NSNotification *note) {
                    SKYContainer *container = [SKYContainer defaultContainer];
                    SKYRecord *user = [[container auth] currentUser];
                }];
```

```swift
NotificationCenter.default.addObserver(forName: Notification.Name.SKYContainerDidChangeCurrentUser,
                                       object: nil,
                                       queue: OperationQueue.main) { (note) in
                                        var user = SKYContainer.default().auth.currentUser
}
```

## Updating a user's username and email

Username and email is saved to the user record. You can modify the username
and email in the same way you modify any other record, which is achieved by
calling [`saveRecord:completion:`].

::: caution

**Caution:** When saving user record, the user record returned by
[`currentUser`] is not automatically updated. To force an update of
the record returned from [`currentUser`], call [`getWhoAmIWithCompletionHandler:`].

:::

To change the username of the current user:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
SKYRecord *record = container.auth.currentUser;
record[@"username"] = @"new-username";
[container.publicCloudDatabase saveRecord:record
                               completion:^(SKYRecord *record, NSError *error) {
                                   if (error != nil) {
                                       NSLog(@"Error: %@", error);
                                       return;
                                   }
                                   NSLog(@"%@", record); // updated user record
                                   console.log('Username is changed to: ', record["username"]);
                                   [container.auth getWhoAmIWithCompletionHandler:nil];
                               }];
```

```swift
let container = SKYContainer.default()
let record = container.auth.currentUser
record?.setObject("new-username", forKey: "username" as NSCopying)
container.publicCloudDatabase.save(record!) { (record, error) in
    if error != nil {
        print(error!);
    }
    print(record!);
    print("Username is changed to: \(record?["username"] ?? "")")
    container.auth.getWhoAmI(completionHandler: nil)
}
```

To change the email of the current user:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
SKYRecord *record = container.auth.currentUser;
record[@"email"] = @"new-email@example.com";
[container.publicCloudDatabase saveRecord:record
                               completion:^(SKYRecord *record, NSError *error) {
                                   if (error != nil) {
                                       NSLog(@"Error: %@", error);
                                       return;
                                   }
                                   NSLog(@"%@", record); // updated user record
                                   console.log('Username is changed to: ', record["username"]);
                                   [container.auth getWhoAmIWithCompletionHandler:nil];
                               }];
```

```swift
let container = SKYContainer.default()
let record = container.auth.currentUser
record?.setObject("new-email@example.com", forKey: "email" as NSCopying)
container.publicCloudDatabase.save(record!) { (record, error) in
    if error != nil {
        print(error!);
    }
    print(record!);
    print("Username is changed to: \(record?["username"] ?? "")")
    container.auth.getWhoAmI(completionHandler: nil)
}
```

You can even change the username and email at the same time:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
SKYRecord *record = container.auth.currentUser;
record[@"username"] = @"new-username";
record[@"email"] = @"new-email@example.com";
[container.publicCloudDatabase saveRecord:record
                               completion:^(SKYRecord *record, NSError *error) {
                                   if (error != nil) {
                                       NSLog(@"Error: %@", error);
                                       return;
                                   }
                                   NSLog(@"%@", record); // updated user record
                                   console.log('Username is changed to: ', record["username"]);
                                   [container.auth getWhoAmIWithCompletionHandler:nil];
                               }];
```

```swift
let container = SKYContainer.default()
let record = container.auth.currentUser
record?.setObject("new-username", forKey: "username" as NSCopying)
record?.setObject("new-email@example.com", forKey: "email" as NSCopying)
container.publicCloudDatabase.save(record!) { (record, error) in
    if error != nil {
        print(error!);
    }
    print(record!);
    print("Username is changed to: \(record?["username"] ?? "")")
    container.auth.getWhoAmI(completionHandler: nil)
}
```

## Updating a user's password

The currently logged-in user can change his/her own password.
This can be done using the [`setNewPassword:oldPassword:completionHandler:`] function.

If the current password is incorrect, the SDK will return an
`InvalidCredentials` error.

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] setNewPassword:@"newPassword"
                     oldPassword:@"oldPassword"
               completionHandler:^(SKYRecord *user, NSError *error) {
    if (error != nil) {
        NSLog(@"can't change password: %@", error);
        // Can be old password not matched?
        return;
    }

    NSLog(@"password change successfully");
}];
```

```swift
SKYContainer.default().auth.setNewPassword("newPassword", oldPassword: "oldPassword") { (user, error) in
    if error != nil {
        print("can't change password: \(error)")
        // Can be old password not matched?
        return
    }

    print("password changed successfully")
}
```

Note: Changing the password of the current user will not cause
[`SKYContainerDidChangeCurrentUserNotification`] notification to be posted.

## Forgot password

Coming soon.

## User Verification

Not yet implemented.

## What's next from here?
You may want to learn more about:

* [Social Login using Skygear (Coming soon)](/guides/auth/social-login/ios/)
* [Setting User Profile](/guides/auth/user-profile/ios/)

[`SKYContainerDidChangeCurrentUserNotification`]: https://docs.skygear.io/ios/reference/v1/Constants.html#/c:@SKYContainerDidChangeCurrentUserNotification
[`SKYRecord`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYRecord.html
[`currentUser`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYAuthContainer.html#/c:objc(cs)SKYAuthContainer(py)currentUser
[`getWhoAmIWithCompletionHandler:`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYAuthContainer.html#/c:objc(cs)SKYAuthContainer(im)getWhoAmIWithCompletionHandler:
[`logoutWithCompletionHandler:`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYAuthContainer.html#/c:objc(cs)SKYAuthContainer(im)logoutWithCompletionHandler:
[`saveRecord:completion:`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYDatabase.html#/c:objc(cs)SKYDatabase(im)saveRecord:completion:
[`signupWithEmail:password:completionHandler:`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYAuthContainer.html#/c:objc(cs)SKYAuthContainer(im)signupWithEmail:password:completionHandler:
[`signupWithUsername:password:completionHandler:`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYAuthContainer.html#/c:objc(cs)SKYAuthContainer(im)signupWithUsername:password:completionHandler:
[`setNewPassword:oldPassword:completionHandler:`]: https://docs.skygear.io/ios/reference/v1/Classes/SKYAuthContainer.html#/c:objc(cs)SKYAuthContainer(im)setNewPassword:oldPassword:completionHandler:
