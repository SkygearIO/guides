---
title: User Authentication Basics
description: User sign up, login, logout / access token / username, email and password management in JS
---

[[toc]]
With Skygear, you can implement a user login system easily using the
authentication functions provided in the SDK.


## User Login Flow

Skygear stores and manages the user credentials (username, email and password)
internally and handles the user login session using an
access token stored in the local storage.

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
   user object (email, username, etc.) and the access token.
4. When a user logs in or signs up successfully, the SDK will set the user
   information with the access token obtained from the Skygear server. The app
   will change from the anonymous state to the logged-in state. The access token
   is sent to the server for authentication in every API call made through the
   SDK.
5. When a user logs out of the access token is found expired during an API
   call to the server, the SDK will clear the user information and the access
   token. The app will change from the logged-in state to the anonymous state.


## User Login Status

### Getting the currently logged-in user

You can retrieve the current user from [`skygear.currentUser`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-get-currentUser).

``` javascript
const user = skygear.currentUser; // if not logged in, it will be null
```

If there is an authenticated user, it will give you a [`User`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/user.js~User.html) object like this:

``` javascript
{
  'id': 'abcdef',
  'username': 'Ben',
  'email': 'ben@skygeario.com',
}
```

Please be reminded that the
[`currentUser`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-get-currentUser)
 object persist locally, and the information (e.g. roles, emails, etc)
 might not sync with the server if it was changed remotely.

To get the latest information of the current user,
you can call [`whoami()`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-whoami):

``` javascript
skygear.whoami().then((user) => {
    console.log(`Oh. I am ${user.username}.`);
}, (err) => {
    // Error handling...
})

```

### Observing user changes

The preferred way for your app to handle any logged-in user change is to
register a callback by using the [`skygear.onUserChanged`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-onUserChanged) method.
The callback will be invoked whenever the user is changed, i.e.
when any of the followings happens:

1. a user logs in;
2. a user logs out (or as logged out due to an expired access token);
3. the current user changes his/her email, even if the new email is the
   same as the old one.

The callback will receive the new user object as the argument.

``` javascript
const handler = skygear.onUserChanged(function (user) {
  if (user) {
    console.log('user logged in or signed up');
  } else {
    console.log('user logged out or the access token expired');
  }
});

handler.cancel(); // The callback is cancelable
```



## Signing up / Logging in / Logging out

### Signing up

A user can sign up using a username or an email, along with a password.
It is done using either [`skygear.signupWithUsername`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-signupWithUsername) or
[`skygear.signupWithEmail`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-signupWithEmail).

Skygear does not allow duplicated usernames or emails. Signing up with a
duplicated identifier will give the error `Duplicated`.

While each of the sign-up functions is resolved with a user object,
in most cases you need not deal with it because
you can access the currently logged-in user using [`skygear.currentUser`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-get-currentUser).

[`signupWithUsername`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-signupWithUsername) sample code:

``` javascript
import skygear from 'skygear';
import skygearError from 'skygear/lib/error';

skygear.signupWithUsername(username, password).then((user) => {
  console.log(user); // user object
}, (error) => {
  console.error(error);
  if (error.error.code === skygearError.Duplicated) {
    // the username has already existed
  } else {
    // other kinds of error
  }
});
```

[`signupWithEmail`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-signupWithEmail) sample code:

``` javascript
import skygear from 'skygear';
import skygearError from 'skygear/lib/error';

skygear.signupWithEmail(email, password).then((user) => {
  console.log(user); // user object
}, (error) => {
  console.error(error);
  if (error.error.code === skygearError.Duplicated) {
    // the email has already existed
  } else {
    // other kinds of error
  }
});
```
#### Anonymous user

Without being authenticated, a user can read data from the public database but
cannot perform most of the other operations, including saving data into the
database.

If you need an authenticated user but do not require a user to
sign up explicitly with a username or email, you can create an anonymous user
by calling [`skygear.signupAnonymously`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-signupAnonymously).

Every anonymous user has a unique user ID, and behaves exactly the same as
any user authenticated with a username or an email. The only difference is that
an anonymous user has no username, email, nor password. Because of the absence
of username and email, the account will be lost when the access token is lost.

