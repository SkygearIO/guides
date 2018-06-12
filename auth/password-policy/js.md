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

``` javascript
import skygear from 'skygear';
skygear.auth.signupWithUsername('john.doe', 'weakpassword').then((user) => {
  console.log(user); // user record
  console.log(user["username"]); // username of the user
}, (error) => {
  console.error(error);
  if (error.error.code === skygear.ErrorCodes.PasswordPolicyViolated) {
    // password does not meet password policy requirements
  } else {
    // other kinds of error
  }
});
```
