---
title: User Authentication Basics
description: User sign up, login, logout / access token / username, email and password management in iOS
---

[[toc]]


## Manage User

### Signing up

The first thing you want a user to do is to sign up. The following code illustrates the sign up process:

```obj-c
[[SKYContainer defaultContainer] signupWithUsername:@"john.doe"
                     password:@"verysecurepasswd"
            completionHandler:^(SKYUser *user, NSError *error) {
    if (error) {
        NSLog(@"error signing up user: %@", error);
        return;
    }

    NSLog(@"sign up successful");
    // do something else
}];
```

```swift
SKYContainer.default().signup(withUsername: "john.doe", password: "verysecurepasswd") { (user, error) in
    if error != nil {
        print ("error signing up user: \(error)")
        return
    }
    
    print ("sign up successful")
    // do something else
}
```

You can use `signupWithEmail:password:` to sign up users with emails as well:

```obj-c
[[SKYContainer defaultContainer] signupWithEmail:@"john.doe@example.com"
                  password:@"verysecurepasswd"
         completionHandler:^(SKYUserRecordID *user, NSError *error) {
    if (error) {
        NSLog(@"error signing up user: %@", error);
        return;
    }

    NSLog(@"sign up successful");
    // do something else
}];
```

```swift
SKYContainer.default().signup(withEmail: "john.doe@example.com", password: "verysecurepasswd") { (user, error) in
    if error != nil {
        print ("error signing up user: \(error)")
        return
    }
    
    print ("sign up successful")
    // do something else
}
```

These two methods will create a new user in your app. Before completing the process, it automatically checks if the username is unique. New users should always be created with either one of the methods.

When a new user is successfully created, a `SKYUser` is returned.

If the signup is not successful, you should read the error object returned. Usually it is just that the username or the email has been taken. You should communicate this to your users and ask them to try a different username. They may also be other kinds of errors that you should take note of.

### Logging in

You can use `loginWithUsername:password:` to let users log in to their accounts by:

```obj-c
[[SKYContainer defaultContainer] loginWithUsername:@"john.doe"
                    password:@"verysecurepasswd"
           completionHandler:^(SKYUserRecordID *user, NSError *error) {
    if (error) {
        NSLog(@"error loggin user in: %@", error);
        return;
    }

    NSLog(@"login successful");
    // do something else
}];
```

```swift
SKYContainer.default().login(withUsername: "john.doe", password: "verysecurepasswd") { (user, error) in
    if error != nil {
        print ("error loggin user in \(error)")
        return
    }
    
    print ("login successful")
    // do something else
}
```

You can also use `loginWithEmail:password:` to log in users with emails:

```obj-c
[[SKYContainer defaultContainer] loginWithEmail:@"john.doe@example.com"
                 password:@"verysecurepasswd"
        completionHandler:^(SKYUserRecordID *user, NSError *error) {
    if (error) {
        NSLog(@"error loggin user in: %@", error);
        return;
    }

    NSLog(@"login successful");
    // do something else
}];
```

```swift
SKYContainer.default().login(withEmail: "john.doe@example.com", password: "verysecurepasswd") { (user, error) in
    if error != nil {
        print ("error loggin user in \(error)")
        return
    }
    
    print ("login successful")
    // do something else
}
```

### Logging out

Of course, you also would want to allow users to log out of their accounts by using `logoutWithCompletionHandler:`.

```obj-c
[[SKYContainer defaultContainer] logoutWithCompletionHandler:^(SKYUser *user, NSError *error) {
    if (error != nil) {
        NSLog(@"Logout error: %@",error);
        return
    }
    NSLog(@"Logout success");
}];
```

```swift
SKYContainer.default().logout { (user, error) in
    if error != nil {
    	print ("Logout error: \(error)")
    	return
    }
    print("Logout success")
}
```

### Changing password

To change the password of the current user:

```obj-c
[[SKYContainer defaultContainer] setNewPassword:@"newPassword" oldPassword:@"oldPassword" completionHandler:^(SKYUser *user, NSError *error) {
    if (error != nil) {
        NSLog(@"can't change password: %@", error);
        // Can be old password not matched?
        return;
    }
    
    NSLog(@"password change successfully");
}];
```

```swift
SKYContainer.default().setNewPassword("newPassword", oldPassword: "oldPassword") { (user, error) in
    if error != nil {
        print ("can't change password: \(error)")
        // Can be old password not matched?
        return
    }
    
    print ("password changed successfully")
}
```



## Current User

### Getting current user id

You can get the `currentUserRecordID` by:

```obj-c
if ([SKYContainer defaultContainer].currentUser != nil) {
    // user is logged in, proceed
    NSString *userID = [[SKYContainer defaultContainer] currentUserRecordID];
} else {
    // ask user to sign up / login
}
```

```swift
if SKYContainer.default().currentUser != nil {
    // user is logged in, proceed
    let userID = SKYContainer.default().currentUserRecordID
} else {
    // ask user to sign up / login
}
```

Please be reminded that the `currentUser` object persist locally, and the
information (e.g. roles, emails, etc) might not sync with the server if it was
changed remotely.

To get the latest information of the current user, you can call `getWhoAmI()`.

```obj-c
[[SKYContainer defaultContainer] getWhoAmIWithCompletionHandler:^(SKYUser *user, NSError *error) {
    if (error) {
        // Error handling...
    } else {
        NSLog(@"Oh. I am %@.", user.username);
    }
}];
```

```swift
SKYContainer.default().getWhoAmI { (user, error) in
    if error != nil {
    	// Error handling...
    } else {
    	print ("Oh. I am \(user.username)")
    }
}
```

### Looking up users by email

Skygear provide a user discovery method by email. Everyone has access to this method without even having to be logged in.

```obj-c
SKYQueryOperation *operation = [SKYQueryOperation queryUsersOperationByEmails:@[@"john.doe@example.com", @"jane.doe@example.com"]];
operation.queryRecordsCompletionBlock = ^(NSArray *users, SKYQueryCursor *queryCursor, NSError *operationError) {
    for (SKYUser *user in users) {
        // do something with the user
    }
}
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
let operation = SKYQueryOperation.queryUsersOperation(byEmails: ["john.doe@example.com", "jane.doe@example.com"])
operation?.queryRecordsCompletionBlock = { (users, queryCursor, error) in
    for user in users as! [SKYUser] {
        // do something with the user
    }
}
operation?.container = SKYContainer.default()
SKYContainer.default().add(operation)
```
