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

```java
skygear.getAuth().signupWithUsername("john.doe", "weakpassword", new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
    String accessToken = skygear.getAuth().getCurrentAccessToken();
    Log.i("Skygear Signup", "onAuthSuccess: Got token: " + accessToken);
  }

  @Override
  public void onAuthFail(Error error) {
    if (error.getCode() == Error.Code.PASSWORD_POLICY_VIOLATED) {
      Log.e("Skygear Signup", "Password policy violated");
    } else {
      Log.e("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
  }
});
```
```kotlin
skygear.auth.signupWithUsername("john.doe", "weakpassword", object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    val accessToken = skygear.auth.currentAccessToken
    Log.i("Skygear Signup", "onAuthSuccess: Got token: $accessToken")
  }

  override fun onAuthFail(error: Error) {
    when (error.code) {
      Error.Code.PASSWORD_POLICY_VIOLATED -> Log.e("Skygear Signup", "Password Policy Violated")
      else -> Log.e("Skygear Signup", "onAuthFail: Fail with reason: ${error.message}")
    }
  }
})
```
