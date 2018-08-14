---
title: Email Verification
description: Verify user email
---

[[toc]]

Skygear allows you to verify your user's email through email verification. To
setup: 

1. Go to [Skygear Portal](https://portal.skygear.io/).
1. On the left menu, click *User Auth*, then click *User Verification*.
1. Complete *General Settings* for user verification and save.
1. Enable email verification in *Email Verification* tab, complete the
settings and save.

::: note

User verification is available to Skygear Server 1.6.1-6 or later. If you
cannot find User verification in the left menu, upgrade your app.

:::


## General Settings

When email verification is enabled, skygear will send verification email to user
record `email` field. After user verify their email, the `email_verified`
field of their user record will turn into `true`. Skygear use `is_verified`
field in user record to identify if a user is verified, `is_verified` will be
updated based on your setup in general settings.

![Email verification general settings screen](/assets/user-verification/general-settings.png)

There are three settings in general settings:

- **Mark user as verified** - Skygear used `is_verified` field in user record
to identify if a user is verified. This setting help you to update `is_verified`
in different scenarios.

- **Only verified users can make user required API calls** - If turns this on,
only users whose `is_verified` is `true` can call the user required API. e.g.
Record save, user required lambda, etc.

- **Send verification to user when**
    - **User sign up** - Send verification email when user sign up.
    - **User change email or phone number** - Send verification email when
      user's email is changed.
    - You can also trigger sending verification email in your application by
      calling `requestVerification('email')` API.

## Email Verification Settings

Enable email verification in **Email Verification Settings**.

You can customize the verification email content, success and error handling here.

![Email verification settings screen](/assets/user-verification/email-verification-screenshot.png)

- **Verification Code format**
    - **Numeric (6 digits number)** - Suitable for user to input verification
      code in your application. You can verify the code by calling
      `verifyUserWithCode` API.
    - **Complex (Random string with number and letters)** - Suitable for
      verification link

- **Verification code expiry** - Verification code will be expired in the
  given hours


## Verify via verification link

#### Verification success page

After user click the verification link in the email and verify successfully. An
successful page will be shown and you can customize the page HTML here.
To redirect user to another page instead of showing the successful page, please
select **Redirect URL** and input the URL.

![Verification Success Page](/assets/user-verification/success-page-screenshot.png)


#### Verification error page

Similarly, if user fail to verify through the verification link. An error page
will be shown, you can customize the page HTML in **Verification Error Page**.
To redirect user to another page instead of showing the error page, please select
**Redirect URL** and input the URL.

![Verification Error Page](/assets/user-verification/error-page-screenshot.png)


## Request verification email API

You can request verification email in your application through API.

```java
Container skygear = Container.defaultContainer(this);
skygear.getAuth().requestVerification("email", new LambdaResponseHandler() {
    @Override
    public void onLambdaSuccess(JSONObject result) {
        Log.i("User verification", "you should receive verification code soon");
    }

    @Override
    public void onLambdaFail(Error error) {
        Log.e("User verification", "fail with reason: " + error.getLocalizedMessage());
    }
});
```

```kotlin
val skygear = Container.defaultContainer(this)
skygear.auth.requestVerification("email", object: LambdaResponseHandler() {
    override fun onLambdaSuccess(result: JSONObject?) {
        Log.i("User verification", "you should receive verification code soon")
    }

    override fun onLambdaFail(error: Error?) {
        Log.e("User verification", "fail with reason: " + error?.localizedMessage)
    }
})
```

## Verify email API

Instead of using verification link, you can do the verification in your
application through verify API with the verification code.

```java
Container skygear = Container.defaultContainer(this);
skygear.getAuth().verifyUserWithCode(code, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(Record user) {
        Log.i("User verification", "verify successfully");
        Log.i("User verification", "user id: " + user.getId());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.e("User verification", "fail with reason: " + error?.localizedMessage)
    }
});
```

```kotlin
val code: String = "User input verification code"
skygear.auth.verifyUserWithCode(code, object: AuthResponseHandler() {
    override fun onAuthSuccess(user: Record?) {
        Log.i("User verification", "verify successfully")
        Log.i("User verification", "user id:" + user?.id)
    }

    override fun onAuthFail(error: Error?) {
        Log.e("User verification", "fail with reason: " + error?.localizedMessage)
    }
})
```
