---
title: User Authentication Basics
description: User sign up, login, logout / access token / username, email and password management in JS
---

[[toc]]

## Introduction

Skygear user authentication is a collection of APIs that helps you manage users in your app.

Skygear stores and manages the user credentials internally and handles the user login session using an access token stored in the local storage.

The following diagram shows the login flow:

<div style="text-align: center;">

```xml
    +-------------------------------------+
    |        Initial page load and        |
    |     Skygear container configured    |
    +-------------------------------------+

                      |
                      |
                      v

    +-------------------------------------+
    |   1. Check if authenticated user    |
    |      exists in local storage        |
    +-------------------------------------+

                      |
                      |
          +-----------+-----------+
          |                       |
   2. No  |                       |  3. Yes
          |                       |
          v                       v

+-------------------+   +---------------------+
|  Anonymous state  |   |   Logged-in state   |
| (no access token) |   | (with access token) |
+-------------------+   +---------------------+

  ^   |                                 ^   |
  |   |    4. login/sign-up success     |   |
  |   +---------------------------------+   |
  |                                         |
  +-----------------------------------------+
   5. logout success or access token expired
```

</div>

1. When the container is configured with the server endpoint and the API key, it
   will check for the existence of an authenticated user in the local storage.
2. If there is none, the SDK will do nothing, the app remains in the anonymous
   state. There is no access token.
3. If there is, the SDK will retrieve the user information, including the
   user record (email, username, etc.) and the access token.
4. When a user logs in or signs up successfully, the SDK will set the user
   information with the access token obtained from the Skygear server. The app
   will change from the anonymous state to the logged-in state. The access token
   is sent to the server for authentication in every API call made through the
   SDK.
5. When a user logs out of the access token is found expired during an API
   call to the server, the SDK will clear the user information and the access
   token. The app will change from the logged-in state to the anonymous state.

## Signing up

### Signing up with email or username

A user can sign up using a username or an email, along with a password.
It is done using either [`skygear.auth.signupWithUsername`] or
[`skygear.auth.signupWithEmail`].

Skygear does not allow duplicated usernames or emails. Signing up with a
duplicated identifier will give the error `Duplicated`.

While each of the sign-up functions is resolved with a user record,
in most cases you need not deal with it because
you can access the currently logged-in user using [`skygear.auth.currentUser`].

[`signupWithUsername`] sample code:

``` javascript
import skygear from 'skygear';

skygear.auth.signupWithUsername(username, password).then((user) => {
  console.log(user); // user record
  console.log(user["username"]); // username of the user
}, (error) => {
  console.error(error);
  if (error.error.code === skygear.ErrorCodes.Duplicated) {
    // the username has already existed
  } else {
    // other kinds of error
  }
});
```

[`signupWithEmail`] sample code:

``` javascript
import skygear from 'skygear';

skygear.auth.signupWithEmail(email, password).then((user) => {
  console.log(user); // user record
  console.log(user["email"]); // email of the user
}, (error) => {
  console.error(error);
  if (error.error.code === skygear.ErrorCodes.Duplicated) {
    // the email has already existed
  } else {
    // other kinds of error
  }
});
```

It is common to add other data to user record when signing up, you can do that
by specifying the data for user record in the third parameter of the signup
functions:

``` javascript
import skygear from 'skygear';

var otherData = { "interest": "reading" };

// sign up with email and also add data to user record
// this also work for signupWithUsername
skygear.auth.signupWithEmail(email, password, otherData).then((user) => {
  console.log(user["interest"]); // print "reading"
}, (error) => {
  console.error(error);
});
```

### Signing up anonymously

Without being authenticated, a user can read data from the public database but
cannot perform most of the other operations, including saving data into the
database.

If you need an authenticated user but do not require a user to
sign up explicitly with a username or email, you can create an anonymous user
by calling [`skygear.auth.signupAnonymously`].

Every anonymous user has a unique user ID, and behaves exactly the same as
any user authenticated with a username or an email. The only difference is that
an anonymous user has no username, email, nor password. Because of the absence
of username and email, the account will be lost when the access token is lost.

``` javascript
import skygear from 'skygear';

skygear.auth.signupAnonymously().then((user) => {
  console.log(user); // user record without email and username
}, (error) => {
  console.error(error);
});
```

## Logging in

The login functions are similar to the sign-up ones.

If the credentials are incorrect, it will give the error of:

- `InvalidCredentials` if the password is incorrect;
- `ResourceNotFound` if the email or username is not found.

While each of the login functions is resolved with a user record,
in most cases you need not deal with it because
you can access the currently logged-in user using [`skygear.auth.currentUser`].

### Logging in using a username

``` javascript
skygear.auth.loginWithUsername(username, password).then((user) => {
  console.log(user); // user record
}, (error) => {
  console.error(error);
  if (error.error.code === skygear.ErrorCodes.InvalidCredentials ||
      error.error.code === skygear.ErrorCodes.ResourceNotFound ) {
    // incorrect username or password
  } else {
    // other kinds of error
  }
})
```

### Logging in using an email

