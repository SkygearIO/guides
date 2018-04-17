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

```javascript
var userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7";
skygear.auth.adminDisableUser(userID).then((userID) => {
  console.log("user is disabled");
}, (error) => {
  console.error(error);
})
```

### Enable a disabled user account

You can enable a disabled user account by calling
`adminEnableUser`:

```javascript
var userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7";
skygear.auth.adminEnableUser(userID).then((userID) => {
  console.log("user is enabled");
}, (error) => {
  console.error(error);
})
```

### Disable a user with a message and expiry period.

You can disable a user and attach a login message to the user account. The
login message can be shown to the user when the user logs in.

You can also set an expiry period. After the expiry period, the user will be
automatically enabled.

```javascript
var userID = "923b3d99-f21f-4e7c-a383-ffa319fe04c7";
skygear.auth.adminDisableUser(
  userID,
   "Account disabled due to TOS violations.",
   new Date((new Date()).getTime() + 5*60000)
).then((userID) => {
  console.log("user is disabled");
}, (error) => {
  console.error(error);
})
```

### Tell user that their account is disabled

When a user is disabled, server may return `UserDisabled` error. This will only
happen if the server action requires an authenticated user. To reliably check if
the user is disabled, call
[`whoami`].

To find out how to handle Skygear Error, check out [Error Handling in SDK].

The `toLocaleString()` function of the returned error will explain to the
user that their account is disabled. If you want to handle the error
differently, you can get the disable message from error info dictionary.

```javascript
skygear.auth.whoami().then((user) => {
    console.log(`Oh. I am ${user.username}.`);
}, (err) => {
    if (err.error.code === UserDisabled) {
        console(err.error.toLocaleString());
    }
    // Error handling...
});
```

[Error Handling in SDK]: /guides/advanced/sdk-error-handling/js/
[`whoami`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-whoami