``` javascript
import skygear from 'skygear';
import skygearError from 'skygear/lib/error';

skygear.signupAnonymously().then((user) => {
  console.log(user); // user object with undefined email and username
}, (error) => {
  console.error(error);
});
```

### Logging in

The login functions are similar to the sign-up ones.

If the credentials are incorrect, it will give the error of:

- `InvalidCredentials` if the password is incorrect;
- `ResourceNotFound` if the email or username is not found.

While each of the login functions is resolved with a user object,
in most cases you need not deal with it because
you can access the currently logged-in user using [`skygear.currentUser`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-get-currentUser).

#### Logging in using a username

``` javascript
skygear.loginWithUsername(username, password).then((user) => {
  console.log(user); // user object
}, (error) => {
  console.error(error);    
  if (error.error.code === skygearError.InvalidCredentials ||
      error.error.code === skygearError.ResourceNotFound ) {
    // incorrect username or password
  } else {
    // other kinds of error
  }
})
```

#### Logging in using an email

``` javascript
skygear.loginWithEmail(email, password).then((user) => {
  console.log(user); // user object
}, (error) => {
  console.error(error);    
  if (error.error.code === skygearError.InvalidCredentials ||
      error.error.code === skygearError.ResourceNotFound ) {
    // incorrect username or password
  } else {
    // other kinds of error
  }
})
```

### Logging out

Logging out the current user is simple using the [`skygear.logout`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-logout) method.

Upon successful logout, the SDK will clear the current user and the access token
from the local storage.

``` javascript
skygear.logout().then(() => {
  console.log('logout successfully');
}, (error) => {
  console.error(error);
});
```


## Changing username/email/password

### Changing the username and email of a user

To change a user's username and email, you can use
the [`skygear.saveUser`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-saveUser) method by
providing the user ID and the new username and/or the new email.
Every user can change his/her own username/email while
only users with the admin role can change the usernames/emails of other users.

::: caution

**Caution:** Changing the username/email of the current user will trigger the callback
registered through [`skygear.onUserChanged`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-onUserChanged), even if the new username/email is
the same as the old one.

:::

To change the username of the current user:

``` javascript
skygear.saveUser({
  id: skygear.currentUser.id,
  username: 'new-username',
}).then((user) => {
  console.log(user); // updated user object
  console.log('Username is changed to: ', user.username);
}, (error) => {
  console.error(error);
});
```

To change the email of the current user:

``` javascript
skygear.saveUser({
  id: skygear.currentUser.id,
  email: 'new-email@example.com',
}).then((user) => {
  console.log(user); // updated user object
  console.log('Email is changed to: ', user.email);
}, (error) => {
  console.error(error);
});
```

To change the username and email at the same time, you can specify both
[`username`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/user.js~User.html#instance-member-username) and [`email`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/user.js~User.html#instance-member-email) when calling [`skygear.saveUser`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-saveUser).

### Changing the password of a user

The currently logged-in user can change his/her own password.
This can be done using the [`skygear.changePassword`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-changePassword) function.

If the current password is incorrect, the SDK will return an
`InvalidCredentials` error.

``` javascript
skygear.changePassword(currentPassword, newPassword).then((user) => {
  console.log(user); // user object
  console.log('Password has been changed');
}, (error) => {
  console.error(error);
  if (error.error.code === skygearError.InvalidCredentials) {
    // the current password is incorrect
  } else {
    // other kinds of error
  }
});
```

Note: Changing the password of the current user will not trigger the callback
registered through [`skygear.onUserChanged`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/container.js~Container.html#instance-method-onUserChanged).

#### Invalidating existing access tokens

If you are using the JWT token store, all existing access tokens
will be invalidated when a user changes his/her password.

[Not yet implemented]
If you are not using the JWT token store, you can invalidate existing tokens by
setting `invalidate` to `true`:

``` javascript
skygear.changePassword(currentPassword, newPassword, invalidate=true)
.then((user) => {
  console.log(user); // user object
  console.log('The user has got a new access token');
}, (error) => {
  console.error(error);
});
```

### Password reset

Not yet implemented.


## User Verification

Not yet implemented.

## What's next from here?
You may want to learn more about:

* [Social Login using Skygear](/guides/auth/social-login/js/)
* [Setting User Profile](/guides/auth/user-profile/js/)
