---
title: Password Policy
description: Define password policy to enhance computer security with stronger passwords.
---

[[toc]]

You can define a password policy so you can encourage your users to employ
stronger passwords. You can define a password policy by going to Skygear
Developer Portal:

1. Go to [Skygear Portal](https://portal.skygear.io/).
2. Login to your account.
3. Click your app in *My Apps* list.
4. On the left menu, click *User Auth*, then click *Password Policy*.
5. Make changes to password policy.
6. Click *Save Changes*.

![Password policy screen](/assets/common/password-policy-screen.png)

::: note

Password Policy is available to Skygear Server 1.5 or later. If you cannot find
Password Policy in the left menu, upgrade your app.

:::

## Handling password policy violations

You do not need to write code to enforce password policy, but it is a good idea
to let your user know that their chosen password does not meet the password
requirement.

In your app, it is recommended you check for password policy violation error
when signing up a user or when changing user password.

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] signupWithUsername:@"john.doe"
                            password:@"weakpassword"
                   completionHandler:^(SKYRecord *user, NSError *error) {
    if (error && error.code == SKYErrorPasswordPolicyViolated) {
        NSLog(@"password policy violated: %@", error);
        NSLog(error.localizedDescription); // Prints user-friendly error message
        return;
    } else if (error) {
        NSLog(@"other error occurred: %@", error);
        return;
    }

    NSLog(@"sign up successful");
    NSLog(@"user record: %@", user);
    NSLog(@"username: %@", user[@"username"]);
}];
```

```swift
SKYContainer.default().auth.signup(withUsername: "john.doe", password: "weakpassword") { (user, error) in
    if error != nil && error.code == SKYErrorPasswordPolicyViolated {
        print("password policy violated: \(error)")
        print(error.localizedDescription) // Prints user-friendly error message
        return
    } else if (error != nil) {
        print("other error occurred: \(error)")
        return
    }

    print("sign up successful")
    print("user record: \(user)")
    print("username: \(user!["username"])")
    // do something else
}
```
