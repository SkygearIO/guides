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
`adminDisableUser`:

```java
Container skygear = Container.defaultContainer(this);
String userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7";

skygear.getAuth().adminDisableUser(
    userID,
    new SetDisableUserResponseHandler() {
        @Override
        public void onSetSuccess() {
            Log.i("Skygear User", "user is disabled");
        }

        @Override
        public void onSetFail(Error error) {
            Log.i("Skygear User", "onSetFail: Fail with reason:" + error.getLocalizedMessage());
        }
    }
);
```

```kotlin
val userID: String = "923b3d99-f21f-4e7c-a383-ffa319fe04c7"

skygear.auth.adminDisableUser(userID, object : SetDisableUserResponseHandler() {
    override fun onSetSuccess() {
        Log.i("Skygear User", "user is disabled")
    }

    override fun onSetFail(error: Error) {
        Log.i("Skygear User", "onSetFail: Fail with reason: ${error.getLocalizedMessage()}")
    }
})
```

### Enable a disabled user account

You can enable a disabled user account by calling
`adminEnableUser`:

```java
Container skygear = Container.defaultContainer(this);
String userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7";

skygear.getAuth().adminEnableUser(
    userID,
    new SetDisableUserResponseHandler() {
        @Override
        public void onSetSuccess() {
            Log.i("Skygear User", "user is enabled");
        }

        @Override
        public void onSetFail(Error error) {
            Log.i("Skygear User", "onSetFail: Fail with reason:" + error.getLocalizedMessage());
        }
    }
);
```

```kotlin
val userID: String = "923b3d99-f21f-4e7c-a383-ffa319fe04c7"

skygear.auth.adminEnableUser(userID, object : SetDisableUserResponseHandler() {
    override fun onSetSuccess() {
        Log.i("Skygear User", "user is enabled")
    }

    override fun onSetFail(error: Error) {
        Log.i("Skygear User", "onSetFail: Fail with reason: ${error.getLocalizedMessage()}")
    }
})
```

### Disable a user with a message and expiry period.

You can disable a user and attach a login message to the user account. The
login message can be shown to the user when the user logs in.

You can also set an expiry period. After the expiry period, the user will be
automatically enabled.

```java
Container skygear = Container.defaultContainer(this);
String userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7";

skygear.getAuth().adminDisableUser(
    userID,
    "Account disabled due to TOS violations.",
    new Date(System.currentTimeMillis()+5*60*1000),
    new SetDisableUserResponseHandler() {
        @Override
        public void onSetSuccess() {
            Log.i("Skygear User", "user is disabled");
        }

        @Override
        public void onSetFail(Error error) {
            Log.i("Skygear User", "onSetFail: Fail with reason:" + error.getLocalizedMessage());
        }
    }
);
```

```kotlin
val userID: String = "923b3d99-f21f-4e7c-a383-ffa319fe04c7"

skygear.auth.adminDisableUser(
    userID,
    "Account disabled due to TOS violations.",
    new Date(System.currentTimeMillis()+5*60*1000),
    object : SetDisableUserResponseHandler() {
        override fun onSetSuccess() {
            Log.i("Skygear User", "user is disabled")
        }

        override fun onSetFail(error: Error) {
            Log.i("Skygear User", "onSetFail: Fail with reason: ${error.getLocalizedMessage()}")
        }
})
```

### Tell user that their account is disabled

When a user is disabled, server may return `USER_DISABLED` error. This will only
happen if the server action requires an authenticated user. To reliably check if
the user is disabled, call
[`whoami`].

To find out how to handle Skygear Error, check out [Error Handling in SDK].

The `getLocalizedMessage()` function of the returned error will explain to the
user that their account is disabled. If you want to handle the error
differently, you can get the disable message from error info dictionary.

```java
Container skygear = Container.defaultContainer(this);

skygear.getAuth().whoami(new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(Record user) {
        Log.i("Skygear User", "I am " + user.getUsername());
    }

    @Override
    public void onAuthFail(Error error) {
        if (error.getCode() == Error.Code.USER_DISABLED) {
            Log.i("Skygear User", "onAuthFail: Fail with reason:" + error.getLocalizedMessage());
        }
    }
});
```

```kotlin
val skygear = Container.defaultContainer(this)

skygear.auth.whoami(object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    Log.i("Skygear User", "I am ${user.get("username")}")
  }

  override fun onAuthFail(error: Error) {
    when (error.code) {
      Error.Code.USER_DISABLED -> Log.e("Skygear Login", "User disabled")
      else -> Log.e("Skygear Login", "onAuthFail: Fail with reason: ${error.code}")
    }
  }
})
```

[Error Handling in SDK]: /guides/advanced/sdk-error-handling/android/
[`whoami`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/AuthContainer.html#whoami-io.skygear.skygear.AuthResponseHandler-