``` javascript
skygear.auth.loginWithEmail(email, password).then((user) => {
  console.log(user); // user record
}, (error) => {
  console.error(error);
  if (error.error.code === skygear.ErrorCodes.InvalidCredentials ||
      error.error.code === skygear.ErrorCodes.ResourceNotFound ) {
    // incorrect email or password
  } else {
    // other kinds of error
  }
})
```

## Logging out

Logging out the current user is simple using the [`skygear.auth.logout`] method.

Upon successful logout, the SDK will clear the current user and the access token
from the local storage.

``` javascript
skygear.auth.logout().then(() => {
  console.log('logout successfully');
}, (error) => {
  console.error(error);
});
```

## Getting the current user

You can retrieve the current user from [`skygear.auth.currentUser`].

``` javascript
const user = skygear.auth.currentUser; // if not logged in, it will be null
```

If there is an authenticated user, it will give you a [`Record`] which
is the user record. The user record looks like this:

``` json
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
you can call [`whoami`]:

``` javascript
skygear.auth.whoami().then((user) => {
    console.log(`Oh. I am ${user.username}.`);
}, (err) => {
    // Error handling...
})

```

## Observing user changes

The preferred way for your app to handle any logged-in user change is to
register a callback by using the [`skygear.auth.onUserChanged`] method.
The callback will be invoked whenever the user is changed, i.e.
when any of the followings happens:

1. a user logs in;
2. a user logs out (or as logged out due to an expired access token);

The callback will receive the new user record as the argument.

``` javascript
const handler = skygear.auth.onUserChanged(function (user) {
  if (user) {
    console.log('user logged in or signed up');
  } else {
    console.log('user logged out or the access token expired');
  }
});

handler.cancel(); // The callback is cancelable
```

## Updating a user's username and email

Username and email is saved to the user record. You can modify the username
and email in the same way you modify any other record, which is achieved by
calling [`skygear.publicDB.save`].

::: caution

**Caution:** When saving user record, the user record returned by
[`currentUser`] is not automatically updated. To force an update of
the record returned from [`currentUser`], call [`whoami`].

:::

To change the username of the current user:

``` javascript
var user = skygear.auth.currentUser;
user["username"] = "new-username"
skygear.publicDB.save(user).then((user) => {
  console.log(user); // updated user record
  console.log('Username is changed to: ', user["username"]);
  return skygear.auth.whoami();
}, (error) => {
  console.error(error);
});
```

To change the email of the current user:

``` javascript
var user = skygear.auth.currentUser;
user["email"] = "new-email@example.com"
skygear.publicDB.save(user).then((user) => {
  console.log(user); // updated user record
  console.log('Email is changed to: ', user["email"]);
  return skygear.auth.whoami();
}, (error) => {
  console.error(error);
});
```

You can even change the username and email at the same time:

``` javascript
var user = skygear.auth.currentUser;
user["username"] = "new-username"
user["email"] = "new-email@example.com"
skygear.publicDB.save(user).then((user) => {
  console.log(user); // updated user record
}, (error) => {
  console.error(error);
});
```

## Updating a user's password

The currently logged-in user can change his/her own password.
This can be done using the [`skygear.auth.changePassword`] function.

If the current password is incorrect, the SDK will return an
`InvalidCredentials` error.

``` javascript
skygear.auth.changePassword(currentPassword, newPassword).then((user) => {
  console.log(user); // user record
  console.log('Password has been changed');
}, (error) => {
  console.error(error);
  if (error.error.code === skygear.ErrorCodes.InvalidCredentials) {
    // the current password is incorrect
  } else {
    // other kinds of error
  }
});
```

Note: Changing the password of the current user will not trigger the callback
registered through [`skygear.auth.onUserChanged`].

## Invalidating existing access tokens

If you are using the JWT token store, all existing access tokens
will be invalidated when a user changes his/her password.

[Not yet implemented]
If you are not using the JWT token store, you can invalidate existing tokens by
setting `invalidate` to `true`:

``` javascript
skygear.auth.changePassword(currentPassword, newPassword, invalidate=true)
.then((user) => {
  console.log(user); // user object
  console.log('The user has got a new access token');
}, (error) => {
  console.error(error);
});
```

## Forgot password

Coming soon.

## User Verification

Not yet implemented.

## What's next from here?
You may want to learn more about:

* [Social Login using Skygear](/guides/auth/social-login/js/)
* [Setting User Profile](/guides/auth/user-profile/js/)

[`Record`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/record.js~Record.html
[`currentUser`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-get-currentUser
[`email`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/user.js~User.html#instance-member-email
[`signupWithEmail`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-signupWithEmail
[`signupWithUsername`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-signupWithUsername
[`skygear.auth currentUser`]: https://docs.skygear.iohttps://docsskygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-get-currentUser
[`skygear.auth.changePassword`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-changePassword
[`skygear.auth.logout`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-logout
[`skygear.auth.onUserChanged`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-onUserChanged
[`skygear.auth.saveUser`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-saveUser
[`skygear.auth.saveUser`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-saveUsery
[`skygear.auth.signupAnonymously`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-signupAnonymously
[`skygear.auth.signupWithEmail`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-signupWithEmail
[`skygear.auth.signupWithUsername`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-signupWithUsername
[`username`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/user.js~User.html#instance-member-username
[`whoami`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-whoami
[`save`]: https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/database.js~Database.html#instance-method-save
